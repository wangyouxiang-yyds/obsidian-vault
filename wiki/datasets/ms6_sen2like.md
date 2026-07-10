---
type: dataset
name: MS6_sen2like
location: /home/alanyh/oil_dataset/new/full_band/
tags: [sen2like, 8-band, multi-spectral]
updated: 2026-07-10
---

# MS6_sen2like 資料集

## 1. 基本資訊
這是目前專案主要使用的 8 波段多光譜資料集，由 Sen2Like 處理後生成。影像對齊至 10m 解析度（10980×10980），並與 Mask TIF 座標系一致。

## 2. 關鍵路徑
- **原始影像 (8-band TIF)**: `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like/`
- **VRT 虛擬影像**: `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like_vrt/`（⚠️ 2026-07-10 查證：`/mnt/backup/oil_dataset/new/full_band/MS6_sen2like_vrt/` 亦真實存在同一份，兩份並存；何者為 canonical 尚未確認，見 [[VRT_pipeline_01_前處理]]）
- **Mask 標籤**: `/home/alanyh/oil_dataset/new/full_band/mask/`
- **JSON 標註**: ~~`/home/alanyh/oil_dataset/new/full_band/JSON/`~~ 已修正：實際分佈在**兩處**，`build_manifest.py` 兩處都查（場景資料夾優先）：
  - 集中目錄：`/home/alanyh/oil_dataset/new/full_band/MS6_sen2like_JSON/`
  - 各場景資料夾內：`MS6_sen2like/{LOCATION}/{YEAR}/{SCENE_NAME}/{SCENE_NAME}.json`
  - 2026-07-10 manifest 盤點（445 場景）：163 個只在集中目錄、6 個只在場景資料夾內（含 5 張新入庫的 2026 NOAA Atlantic 場景 + 1 張 2023年菲律賓MT公主皇后號_20230430_L8_51PUQ）、276 個兩處都有；曾誤報菲律賓場景缺 JSON，已更正

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
- **格式**: ~~已轉換為 **COG (Cloud Optimized GeoTIFF)** 格式~~ 已修正（2026-07-10）：實際已跑轉檔（tiled 256 + deflate，`cog_convert.log` 3080 檔全 OK），但**未建 overviews**，嚴格來說是 **tiled GeoTIFF**，非完整 COG 規格。
- **單位**: ~~數值已除以 10000 轉換為反射率 (Reflectance)~~ 已修正（2026-07-10）：檔案本體是 **uint16 DN（0-10000 尺度）**，`/10000` 是 dataloader 讀取時才做的轉換（`deeplab_adapter.py` 讀 window 後 `/10000.0`，另有 nodata 歸零處理），並非儲存時就已轉好的反射率。
- **統計值**: `mean` 與 `std` 已依 8 波段重新計算。
- **場景總數（2026-07-10 manifest 盤點）**：445 場景，8-band TIF 齊全、mask/JSON/VRT 齊全，manifest issues 欄全空（0 個異常）。dirty（`dirty.txt`/`blur.txt` 標記）85 張。訓練用 358clean 3-fold split（`3_fold_stratified_v2`）不受影響，維持不變。
- **VRT 雙路徑覆蓋率**：220 場景 home（`/home/alanyh/...`）+ backup（`/mnt/backup/oil_dataset/...`）兩份皆有，225 場景（含新入庫的 5 張 2026 場景）只有 home 份，backup 補齊為待辦。

## 6. Dataset Manifest 制度（2026-07-10 新增）

**腳本**：`OIL_PROJECT_MutiBand_GT_expand/preprocess/build_manifest.py`

**作用**：掃描整個 full_band 資料集，產出**單一真相帳本** `/home/alanyh/oil_dataset/new/full_band/manifest.csv`，每場景一列。

**欄位**：`scene_id` / `sensor` / `date` / `tile` / `n_band_tifs` / `vrt_home` / `vrt_backup_exists` / `mask_path` / `mask_md5` / `json_exists` / `dirty` / `oil_px` / `splits_sv2` / `issues` 等。

**機制**：
- 8-band TIF / VRT / mask / JSON 缺一即在 `issues` 欄標記，完整性一目瞭然
- JSON 檢查場景資料夾與集中目錄兩處（場景資料夾優先），見上方第 2 節
- 重跑即 idempotent 更新；若已有舊版，先備份成 `manifest.prev.csv`，並印出與上一版的差異（新增/移除/`mask_md5` 變更）
- `mask_md5` 一旦變動 → 警告「該場景 patch 座標已過期，需重生 `patch_level_gt_expand`」，用於追蹤標註修改後下游是否同步

**標準動作**：資料集增刪改場景後，一律重跑 `build_manifest.py` 作為入口，而非手動核對。

## 7. 2026 新場景入庫記錄（2026-07-10）

5 張 2026 NOAA Atlantic（19TDE tile）場景完成入庫：`20260309_S2`、`20260329_S2`、`20260414_L9`、`20260425_S2`、`20260508_L8`。

- 場景資料夾內原有 `oil.gpkg` + `background.gpkg` 標註，經 `batch_event_gpkg_to_mask.process_scene`（gpkg → LabelMe JSON → mask TIF）+ `build_vrt_ms6.py`（8-band VRT，B02 10m 參考）完成前處理
- 驗證：mask = VRT = 10980×10980，mask 值僅 0（oil）/1（bg）/255（未標記）
- 油汙像素數（依序）：13,663 / 345 / 3,494 / 2,286 / 1,884（偏小型油汙）
- JSON 只存在於場景資料夾內，尚未同步到集中目錄 `MS6_sen2like_JSON/`（見上方第 2 節）
- VRT 只有 home 份，無 backup
- **尚未加入任何 split**：`3_fold_stratified_v2` 不含這 5 張；若要進訓練須重生 split / patch 座標，另行決策
