---
date: 2026-05-21
type: experiment
stage: preprocess
status: in_progress
tags: [sen2like, deeplabv3+, vrt, patching, fold-split, GB1.0, 8band, trial-run]
---

# 新批 Sen2Like Pipeline 執行紀錄（2026-05-21）

承接 [20260520 計畫文件](20260520_新批Sen2Like資料Pipeline重建計畫.md)，本頁記錄實際執行的過程、產出與發現的問題。

---

## 執行摘要

| 步驟 | 狀態 | 說明 |
|---|---|---|
| Step 2：建 VRT | ✅ 完成 | 220 個 8-band VRT，新腳本 `build_vrt_ms6.py` |
| Step 3：scene-level fold split | ✅ 完成 | 新腳本 `build_scene_splits_stratified.py` |
| Step 4：patch 座標生成 | ✅ 完成 | 新腳本 `generate_patch_coords.py` |
| Step 5：YAML 更新 | ✅ 完成 | `in_channels=8`、新路徑全部到位 |
| 試跑 Fold 1 | ⛔ 卡住 | 見下方「問題紀錄」 |

> Step 0（GPKG→JSON）、Step 1（mask 重生）已在更早的 session 完成，本次從 Step 2 開始。

---

## Step 2：建立 8-band VRT

### 新腳本

**`preprocess/build_vrt_ms6.py`**

由於 conda 環境內無法使用 `gdalbuildvrt` CLI，改用 `rasterio` 讀取 metadata 後手動組裝 VRT XML。

**波段順序（依波長升序）：**

| Band index（1-indexed） | 波段 | 波長(nm) |
|---|---|---|
| 1 | B01 | 443 |
| 2 | B02 | 492 |
| 3 | B03 | 560 |
| 4 | B04 | 665 |
| 5 | B08 | 833 |
| 6 | B8A | 865 |
| 7 | B11 | 1614 |
| 8 | B12 | 2202 |

**輸出目錄：**
```
/mnt/backup/oil_dataset/new/full_band/MS6_sen2like_vrt/
    NOAA_Atlantic_Ocean_High_Confidence/
        {SCENE_NAME}.vrt
    NOAA_Gulf_of_Mexico_High_Confidence/
        {SCENE_NAME}.vrt
```

> ⚠️ 腳本內 `VRT_OUT_DIR` 仍指向舊路徑 `vrt/`，VRT 是事後手動移動到 `MS6_sen2like_vrt/` 的。若重新執行腳本，需先更新 `VRT_OUT_DIR`。

### 結果

- **成功：220 個 VRT**
- **失敗：1 個**（`20250529_S2_15RXM`，缺少 B02 波段，已排除在所有 fold TXT 外）

### 附帶：NIR-R-G 視覺化 PNG

`preprocess/generate_nirRG_png.py`

為 66 個 test 場景生成 NIR-R-G 假彩色 PNG，供 Reconstruction 視覺化疊圖用。

- Band 5（B08/NIR） → R channel
- Band 4（B04/Red） → G channel
- Band 3（B03/Green） → B channel
- Clip 範圍：raw 值 × (1/10000) 後 clip 至 0–0.3
- 輸出：`/mnt/backup/oil_dataset/new/full_band/NIR_R_G_Output_png/test_ms6/`（66/66 完成）

---

## Step 3：Stratified Scene-Level Fold Split

### 新腳本

**`preprocess/build_scene_splits_stratified.py`**

- 掃描 MS6_sen2like 目錄，只保留 mask TIF 已存在的場景
- 以 `(location, year_month)` 為分組單位（防止同事件跨 train/val）
- 2025 場景 → 固定 test set
- 非 2025 場景 → 按 `(location, year)` 分層，月份排序後 round-robin 分 fold

### 結果

```
輸出目錄：/mnt/backup/oil_dataset/new/full_band/data_split/5_fold/scene_level/
  test_fold1~5.txt：66 個場景（內容完全相同）
  train_fold1.txt：132 scenes
  train_fold2.txt：132 scenes
  train_fold3.txt：119 scenes
  train_fold4.txt：132 scenes
  train_fold5.txt：132 scenes
  val_fold1.txt：22 scenes
  val_fold2.txt：22 scenes
  val_fold3.txt：35 scenes
  val_fold4.txt：22 scenes
  val_fold5.txt：22 scenes
總計：66（test）+ 154（非 2025，train+val）= 220 場景
```

---

## Step 4：Patch 座標生成（GB1.0）

### 新腳本

**`preprocess/generate_patch_coords.py`**

