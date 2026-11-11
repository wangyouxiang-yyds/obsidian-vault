---
date: 2026-10-24
type: experiment
stage: supervised
status: completed
tags: [dads-sam, three-fold, full-data, tversky, gt-expand, experiment-card, descriptive]
---

# DADS-SAM ＋ Tversky loss 全資料 3-fold：同一 DADS-SAM 設定只把 loss 換成基線的 Tversky，逐折平均 0.3688 落在 DiceCE／Focal 三次範圍內、低於基線 0.4399

> 本頁為「實驗九步驟卡」制度（`.agents/institution/templates/experiment_card.md`）下的卡，2026-10-24 以進行中卡建立，2026-10-25 結案。第 1–6 步（開跑前凍結）取自 repo 內事前紀錄 `analysis/dads_sam_3fold_tversky_20261024/CARD.md`，在任何結果出現前寫成，結案時未改；第 7–8 步為 2026-10-25 填入的結果與描述，第 9 步仍待使用者。即時狀態紀錄另見 `log.md` 與 `.agents/handoff.md`。
>
> 本組承接 [[20261019_DADS_SAM_公開碼loss_全資料3fold]]：該卡用 DADS-SAM ＋ 0.7 DiceCE ＋ 0.3 Focal 跑全資料 3-fold，三次（seed 42／43／44）逐折平均 0.3787／0.3555／0.3706，皆低於 Prithvi＋DeepLabV3 基線 0.4399。本組設定與該卡 seed 42 那次完全相同，只把 loss 換回基線用的 Tversky。該卡的結果與描述不因本卡更改。

**實驗編號**：無 P 編號（DADS-SAM Tversky 全資料 3-fold）
**凍結日期**：2026-10-24（啟動 22:03 Asia/Taipei 之前）
**結案日期**：2026-10-25
**事後補寫**：否
**取代哪張卡**：無
**協商狀態**：未經 Claude × Codex 協商
**授權**：使用者指示（2026-10-24，原話）：「我覺得你這邊先幫我改成tversky loss然後跑跑看  先跑個一次好了」。

**目前狀態**：已完成。2026-10-24 22:03 啟動（launch gate PASS，tmux `dads_sam_3fold_tversky`，wandb fold 1 `ecepktmm`；fold 1 log 顯示 `TverskyLoss(alpha=0.3, beta=0.7)`），三折皆完成、2026-10-25 18:39 `EXIT=0`，GPU 已釋放；結果於 2026-10-25 補入第 7–8 步，第 9 步未填（待使用者）。

---

## 前半：開跑前凍結（1-6）

### 1. 目的

[[20261019_DADS_SAM_公開碼loss_全資料3fold]] 的 DADS-SAM（0.7 DiceCE ＋ 0.3 Focal）三次全資料 3-fold 都低於 Prithvi＋DeepLabV3 基線。使用者要看：同一個 DADS-SAM 設定，loss 換成基線用的 Tversky，各折測試 Oil IoU 會變多少，以及換了之後和基線還差多少。

### 2. 問題

(1) 同一 DADS-SAM 設定，loss 從 0.7 DiceCE ＋ 0.3 Focal 換成 Tversky（α 0.3、β 0.7），全資料 3-fold 各折測試 Oil IoU 變多少。
(2) 換 loss 之後，與 Prithvi＋DeepLabV3 3-fold 基線還差多少。此時兩邊剩下的差別是模型（含輸入放大到 1024）與模型專屬的學習率。

### 3. 做什麼

DADS-SAM（`dads_sam_public6`）全資料 3-fold，設定與 DiceCE／Focal 3-fold seed 42 那次完全相同，只把 loss 從 0.7 DiceCE ＋ 0.3 Focal 換成基線用的 Tversky（α 0.3、β 0.7）；跑一次（base seed 42，每折 seed 43／44／45）。設定檔 `main/experiments_DADS_SAM_3fold_tversky.yaml`（由 `analysis/dads_sam_3fold_tversky_20261024/build_config.py` 產生）。

| 項目 | 3-fold 基線（`_dlv3_det`） | DiceCE／Focal 3-fold（seed 42） | 本組 |
|---|---|---|---|
| 模型 | Prithvi-EO-V2 300M ＋ DeepLabV3 ASPP head | DADS-SAM（`dads_sam_public6`，SAM ViT-B），六波段，prompt 固定為整張圖的框 | 同左 |
| 模型專屬的學習率 | Prithvi encoder 0.1×（`backbone_lr_mult 0.1`） | 全部 1× | 同左（全部 1×） |
| Loss | Tversky（α 0.3、β 0.7） | 0.7 × DiceCE ＋ 0.3 × Focal（γ 2） | **Tversky（α 0.3、β 0.7）** |
| 其餘訓練設定 | 同 DiceCE／Focal 3-fold | 同左 | 同左 |
| 資料 | `3_fold_stratified_v2/patch_level_gt_expand`；每折 seed＝base seed＋折號；每折訓練一次 | 同左 | 同左（base seed 42，每折 seed 43／44／45） |

