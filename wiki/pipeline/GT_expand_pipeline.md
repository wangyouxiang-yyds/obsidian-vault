---
date: 2026-07-02
updated: 2026-07-10
type: pipeline
tags: [gt-expand, deeplabv3+, oil-detection, gt-centric, patch-strategy, 3-fold-cv]
project: OIL_PROJECT_MutiBand_GT_expand
---

# GT_expand Pipeline — GT-centric Patch 策略說明

> 本頁說明 `OIL_PROJECT_MutiBand_GT_expand` 的完整 pipeline 設計，並與論文主線 [[OIL_PROJECT_MutiBand_0422_VRT_training]] 的 sliding-window 策略作對比。

---

## 一、為什麼 fork？問題背景

主線 0422 版本（sliding-window + 全圖重組）存在嚴重的類別不平衡問題：
- 整張大圖按 stride 切 patch，約 5 萬量級，油汙 patch 極稀少、多為純背景
- class_weights=[13,1] 雖然補償，但模型仍大量接觸「幾乎全背景」的 patch
- 重組評估（含大量 BG）的 IoU 受背景主導

GT_expand 的出發點：**每個 patch 都包含油汙**，讓模型集中學油汙邊緣的辨識，估計監督式分割的效能上限。

> ⚠️ **2026-07-10 更新**：此假設已不成立。後續加入了背景 patch 機制（見第三節 b），訓練集實際混有純背景 patch，不再是「每個 patch 都含油汙」。

---

## 二、Pipeline 架構對比

| 維度 | 版本 A（0422 VRT training）| 版本 B（GT_expand）|
|------|--------------------------|---------------------|
| repo | OIL_PROJECT_MutiBand_0422_VRT_training | OIL_PROJECT_MutiBand_GT_expand |
| Patch 策略 | 全圖 sliding-window（stride 切）| GT-centric：bbox 中心取 patch，大 bbox 改 256-grid 鋪磚（見第三節）|
| Patch 數量 | ~5 萬量級，油汙 patch 稀少 | 2,893 個 patch（每 fold train+val+test 皆 2,893；~~原每個都含油汙~~ 已混入背景 patch，見第三節 b）|
| 資料讀取 | VRT 動態讀取（rasterio.Window）| 同左 |
| 評估方式 | reconstruct_module 拼回大圖，整圖 IoU（含大量 BG）| **每 fold 訓練完自動跑 gt_aware reconstruction（post_test）**，主指標 = recon pooled_oil_iou + per-scene oil IoU（詳見第五節）|
| post_tests | reconstruct_module | **gt_aware reconstruction**（~~[]（不做重組）~~ 已於 2026-07 前後啟用，此欄過時）|
| class_weights | [13, 1]（適配稀疏 patch）| baseline **[1.0, 1.0]**；~~已重校：GT_expand 油汙佔比 ~42:58，接近自然分佈，無需加權~~ 此前提已被背景 patch 機制打破（實測 oil 像素佔比僅 ≈0.059），現有 recall 導向的 cw31=[3,1] 實驗線（見第三節 c）|
| 論文角色 | **主軸**（整圖偵測、真實部署情境）| **探索**（效能上限估計、分布分析）|

---

## 三、GT-centric Patch 取法

**腳本**：`preprocess/generate_patch_coords_gt_expand.py`

```
油汙 GT Mask（全場景 TIF）
    ↓ cv2.connectedComponentsWithStats 找每個 component 的 bbox
    ↓ bbox ≤ 256 px  → 取 bbox 中心 1 個 256×256 patch
    ↓ bbox > 256 px  → 用 256-grid 鋪磚，覆蓋整個 bbox（多個 patch）★"expand" 之名由來
    ↓ 座標 clip 到影像邊界
    ↓ 排除帶 dirty.txt / blur.txt 標記的髒場景
```

輸出格式為檔名編碼 `{SCENE_NAME}_patch_x{COL}_y{ROW}`（非獨立的 x/y/w/h 座標欄位）。

> ⚠️ 舊版本文描述「以 bbox 中心 ±128 clamp」只涵蓋了 bbox ≤256 的情況，遺漏了大油汙 bbox 會用 256-grid 鋪磚展開成多個 patch 這半套機制——這正是 GT_expand 名稱中「expand」的來源，2026-07-10 補上。

> ⚠️ **2026-07-10 硬化**：`generate_patch_coords_gt_expand.py` 與 `generate_bg_patches.py`（見第三-b節）的 `SPLIT_DIR`/`OUT_DIR` 已改為**必填環境變數**，裸跑會直接報錯退出。理由：舊版預設值指向已過期的 `5_fold` split，裸跑會靜默用過期 split 重生 patch 座標（footgun），已封堵。

