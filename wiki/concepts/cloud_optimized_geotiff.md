---
type: concept
tags: [optimization, io, geotiff]
related: [wiki/datasets/ms6_sen2like.md]
---

# Cloud Optimized GeoTIFF (COG)

## 一句話定義
COG 是一種特殊的 GeoTIFF 檔案格式，它透過內部 Tiling（分塊）與 Overviews（預覽圖）優化，使得支援 HTTP Range Requests 或動態讀取的系統（如 GDAL/Rasterio）可以只讀取所需的局部區域，而不需要下載或解壓整個檔案。

## 詳細說明
傳統的 TIF 檔案若採用 **Strip (掃描線)** 格式儲存，當我們只想讀取中間的一個 256x256 patch 時，程式必須讀取並解壓整行（或多行）的所有像素。
*   **Strip 格式**: 讀取 256x256 需解壓 10980x256 的資料。
*   **COG (Tiled) 格式**: 內部以 256x256 或 512x512 的小塊儲存，讀取 256x256 時幾乎只需讀取目標塊，效能提升極大。

## 在本專案的應用
在 MS6 Pipeline 中，我們發現 10m 解析度的 Strip TIF 導致 I/O 成為訓練瓶頸（epoch 時間 > 9 分鐘）。
*   **優化方案**: 使用 `convert_to_cog.py` 將所有波段 TIF 轉為 COG 格式。
*   **實測效果**: 單波段讀取時間從 **397ms 降至 43ms**（約 9 倍加速）。
*   **訓練影響**: 預計將每個 epoch 的訓練時間從 9 分鐘縮短至 2 分鐘內。

## 相關頁面
- [20260521_MS6_Pipeline執行紀錄.md](../experiments/20260521_MS6_Pipeline執行紀錄.md)
