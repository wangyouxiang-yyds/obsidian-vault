---
type: dataset
name: MS6_sen2like
location: /home/alanyh/oil_dataset/new/full_band/
tags: [sen2like, 8-band, multi-spectral]
---

# MS6_sen2like 資料集

## 1. 基本資訊
這是目前專案主要使用的 8 波段多光譜資料集，由 Sen2Like 處理後生成。影像對齊至 10m 解析度（10980×10980），並與 Mask TIF 座標系一致。

## 2. 關鍵路徑
- **原始影像 (8-band TIF)**: `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like/`
- **VRT 虛擬影像**: `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like_vrt/`
- **Mask 標籤**: `/home/alanyh/oil_dataset/new/full_band/mask/`
- **JSON 標註**: `/home/alanyh/oil_dataset/new/full_band/JSON/`

## 3. 波段定義 (8 Bands)
波段順序依波長升序排列：

| Index | 波段 | 波長(nm) | 解析度 | 訓練說明 |
|---|---|---|---|---|
| 1 | B01 | 443 | 60m → 10m | Coastal aerosol |
| 2 | B02 | 492 | 10m | Blue |
| 3 | B03 | 560 | 10m | Green |
| 4 | B04 | 665 | 10m | Red |
| 5 | B08 | 833 | 10m | NIR |
| 6 | B8A | 865 | 20m → 10m | Narrow NIR |
| 7 | B11 | 1614 | 20m → 10m | SWIR 1 |
| 8 | B12 | 2202 | 20m → 10m | SWIR 2 |

## 4. Mask 定義
| 像素值 | 語意 | 訓練映射 (Pixel Mapping) |
|---|---|---|
| 0 | Oil | class 0 |
| 1 | Background | class 1 |
| 2 | Others | class 1 (ignore if needed) |
| 255 | Unannotated | ignore_index=255 |

## 5. 目前狀態
- **格式**: 已轉換為 **COG (Cloud Optimized GeoTIFF)** 格式，提升 I/O 讀取效能。
- **單位**: 數值已除以 10000 轉換為反射率 (Reflectance)。
- **統計值**: `mean` 與 `std` 已依 8 波段重新計算。
