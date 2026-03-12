# 2016 美濃地震 GNSS 垂直同震位移（WebGIS Demo）
使用 Leaflet 製作 WebGIS 範例，展示 2016 美濃地震 GNSS 測站垂直同震位移（位移量單位：mm；正值抬升、負值沉陷）。

## Demo
- GitHub Pages：https://gary870615.github.io/WebGIS/

## 功能
- 讀取兩個 GeoJSON 圖層：GNSS 固定站 / GNSS 移動站
- 自訂 marker 顏色：固定站（藍）/ 移動站（紅）
- Popup 顯示測站代碼（GNSS）與位移量（mm）、座標(Lon,Lat)
- bbox 篩選：依目前地圖視窗範圍篩選顯示點位
- 圖層開關：右上角可切換顯示固定站/移動站/地震震央

## 資料與檔案
- `index.html`：前端主程式
- `data/2016_Meinong_u_CGNSS.geojson`：GNSS固定站資料
- `data/2016_Meinong_u_SGNSS.geojson`：GNSS移動站資料
- 資料來源：中央地質調查所

## GeoJSON 欄位（properties）
- `GNSS`：測站代碼（Popup 標題）
- `位移量`：垂直位移量（mm）
- `Lon`, `Lat`：座標（WGS84）