與原本的 `patch_from_stacked_tif_0421.py` 不同，本腳本不切實體 patch 圖，而是生成座標 TXT（`{SCENE_NAME}_patch_x{x}_y{y}` 格式），供 VRT 模式動態讀取用。

**關鍵參數：**

| 參數 | 值 |
|---|---|
| PATCH_SIZE | 256 |
| STRIDE | 192（overlap=64） |
| OIL_TO_BG_RATIO | 1.0（GB1.0） |
| MIN_ANNOTATED_RATIO | 0.7（BG patch 至少 70% 已標注） |
| MASK_DIR | `/home/alanyh/oil_dataset/new/full_band/mask`（395 個 TIF） |

**Oil patch 判斷**：至少 1 個像素值為 0（oil）
**BG patch 判斷**：無 oil 且標注像素比例 ≥ 70%

### 結果（Fold 1）

```
輸出目錄：/mnt/backup/oil_dataset/new/full_band/data_split/5_fold/patch_level_GB1.0/
  train_fold1.txt：2044 patches（oil + bg 1:1）
  val_fold1.txt：362 patches
  test（所有 fold）：1334 patches
```

---

## Step 5：YAML 更新

`main/experiments_CV.yaml` 最終狀態：

```yaml
_paths:
  stacked_tif_dir: &stacked_tif_dir '/mnt/backup/oil_dataset/new/full_band/MS6_sen2like_vrt'

architecture_cfg:
  in_channels: 8        # ← 從 11 改為 8

cross_validation:
  num_folds: 1          # ← 試跑改 1，正式跑改回 5
  split_dir: '/mnt/backup/oil_dataset/new/full_band/data_split/5_fold/patch_level_GB1.0'

dataset:
  labels_dir: '/home/alanyh/oil_dataset/new/full_band/mask'
  vrt_dir: *stacked_tif_dir
  read_images_from_vrt: true
  pixel_mapping: {0: 0, 1: 1, 2: 1, 255: 1}

post_tests:
  reconstruction:
    vrt_dir: *stacked_tif_dir
    scene_list_txt: '/mnt/backup/oil_dataset/new/full_band/data_split/5_fold/scene_level'
    vis_image_dir: '/mnt/backup/oil_dataset/new/full_band/NIR_R_G_Output_png/test_ms6'
    json_dir: '/home/alanyh/oil_dataset/new/full_band/JSON_ms6'
```

### Reconstruction 額外處理

`reconstruct_module` 的 `fuzzy_find_file()` 使用非遞迴 glob，無法掃描巢狀目錄。新 MS6 的 JSON 存在各場景子資料夾內，因此建立了一個**扁平化 symlink 目錄**：

```
/home/alanyh/oil_dataset/new/full_band/JSON_ms6/{SCENE_NAME}.json
```
共 221 個 symlink，指向各場景資料夾內的實際 JSON 檔案。

---

## 問題紀錄

### 試跑 Fold 1 卡住

**現象：**
- 兩個 Python process（PID 20051、20136）執行超過 1 小時，GPU 使用率始終 0%
- CPU 累計時間只有 5 秒（代表幾乎沒有做任何計算）
- SIGTERM/SIGKILL 均無效（可能是 D-state 或容器 PID 命名空間問題）
- 需要重啟 Docker container 才能清除

**根本原因（待確認）：**

發現一個重要的**資料單位 Bug**：

MS6_sen2like TIF 的原始像素值為整數反射率格式（`raw value × 10000`），例如 B01 band mean ≈ 227（對應反射率 0.0227）。

但 `deeplab_adapter.py` 的 `_get_pos_vrt_item` 和 `_auto_calculate_stats_vrt` 在讀取後有：
```python
patch[patch > 100.0] = 0.0   # 原意是清除 nodata fill（9.97e36）
```

此 clip 針對 0–1 範圍的反射率設計；但 raw 值在 0–10000 範圍，導致**幾乎所有有效像素都被歸零**。

**正確做法**：讀取後除以 10000：
```python
patch = src.read(...).astype(np.float32) / 10000.0
```

這與 `generate_nirRG_png.py` 的做法一致（`/ 10000.0`）。

**卡住的直接原因尚未完全確認**，可能是：
1. 資料全歸零後 Normalize 的 std ≈ 0 → 除零或 inf 值引發 hang
2. DataLoader multiprocessing 在 rasterio 讀取時的 fork deadlock

---

## Bug 2：VRT 解析度與 Mask 不對齊（2026-05-21 發現）

### 現象

修正 Bug 1、重建 VRT、重啟訓練後，62 個 epoch 的 `val_iou_Oil` 始終為 0。

### 根本原因