與 DiceCE／Focal 3-fold seed 42 那次只差 loss。與 Prithvi＋DeepLabV3 3-fold 基線只差模型與模型專屬學習率（DADS-SAM 全部 1×，基線 encoder 0.1×）。

### 4. 開跑前的檢查

- launch gate PASS。
- **未另跑 smoke**：資料、模型與流程和 DiceCE／Focal 3-fold 相同，且該 3-fold 三次（seed 42／43／44）共 9 折都正常完成；DADS-SAM ＋ Tversky 在 GoM-only B 組三個 seed 也都正常跑完（同一個 `architecture_cfg`，已另行比對相同）。
- `build_config.py` 內 assert 兩件事：與 DiceCE／Focal 3-fold block 只差 loss 與名稱；與基線 block 只差模型（`architecture_cfg`）、名稱與重建的預測檔選項。
- 啟動後確認：fold 1 log 顯示 `TverskyLoss(alpha=0.3, beta=0.7)`，每步約 1.8 秒；粗估 1–1.5 天。

### 5. 怎麼報告（事前寫定）

- **主要**：各折測試集 gt_aware 重建的 Oil IoU（每折內像素加總算一次），三折平均；另報全語料 pooled（三折混淆矩陣加總算一次）。
- **主要對照**：DiceCE／Focal 同 seed（42）那次，各折 0.4431／0.3947／0.2984，逐折平均 0.3787（只差 loss）；並描述本組是否落在 DiceCE／Focal 三次範圍內（逐折平均 0.3555–0.3787）。
- **另對照基線**：Prithvi＋DeepLabV3 0.4945／0.4774／0.3479，逐折平均 0.4399、全語料 pooled 0.4335。
- 只描述、不訂門檻；只跑一次，自身訓練間變動未知。

### 6. 限制與可以／不可以主張

- 與基線的差距只能歸因於「模型＋模型專屬學習率」整包，不能拆開說是架構。
- 與 DiceCE／Focal 的差只代表 loss 這一項在本設定下的效果（單次）。
- 這是 3-fold 交叉驗證，不是外部測試。
- **沒有加臺灣 19 張測試**：天使輪 7 張在 3-fold 語料內，測了會洩漏。
- 只跑一次（base seed 42），沒有 seed 重複；與 DiceCE／Focal 的逐折差無法和訓練間雜訊比較（DiceCE／Focal 三次逐折平均 SD 0.0118，僅供參考，不是本組的變動估計）。
- DiceCE／Focal 三次與基線結果在開跑前都已看過，不是盲測。
- 未經 Claude × Codex 協商。

---

## 後半：結案才填（7-9）

### 7. Evidence / Result（實際結果）

來源：GT_expand worktree 內 `analysis/dads_sam_3fold_tversky_20261024/CARD.md` 末節「結果（2026-10-25 補記）」與 `results.json`（commit `8de1b42`，branch `claude/dads_sam_3fold_20261019`，未併入 main）。DiceCE／Focal 數字讀自同 worktree `analysis/dads_sam_3fold_20261019/results_seeds.json`（見 [[20261019_DADS_SAM_公開碼loss_全資料3fold]]）。

**流程與完整性**

- 三折皆完成，2026-10-25 18:39 `EXIT=0`，GPU 已釋放；每折 loss 皆為有限值，`best.pt` 存在。
- 重建測試的景數與基線各折相同（119／118／118 景），GT 油汙像素數與覆蓋像素數也與基線相同；預測檔算出的 Oil IoU 與各折重建 `run_metrics.json` 一致。
- wandb `ecepktmm`／`pa56otf0`／`yn29axpk` 皆 finished，列數＝`results.csv` 列數（65／56／63）。

**各折 gt_aware 重建 Oil IoU（每折像素加總）**

| | fold 1 | fold 2 | fold 3 | 逐折平均 | 全語料 pooled |
|---|---|---|---|---|---|
| **本組（Tversky，seed 42）** | 0.4256 | 0.3671 | 0.3139 | **0.3688** | **0.3632** |
| DiceCE／Focal seed 42 | 0.4431 | 0.3947 | 0.2984 | 0.3787 | 0.3669 |
| 本組 − DiceCE／Focal S42 | −0.0176 | −0.0276 | +0.0155 | −0.0099 | −0.0037 |
| DiceCE／Focal 三次範圍 | 0.4077–0.4431 | 0.3383–0.3947 | 0.2984–0.3205 | 0.3555–0.3787 | 0.3479–0.3669 |
| 基線 Prithvi＋DeepLabV3 | 0.4945 | 0.4774 | 0.3479 | 0.4399 | 0.4335 |
| 本組 − 基線 | −0.0689 | −0.1103 | −0.0340 | −0.0711 | −0.0703 |

**Precision／Recall 與訓練概況**

