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
| 模型 | DeepLabV3+（ResNet-50, 8-ch, from-scratch）|
| 資料集 | 358 clean scenes，3-fold CV |
| Patch 策略 | GT-centric（±128 bbox center，共 2,897 patches）|
| Epochs | 300 |
| class_weights | [13, 1]（繼承自 0422，待重校）|
| 實驗記錄格式 | per-fold run_metrics.json |
| 結果彙整 | result_CV_358clean_gt_expand_log_restructured.xlsx |

> class_weights=[13,1] 繼承自 0422，在 GT-centric 分布下（所有 patch 都含油汙）可能過度加強 Oil 的 loss 貢獻。等 fold3 完成後再評估是否需重校（見 `TODO/01_distribution_shift_recheck.md`）。

---

## 進度總覽（截至 2026-07-02）

| Fold | 狀態 | 完成時間 | Best Val mIoU | Oil IoU | Background IoU |
|------|------|---------|--------------|---------|----------------|
| fold1 | 完成 | 2026-07-01 | （待補）| （待補）| （待補）|
| fold2 | 完成 | 2026-07-01 | （待補）| （待補）| （待補）|
| fold3 | **進行中** | — | ≈0.633（epoch 43/300）| ≈0.30 | ≈0.96 |

> fold1/2 的具體指標數字待使用者從 run_metrics.json 補入。

---

## 待動工 TODO 清單

| 文件（repo 內路徑）| 狀態 | 說明 |
|--------------------|------|------|
| `TODO/00_code_review_2026-06-25.md` | 未開始 | 動工前必修 7 項（seed、stats filter、collate None、pixel_mapping ignore、aug 過裁、eval bias 等）|
| `TODO/01_distribution_shift_recheck.md` | 未開始 | 因 patch 分布大改，重校 class_weights / loss / aug 強度 |
| `TODO/02_eval_protocol_comparison.md` | 未開始 | 與 0422 sliding-window 公平比較的評估協議設計 |

建議執行順序：fold3 完成 → `00_code_review`（必修 bug fix）→ 評估是否重跑 3-fold → `01_distribution_shift_recheck` → `02_eval_protocol_comparison`。

---

## 已知繼承 Bug（影響本輪結果可信度）

| Bug | 說明 | 對結果的影響 |
|-----|------|------------|
| C2 mean/std 污染 | YAML 直接沿用 0422 的 mean/std，等於繼承從全場景（可能含 test fold）抽樣的統計 | 輕微污染，不會讓結果完全失效但不算嚴格乾淨 |
| C3 collate None | DataLoader batch 行為可能非預期 | 不確定是否影響梯度計算，需驗證 |
| C4 evaluation else 走 YOLO | val 階段可能未正確計算分割 IoU | 若確認有問題，目前的 val IoU 數字會有偏差 |

> 以上三點尚未修正，本輪 CV 結果作為「基準觀察」使用，`00_code_review` 修完後重跑才算正式結果。

---

## 初步觀察

- fold3 epoch 43 的 Oil IoU ≈ 0.30，Background IoU ≈ 0.96；與 0422 sliding-window baseline（avg Oil IoU 0.24~0.28）相當，需等完整 3-fold 平均才有意義。
- Background IoU 高（0.96）在 GT-centric 分布下需謹慎解讀：patch 本身油汙佔比高，背景像素相對少，高 BG IoU 不等於整圖上的 BG 辨識能力。
- 本輪數字**不可直接與 0422 比較**（評估分母不同），需等 `TODO/02_eval_protocol_comparison.md` 的公平比較協議完成後再做跨版本對比。

---

## 下一步

- [ ] fold3 訓練完成後，補上三個 fold 的完整 metrics（val mIoU / Oil IoU / BG IoU / best epoch）
- [ ] 執行 `TODO/00_code_review_2026-06-25.md`（7 項必修 bug fix）
- [ ] 評估 bug fix 後是否需重跑 3-fold
- [ ] 執行 `TODO/01_distribution_shift_recheck.md`（重校 class_weights / loss / aug）
- [ ] 完成 `TODO/02_eval_protocol_comparison.md`（公平比較協議設計）

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
