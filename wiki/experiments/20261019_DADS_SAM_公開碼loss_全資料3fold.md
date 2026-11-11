---
date: 2026-10-19
type: experiment
stage: supervised
status: completed
tags: [dads-sam, three-fold, full-data, dice-ce-focal, public-code-loss, gt-expand, experiment-card, descriptive]
---

# DADS-SAM ＋ 公開碼 loss（0.7 DiceCE ＋ 0.3 Focal）全資料 3-fold：三折測試 Oil IoU 皆低於 Prithvi＋DeepLabV3 基線

> 本頁為「實驗九步驟卡」制度（`.agents/institution/templates/experiment_card.md`）下的卡（2026-10-19 以進行中卡建立，2026-10-20 結案）。第 1–6 步（開跑前凍結）取自 repo 內事前紀錄 `analysis/dads_sam_3fold_20261019/CARD.md`，在任何結果出現前寫成，結案時未改；第 7–8 步為 2026-10-20 填入的結果與描述，第 9 步仍待使用者。即時狀態紀錄另見 `log.md` 與 `.agents/handoff.md`。
>
> 本組承接 [[20261016_DADS_SAM_B_只換loss_三seed_與B無可分辨差異]]：GoM-only 實驗得出，DADS-SAM 用基線訓練設定時，loss 從 Tversky 換成 0.7 DiceCE＋0.3 Focal，在 GoM 驗證上沒有可分辨的差異。該卡與 10-10 凍結卡的結論不因本卡更改。

**實驗編號**：無 P 編號（DADS-SAM 公開碼 loss 全資料 3-fold）
**凍結日期**：2026-10-19（啟動 18:43 Asia/Taipei 之前）
**結案日期**：2026-10-20
**事後補寫**：否
**取代哪張卡**：無
**協商狀態**：未經 Claude × Codex 協商
**授權**：使用者指示（2026-10-19，原話）：「現在有在跑的實驗你先幫我停掉，並且幫我刪掉有關的紀錄好了，我覺得暫時沒有必較在繼續追下去，就先以目前結論來看的話 他原論文上的 Loss方法  在搭上我們的optimizer 與 lr好了 那我覺得是要去測試他的效果，請你去幫我跑跑看全資料訓練全資料測試，也就是3fold的訓練」。（前半句所指的實驗已停止並刪除，見 `log.md` 同日條目；該兩組沒有結論，往後不要引用或重跑。）

**目前狀態**：已完成。2026-10-19 18:43 啟動（launch gate PASS，tmux `dads_sam_3fold`），三折皆完成、`EXIT=0`；結果於 2026-10-20 補入第 7–8 步，第 9 步未填（待使用者）。

---

## 前半：開跑前凍結（1-6）

### 1. 目的

GoM-only 實驗得出：DADS-SAM 用基線訓練設定時，loss 從 Tversky 換成公開碼的 0.7 DiceCE＋0.3 Focal，GoM 驗證沒有可分辨的差異。使用者據此決定 DADS-SAM 採用「公開碼 loss＋我們的 optimizer 與學習率」，並測它在全資料 3-fold 上的表現。

### 2. 問題

這個設定在全資料 3-fold（355 景、GT-centric patch）上，各折測試集的重建 Oil IoU 是多少，跟現行 Prithvi＋DeepLabV3 3-fold 基線比起來如何。

### 3. 做什麼

設定檔 `main/experiments_DADS_SAM_3fold_dicece_focal.yaml` 由 `build_config.py` 產生：以 3-fold 基線的 block（`main/experiments_CV_358clean_gt_expand_prithvi300m_dlv3_det.yaml`）為底，換掉 `architecture_cfg`（整段取自 GoM-only「B 只換 loss」seed 42 那組）、loss、名稱、重建的預測檔選項與 wandb 說明；其餘逐鍵比對與基線相同。

