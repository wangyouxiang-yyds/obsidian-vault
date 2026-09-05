---
date: 2026-09-05
updated: 2026-09-05
type: pipeline
tags: [gt-expand, onboarding, workflow, dataset, eight-band]
project: OIL_PROJECT_MutiBand_GT_expand
---

# GT_expand Workflow 新人導覽

> 本頁是一份隨 workflow trace 逐段補寫的新人文件。內容以目前實際資料、程式與設定為準；尚未確認的環節會明確標為待確認，不以歷史筆記補空白。

## 1. 文件範圍與閱讀方式

本文件的起點是**已完成上游處理、整理成固定八波段的場景資料**。原始 11-band 衛星產品、Sen2Like 大氣校正／光譜調和／Fusion 等流程目前位於另一個專案，暫不納入本頁；待該專案移入後再向上補接。

目前研究與程式正本：

`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand`

閱讀時要區分三個層級：

1. **資料池**：磁碟上所有已整理場景。
2. **實驗語料**：被某一版 split 與 patch index 引用的場景。
3. **模型輸入**：dataloader 讀取後，實際送進特定模型的通道與張量。

三者數量與波段數不一定相同。例如資料層固定保存 8 bands，但目前 Prithvi 路徑之後只選其中 6 bands 餵入 backbone；具體選法留待「模型輸入」章節 trace。

## 2. Workflow 全貌（逐段補寫）

```text
已整理的 8-band 場景
  ├─ 影像流：單波段 TIF → 8-band VRT
  └─ 標註流：JSON / GPKG → 全場景 Mask TIF
                         ↓
                 manifest / preflight
                         ↓
                scene split / patch index
                         ↓
              dataloader / normalization / augmentation
                         ↓
                    model / training
                         ↓
                inference / reconstruction
                         ↓
                  evaluation / output
```

後續待逐段補寫：

- [ ] VRT、Mask、manifest 與 preflight
- [ ] Scene split、GT-expand 正樣本與背景 patch
- [ ] Dataloader、正規化與 augmentation
- [ ] 模型結構、loss、訓練與 checkpoint
- [ ] Patch inference、TTA 與場景 reconstruction
- [ ] Evaluation Contract、prediction artifacts 與結果輸出

## 3. 資料特性

### 3.1 一筆資料的基本單位是場景

場景識別字目前遵循：

`{來源或事件}_{YYYYMMDD}_{S2|L8|L9}_{MGRS_TILE}`

例如：

`NOAA_Atlantic_Ocean_High_Confidence_20190203_L8_18TYK`

實體目錄存在兩種主要深度：

```text
# 一般事件與台灣資料
MS6_sen2like/{事件或資料群}/{SCENE_ID}/

# NOAA 資料
MS6_sen2like/{NOAA資料群}/{YEAR}/{SCENE_ID}/
```

因此下游程式不應假設所有場景都有固定目錄深度；現行 manifest 以搜尋 `*_B02.tif` 的父目錄辨認場景。

### 3.2 固定八波段契約

每個場景應包含以下八個單波段 TIF，順序不可自行調換：

| VRT index | 波段 | 說明 |
|---:|---|---|
| 1 | B01 | Coastal aerosol |
| 2 | B02 | Blue；空間參考波段 |
| 3 | B03 | Green |
| 4 | B04 | Red |
| 5 | B08 | Broad NIR |
| 6 | B8A | Narrow NIR |
| 7 | B11 | SWIR 1 |
| 8 | B12 | SWIR 2 |

檔名契約：`{SCENE_ID}_{BAND}.tif`。

單波段影像目前實查為 `uint16`，反射率仍採約 0–10000 的整數尺度，並宣告 `nodata=0`。`/10000` 是下游 dataloader／reconstruction 讀取 window 時才執行，不是檔案已預先存成 0–1 浮點數。

主要路徑：

| 內容 | 路徑 |
|---|---|
| 單波段影像 | `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like/` |
| 8-band VRT | `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like_vrt/` |
| 全場景 Mask | `/home/alanyh/oil_dataset/new/full_band/mask/` |
| 集中式 LabelMe JSON | `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like_JSON/` |
| Dataset manifest | `/home/alanyh/oil_dataset/new/full_band/manifest.csv` |
| 現行 split 根目錄 | `/mnt/backup/oil_dataset/new/full_band/data_split/3_fold_stratified_v2/` |