**關鍵特性**：
- 每個 oil-bearing patch 保證含至少一部分油汙 component
- oil-pixel 佔比顯著高於 sliding-window patch
- 訓練集共 2,893 個 patch（每 fold train+val+test 皆 2,893；358 clean scenes，3-fold CV；~~原 2,897~~ 已於 2026-07-10 依現行 code 校正）
  - fold1 = 1,418 / 560 / 915（train/val/test）
  - fold2 = 1,738 / 294 / 861
  - fold3 = 1,436 / 340 / 1,117

### 三-b：背景 patch 機制（2026-07-10 補充，原本文未涵蓋）

上面的取法只涵蓋含油汙的 patch。實際訓練資料另外混入了純背景 patch，「每個 patch 都含油汙」的原始設計假設已不成立：

**腳本**：`preprocess/generate_bg_patches.py`

- 在同場景油汙範圍**外**滑窗取樣（256×256, stride 512）
- 篩選條件：零 oil 像素、valid 背景比例 ≥30%、ignore 比例 <50%
- 輸出：3,510 筆座標 → `data_split/3_fold_stratified_v2/patch_level_gt_expand_bg/bg_coords.tsv`
- 由 `deeplab_adapter.py` 的 `bg_coord_txt` 參數 + scene_filter 機制，按 fold 場景混入訓練集

**實測（fold1 train）**：pos 1,168 / neg 2,110 ≈ 1 : 1.8，油汙像素佔比 ≈ 0.059（遠低於舊筆記估計的 42:58 自然分佈）。

### 三-c：class_weights / recall 導向實驗線（2026-07-10 補充）

baseline 仍用 `[1.0, 1.0]`，但既然真實 oil 像素佔比只有 ≈0.059，加權可能有空間，因此衍生出兩條實驗線：

| 實驗 | 設定 | 現況（2026-07-10）|
|------|------|------|
| cw31 | `class_weights: [3.0, 1.0]` | 三折已跑完：pooled_oil_iou = **0.394**，三折同向勝 baseline（0.332）。細看是 **trade-off 而非全面提升**：大油汙（>50k px, n=51）顯著變好（p=0.032，recall 0.474→0.608）；tiny 油汙（<1k px）顯著變差（p=0.007）；整體 per-scene Wilcoxon p=0.643（未達顯著）|
| tversky | region-based loss，α=0.3 / β=0.7 | 跑步中，尚無結果 |

> **2026-07-11 補充**：tversky 三折已跑完，pooled_oil_iou = **0.3915**，三個實驗線（baseline 0.332 / cw31 0.394 / tversky 0.3915）中**唯一同時通過雙 gate**（per-scene Wilcoxon p=9.5e-14），判定優於 cw31（cw31 均值雖高但 Wilcoxon 不顯著且 tiny 油汙場景顯著變差）。已成為標竿 loss 設定，帶入下一輪架構消融（`tversky_v3plus`）。α=0.3/β=0.7 這組超參數與原始文獻 [[Salehi2017_TverskyLoss]]（Salehi et al. 2017）在 MS 病灶分割任務上實測的最佳點一致，見該文獻頁與 [[分割損失函數與類別不平衡.md]] 概念頁的交叉整理。

### 三-d：Split 策略（3_fold_stratified_v2，現行主 split，2026-07-10 補充）

**腳本**：`preprocess/resplit_stratified_v2.py`

```
排除髒場景（dirty.txt / blur.txt）
    ↓ 讀 mask 算每場景 oil pixel count
    ↓ 依面積分 3 個 tertile（分層依據）
    ↓ StratifiedKFold(3)
    ↓ 每 fold 的 train+val 再依 80/20 分層抽（seed 42）
```

輸出目錄：`/mnt/backup/oil_dataset/new/full_band/data_split/3_fold_stratified_v2/`

---

## 四、Code Review 實況（已對照 code 驗證）

> **注意**：repo 內 `TODO/` 目錄的 markdown 文件（00/01/02）撰寫後未回頭更新，**已過時**，不反映 code 實況。以下以實際 code 驗證結果為準。

### 00_code_review（7 項）— 幾乎全做了