| 項目 | 3-fold 基線（`_dlv3_det`） | 本組 |
|---|---|---|
| 模型 | Prithvi-EO-V2 300M ＋ DeepLabV3 ASPP head | DADS-SAM 公開碼架構（`dads_sam_public6`，SAM ViT-B），六波段 B02／B03／B04／B8A／B11／B12，prompt 固定為整張圖的框 |
| 模型專屬的學習率 | Prithvi encoder 0.1×（`backbone_lr_mult 0.1`） | 全部 1×（同 GoM-only DADS-SAM 各組） |
| Loss | Tversky（α 0.3、β 0.7） | **0.7 × DiceCE ＋ 0.3 × Focal（γ 2）** |
| 其餘訓練設定 | AdamW（weight decay 1e-3）、lr 5e-5 cosine、batch 16、最多 300 epoch、patience 25、取驗證 mIoU 最高的 epoch、資料增強、clipping 1.0、fp16 混合精度 | 同左 |
| 資料 | `3_fold_stratified_v2/patch_level_gt_expand`；每折 seed＝42＋折號；每折訓練一次 | 同左 |
| 測試 | 各折 `test_fold` 的 gt_aware 重建（argmax） | 同左；另存逐像素預測檔 |

與基線不同的地方：模型、loss、模型專屬學習率。與 GoM-only「B 只換 loss」的差別：資料換成 3-fold；patience 由 50 改為 25，對齊 3-fold 基線。

### 4. 開跑前的檢查

smoke（`analysis/dads_sam_3fold_20261019/smoke.py`，結果 `smoke.json`）：**PASS**。
- 資料：三折 train／val／test 的油汙與背景 patch 數、訓練集油汙／背景像素數，與基線三折 `run_metrics.json` 完全相同（例如 fold 1 訓練 1,168＋2,110 patch）。
- 訓練：fold 1 跑 200 步加完整驗證；loss 皆為有限值；顯存峰值 17.1 GiB；訓練加驗證 6.3 分鐘。

### 5. 怎麼報告（事前寫定）

- **主要**：各折測試集 gt_aware 重建的 Oil IoU（每折內像素加總算一次），再三折平均。對照基線 0.4945／0.4774／0.3479，平均 **0.4399**。
- 另報全語料 pooled（三折混淆矩陣加總算一次），對照基線 **0.4335**。
- 每折另報 Precision／Recall／F1／BG IoU／mIoU、混淆矩陣、最佳 epoch 與停止 epoch、訓練時間。
- 不訂門檻，只描述逐折差與方向。

### 6. 限制與可以／不可以主張

- 不能把與基線的差距歸因於架構：loss 也不同（本組 DiceCE＋Focal，基線 Tversky），模型專屬學習率也不同。
- 這是 3-fold 交叉驗證，不是外部測試；語料含天使輪 7 張臺灣影像，不能據此主張外部泛化。
- **沒有加臺灣 10 景 19 張測試**：原臺灣測試集（5 景 9 張）中的天使輪 7 張就在 3-fold 語料裡，測了會洩漏；基線也沒有這項測試。
- 每折只訓練一次，沒有 seed 重複（與基線相同）。
- GoM-only 各組結果與 3-fold 基線結果在開跑前都已看過，不是盲測。
- 未經 Claude × Codex 協商。

---

## 後半：結案才填（7-9）

### 7. Evidence / Result（實際結果）

來源：GT_expand worktree 內 `analysis/dads_sam_3fold_20261019/CARD.md` 末節「結果（2026-10-20 補記）」與 `results.json`（commit `39e3e45`，branch `claude/dads_sam_3fold_20261019`，未併入 main）。基線數字由 P4 逐像素 atoms（τ 0.5）重算，與正典 0.4945／0.4774／0.3479、0.4335 相同。

**流程與完整性**

- 三折皆完成，`EXIT=0`；每折 loss 皆為有限值，`best.pt` 存在。
- 重建測試的景數與基線各折相同（119／118／118 景），GT 油汙像素數與覆蓋像素數也與基線相同。
- wandb `am48y66t`／`58dwjjkl`／`ku5a1wiq` 皆 finished，列數＝`results.csv` 列數（41／127／54）。

**主要：各折測試集 gt_aware 重建 Oil IoU（每折內像素加總算一次）**

| 折 | 基線 Oil IoU | 本組 Oil IoU | 本組 − 基線 |
|---|---:|---:|---:|
| fold 1 | 0.4945 | 0.4431 | −0.0514 |
| fold 2 | 0.4774 | 0.3947 | −0.0826 |
| fold 3 | 0.3479 | 0.2984 | −0.0495 |
| 逐折平均 | 0.4399 | **0.3787** | −0.0612 |
| 全語料 pooled（三折混淆矩陣加總） | 0.4335 | **0.3669** | −0.0666 |

