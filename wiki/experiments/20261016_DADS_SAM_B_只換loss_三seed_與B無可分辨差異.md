---
date: 2026-10-16
type: experiment
stage: supervised
status: completed
tags: [dads-sam, gom-only, taiwan-holdout, loss-only, dice-ce-focal, tversky, three-seed, experiment-card, descriptive, no-distinguishable-difference]
---

# DADS-SAM B 只換 loss（三個 seed）：GoM 驗證與臺灣的 Oil IoU 都與 B 的範圍重疊，沒有可分辨的差異

> 本頁為「實驗九步驟卡」制度（`.agents/institution/templates/experiment_card.md`）下的結案卡，只填第 7–9 步。第 1–6 步（開跑前凍結：目的、問題、做什麼、開跑前檢查、怎麼報告、限制）正本為事前紀錄（vault 根目錄下），本頁不重複抄錄、不修改它：
> - 三個 seed 與報告方式：`.agents/decisions/20261015_DADS_SAM_B_只換loss_DiceCE_Focal_設計.md`
>
> 本組是前一張卡 [[20261015_DADS_SAM_B_公開碼訓練設定_三seed_臺灣皆低於B範圍]] 的後續：那一組同時差了九項訓練設定、臺灣都低於 B；本組只把 B 的 loss 換成公開碼的一項，回答差距有多少來自 loss。前一張卡與 10-10 凍結卡的結論不因本卡更改。

**實驗編號**：無 P 編號（DADS-SAM B 只換 loss）
**凍結日期**：2026-10-15 13:35 Asia/Taipei，見上引事前紀錄（repo 內 `analysis/dads_sam_gomonly_20261005/B_DICECE_FOCAL.md`，commit `7eb2c3b`）
**結案日期**：2026-10-16
**事後補寫**：否
**取代哪張卡**：無
**協商狀態**：未經 Claude × Codex 協商

跑的是：DADS-SAM B（公開碼架構、六波段輸入、固定整張圖的 box prompt，Tversky 那組）只把 loss 換成公開碼的 0.7 × DiceCE ＋ 0.3 × Focal；其他照 baseline 訓練設定（AdamW 學習率 5e-5 cosine、batch size 16、資料增強、gradient clipping 1.0、fp16 混合精度、最多 300 epoch／patience 50 早停、取 GoM 驗證 mIoU 最佳的權重）。三個 seed：42、43、44。使用者授權（2026-10-15）：「好吧，那就幫我安排下去 跑這個實驗」。事前紀錄不訂判定門檻，結果只作描述。

---

## 前半：開跑前凍結（1-6）

見正本 `.agents/decisions/20261015_DADS_SAM_B_只換loss_DiceCE_Focal_設計.md`，本頁不重複抄錄。

---

## 後半：結案才填（7-9）

### 7. Evidence / Result（實際結果）

來源：各 run 目錄內的 `results.csv`、`run_record.json`、`run_metrics.json`，臺灣 prediction manifest，以及 wandb（路徑見文末）。臺灣每個 run 只評一次，用的權重是該 run 驗證 mIoU 最高的 `best.pt`。

**流程與完整性**

- 設定檔 `main/experiments_DADS_SAM_gomonly_B_dicece_focal.yaml`，GT_expand commit `7eb2c3b`；同一個 main_runner 依序 S42 → S43 → S44（tmux `dads_sam_b_dicece_focal`）；2026-10-15 13:35 啟動，2026-10-16 23:34 全部結束，log 末行 `EXIT=0`。
- 三個 run 都在 patience 50 早停（沒有跑滿 300 epoch）：早停 epoch 158／118／112，最佳 epoch 108／68／62；訓練 829.2／619.0／587.4 分鐘，平均 678.5 ± 131.4 分鐘。
- 驗收項目三個 run 全數通過：`results.csv` 全為有限值；學習率為 cosine；`run_record` status trained；臺灣 9/9 景 prediction manifest complete；臺灣測試用的權重就是 `best.pt`，且 sha256 與 manifest 記錄一致。
- wandb：三個 run 都已 finished，history 列數與 `results.csv` 列數相同且逐列一致，summary 內有臺灣指標。

| seed | 早停 epoch | 最佳 epoch | 訓練分鐘 | best.pt sha256 前綴 | wandb run |
|---|---:|---:|---:|---|---|
| S42 | 158 | 108 | 829.2 | `8f3b8379` | `d5w2sjbn` |
| S43 | 118 | 68 | 619.0 | `395353ce` | `uuq71rcq` |
| S44 | 112 | 62 | 587.4 | `8290423c` | `3spqtugu` |
| 平均 ± 標準差 | | | 678.5 ± 131.4 | | |