| 代號 | 問題描述 | 狀態 | 說明 |
|------|---------|------|------|
| C1 | seed 缺失 | ✅ 已修 | `main_runner.py` 有 `set_global_seed()`，每 fold 用 `base_seed+fold_idx`，cudnn deterministic，DataLoader seed-aware |
| C2 | mean/std 沿用 0422 污染版 | ✅ 已修 | `deeplab_adapter` 加了 nan>10%/0值 過濾；`_auto_calculate_stats_vrt` 從 GT_expand 自有座標重算，不再繼承 0422 stats |
| C3 | collate None | ✅ 已修 | `oil_collate_fn` 永遠掛上，並加「整批皆 None → return None」防護 |
| C4 | evaluation else 走 YOLO | ⚠️ 刻意 revert | YAML 標記 `[revert-C4]`，維持 255:1（ignore 也算 bg），理由：訓練有效 pixel 12%→100% 且與 eval 指標對齊。**非未做，而是評估後刻意保留** |
| C5 | RandomResizedCrop 過裁 | ✅ 已修 | scale 從 0.75 收緊至 0.9，並降頻率（避免把 bbox 貼邊的油汙 crop 掉）|
| C6 | eval else 靜默丟棄 | ✅ 已修 | `evaluation_module` else 改 raise；shape 不符不再靜默跳過 |
| C7 | eval_protocol 欄位缺失 | ✅ 已修 | `main_runner.py` 加了評估協議標註 |

### 01_distribution_shift_recheck — 實質已做

| 項目 | 0422 設定 | GT_expand 實際設定 |
|------|----------|--------------------|
| class_weights | [13.0, 1.0] | **[1.0, 1.0]**（GT_expand 油汙佔比 ~42:58，接近自然分佈）|
| patience | 50 | **25**（GT_expand patch 較少，早收斂）|

### A1 ImageNet 預訓 backbone — 已做

YAML 設定 `torchvision_weights: 'DEFAULT'`，已啟用 ImageNet pretrained ResNet-50。

### 02_eval_protocol_comparison — 部分

C7 的 eval_protocol 欄位標註已完成；與 0422 sliding-window 的完整「公平比較協議」設計（跨版本 IoU 可比性）尚未確認完成，仍待動工。

### 分布改變後不再需要的項目

| 0422 TODO 項目 | GT_expand 中的情況 |
|---------------|-------------------|
| A4 hard-positive oversampling | 所有 patch 已是 hard positive，**無需** |
| TTA / sliding-window prob 混用 | 暫不適用（無重組流程）|

---

## 五、評估協議說明

> **2026-07-10 更新**：評估方式已全面改變，以下取代舊版「per-patch IoU on oil-bearing patches」敘述。

- 每個 fold 訓練完成後，自動觸發 **gt_aware reconstruction（post_test）**，把 patch 級預測拼回場景做評估
- **主要指標**：recon **pooled_oil_iou** + per-scene oil IoU；~~per-patch IoU on oil-bearing patches~~ 降為輔助指標
- **正式驗收採雙 gate**：
  1. `pooled_oil_iou > 0.362`
  2. per-scene 配對 Wilcoxon 檢定 vs baseline，p<0.05 且方向為正
- **Baseline 三折基準**（run `20260701-133158` / `20260701-204746` / `20260702-032032`）：pooled_oil_iou = 0.3695 / 0.3119 / 0.3157，mean = **0.332**
- 注意：GT_expand 的 IoU 數字仍不可直接與 0422 的整圖 IoU 比較（分母不同）
- 與 0422 公平比較的完整協議尚未完成（`TODO/02_eval_protocol_comparison.md` 內容已過時，見第四節說明）

---

## 六、實驗記錄格式

- 結果目錄：`result-seg/CV_358clean_gt_expand/`（在 repo 根目錄下）
- 記錄格式：per-fold `run_metrics.json`
- 彙整腳本：`restructure_excel.py` → `result_CV_358clean_gt_expand_log_restructured.xlsx`

當前進行中的實驗見 [[20260702_CV_358clean_gt_expand_進行中]]。

---

## 七、相關頁面

- [[OIL_PROJECT_MutiBand_0422_VRT_training]] — 主線版本入口索引
- [[VRT_pipeline_01_前處理]] — 共用前處理（VRT 建立、場景、Mask、Dataset Manifest 制度）
- [[ms6_sen2like]] — MS6 資料集現況（manifest 盤點數字、2026 新場景入庫記錄）
- [[VRT_pipeline_02_模型訓練]] — 主線訓練 pipeline 詳細設定（本 fork 大部分繼承）
- [[deeplabv3plus_358clean_overfitting_改善方向]] — A1/A2/C1~C4 改善方向分析（兩版本共用）
- [[20260702_CV_358clean_gt_expand_進行中]] — 當前實驗現況