**Precision／Recall／F1／BG IoU／mIoU**

| 指標 | 組 | fold 1 | fold 2 | fold 3 | 全語料 pooled |
|---|---|---:|---:|---:|---:|
| Precision | 本組 | 0.736 | 0.785 | 0.585 | 0.691 |
| Precision | 基線 | 0.633 | 0.699 | 0.644 | 0.660 |
| Recall | 本組 | 0.527 | 0.443 | 0.379 | 0.439 |
| Recall | 基線 | 0.694 | 0.601 | 0.431 | 0.558 |
| F1 | 本組 | 0.614 | 0.566 | 0.460 | 0.537 |
| F1 | 基線 | 0.662 | 0.646 | 0.516 | 0.605 |
| BG IoU | 本組 | 0.953 | 0.929 | 0.920 | 0.933 |
| BG IoU | 基線 | 0.949 | 0.930 | 0.927 | 0.935 |
| mIoU | 本組 | 0.698 | 0.662 | 0.609 | 0.650 |
| mIoU | 基線 | 0.722 | 0.704 | 0.638 | 0.684 |

**停止／最佳 epoch、最佳驗證 mIoU、訓練時間**

| 折 | 停止 epoch | 最佳 epoch | 最佳驗證 mIoU（本組） | 最佳驗證 mIoU（基線） | 訓練時間（本組，分鐘） |
|---|---:|---:|---:|---:|---:|
| fold 1 | 41 | 16 | 0.6206 | 0.6620 | 261 |
| fold 2 | 127 | 102 | 0.7374 | 0.7471 | 863 |
| fold 3 | 54 | 29 | 0.6956 | 0.6713 | 340 |

基線的訓練時間是在 CPU 節流期間記錄的，不能和本組比較，故不列。

**混淆矩陣（像素）**

| 組 | 折 | TP | FP | FN | TN |
|---|---|---:|---:|---:|---:|
| 本組 | fold 1 | 1,465,145 | 526,350 | 1,314,826 | 37,354,673 |
| 本組 | fold 2 | 1,771,836 | 486,437 | 2,230,264 | 35,517,083 |
| 本組 | fold 3 | 1,641,584 | 1,165,768 | 2,694,530 | 44,583,835 |
| 基線 | fold 1 | 1,928,513 | 1,120,035 | 851,458 | 36,760,988 |
| 基線 | fold 2 | 2,406,318 | 1,038,644 | 1,595,782 | 34,964,876 |
| 基線 | fold 3 | 1,867,284 | 1,031,684 | 2,468,830 | 44,717,919 |

### 8. Answer / Finding（結論）

事前紀錄不訂門檻，以下只作描述。

**描述**

- 三折的測試 Oil IoU 都低於基線，方向一致，差 −0.05 到 −0.08；逐折平均低 0.0612（0.3787 對 0.4399），全語料 pooled 低 0.0666（0.3669 對 0.4335）。
- 三折的 Recall 都低於基線（fold 1 0.527 對 0.694、fold 2 0.443 對 0.601、fold 3 0.379 對 0.431；全語料 0.439 對 0.558）；差距主要來自漏判。
- fold 3 的最佳驗證 mIoU 高於基線（0.6956 對 0.6713），但測試 Oil IoU 較低（0.2984 對 0.3479）。
- 每折只訓練一次，差距無法和訓練間雜訊比較。

**不可主張**

- 不能把與基線的差距全歸因於架構：loss（本組 DiceCE＋Focal，基線 Tversky）與模型專屬學習率也不同。
- 這是 3-fold 交叉驗證，不是外部測試；語料含天使輪 7 張臺灣影像，不能據此主張外部泛化。
- 沒有臺灣 10 景 19 張測試（理由見第 6 步）。

**缺什麼／限制**

- 每折訓練一次，沒有 seed 重複。
- GoM-only 各組結果與 3-fold 基線結果在開跑前都已看過，不是盲測。
- 未經 Claude × Codex 協商。

