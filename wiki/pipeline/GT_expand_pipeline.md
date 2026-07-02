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
| class_weights | [13, 1]（適配稀疏 patch）| 繼承 [13, 1]，待重校（過度加權風險）|
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

## 四、從 0422 繼承的已知 Bug（待修清單）

> 詳見 repo 內 `TODO/00_code_review_2026-06-25.md`（動工前必修 7 項）

| 代號 | 問題 | 嚴重度 | GT_expand 是否適用 |
|------|------|--------|-------------------|
| C1 | seed 缺失（可重現性問題）| 中 | 是 |
| C2 | mean/std 直接沿用 0422 stats（繼承抽樣污染）| 高 | 是（YAML 直接沿用，等於繼承污染）|
| C3 | collate None（DataLoader 未正確 batch）| 高 | 是 |
| C4 | evaluation else 走 YOLO-style 而非分割評估 | 中 | 是 |
| — | pixel_mapping ignore 設定問題 | 低-中 | 是 |
| — | aug 過裁（crop 尺寸設定）| 低-中 | 是 |

**從 0422 TODO 不繼承的項目**（分布已改，不適用）：

| 0422 TODO 項目 | GT_expand 中的情況 |
|---------------|-------------------|
| A4 hard-positive oversampling | 所有 patch 已是 hard positive，**無需** |
| class_weights=[13,1] 設定 | 需**重校**（詳見 `TODO/01_distribution_shift_recheck.md`）|
| TTA / sliding-window prob 混用 | 暫不適用（無重組流程）|

**從 0422 TODO 仍然適用的項目**：
- A1 ImageNet pretrained backbone → 適用，優先順序不變

---

## 五、評估協議說明

- 主要指標：**per-patch IoU on oil-bearing patches**
- 注意：GT_expand 的 Oil IoU 數字不可直接與 0422 的整圖 IoU 比較（分母不同）
- 與 0422 公平比較的協議設計中，詳見 repo `TODO/02_eval_protocol_comparison.md`

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
