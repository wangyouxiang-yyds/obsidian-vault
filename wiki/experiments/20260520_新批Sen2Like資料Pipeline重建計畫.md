---
date: 2026-05-20
type: experiment
stage: preprocess
status: planning
tags: [sen2like, deeplabv3+, patching, fold-split, GB1.0, 8band]
---

# 新批 Sen2Like 資料完整 Pipeline 重建計畫

## 背景與動機

引入新一批 Sen2Like 處理後的多光譜衛星影像，同時部分 JSON 標注已有修改。這兩件事都導致現有的 patch split TXT 無法沿用，必須從 mask 生成開始全部重跑。

本次同步調整 fold 分割策略，確保訓練集設計更嚴謹、各 fold 難度更均衡。

> ⚠️ **重要：波段數從 11 改為 8**
> 新批 Sen2Like 只輸出 B01/02/03/04/08/8A/11/12（8 個波段），不含原本的 B05/B06/B07（紅邊波段）。全流程中凡涉及波段設定的地方均需更新，詳見各步驟說明。

---

## 本次設計原則

| 項目 | 設定 |
|---|---|
| 模型 | DeepLabV3+ |
| 輸入波段數 | **8 bands**（B01/02/03/04/08/8A/11/12） |
| Oil : Background 比例 | **1 : 1**（GB1.0） |
| Test set | **2025 年全部場景**（5 個 fold 共用固定 test set） |
| Train / Val | 非 2025 年資料做 5-fold 輪換 |
| 分組單位 | **(地點 × 月份)**，同地點同月份必須整包在同一側，不可拆散 |
| 平衡目標 | 每個 fold 的 Gulf/Atlantic 比例、2019/2020/2024 年份比例均勻 |

### 為什麼以月份為分組單位？

同一地點在同一個月份內的多次衛星過境，極可能是在觀測同一次油汙事件（只是不同日期或不同衛星）。若這些場景被拆到 train 和 test 兩側，模型可能「認得」同一事件的背景紋理，造成測試指標虛高。

因此分組單位定為 **(location, year_month)**，例如 `Gulf of Mexico 2025-06` 的所有場景整包進 test，不拆散。

### 為什麼 2025 年作為 test？

2025 年的資料是最新一批（含 S2C 新衛星），與訓練資料在時間上完全隔離，最能反映模型對未見過資料的真實泛化能力。

---

## 完整流程

```
GPKG 標注（新場景）
    ↓ Step 0：gpkg_to_labelme.py（以 B02 為 ref_tif）
JSON 標注（統一格式）
    ↓ Step 1：json_to_mask_tif.py
全場景 mask TIF（重新生成）
    ↓ Step 2：stack_multiband_tif.py（8-band）+ build_vrt.py（新場景就位後）
8-band VRT 虛擬影像
    ↓ Step 3：build_scene_splits_stratified.py（新腳本）
scene_level fold TXT（stratified 分配）
    ↓ Step 4：patch_from_stacked_tif_0421.py（OIL_TO_BG_RATIO=1.0）
patch_level_GB1.0 fold TXT（座標清單）
    ↓ Step 5：main_runner.py + experiments_CV.yaml
DeepLabV3+ 5-fold CV 訓練
```

---

## Step 0：GPKG 標注 → LabelMe JSON 轉換

**只適用於新批資料中帶有 `.gpkg` 標注的場景；已是 `.json` 格式的舊場景直接跳過。**

使用已有腳本：`file_label_temp/gpkg_to_labelme.py`

**ref_tif**：用 Sen2Like 輸出的 **B02 單波段 TIF**（10m native，不需等 stack 完成，CRS/transform 與 stacked TIF 完全一致）。

```bash
# 對每個有 GPKG 標注的場景執行一次（場景資料夾模式）
python file_label_temp/gpkg_to_labelme.py \
    --scene_dir /path/to/scene_gpkg_folder/ \
    --ref_tif   /path/to/scene/B02.tif \
    --output    /mnt/backup/oil_dataset/new/full_band/JSON/SCENE_NAME.json
```

**檔名命名規則**（`infer_label` 自動識別）：

| GPKG 檔名 | 推斷 label |
|---|---|
| `*_oil.gpkg` | oil |
| `*_oil_1.gpkg`、`*_oil_2.gpkg` | oil |
| `*_background.gpkg` | background |