### 9. Decision / Next（下一步）

- **這條路線**：未填（待使用者）。
- **下一張卡**：未填（待使用者）。

---

## 實驗檔案位置

| 類型 | 絕對路徑 |
|---|---|
| 程式（GT_expand worktree） | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019`（branch `claude/dads_sam_3fold_20261019`，開跑 commit `36ed46b`，結果補記 commit `39e3e45`，未併入 main） |
| 設定檔 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019/main/experiments_DADS_SAM_3fold_dicece_focal.yaml` |
| 事前紀錄與結果補記（repo 內） | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019/analysis/dads_sam_3fold_20261019/CARD.md`（前段為事前紀錄，末節「結果（2026-10-20 補記）」為結果） |
| 結果數字 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019/analysis/dads_sam_3fold_20261019/results.json`（由同目錄 `summarize_results.py` 產生） |
| 結果目錄 | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_dicece_focal_20261019/` |
| run 目錄（fold 1） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_dicece_focal_20261019/20261019-184358_MS6_DADS_SAM_3fold_DiceCEFocal_fold1/` |
| run 目錄（fold 2） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_dicece_focal_20261019/20261019-231750_MS6_DADS_SAM_3fold_DiceCEFocal_fold2/` |
| run 目錄（fold 3） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_dicece_focal_20261019/20261020-135337_MS6_DADS_SAM_3fold_DiceCEFocal_fold3/` |
| restructured Excel | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_dicece_focal_20261019/result_DADS_SAM_3fold_dicece_focal_restructured.xlsx` |
| tmux session | `dads_sam_3fold` |
| wandb | fold 1：[am48y66t](https://wandb.ai/tree8454-national-central-university/oil-gt-expand/runs/am48y66t)；fold 2：`58dwjjkl`；fold 3：`ku5a1wiq`（皆 finished） |
| institution 現況檔回寫 | institution commit `7f8be8d` |

## 相關頁面

- [[20261016_DADS_SAM_B_只換loss_三seed_與B無可分辨差異]] — GoM-only 只換 loss 的結案卡；本組的前置依據
- [[20261011_DADS_SAM_GoM_only_正式C與B_皆不可判定]] — DADS-SAM GoM-only 正式 C／B 結案卡（10-10 凍結卡的結案）
- [[20260731_ASPP_rate適配實驗]] — 3-fold 基線 `_dlv3_det`（0.4945／0.4774／0.3479、平均 0.4399）的出處
- [[GT_expand_pipeline]] — GT_expand 資料與 3_fold_stratified_v2 split

---

## 後續：再跑兩次（base seed 43、44，已完成）

> 2026-10-21 追加。上方第 1–9 步與 seed 42 那次的結果（各折 Oil IoU 0.4431／0.3947／0.2984）不因本節更改。本節只記啟動與事前寫定的報告方式，**尚無 seed 43、44 的任何結果**。事前紀錄正本為 repo 內 `analysis/dads_sam_3fold_20261019/CARD_SEEDS.md`（在這兩次的任何結果出現前寫成）。

**授權**：使用者指示（2026-10-21，原話）：「你在幫我跑一組好了  多跑個兩次」。解讀（Claude 的）：同一組設定再跑兩次，換 base seed 43、44（專案慣例 S42／43／44），連同 10-19 的 seed 42 共三次。未經 Claude × Codex 協商。

**目前狀態**：已完成（2026-10-24 14:07 跑完，`EXIT=0`，GPU 已釋放；結果見本頁最末「再跑兩次的結果（2026-10-24 補記）」）。2026-10-21 10:53 啟動，launch gate PASS，tmux `dads_sam_3fold_seeds`，seed 43 fold 1 的 wandb run `kjsmqnjk`。兩次依序跑，預估約 2 天（1.5–3 天）。

**做什麼**

- 設定與 seed 42 那次逐鍵相同，只改 seed、實驗名、重建測試名與 wandb 說明。
- 每折實際 seed＝base seed＋折號：base seed 43 → 44／45／46，base seed 44 → 45／46／47。

**怎麼報告（事前寫定）**

- 每次各折 Oil IoU、逐折平均、全語料 pooled 等，用與 seed 42 相同的算法。
- 三次（seed 42／43／44）的逐折平均與全語料 pooled 報平均 ± 樣本標準差。
- 對基線（只有一次訓練）只描述：三次中有幾次的逐折平均低於 0.4399；各折三次的範圍是否都低於基線該折（0.4945／0.4774／0.3479）。不訂門檻。

**限制**

- 基線沒有重複，所以三次的變動只代表 DADS-SAM 這一邊的訓練間變動，不能當成兩邊差距的統計檢定。
- 可以與不可以主張同上方第 6 步；seed 42 的結果已看過，這兩次不是盲測。

**檔案位置（本節）**

| 類型 | 絕對路徑 |
|---|---|
| 設定檔 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019/main/experiments_DADS_SAM_3fold_dicece_focal_seeds.yaml`（由同 worktree `analysis/dads_sam_3fold_20261019/build_seeds.py` 產生） |
| 事前紀錄（repo 內） | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019/analysis/dads_sam_3fold_20261019/CARD_SEEDS.md`（GT_expand worktree，branch `claude/dads_sam_3fold_20261019`，commit `dd4ebd9`，未併入 main） |
| 結果目錄 | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_dicece_focal_seeds_20261021/` |
| tmux session | `dads_sam_3fold_seeds` |
| wandb | seed 43 fold 1：`kjsmqnjk` |
| institution 現況檔回寫 | institution commit `7b603d6` |