`build_vrt_ms6.py` 以 **B01（60m，1830×1830）** 作為 VRT 參考波段：

```python
with rasterio.open(str(band_files[0])) as src:  # band_files[0] 是 B01
    h, w = src.height, src.width   # 1830 × 1830
```

導致 VRT 解析度為 **60m（1830×1830）**，但：

| 檔案 | 解析度 | 像素座標系 |
|------|--------|-----------|
| Mask TIF | 10m (10980×10980) | 最大座標 10979 |
| Patch TXT 座標 | 10m 像素空間 | x,y 最大 ~9500 |
| VRT（舊） | 60m (1830×1830) | 最大座標 1829 |

Patch 座標（如 x=7296, y=5184）遠超 VRT 邊界 → `rasterio` `boundless=True` 填 0 → **模型訓練的所有影像都是空白（全零）**。

### 驗證

```python
# 在已知有 63419 個油污像素的 patch 讀取影像
patch = src.read(window=Window(7296, 5184, 256, 256), ...)
# 舊 VRT（60m）：min=0, max=0  ← 全零！
# 新 VRT（10m）：mean≈0.13      ← 正常反射率
```

### 修復

**`preprocess/build_vrt_ms6.py`**：以 `band_files[1]`（B02，10m）為 VRT 參考，並為每個波段讀取其實際尺寸作為 SrcRect，VRT 尺寸作為 DstRect：

```python
# 修改前（錯誤）
with rasterio.open(str(band_files[0])) as src:   # B01, 60m
    h, w = src.height, src.width  # 1830×1830

_BAND_TMPL: SrcRect={w}×{h}, DstRect={w}×{h}  # 所有 band 用同一尺寸

# 修改後（正確）
with rasterio.open(str(band_files[1])) as src:   # B02, 10m
    h, w = src.height, src.width  # 10980×10980

# 每個 band 讀其真實尺寸作 SrcRect，VRT 尺寸作 DstRect
for band_file in band_files:
    with rasterio.open(band_file) as src:
        src_w, src_h = src.width, src.height
    # SrcRect=(0,0,src_w,src_h), DstRect=(0,0,10980,10980)
    # GDAL 自動處理 resample（B01 6x upsample、B8A/B11/B12 2x upsample）
```

VRT_OUT_DIR 同時從 `/mnt/backup/.../vrt` 改為直接輸出到本機 SSD：`/home/alanyh/oil_dataset/new/full_band/MS6_sen2like_vrt`。

### 處理步驟

1. 刪除 220 個舊 VRT（`find ... -name "*.vrt" -delete`）
2. 執行修正後的腳本重建（約 35 秒，220 成功 / 1 失敗（B02 缺失）)
3. 重算 mean/std（只計算非零像素，排除 nodata fill）

### 修正後的 mean/std（2026-05-21，500 patch 抽樣）

| 波段 | mean | std |
|------|------|-----|
| B01 | 0.190978 | 0.136939 |
| B02 | 0.184417 | 0.130860 |
| B03 | 0.177819 | 0.129367 |
| B04 | 0.171368 | 0.131349 |
| B08 | 0.171265 | 0.136126 |
| B8A | 0.169873 | 0.131062 |
| B11 | 0.151570 | 0.072673 |
| B12 | 0.145186 | 0.062876 |

（舊值 mean≈0.005 是因為大部分像素讀出全零）

---

## 效能優化 Round 1：DataLoader file handle cache（2026-05-21）

### 現象

VRT 修復後訓練有效（Oil IoU ≠ 0），但每個 epoch 約 7–9 分鐘（300 epoch ≈ 35–45 小時）。
診斷：GPU 使用率 0%，8 個 DataLoader workers 各佔 ~49% CPU → 資料讀取是瓶頸。

### 第一層原因（已修）：每 epoch 重建 worker process

PyTorch DataLoader 預設 `persistent_workers=False`，每個 epoch 結束就殺掉 worker process，module-level `_ds_cache` 清空。

**修復（`main/deeplab_adapter.py`）：**

```python
_ds_cache: dict = {}

def _cached_rasterio_open(path: str):
    if path not in _ds_cache:
        _ds_cache[path] = rasterio.open(path)
    return _ds_cache[path]
```

`_get_pos_vrt_item` 改用 `_cached_rasterio_open()`；DataLoader 加 `persistent_workers=True`。

### 效果

- 有改善，但 epoch 仍約 8–9 分鐘，GPU 仍 0%
- 表示瓶頸不在 file open 次數，而在讀取本身

---

## 效能優化 Round 2：TIF 格式根本原因診斷（2026-05-22）

### Benchmark