輸出放到 `/mnt/backup/oil_dataset/new/full_band/JSON/`，與舊資料 JSON 放同一目錄。

---

## Step 1：重新生成 mask TIF

**條件：JSON 標注有改動 → 必須先清除舊 mask，避免新舊版本混用。**

```bash
# 刪除舊 mask
rm -rf /mnt/backup/oil_dataset/new/full_band/mask/

# 重新生成
conda run -n yolo11 python preprocess/json_to_mask_tif.py
```

**輸出格式（v2，default=255）：**

| 像素值 | 語意 |
|---|---|
| 0 | Oil（明確標注） |
| 1 | Background |
| 2 | Others（look-alike） |
| 255 | 未標注（ignore） |

> ⚠️ 若直接沿用舊 mask TIF（default=1 版本），訓練時 ignore_index=255 的邊界處理會出錯，務必重生。

---

## Step 2：波段堆疊 + 建 VRT

**等新資料完全上傳就位後執行。**

```bash
conda run -n yolo11 python preprocess/stack_multiband_tif.py
conda run -n yolo11 python preprocess/build_vrt.py
```

- 輸出：`/mnt/backup/oil_dataset/new/full_band/vrt/*.vrt`
- 格式：**8 波段**，依波長升序排列，10m 解析度

| Index | 波長(nm) | 對應波段 |
|---|---|---|
| 1 | 443 | B01 |
| 2 | 492 | B02 |
| 3 | 560 | B03 |
| 4 | 665 | B04 |
| 5 | 833 | B08 |
| 6 | 865 | B8A |
| 7 | 1614 | B11 |
| 8 | 2202 | B12 |

> ⚠️ `stack_multiband_tif.py` 的波段清單需從原本的 11 個更新為上表這 8 個。

---

## Step 3：Stratified Scene-Level Fold Split（核心新邏輯）

**新腳本**：`preprocess/build_scene_splits_stratified.py`

現有的 `auto_split_scenes.py` 只做 random shuffle（seed=42），不符合本次需求，需要另寫新腳本。

### 演算法

**① 解析所有場景**

從 VRT 目錄掃描，對每個場景解析：
- `location`：`Gulf_of_Mexico` 或 `Atlantic_Ocean`
- `year`：場景名稱中的年份（2019/2020/2024/2025）
- `year_month`：年 + 月（例如 `202506`）

**② 建立場景組**

以 `(location, year_month)` 為 key，把同組場景收在一起。
這保證同地點同月份的場景一定整包分配，不拆散。

**③ 分離 2025 → 固定 test set**

所有 2025 年的場景組 → 直接進 test（5 個 fold 共用）。

**④ 非 2025 場景組 → Stratified 5-fold**

按 `(location, year)` 分層，每一層內依 month 排序後 round-robin 分給 5 個 fold：

```
Gulf × 2019: [G19_01, G19_02, G19_03, G19_04, G19_05, ...]
              fold:  [  1,     2,     3,     4,     5,  1, 2, ...]

Gulf × 2020: [G20_01, G20_02, ...]
              fold:  [  1,     2, ...]

Gulf × 2024: ...
Atlantic × 2019: ...
（以此類推）
```

這樣每個 fold 的 val 集都會有接近的 Gulf/Atlantic 比例與 2019/2020/2024 年份比例。

**⑤ 輸出 TXT 檔**

```
對每個 fold i（1~5）：
  train_fold{i}.txt = 屬於 fold ≠ i 的非2025場景（所有場景路徑）
  val_fold{i}.txt   = 屬於 fold == i 的非2025場景
  test_fold{i}.txt  = 所有2025場景（5個fold內容相同）
```

格式與現有 `scene_level/` TXT 相容，每行一個場景路徑：
```
NOAA_Gulf_of_Mexico_High_Confidence/NOAA_Gulf_of_Mexico_High_Confidence_20200615_S2
```

**輸出目錄**：`/mnt/backup/oil_dataset/new/full_band/data_split/5_fold/scene_level/`

### 執行後驗證

```bash
cd /mnt/backup/oil_dataset/new/full_band/data_split/5_fold/scene_level/

# 1. 確認 test 5 個 fold 內容相同
diff test_fold1.txt test_fold2.txt   # 應無差異

# 2. 確認 2025 場景未出現在 train/val
grep "2025" train_fold1.txt          # 應無輸出
grep "2025" val_fold1.txt            # 應無輸出

# 3. 確認各 fold 場景數量
wc -l train_fold*.txt val_fold*.txt test_fold*.txt

# 4. 確認 Gulf/Atlantic 分布
grep -c "Gulf" val_fold*.txt
grep -c "Atlantic" val_fold*.txt
```

