---
type: pipeline
tags: [preprocess, vrt, sen2like, patch, fold-split, hnm, 8band, oil-detection]
project: OIL_PROJECT_MutiBand_VRT
updated: 2026-07-10
---

# VRT Pipeline — 01 前處理（Preprocessing）

> 系列文件：本文 | [[VRT_pipeline_02_模型訓練]] | [[VRT_pipeline_03_重組評估]]

---

## 概覽

將 Sen2Like 處理後的衛星影像，經過 VRT 虛擬化、GT Mask 生成、Patch 切分、Fold 切分，最終產出供模型訓練使用的 patch 座標索引與 split 清單。

```
Sen2Like 8-band TIF（每場景 8 個 ~125 MB TIF）
    ↓ build_vrt_ms6.py             → .vrt 虛擬多波段檔（5.3 KB / scene）
    ↓ json_to_mask_tif.py          → 10980×10980 GT Mask TIF
    ↓ generate_patch_coords.py     → 256×256 patch 座標 TXT（GB1.5 平衡採樣）
    ↓ build_scene_splits.py        → 3-fold scene-level split TXT
    ↓ Hard Negative Mining（選用）  → HN patch 座標加入 train set
    ↓ generate_nirRG_png.py        → NIR-R-G 假色背景圖（重組視覺化用）
```

---

## 資料規模

| 項目 | 數值 |
|------|------|
| 場景總數 | 439（NOAA 403 + 事件 36） |
| 影像大小 | 10980 × 10980 像素（10m 解析度）|
| 覆蓋範圍 | 110 × 110 km / 場景 |
| 輸入頻段 | 8 bands（B01/02/03/04/08/8A/11/12）|
| 波長範圍 | 443 nm → 2202 nm（Sen2Like 預處理後）|

---

## 波段定義（8-band MS6）

| Band index（1-indexed）| 波段 | 波長(nm) | 原始解析度 | VRT 解析度 |
|---|---|---|---|---|
| 1 | B01 | 443 | 60m | 10m（bilinear resize）|
| 2 | B02 | 492 | 10m | 10m |
| 3 | B03 | 560 | 10m | 10m |
| 4 | B04 | 665 | 10m | 10m |
| 5 | B08 | 833 | 10m | 10m |
| 6 | B8A | 865 | 20m | 10m |
| 7 | B11 | 1614 | 20m | 10m（Fusion）|
| 8 | B12 | 2202 | 20m | 10m（Fusion）|

---

## 步驟 A：VRT 生成（build_vrt_ms6.py）

**腳本**：`preprocess/build_vrt_ms6.py`

**作用**：把每個場景 8 個 Sen2Like band TIF 組成一個 GDAL VRT 虛擬多波段檔，避免建立實體疊加圖（可省 > 1 TB 磁碟空間）。

**機制**：
- 用 rasterio 讀取每個 band 的 metadata，手動組裝 VRT XML（conda 環境不可用 `gdalbuildvrt` CLI）
- **參考 band 固定為 B02（10m）**，決定 VRT 的 RasterXSize/RasterYSize 與 GeoTransform
  - ⚠️ 歷史 bug（已修）：舊版以 B01（60m）為參考，VRT 聲明大小與 TIF 不符，導致讀出全零
- B01（60m → 10m）：VRT 內聲明 `cv2.resize` 到 10m；讀取時 rasterio 自動處理重採樣

**TIF 來源**：`/home/alanyh/oil_dataset/new/full_band/MS6_sen2like/`（8 個 band TIF / 場景，LZW 壓縮，約 125 MB 各）

**輸出**：
```
/mnt/backup/oil_dataset/new/full_band/MS6_sen2like_vrt/
    {EVENT_TYPE}/
        {SCENE_NAME}.vrt    （5.3 KB，指向本機 TIF 絕對路徑）
```

> ⚠️ **2026-07-10 補充**：`/home/alanyh/oil_dataset/new/full_band/MS6_sen2like_vrt/` 亦真實存在同一份輸出，兩份並存；分支 B（GT_expand）的 yaml 實際指向 `/home/alanyh` 這份。何者為 canonical 尚未確認，如實記錄待統一，不下結論。

---

## 步驟 B：GT Mask 生成（json_to_mask_tif.py）

**腳本**：`preprocess/json_to_mask_tif.py`

**作用**：把人工標註的 LabelMe JSON polygon 轉成與影像同大小的 GT Mask TIF。

**輸入**：`/home/alanyh/oil_dataset/new/full_band/MS6_sen2like_JSON/`（439 個 JSON，以場景資料夾名命名）

**Mask 像素值定義**：

| 像素值 | 意義 | 訓練用途 |
|--------|------|---------|
| 0 | Oil（油汙，正樣本） | Class 0 |
| 1 | Annotated BG（已標記背景）| Class 1（bg）|
| 2 | Others（船隻等次要類別）| Class 1（bg）|
| 255 | Unannotated（未標記區域）| 訓練時 ignore |

**輸出**：同場景目錄下的 `.mask.tif`（10980 × 10980，uint8，單 channel）

---

## 步驟 C：Patch 座標切分（generate_patch_coords.py）

**機制**：對每個場景做 sliding window，記錄含油 patch 與背景 patch 的左上角座標到 TXT。

