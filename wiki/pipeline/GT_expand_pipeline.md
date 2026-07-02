---
date: 2026-07-02
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

---

## 二、Pipeline 架構對比

| 維度 | 版本 A（0422 VRT training）| 版本 B（GT_expand）|
|------|--------------------------|---------------------|
| repo | OIL_PROJECT_MutiBand_0422_VRT_training | OIL_PROJECT_MutiBand_GT_expand |
| Patch 策略 | 全圖 sliding-window（stride 切）| GT-centric：以油汙 bbox 中心 ±128 取 patch |
| Patch 數量 | ~5 萬量級，油汙 patch 稀少 | 2,897 個 patch，每個都含油汙 |
| 資料讀取 | VRT 動態讀取（rasterio.Window）| 同左 |
| 評估方式 | reconstruct_module 拼回大圖，整圖 IoU（含大量 BG）| per-patch IoU on oil-bearing patches |
| post_tests | reconstruct_module | []（不做重組）|
| class_weights | [13, 1]（適配稀疏 patch）| **[1.0, 1.0]**（已重校：GT_expand 油汙佔比 ~42:58，接近自然分佈，無需加權）|
| 論文角色 | **主軸**（整圖偵測、真實部署情境）| **探索**（效能上限估計、分布分析）|

---

## 三、GT-centric Patch 取法

```
油汙 GT Mask（全場景 TIF）
    ↓ connected-component analysis
    ↓ 取每個 component 的 bbox 中心
    ↓ 以中心為基準 ±128 pixel → 256×256 patch
    ↓ 邊界處理：超出影像範圍 → clamp 至有效區域
```

**關鍵特性**：
- 每個 patch 保證含至少一個油汙 component
- oil-pixel 佔比顯著高於 sliding-window patch
- 訓練集共 2,897 個 patch（358 clean scenes，3-fold CV）

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

- 主要指標：**per-patch IoU on oil-bearing patches**
- 注意：GT_expand 的 Oil IoU 數字不可直接與 0422 的整圖 IoU 比較（分母不同）
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
- [[VRT_pipeline_01_前處理]] — 共用前處理（VRT 建立、場景、Mask）
- [[VRT_pipeline_02_模型訓練]] — 主線訓練 pipeline 詳細設定（本 fork 大部分繼承）
- [[deeplabv3plus_358clean_overfitting_改善方向]] — A1/A2/C1~C4 改善方向分析（兩版本共用）
- [[20260702_CV_358clean_gt_expand_進行中]] — 當前實驗現況