---

## Step 4：切 Patch（GB1.0）

**修改參數**：`patch_from_stacked_tif_0421.py` 第 45 行

```python
# 原本
OIL_TO_BG_RATIO = 1.5

# 改為
OIL_TO_BG_RATIO = 1.0
```

**執行**：

```bash
conda run -n yolo11 python preprocess/patch_from_stacked_tif_0421.py
```

**輸出**：`/mnt/backup/oil_dataset/new/full_band/data_split/5_fold/patch_level_GB1.0/`

**切割策略**：
- Oil patch：步長 192（PATCH_SIZE 256 - OVERLAP 64），密集掃描，全保留
- BG patch：步長 256（不重疊），採樣至 oil patch 數量的 1.0 倍

**執行後驗證**：

```bash
# 確認 oil:bg 比例（腳本執行完會印出 Actual_Ratio，應接近 1.0）
wc -l patch_level_GB1.0/train_fold*.txt
```

---

## Step 5：更新 YAML 並啟動訓練

**修改** `main/experiments_CV.yaml`：

```yaml
dataset:
  split_dir: /mnt/backup/oil_dataset/new/full_band/data_split/5_fold/patch_level_GB1.0
```

**確認其他設定**：
- `architecture: deeplab`
- `in_channels: 8`（原本 11 → 改 8）
- `read_images_from_vrt: true`
- `cross_validation.enabled: true`、`num_folds: 5`
- `mean: []`、`std: []`（清空，讓程式從訓練 fold 自動重算 8 個值，算完印出後貼回 YAML 固定）

**啟動訓練**：

```bash
conda run -n yolo11 python main/main_runner.py \
    --config main/experiments_CV.yaml
```

---

## 關鍵檔案索引

| 檔案 | 動作 | 備註 |
|---|---|---|
| `file_label_temp/gpkg_to_labelme.py` | 執行 | 新 GPKG 場景轉 JSON，ref_tif 用 B02 |
| `preprocess/json_to_mask_tif.py` | 重跑 | 先清除舊 mask/ |
| `preprocess/stack_multiband_tif.py` | **修改 + 重跑** | 波段清單改為 8 個（B01/02/03/04/08/8A/11/12） |
| `preprocess/build_vrt.py` | 重跑 | 新場景就位後 |
| `preprocess/build_scene_splits_stratified.py` | **新建** | 本次核心新腳本 |
| `preprocess/patch_from_stacked_tif_0421.py` | 修改第 45 行 | `OIL_TO_BG_RATIO = 1.0` |
| `main/experiments_CV.yaml` | 修改多處 | `split_dir`、`in_channels: 8`、`mean/std` 清空 |

---

## 執行時序

| 步驟 | 等待條件 | 預估耗時 |
|---|---|---|
| Step 0（GPKG → JSON） | 無（可立即執行，只需 B02） | 數分鐘 |
| Step 1（mask 重生） | Step 0 完成 | 依場景數量 |
| Step 2（stack + VRT） | 新資料完全上傳 | 依資料量 |
| Step 3（fold split） | Step 2 完成 | 數分鐘 |
| Step 4（patching） | Step 1 + Step 3 完成 | 數小時 |
| Step 5（訓練） | Step 4 完成 | 數天 |

---

## 重新分配場景的時機

`build_scene_splits_stratified.py` 設計為可重複執行：每次執行會掃描當下所有已存在的場景，完整重建所有 fold TXT 檔。

**若後續有新場景加入或場景有異動，只需重跑 Step 3（+ Step 4）即可，不需重跑 Step 1/2。**

> ⚠️ 建議等所有新資料**完全上傳就位**後再執行 Step 3，避免因資料未齊而多次重跑。

---

## 下一步

1. 等新資料全部上傳完畢
2. 對新 GPKG 場景執行 Step 0（gpkg → JSON 轉換）
3. 實作 `build_scene_splits_stratified.py`，先在 dry-run 模式印出分配結果讓你確認
4. 確認分配結果後，依序執行 Step 1 → 2 → 3 → 4 → 5