> ⚠️ **2026-07-10 修正**：座標 TXT 的實際格式是**檔名編碼** `{SCENE_NAME}_patch_x{COL}_y{ROW}`，不是獨立的「x, y, w, h」欄位（[[VRT_pipeline_02_模型訓練]] 舊版的資料 Pipeline 段落也曾誤寫成 `x, y, w, h` 格式，已一併修正）。

| 參數 | 值 |
|------|----|
| Patch Size | 256 × 256 |
| Stride | 192（訓練時；推論重組時相同）|
| 取樣策略 | GB1.5：無油 patch : 含油 patch = 1.5 : 1 |

**Fold2 統計（3fold_random 主要使用的 fold）**：

| Split | Scene 數 | 含油 patch | 純背景 patch | 總計 |
|-------|---------|-----------|------------|------|
| Train | 234 | 2,321 | 3,482 | 5,803 |
| Val | 59 | 638 | 957 | 1,595 |
| Test | 146 | 1,536 | 1,304 | 3,840 |

**輸出**：
```
/mnt/backup/oil_dataset/new/full_band/data_split/3fold_random/patch_level_GB1.5/
    fold1/train.txt  val.txt  test.txt
    fold2/train.txt  val.txt  test.txt
    fold3/train.txt  val.txt  test.txt
```

---

## 步驟 D：Scene-level Fold 切分策略

**Split 目錄**：`/mnt/backup/oil_dataset/new/full_band/data_split/`

目前有以下幾種切分策略（均基於 439 場景 scene-level，再產出 patch-level TXT）：

| 策略名稱 | 說明 | 備註 |
|---------|------|------|
| `3fold_random` | 純隨機 3-fold，**當前主用** | Fold 2 為最新交付基準 |
| `3fold_temporal` | 依時間序列切（Expanding window，只用 NOAA 403 scenes）| 模擬實際部署情境 |
| `3fold_event` | 依事件留出 | 測試跨事件泛化能力 |
| `3fold_random_noGulf` | 排除 NOAA Gulf of Mexico（191 scenes）| 研究區域偏差 |
| `3fold_all220` | 舊 220-scene 版本（已廢棄）| 僅供歷史對照 |
| `5_fold` | 5-fold 版本 | 早期實驗，未成為主線 |

---

## 步驟 E：Hard Negative Mining（HNM，選用）

**目錄**：`preprocess/Hard_Negative_Mining/`

**流程**：
1. 用 ConvNeXt 分類器在純背景 patch 上做 inference
2. 挑出 confidence > threshold 的誤判 patch（False Positives）
3. 加入訓練集（組裝為「1:1 原始 + 0.5 HNM = 1:1.5」）

**現況**：cfHNM 實驗完成（avg Oil IoU=0.224），未超越基準 A 組（0.242），HNM 不是當前主線。

---

## 步驟 F：NIR-R-G 背景圖生成（generate_nirRG_png_all220_local.py）

**腳本**：`generate_nirRG_png_all220_local.py`（**2026-06-23 新版，本機輸出**）

**作用**：產出假色衛星底圖，供重組階段的 overlay 視覺化使用。

**假色組合**：B08（NIR）→ R，B04（R）→ G，B03（G）→ B

| 參數 | 值 |
|------|----|
| Reflectance clip | 0 ~ 0.3（之後 stretch 至 0-255）|
| 輸出格式 | JPEG |
| 輸出目錄 | `/home/alanyh/oil_dataset/new/full_band/NIR_R_G_Output_png/all_ms6/` |
| 輸出數量 | 440 張，共 7.2 GB |
| fold2 覆蓋率 | 146 / 146（100%）|

---

## 重要路徑彙整

| 項目 | 路徑 |
|------|------|
| 8-band TIF | `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like/` |
| VRT 檔 | `/mnt/backup/oil_dataset/new/full_band/MS6_sen2like_vrt/` **與** `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like_vrt/`（2026-07-10 查證：兩份皆真實存在，B 分支 yaml 用 `/home/alanyh` 這份；何者為 canonical 尚未確認，兩份並存待統一）|
| Mask TIF | ~~與各場景 TIF 同目錄~~ 已修正：實際集中於 `/home/alanyh/oil_dataset/new/full_band/mask/` |
| JSON 標註（flat）| `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like_JSON/` |
| Split TXT | `/mnt/backup/oil_dataset/new/full_band/data_split/` |
| NIR-R-G 背景圖 | `/home/alanyh/oil_dataset/new/full_band/NIR_R_G_Output_png/all_ms6/` |
| 專案根目錄 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_0422_VRT_training/` |

---

## 相關頁面

- [[VRT_pipeline_02_模型訓練]] — 訓練架構與超參數
- [[VRT_pipeline_03_重組評估]] — Reconstruction 與交付指標
- [[OIL_PROJECT_VRT_0422]] — 早期版本架構說明（0422 初版，部分資訊已過時）
- [[ms6_sen2like]] — MS6 資料集說明
- [[annotation_workflow]] — LabelMe JSON 標注工作流程
- [[20260605_資料集修正與三策略重跑]] — 場景擴充至 439 的完整紀錄
- [[hard_negative_mining]] — HNM 概念說明