| | P | R | F1 | BG IoU | mIoU | 停止／最佳 epoch | 訓練時間（分鐘） |
|---|---|---|---|---|---|---|---|
| fold 1 | 0.5511 | 0.6515 | 0.5971 | 0.9371 | 0.6813 | 65／40 | 415 |
| fold 2 | 0.6823 | 0.4428 | 0.5370 | 0.9201 | 0.6436 | 56／31 | 382 |
| fold 3 | 0.5574 | 0.4180 | 0.4778 | 0.9179 | 0.6159 | 63／38 | 398 |
| 逐折平均 | 0.5969 | 0.5041 | 0.5373 | 0.9250 | 0.6469 | | |
| DiceCE／Focal S42 逐折平均 | 0.7017 | 0.4494 | 0.5466 | 0.9341 | 0.6564 | | |
| 基線逐折平均 | 0.6584 | 0.5752 | 0.6081 | 0.9355 | 0.6877 | | |

混淆矩陣（像素，TP／FP／FN／TN）：fold 1 1,811,037／1,475,479／968,934／36,405,544；fold 2 1,772,148／825,330／2,229,952／35,178,190；fold 3 1,812,640／1,439,260／2,523,474／44,310,343。

### 8. Answer / Finding（結論）

事前紀錄不訂門檻，以下只作描述。

- 與 DiceCE／Focal 同 seed（42）那次比：fold 1、2 較低、fold 3 較高，方向不一致；逐折平均差 −0.0099。三折與逐折平均都落在 DiceCE／Focal 三次範圍內 → 單次結果下，3-fold 上 loss 換成 Tversky 的效果與重跑一次的變動分不開（與 GoM-only「B 只換 loss」結論同向，見 [[20261016_DADS_SAM_B_只換loss_三seed_與B無可分辨差異]]）。
- Tversky 這次 Precision 較低、Recall 較高（逐折平均 P 0.597 對 0.702，R 0.504 對 0.449），兩者抵銷，Oil IoU 相近。
- 與基線比：三折都較低（−0.034 到 −0.110），逐折平均低 0.071，全語料 pooled 低 0.070。此時兩邊剩下的差別是「模型＋模型專屬學習率（DADS-SAM 全部 1×、基線 encoder 0.1×）」整包，不能單獨歸因於架構。
- 本組只跑一次，自身訓練間變動未知。
- 限制同第 6 步：這是 3-fold 交叉驗證，不是外部測試；DiceCE／Focal 三次與基線結果在開跑前都已看過，不是盲測。

### 9. Decision / Next（下一步）

- **這條路線**：未填（待使用者）。
- **下一張卡**：未填（待使用者）。

---

## 實驗檔案位置

| 類型 | 絕對路徑 |
|---|---|
| 程式（GT_expand worktree） | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019`（branch `claude/dads_sam_3fold_20261019`；啟動時 commit `58c15c0`，結果補記 commit `8de1b42`；未併入 main） |
| 設定檔 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019/main/experiments_DADS_SAM_3fold_tversky.yaml` |
| 事前紀錄（repo 內） | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019/analysis/dads_sam_3fold_tversky_20261024/CARD.md` |
| 結果目錄 | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_tversky_20261024/` |
| run 目錄（fold 1） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_tversky_20261024/20261024-220321_MS6_DADS_SAM_3fold_Tversky_fold1/` |
| run 目錄（fold 2） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_tversky_20261024/20261025-051138_MS6_DADS_SAM_3fold_Tversky_fold2/` |
| run 目錄（fold 3） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_tversky_20261024/20261025-114611_MS6_DADS_SAM_3fold_Tversky_fold3/` |
| 結果 Excel | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_tversky_20261024/result_DADS_SAM_3fold_tversky_restructured.xlsx` |
| 結果數字 | worktree `analysis/dads_sam_3fold_tversky_20261024/results.json`（由同目錄 `summarize_results.py` 產生） |
| 訓練 log | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_tversky_20261024_main_runner.log`（結束時末行 `EXIT=`） |
| tmux session | `dads_sam_3fold_tversky` |
| wandb | fold 1：`ecepktmm`；fold 2：`pa56otf0`；fold 3：`yn29axpk` |
| institution 現況檔回寫 | 啟動：institution commit `17cf011`；結案：institution commit `e92b02e` |

## 相關頁面

- [[20261019_DADS_SAM_公開碼loss_全資料3fold]] — 同設定、loss 為 0.7 DiceCE ＋ 0.3 Focal 的全資料 3-fold 三次（本組的主要對照）
- [[20261016_DADS_SAM_B_只換loss_三seed_與B無可分辨差異]] — GoM-only 上 Tversky 換成 DiceCE／Focal 的結案卡
- [[20261011_DADS_SAM_GoM_only_正式C與B_皆不可判定]] — DADS-SAM GoM-only 正式 C／B 結案卡（B 組含 Tversky，三個 seed 正常跑完）
- [[20260731_ASPP_rate適配實驗]] — 3-fold 基線 `_dlv3_det`（0.4945／0.4774／0.3479、平均 0.4399）的出處
- [[GT_expand_pipeline]] — GT_expand 資料與 3_fold_stratified_v2 split