**GoM 驗證（主要；最佳 epoch，三 seed 平均 ± 樣本標準差 ddof=1）**

| seed | 驗證 Oil IoU | 驗證 mIoU |
|---|---:|---:|
| S42 | 0.4810 | 0.7261 |
| S43 | 0.4986 | 0.7348 |
| S44 | 0.4937 | 0.7317 |
| 平均 ± 標準差 | **0.4911 ± 0.0091** | 0.7309 ± 0.0044 |

對照（同一選法）：

| 組 | 各 seed 驗證 Oil IoU | 平均 ± 標準差 | 各 seed 驗證 mIoU | 平均 ± 標準差 |
|---|---|---:|---|---:|
| 本卡（B 只換 loss） | 0.4810／0.4986／0.4937 | **0.4911 ± 0.0091** | 0.7261／0.7348／0.7317 | 0.7309 ± 0.0044 |
| B（Tversky、baseline 訓練設定） | 0.5081／0.5231／0.4824 | 0.5045 ± 0.0206 | 0.7394／0.7468／0.7247 | 0.7370 ± 0.0112 |
| B × 公開碼訓練設定（九項同時改） | 0.4333／0.4350／0.4237 | 0.4307 ± 0.0061 | 0.6985／0.6989／0.6924 | 0.6966 ± 0.0036 |

**臺灣（每個 run 9 景像素加總算一次，再跨 seed 取平均 ± 樣本標準差 ddof=1；依事前紀錄不拿來選 loss）**

| 指標 | S42 | S43 | S44 | 平均 ± 標準差 |
|---|---:|---:|---:|---:|
| **Oil IoU** | 0.5394 | 0.4671 | 0.5354 | **0.5140 ± 0.0406** |
| Precision | 0.6832 | 0.5425 | 0.5993 | 0.6083 ± 0.0708 |
| Recall | 0.7193 | 0.7707 | 0.8339 | 0.7746 ± 0.0574 |
| F1 | 0.7008 | 0.6367 | 0.6974 | 0.6783 ± 0.0361 |
| BG IoU | 0.9498 | 0.9278 | 0.9403 | 0.9393 ± 0.0110 |
| mIoU | 0.7446 | 0.6974 | 0.7378 | 0.7266 ± 0.0255 |
| 9 景各算再平均 Oil IoU | 0.4418 | 0.4066 | 0.4673 | 0.4386 ± 0.0305 |

混淆矩陣（臺灣 9 景加總，像素）：

| seed | TP | FP | FN | TN |
|---|---:|---:|---:|---:|
| S42 | 218,429 | 101,266 | 85,247 | 3,527,936 |
| S43 | 234,036 | 197,393 | 69,640 | 3,431,809 |
| S44 | 253,250 | 169,340 | 50,426 | 3,459,862 |

**對照（同為臺灣 9 景像素加總，同一算法）**

| 組 | 各 seed 的 Oil IoU | 平均 ± 標準差 | Precision | Recall | F1 | BG IoU | mIoU |
|---|---|---:|---:|---:|---:|---:|---:|
| 本卡（B 只換 loss） | 0.5394／0.4671／0.5354（seed 42／43／44） | **0.5140 ± 0.0406** | 0.6083 | 0.7746 | 0.6783 | 0.9393 | 0.7266 |
| 正式 B（Tversky、baseline 訓練設定） | 0.4434／0.4695／0.4355 | 0.4495 ± 0.0178 | 0.5695 | 0.6825 | 0.6201 | 0.9319 | 0.6907 |
| B × 公開碼訓練設定（九項同時改） | 0.1807／0.3709／0.3614 | 0.3043 ± 0.1071 | 0.5946 | 0.3770 | 0.4594 | 0.9304 | 0.6174 |
| 基線 P15 Original（Prithvi＋DeepLabV3，GoM-only） | 0.4808／0.4379／0.3704 | 0.4297 ± 0.0557 | 0.5473 | 0.6718 | 0.5997 | 0.9278 | 0.6788 |

（後三列的 Precision 等欄位取自前一張結案卡與 10-10 凍結卡的對照表，未重算。）

**S44 逐景 Oil IoU（臺灣 9 景；景名依前一張卡的寫法）**