### 3.3 實體影像與 VRT 邏輯影像不同

八個實體 TIF 並非全部都是 `10980 × 10980`。例如實查 Sentinel-2 場景可見 B01 為 `1830 × 1830`、B8A/B11/B12 為 `5490 × 5490`；Landsat 場景也可能出現 B01 `3660 × 3660`、B08 `7320 × 7320` 等尺寸。

`build_vrt_ms6.py` 以 B02 的 CRS、bounds、transform 與大小作為空間基準，把八個來源波段描述成邏輯上的 `8 × 10980 × 10980` raster。VRT 是引用來源檔與重採樣規則的 XML，**不是另外複製一份八波段大型影像**；模型之後透過 `rasterio.Window` 按 patch 動態讀取。

因此本專案有兩種不同的「對齊」：

- 上游對齊：跨感測器的光譜／產品語意整理；不在本頁目前範圍內。
- VRT 對齊：把八個實體波段映射到同一個 B02 像素網格；屬於現行 GT_expand workflow。

### 3.4 原始 Mask 類別語意

全場景 Mask 是單波段 `uint8 GeoTIFF`，原始像素值為：

| 原始值 | 資料語意 |
|---:|---|
| 0 | Oil |
| 1 | Background |
| 2 | Others |
| 255 | 未標註 |

這張表只描述**資料儲存語意**。訓練與重建時，YAML 的 `pixel_mapping` 還會把原始值轉成模型 class；目前 P10 config 明確採用 `0→0`、`1→1`、`2→1`、`255→1`，也就是把 `Others` 與未標註像素都送入 Background class。這裡先記錄程式現況；是否長期維持這項資料政策，仍待後續 trace 確認。

#### 已確認的 metadata 陷阱：`nodata=0` 與 Oil 類別衝突

兩個 Mask writer 都先複製 B02 的 raster profile，再修改 dtype/count/compression，但沒有重設 `nodata`。抽樣 Mask 因此宣告 `nodata=0`，恰好又與 `Oil=0` 衝突。

現行 GT_expand 訓練會依路徑使用 `tifffile` 或 Rasterio 讀成一般 ndarray；Rasterio 路徑與重建皆未啟用 masked read（等同 `masked=False`）。因此目前程式不會因 metadata 自動丟掉 Oil。然而，若新人改用 GIS、`rasterio.read(masked=True)` 或其他遵守 nodata metadata 的工具，可能把 Oil 像素誤當成無效值。

本頁只記錄此風險，**尚未修改任何 Mask 或 writer**；是否修正及如何維持既有 artifact 相容性，留待後續審查。

### 3.5 資料池不等於目前訓練語料

依 `manifest.csv` 的 2026-09-05 唯讀快照：

| 項目 | 數量 | 解讀 |
|---|---:|---|
| Manifest 場景總數 | 789 | 磁碟資料池，不是訓練數 |
| dirty／blur 標記 | 85 | 是否排除由 split／生成流程決定 |
| 現行 `3_fold_stratified_v2` 成員 | 355 | 目前 GT_expand 實驗語料範圍 |
| `no_json` | 342 | Manifest 完整性問題；不可自動視為有 GT |

新增場景、標註或 Mask 改動後，應重新產生 manifest。Manifest 會記錄八波段完整性、VRT、Mask、JSON、dirty marker、split membership 與 `mask_md5`；若 Mask hash 改變，既有 patch 座標可能已過期。

## 4. 本輪待確認

1. 現行 GT_expand 訓練及 reconstruction 已把原始 `2=Others` 與 `255=未標註` 映射為 Background；待確認這是要維持的正式資料政策，還是現行實驗設定。
2. Mask 的 `nodata=0` metadata 是否在後續另立相容性修正，或只作使用限制揭露。

## 5. 本節證據入口

- 八波段與 VRT 幾何契約：`preprocess/build_vrt_ms6.py`
- JSON → Mask：`preprocess/json_to_mask_tif.py`
- GPKG/SHP → JSON → Mask：`preprocess/batch_event_gpkg_to_mask.py`
- Manifest：`preprocess/build_manifest.py`
- 訓練資料讀取：`main/deeplab_adapter.py`
- Reconstruction 資料讀取：`main/recon_gt_aware_module.py`