---

## 再跑兩次的結果（2026-10-24 補記）

> 上方第 1–9 步、seed 42 那次的結果與「後續：再跑兩次」一節的事前寫定內容不因本節更改；第 9 步仍為「未填（待使用者）」。

- seed 43、44 兩次 10-24 14:07 跑完（EXIT=0），GPU 已釋放。六折驗收通過：loss 有限、best.pt 在、測試景數與 GT 像素和基線相同、wandb 六個 run finished 且列數＝results.csv。
- 各折 gt_aware 重建 Oil IoU（每折像素加總）：

| | fold 1 | fold 2 | fold 3 | 逐折平均 | 全語料 pooled |
|---|---|---|---|---|---|
| seed 42 | 0.4431 | 0.3947 | 0.2984 | 0.3787 | 0.3669 |
| seed 43 | 0.4077 | 0.3383 | 0.3205 | 0.3555 | 0.3479 |
| seed 44 | 0.4367 | 0.3722 | 0.3027 | 0.3706 | 0.3603 |
| 三次平均 ± SD（ddof=1） | | | | 0.3683 ± 0.0118 | 0.3584 ± 0.0097 |
| 基線 Prithvi＋DeepLabV3（一次） | 0.4945 | 0.4774 | 0.3479 | 0.4399 | 0.4335 |

- 逐折平均的其他指標三次平均：P 0.6959／R 0.4382／F1 0.5361／BG IoU 0.9330／mIoU 0.6506（基線 0.6584／0.5752／0.6081／0.9355／0.6877）。
- 描述（不訂門檻）：三次逐折平均都低於 0.4399（3/3）；各折三次範圍（0.4077–0.4431／0.3383–0.3947／0.2984–0.3205）都低於該折基線；Recall 都較低，差距主要來自漏判。DADS-SAM 三次全距 0.023 小於與基線差距 0.072，但基線只訓練一次，不是統計檢定；loss 與模型專屬學習率也不同，不能全歸因於架構；這是交叉驗證，不是外部測試。
- seed 43 fold 3 的訓練時間 1,968 分鐘含 10-22 12:50～10-23 12:28 的 23.6 小時停擺（原因未定），扣除後約 555 分鐘；訓練數值不受影響。
- 來源：worktree `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019` 的 `analysis/dads_sam_3fold_20261019/CARD_SEEDS.md` 末節與 `results_seeds.json`（commit `2ac15c2`，branch `claude/dads_sam_3fold_20261019`，未併 main）；institution 現況檔 commit `085cff7`；結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_dicece_focal_seeds_20261021/`。
- wandb（皆 finished）：seed 43 fold 1／2／3 為 `kjsmqnjk`／`e4eyeymq`／`gx5s2z3j`；seed 44 fold 1／2／3 為 `j1ydpork`／`uet39881`／`koqs7kd9`。