| 事件 | 景 | S44 Oil IoU |
|---|---|---:|
| 天使輪 | 0801_L9_50QRK | 0.397 |
| 天使輪 | 0801_L9_50QRL | 0.273 |
| 天使輪 | 0912_S2_50QRL | 0.644 |
| 天使輪 | 0918_L9_50QRK | 0.618 |
| 天使輪 | 0918_L9_50QRL | 0.605 |
| 天使輪 | 0922_S2_50QRK | 0.264 |
| 天使輪 | 0922_S2_50QRL | 0.239 |
| 05-22 | 0522_L8_51QTD | 0.573 |
| 05-22 | 0522_L8_51QUD | 0.592 |

0922 兩景的 Precision 約 0.24–0.27、Recall 約 0.99，即過度預測。（本卡只整理了 S44 的逐景數字；S42、S43 的逐景值在各自 run 目錄的 `per_scene_iou.csv`。）

**loss 佔差距的比例（描述性，事前寫定）** ＝（B 平均 − 本組平均）／（B 平均 − 公開碼設定平均）

| 指標 | B 平均 | 本組平均 | 公開碼設定平均 | loss 佔差距比例 |
|---|---:|---:|---:|---:|
| GoM 驗證 Oil IoU | 0.5045 | 0.4911 | 0.4307 | 0.18 |
| GoM 驗證 mIoU | 0.7370 | 0.7309 | 0.6966 | 0.15 |
| 臺灣 Oil IoU | 0.4495 | 0.5140 | 0.3043 | −0.44（本組平均反而高於 B） |

事前紀錄註明：公開碼設定的臺灣數字用的是最後一個 epoch 的權重、seed 間差異大，這個比例很不穩，不單獨據此下結論。

**已知小問題（未修，待使用者決定）**：訓練期間 wandb 曲線比 `results.csv` 慢一個 epoch，原因是 `main/wandb_logging.py` 的 `run.log(row, step=epoch)` 沒給 `commit=True`；最後一列在 run 結束時補寫，沒有資料遺失。

### 8. Answer / Finding（結論）

**依事前紀錄的描述規則（GoM 驗證 Oil IoU、臺灣 Oil IoU 各自套用；不訂判定門檻，只作描述）**

| 指標 | B 的範圍 | 本組三個 seed | 與 B 範圍的關係 | 寫法 |
|---|---|---|---|---|
| GoM 驗證 Oil IoU | 0.4824–0.5231 | 0.4810／0.4986／0.4937 | S42 比 B 最小值低 0.0014；S43、S44 在範圍內 → 重疊 | **只換 loss 時與 B 沒有可分辨的差異** |
| 臺灣 Oil IoU | 0.4355–0.4695 | 0.5394／0.4671／0.5354 | S42、S44 高於 B 最大值；S43 在範圍內 → 重疊（不是三個全高） | **只換 loss 時與 B 沒有可分辨的差異**（臺灣依事前紀錄不拿來選 loss） |
| GoM 驗證 mIoU（補充） | 0.7247–0.7468 | 0.7261／0.7348／0.7317 | 三個都在範圍內 | 同上 |

**loss 在差距中的位置（描述性）**：GoM 驗證上，loss 佔「B → 公開碼訓練設定」差距的比例約 0.18（Oil IoU）、0.15（mIoU）；臺灣為 −0.44（本組平均高於 B）。因此公開碼訓練設定與 B 的差距，大部分不是只換 loss 造成，而是另外八項設定（optimizer 與學習率排程、batch size、訓練長度與選權重方式、資料增強、clipping、精度等）；這八項是同時改的，無法再細分到單項。

**觀察（描述性）**
- 兩個口徑的平均方向相反：GoM 驗證 Oil IoU 本組平均 0.4911 低於 B 的 0.5045，臺灣 Oil IoU 本組平均 0.5140 高於 B 的 0.4495；依描述規則兩者都是「與 B 範圍重疊」。
- seed 間差異：臺灣 Oil IoU 標準差 0.0406（B 為 0.0178、公開碼設定為 0.1071、基線為 0.0557）；GoM 驗證 Oil IoU 標準差 0.0091（B 為 0.0206、公開碼設定為 0.0061）。
- seed 間 Precision 範圍 0.5425–0.6832、Recall 範圍 0.7193–0.8339；S44 的 0922 兩景是過度預測（Precision 約 0.24–0.27、Recall 約 0.99）。

