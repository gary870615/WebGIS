<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <title>WebGIS Demo (Leaflet + GeoJSON)</title>

  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

  <style>
    html, body { height: 100%; margin: 0; }
    #map { height: 100vh; }

    /* 簡單 UI 面板 */
    .panel {
      position: absolute;
      top: 12px;
      left: 12px;
      z-index: 9999;
      background: white;
      padding: 10px 12px;
      border-radius: 10px;
      box-shadow: 0 4px 16px rgba(0,0,0,0.15);
      font-family: Arial, sans-serif;
      width: 280px;
    }
    .panel h3 { margin: 0 0 8px 0; font-size: 16px; }
    .panel button {
      width: 100%;
      padding: 8px 10px;
      margin: 6px 0;
      cursor: pointer;
    }
    .panel .row { font-size: 13px; margin-top: 6px; line-height: 1.4; }
    .muted { color: #666; }
    .ok { color: #0a7; font-weight: bold; }
    .bad { color: #d33; font-weight: bold; }
    code { background: #f4f4f4; padding: 1px 4px; border-radius: 4px; }
  </style>
</head>

<body>
<div id="map"></div>

<div class="panel">
  <h3>Leaflet 靜態 Demo</h3>

  <div class="row">
    資料來源：<code id="dataSrc">./data/features.geojson</code>
  </div>

  <button id="btnLoadAll">載入全部點位</button>
  <button id="btnLoadBbox">依目前地圖視窗 bbox 篩選</button>

  <div class="row muted">
    提示：此版本可直接部署到 GitHub Pages（不需要後端）。
    若要連 FastAPI/DB，之後再改 API_BASE 指向雲端後端即可。
  </div>

  <div class="row muted" id="status">狀態：待命</div>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script>
  const DATA_URL = "./data/features.geojson";
  document.getElementById("dataSrc").textContent = DATA_URL;

  // 1) 初始化地圖
  const map = L.map('map').setView([23.8, 121.0], 7);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19
  }).addTo(map);

  // 2) GeoJSON layer（每次重新載入就清掉再加）
  let geoLayer = null;
  let allGeoJSON = null; // 讀入後保存在記憶體，bbox 篩選用

  function setStatus(msg) {
    document.getElementById("status").textContent = "狀態：" + msg;
  }

  function clearLayer() {
    if (geoLayer) {
      geoLayer.remove();
      geoLayer = null;
    }
  }

  function addGeoJSON(geojson) {
    clearLayer();
    geoLayer = L.geoJSON(geojson, {
      onEachFeature: (feature, layer) => {
        const p = feature.properties || {};
        const name = p.name ?? "";
        const type = p.type ?? "";
        layer.bindPopup(`<b>${name}</b><br>type: ${type}`);
      }
    }).addTo(map);

    const n = (geojson.features || []).length;
    setStatus(`載入完成：${n} 筆`);
    if (n > 0) map.fitBounds(geoLayer.getBounds());
  }

  async function fetchGeoJSON() {
    const res = await fetch(DATA_URL, { cache: "no-store" });
    if (!res.ok) {
      const text = await res.text();
      throw new Error(`讀取 GeoJSON 失敗 (HTTP ${res.status}): ${text}`);
    }
    const geojson = await res.json();

    // 基本檢查
    if (!geojson || geojson.type !== "FeatureCollection" || !Array.isArray(geojson.features)) {
      throw new Error("GeoJSON 格式不正確：需要 FeatureCollection + features[]");
    }
    return geojson;
  }

  // 依 bbox 在前端本地篩選（不靠 API）
  function filterByMapBounds(geojson) {
    const b = map.getBounds();
    const filtered = {
      type: "FeatureCollection",
      features: (geojson.features || []).filter(f => {
        if (!f || !f.geometry) return false;
        if (f.geometry.type !== "Point") return true; // 非點先保留（你未來若放線面）
        const c = f.geometry.coordinates;
        if (!Array.isArray(c) || c.length < 2) return false;
        const lon = Number(c[0]);
        const lat = Number(c[1]);
        if (!Number.isFinite(lon) || !Number.isFinite(lat)) return false;
        return b.contains([lat, lon]); // Leaflet 用 [lat, lon]
      })
    };
    return filtered;
  }

  // 載入全部
  document.getElementById("btnLoadAll").addEventListener("click", async () => {
    try {
      setStatus("載入 GeoJSON...");
      allGeoJSON = await fetchGeoJSON();
      addGeoJSON(allGeoJSON);
    } catch (e) {
      setStatus("載入失敗：" + e.message);
      alert(e.message);
    }
  });

  // bbox 篩選（本地）
  document.getElementById("btnLoadBbox").addEventListener("click", async () => {
    try {
      if (!allGeoJSON) {
        setStatus("尚未載入資料，先載入 GeoJSON...");
        allGeoJSON = await fetchGeoJSON();
      }
      setStatus("依地圖視窗 bbox 篩選...");
      const filtered = filterByMapBounds(allGeoJSON);
      addGeoJSON(filtered);
    } catch (e) {
      setStatus("bbox 篩選失敗：" + e.message);
      alert(e.message);
    }
  });

  // 預設先載入一次
  (async () => {
    try {
      setStatus("預設載入 GeoJSON...");
      allGeoJSON = await fetchGeoJSON();
      addGeoJSON(allGeoJSON);
    } catch (e) {
      setStatus("預設載入失敗：" + e.message);
    }
  })();
</script>
</body>
</html>