```python
# 讀取 256×256 window from B03.tif（10m, 10980×10980）
B03 strip read (10 次平均): 397ms
B03 COG tiled read (10 次平均): 43.6ms  ← 9× 加速
```

### 根本原因

MS6_sen2like TIF 的內部結構：

```
block_shapes: [(1, 10980)]   ← strip 格式：每行儲存一條
compress: LZW
```

讀取一個 256×256 的 patch → GDAL 必須解壓 256 條 strip，每條 10980 pixels → 實際 I/O 是需求量的 **43 倍**。

| 波段 | 格式 | 每 sample 讀取時間 |
|------|------|------------------|
| B01 (60m, 1830×1830) | strip | ~46ms（小檔，尚可）|
| B02/03/04/08 (10m) | strip | ~330ms 各 ← 主要瓶頸 |
| B8A/11/12 (20m, 5490×5490) | strip | ~115ms 各 |
| **VRT 8 band 合計** | | **~1600ms/sample** |

**舊的「1:20/epoch」是虛假速度**：60m VRT 版本所有 patch 讀出全零（座標超出邊界），根本沒有真正的 I/O。

### 修復：in-place COG 轉換

將所有 B02/03/04/08（10m）及 B8A/11/12（20m）的 TIF 轉成 **256×256 tile 的 COG**（Cloud Optimized GeoTIFF）。

- **Lossless**：DEFLATE + predictor=2，uint16 資料完整保留
- **VRT 路徑不變**：in-place 轉換（tmp → rename），VRT 與 deeplab_adapter.py 不需修改
- **轉換腳本**：`/home/alanyh/oil_dataset/new/full_band/convert_to_cog.py`
  - 8 workers 並行，每檔寫入 `.cog_tmp.tif` 後驗證 pixel 值一致再 rename
  - Log：`/home/alanyh/oil_dataset/new/full_band/cog_convert.log`

### 預計效果

| | 現況 | COG 後（估算）|
|---|---|---|
| B02/03/04/08 各 | ~330ms | ~44ms |
| B8A/11/12 各 | ~115ms | ~12ms |
| **每 sample 合計** | ~1600ms | ~258ms |
| **epoch 時間** | ~9 min | **~2 min（估算）** |

### 轉換結果（2026-05-22 完成）

- **1546 / 1546 成功，0 失敗**
- 耗時：234 分鐘（3.9 小時）
- 所有檔案逐一 pixel 驗證通過（`np.array_equal`）後才 rename
- 實測讀速：B03 397ms → **43ms（9.2×）**；VRT 8-band 1600ms → **450ms（3.6×）**

---

## 訓練 Fold 1（COG 後，2026-05-22 重啟）

### 設定變更

- 加入 `class_weights: [13.0, 1.0]`（Oil:BG 像素比 1:13.8，inverse frequency）
- 舊 checkpoint 清除，從頭訓練
- 其他設定不變：`num_folds=1`, `epochs=300`, `patience=50`, `workers=8`, `batch=16`, `lr=5e-5`

### Epoch 速度

~3 min/epoch（vs COG 前 9 min，vs strip TIF 假速度 1:20）

**GPU 觀察**：使用率 0%，VRAM 僅 6.4/32GB → 訓練仍為 I/O bound，GPU 持續在等資料。

### 訓練結果

| Epoch | Oil IoU | BG IoU | Val mIoU | Best |
|-------|---------|--------|---------|------|
| 1 | 0.028 | 0.758 | 0.393 | ★ |
| 2 | 0.049 | 0.935 | 0.492 | ★ |
| 3 | 0.041 | 0.808 | — | |
| 4 | 0.074 | 0.896 | — | |
| 5 | 0.124 | 0.960 | 0.542 | ★ |
| 6 | 0.111 | 0.959 | — | |
| 7 | 0.127 | 0.940 | — | |
| 8 | 0.164 | 0.953 | 0.558 | ★ |
| 9 | 0.115 | 0.921 | — | |

class_weights 有效：Epoch 1 就出現 Oil IoU > 0（上一版無 class_weight 要到 Epoch 3 才出現）。

---

## 目前狀態（2026-05-22）

- Fold 1 訓練中（Epoch 9+），GPU I/O bound，epoch ~3 min
- 下一步優化方向：AMP（GPU 未啟用 fp16）、pre-stacked 8-band COG（消除 VRT 多檔開銷）

---

## 下一步

1. ✅ COG 轉換完成
2. ✅ Fold 1 重啟（class_weights 加入）
3. 評估是否啟用 AMP + 增大 batch_size
4. Fold 1 完成後 `num_folds: 5` 啟動完整 5-fold CV