**不可主張**
- 不可寫「Tversky 比 DiceCE＋Focal 好」或反之：兩個口徑都落在重疊，且平均方向不一致。
- 不可用臺灣數字選 loss：事前紀錄寫定臺灣只照例報告。
- 不可把結果推到其他模型或公開碼設定下：loss 與 batch size、學習率等可能有交互作用；batch size、資料增強、取權重方式等另外八項各自的影響本組沒有測。
- 不可單獨據「loss 佔差距比例」下結論：臺灣一項的分母（公開碼設定）本身很不穩。

**缺什麼 / 限制**
- 三個 seed（n=3），平均 ± 標準差只是描述；B 與公開碼設定兩組的結果在開跑前都已看過，不是盲測。
- DiceCE／Focal 的權重（0.7／0.3、γ 2）照公開碼，沒有調；Tversky 的 α／β 也是沿用 baseline，沒有為 DADS-SAM 調過。
- 早停以驗證 mIoU 為準，loss 不同可能讓停的時間不同（本組停在第 158／118／112 個 epoch；事前紀錄記載 B 三個 seed 停在第 133／151／121 個 epoch）。
- prompt 與公開碼不同（固定整張圖 box）；臺灣只有 9 景、兩個事件。
- 未經 Claude × Codex 協商。

### 9. Decision / Next（下一步）

- **這條路線**（DADS-SAM 線續跑或收線）：未填（待使用者）。
- **投影片寫法**：未填（待使用者）。
- **是否另外單獨測 batch size 等其他設定**：未填（待使用者）；本組結果顯示另外八項合起來占差距的大部分，但單項影響本卡沒有資料。
- **wandb 曲線慢一個 epoch**：是否修 `main/wandb_logging.py`（`run.log(row, step=epoch)` 加 `commit=True`）：未填（待使用者）。
- **未被此實驗排除的替代解釋**：
  - 另外八項訓練設定同時改，無法再細分；loss 與它們之間的交互作用未測。
  - seed 間差異（尤其臺灣）使 n=3 對這個變異量的刻畫有限。
  - 非盲測（B 與公開碼設定的結果已先看過）。
  - prompt 與資料和公開碼不同（固定整張圖的 box 對標註外接框；GoM 衛星影像對無人機 RGB）。
- **下一張卡**：未填（待使用者）。
- **回寫狀態（非判定內容）**：institution 現況檔已回寫（institution commit `f1f0b29`）。

---

## 實驗檔案位置

| 類型 | 絕對路徑 |
|---|---|
| 專案 repo | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand` |
| 設計決議（1-6 正本） | `.agents/decisions/20261015_DADS_SAM_B_只換loss_DiceCE_Focal_設計.md`（vault 根目錄下） |
| repo 內事前紀錄 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/dads_sam_gomonly_20261005/B_DICECE_FOCAL.md`（commit `7eb2c3b`） |
| 設定檔 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/main/experiments_DADS_SAM_gomonly_B_dicece_focal.yaml` |
| launcher | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/dads_sam_gomonly_20261005/launch_B_dicece_focal.sh`（tmux `dads_sam_b_dicece_focal`） |
| 結果目錄 | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_dicece_focal/` |
| run 目錄（S42） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_dicece_focal/20261015-133542_MS6_DADS_SAM_B_public6_GoM_DiceCEFocal_S42/` |
| run 目錄（S43） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_dicece_focal/20261016-032542_MS6_DADS_SAM_B_public6_GoM_DiceCEFocal_S43/` |
| run 目錄（S44） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_dicece_focal/20261016-134529_MS6_DADS_SAM_B_public6_GoM_DiceCEFocal_S44/` |
| restructured Excel | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_dicece_focal/result_DADS_SAM_gomonly_B_dicece_focal_restructured.xlsx` |
| 序列 log（結束時末行 `EXIT=`） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_dicece_focal_main_runner.log` |
| institution 現況檔回寫 | institution commit `f1f0b29` |

## 相關頁面

- [[20261015_DADS_SAM_B_公開碼訓練設定_三seed_臺灣皆低於B範圍]] — 前一組：B × 公開碼訓練設定（九項同時改）三個 seed 的結案卡；本卡是只換 loss 的後續
- [[20261014_DADS_SAM_B_公開碼訓練設定_臺灣低於B三seed範圍]] — 公開碼訓練設定 seed 42 的單獨結案卡
- [[20261011_DADS_SAM_GoM_only_正式C與B_皆不可判定]] — 正式 B（seed 42）與 C 的 10-10 凍結卡結案；本卡以其 B 設定為底
