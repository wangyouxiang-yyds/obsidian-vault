---
date: 2026-07-02
type: experiment
stage: training
status: running
tags: [gt-expand, deeplabv3+, 3-fold-cv, oil-detection, 358clean, gt-centric]
---

# CV_358clean_gt_expand 訓練進度（2026-07-02）

## 目標

驗證 GT-centric patch 策略（以油汙 GT bbox 中心 ±128 取 256×256 patch，每個 patch 保證含油汙）下，DeepLabV3+ 在 per-patch IoU 評估上的表現，作為監督式分割效能上限的參考。

---

## 設定

| 項目 | 設定 |
|------|------|
| 實驗名稱 | CV_358clean_gt_expand |
| 模型 | DeepLabV3+（ResNet-50, 8-ch, **ImageNet pretrained**）|
| 資料集 | 358 clean scenes，3-fold CV |
| Patch 策略 | GT-centric（±128 bbox center，共 2,897 patches）|
| Epochs | 300 |
| Patience | 25（已從 0422 的 50 縮短）|
| class_weights | **[1.0, 1.0]**（已重校：GT_expand 油汙佔比 ~42:58，無需加權）|
| 實驗記錄格式 | per-fold run_metrics.json |
| 結果彙整 | result_CV_358clean_gt_expand_log_restructured.xlsx |

---

## 進度總覽（截至 2026-07-02）

| Fold | 狀態 | 完成時間 | Best Val mIoU | Oil IoU | Background IoU |
|------|------|---------|--------------|---------|----------------|
| fold1 | 完成 | 2026-07-01 | （待補）| （待補）| （待補）|
| fold2 | 完成 | 2026-07-01 | （待補）| （待補）| （待補）|
| fold3 | **進行中** | — | ≈0.633（epoch 43/300）| ≈0.30 | ≈0.96 |

> fold1/2 的具體指標數字待使用者從 run_metrics.json 補入。

---

## Code Review 實況（已對照 code 驗證）

> **重要**：repo 內 `TODO/` 目錄的 markdown 文件（00/01/02）撰寫後**未回頭更新，已過時**。以下以實際 code 驗證結果為準。

| 項目 | 狀態 | 備註 |
|------|------|------|
| C1 seed | ✅ 已修 | `set_global_seed()` + 每 fold `base_seed+fold_idx`，cudnn deterministic |
| C2 mean/std | ✅ 已修 | `_auto_calculate_stats_vrt` 從 GT_expand 座標重算，不再沿用 0422 污染版 |
| C3 collate None | ✅ 已修 | `oil_collate_fn` 永遠掛上，加「整批皆 None → return None」防護 |
| C4 pixel_mapping | ⚠️ 刻意 revert | YAML 標記 `[revert-C4]`，255:1 ignore 算 bg；有效 pixel 12%→100%，與 eval 對齊。**評估後刻意保留，非未做** |
| C5 RandomResizedCrop 過裁 | ✅ 已修 | scale 收緊至 0.9，降頻率 |
| C6 eval else 靜默丟棄 | ✅ 已修 | else 改 raise，shape 不符不再靜默跳過 |
| C7 eval_protocol 欄位 | ✅ 已修 | `main_runner.py` 加評估協議標註 |
| class_weights 重校 | ✅ 已做 | [13,1] → [1.0,1.0]（油汙佔比 ~42:58，無需加權）|
| patience 調整 | ✅ 已做 | 50 → 25 |
| A1 ImageNet backbone | ✅ 已做 | `torchvision_weights: 'DEFAULT'` |
| 02 公平比較協議 | 部分 / 待動工 | C7 eval_protocol 欄位已加；跨版本完整比較協議尚未確認 |

---

## 初步觀察

- fold3 epoch 43 的 Oil IoU ≈ 0.30，Background IoU ≈ 0.96；需等完整 3-fold 平均才有意義。
- Background IoU 高（0.96）在 GT-centric 分布下需謹慎解讀：patch 本身油汙佔比高，背景像素相對少，高 BG IoU 不等於整圖上的 BG 辨識能力。
- 本版本已啟用 ImageNet pretrained backbone + class_weights=[1,1] + patience=25，設定與 0422 baseline 有根本差異，數字**不可直接比較**（評估分母也不同），需等公平比較協議設計完成。

---

## 下一步

- [ ] fold3 訓練完成後，補上三個 fold 的完整 metrics（val mIoU / Oil IoU / BG IoU / best epoch）
- [ ] 完成跨版本公平比較協議設計（GT_expand per-patch IoU vs 0422 整圖 IoU 的可比性）
- [ ] 視 fold3 結果決定是否需針對 C4 revert 做敏感度測試（即加回 ignore mask 後重跑一 fold 比較）

---

## 實驗檔案位置

| 項目 | 路徑 |
|------|------|
| repo 根目錄 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/` |
| 結果目錄 | `result-seg/CV_358clean_gt_expand/` |
| 彙整 Excel | `result_CV_358clean_gt_expand_log_restructured.xlsx` |
| TODO 文件目錄 | `TODO/`（00/01/02 三份）|

---

## 相關頁面

- [[GT_expand_pipeline]] — GT_expand 版本 pipeline 說明、與 0422 策略差異表、TODO 適用性分析
- [[VRT_pipeline_02_模型訓練]] — 主線 0422 訓練設定（本實驗繼承大部分超參數）
- [[deeplabv3plus_358clean_overfitting_改善方向]] — C1~C4 bug fix 以及 A1 pretrained backbone 改善方向
