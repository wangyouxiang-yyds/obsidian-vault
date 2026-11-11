---
date: 2026-11-10
type: experiment
stage: evaluation
status: 完成
tags: [p50, upernet, prithvi-eo-2, h1-prime, roi3840, three-seed, teacher-candidate, supervised, descriptive, taiwan24, execution-record]
---

# P50：H1′（1:1+0.5 HNM+SI）× Prithvi-EO-2.0-300M + UperNet，一般監督式分割三 seed——完成

> 本頁為執行紀錄，內含卡片全文副本（原檔 `analysis/p50_upernet_h1_20261110/P50_CARD.md`，repo commit ab23095）。動機承接 [[20261110_P49_LFMDA搬到ROI_三seed_執行紀錄]]；對照基準見 [[20261108_P48補seed_H1與P44]]、[[20261107_P46大圖版配比與OOF探勘]]、[[20261106_P44_ROI重疊滑窗全球排除臺灣設計]]。**描述性、未經 Codex、無預先登記門檻**。

## 1. 實驗背景

P49 的 LFMDA student 在各子集的強弱幾乎照抄 teacher H1′ S43（DeepLabV3 頭：扣大林 0.139、Landsat 0.142、大林 0.537）。使用者想試 UperNet 當 teacher，先以一般監督式分割（不含 LFMDA）跑三 seed 看成效。使用者 2026-11-10 原話：「1:1.5HNM + SI的部分 使用Prithvi + UperNet的部分，然後我們先試試teacher model 意即 一搬的分割模式 看一下成效如何跑個三次吧」。結果決定是否拿 UperNet 當 P49 的新 teacher（仍需使用者決定）。

## 2. 方法架構 / Pipeline 流程

1. 訓練：同一支 trainer（`train_p46.py` 未改）、同 seed 的同一份 H1′ schedule，只把分割頭由 DeepLabV3 換成 UperNet；原定 S42 與 S43 並行、任一完成後接 S44；11-10 12:44 使用者要求一個一個跑，改為 S42 → S43 → S44 依序訓練（S43 從頭重跑）。
2. 預測：依序對三個 seed 做臺灣 24 張 ROI 預測（P44 凍結算子，predict 程式照抄 P48）。
3. 評分：`evaluate_p45.score`，與同 seed H1′ DLV3 配對比較。

## 3. 關鍵參數

| 項目 | H1′ DLV3 | P50 |
|---|---|---|
| model_family | prithvi_eo_v2_dlv3 | prithvi_eo_v2（`main/prithvi_upernet.py`，官方 terratorch UperNet） |
| feature_indices | [23] | [5, 11, 17, 23] |
| decoder | ASPP rates 6/12/18 | UperNet decoder_channels 256、head_dropout 0.1 |
| 其餘 | Tversky 0.3/0.7、batch 16、lr0 5e-5、AMP、每 epoch 4,000 步、最多 40 epoch、patience 8、VAL 37 ROI 步距 256 選 best | 同左 |

## 4. 量化結果

### 三 seed 臺灣 24 張 ROI 結果（2026-11-11 01:57 評分；描述性）

訓練：S42 11:40–16:23（best ve 12）、S43 16:23–20:59（best ve 14）、S44 20:59–01:24（best ve 13）。`eval_p50.json` sha b31a49ceea50…；P44_best 重算 0.300。

| 臺灣24 ROI pooled OilIoU | UperNet 三 seed | DLV3 三 seed | UPN−DLV3 配對（S42/S43/S44） |
|---|---|---|---|
| 全部 24 張 | 0.299±0.016（.317/.293/.287） | 0.311±0.014 | −0.000／−0.002／−0.032 |
| 扣大林（22） | 0.172±0.005 | 0.184±0.044 | −0.058／+0.030／−0.008 |
| Landsat（11） | 0.147±0.010 | 0.175±0.038 | −0.067／+0.014／−0.033 |
| Sentinel-2（13） | 0.378±0.010 | 0.378±0.015 | +0.009／+0.006／−0.016 |
| 大林（2） | 0.451±0.018 | 0.493±0.042 | +0.014／−0.081／−0.059 |
| 19 張／新增5張 | 0.311±0.008／0.210±0.066 | 0.320±0.014／0.209±0.048 | |
| P／R／pred-gt | 0.44／0.48／1.09 | 0.40／0.60／1.52 | |
| VAL best | 0.188／0.206／0.210 | 0.198／0.223／0.194 | |

逐張勝同 seed DLV3：14／14／13 張（24 張中）。參考 P44_best（自然比例）：扣大林 0.267、Landsat 0.286。

## 5. 觀察與結論

- **判讀（§3 條件）**：「三個 seed 扣大林與 Landsat 都高於同 seed DLV3」不成立（扣大林 −0.058／+0.030／−0.008、Landsat −0.067／+0.014／−0.033）→ **not better，不建議換 UperNet 當 P49 teacher**（最終仍由使用者決定）。
- 全部 24 張 UperNet 0.299±0.016 與 DLV3 0.311±0.014 分不開；Sentinel-2 13 張持平（0.378 vs 0.378），大林兩張偏低（0.451 vs 0.493）。
- UperNet seed 間差較小（扣大林 ±0.005 vs ±0.044）、誤判較少（pred/gt 1.09 vs 1.52、P 0.44 vs 0.40），但漏判較多（R 0.48 vs 0.60），總平均抵銷。
- 兩種頭在扣大林（0.172／0.184）與 Landsat（0.147／0.175）都遠低於 P44（自然比例）的 0.267／0.286；換分割頭並沒有補上這個缺口。
- 限制：三 seed、描述性、未經 Codex；Prithvi 各層同為 1/16 解析度，UperNet 多尺度的細線助益有限（卡 §4 已預期）。

## 6. 後續改進方向

- [x] smoke（S42，60 步、VAL 2 景）通過
- [x] S42、S43 同時 launch（11:40；12:44 停 S43 改依序）
- [x] 12:44 改依序（`launch_seq.sh`，commit 8f9d78a）：S42 → S43 → S44 訓練完成
- [x] 三 seed 臺灣 24 張 ROI 預測與評分完成（11-11 01:57）
- [x] 依評分結果：扣大林與 Landsat 未三 seed 皆升 → 不建議換 P49 teacher
- [ ] 是否仍拿 UperNet 或其他方向重跑 P49 teacher：待使用者決定

## 7. 實驗檔案位置

- 卡片與程式：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p50_upernet_h1_20261110/`（`P50_CARD.md`、`p50.py`、`launch.sh`、`launch_seq.sh`）
- 產物資料夾：`/home/alanyh/oil_dataset/new/_p50_upernet_h1_20261110/`（`eval_p50.json`、`eval_p50_per_scene.csv`、`train_UPN_S42/S43/S44`、`predictions/`；`train_UPN_S43_failed_attempt1/` 為被停掉的並行 S43，epoch 3）

## 執行時間線

| 時間（Asia/Taipei） | 事件 |
|---|---|
| 11-10 稍早 | smoke（S42，60 步、VAL 2 景）：5.5 it/s、峰值 8.6 GiB，loss 有限 |
| 11:40 | launch（S42、S43 並行，之後接 S44 → 臺灣 24 張 ROI 預測 → 評分）；持有共用 GPU lock |
| 11-10 12:44 | 使用者要求「停掉43啦 一個 一個跑」：停掉並行的 S43（epoch 3，移到 `train_UPN_S43_failed_attempt1/`）與原 `launch.sh`，改 `launch_seq.sh`（commit 8f9d78a）依序跑 |
| 11-10 11:40–16:23 | S42 訓練（best ve 12） |
| 11-10 16:23–20:59 | S43 從頭重跑（best ve 14） |
| 11-10 20:59–11-11 01:24 | S44 訓練（best ve 13） |
| 11-11 01:24–01:57 | 三 seed 臺灣 24 張 ROI 預測 |
| 11-11 01:57 | 評分完成（`eval_p50.json` sha b31a49ceea50…）；無 process 在跑 |

## 卡片全文（P50_CARD.md，commit ab23095 副本，含 §5 執行段修改與 §6 結果）

### P50：H1′（1:1+0.5 HNM+SI）× Prithvi-EO-2.0-300M + UperNet，一般監督式分割，三 seed

狀態：使用者 2026-11-10 授權執行。原話：「1:1.5HNM + SI的部分 使用Prithvi + UperNet的部分，然後我們先試試teacher model 意即 一搬的分割模式 看一下成效如何跑個三次吧」。
「看一下成效」解讀為授權跑完後讀臺灣 24 張 GT 評分。未經 Codex 審；描述性實驗，無預先登記門檻。

#### 1. 問題
P49 顯示 LFMDA student 在各子集的強弱幾乎照抄 teacher（H1′ S43，DeepLabV3 頭：扣大林 0.139、Landsat 0.142、大林 0.537）。
P50 問：只把分割頭從 DeepLabV3（ASPP，只吃第 23 層）換成 UperNet（第 5/11/17/23 層多尺度融合），其他完全不變，H1′ 在臺灣 24 張 ROI 上會不會變好，特別是扣掉大林與 Landsat。結果決定是否拿 UperNet 當 P49 的新 teacher。

#### 2. 設定（相對 H1′ DLV3：P46 S42、P48 S43/S44）
| 項目 | H1′ DLV3 | P50 |
|---|---|---|
| architecture_cfg.model_family | prithvi_eo_v2_dlv3 | **prithvi_eo_v2**（`main/prithvi_upernet.py`，官方 terratorch UperNet） |
| feature_indices | [23] | **[5, 11, 17, 23]** |
| decoder | ASPP rates 6/12/18 | **UperNet decoder_channels 256、head_dropout 0.1**（同 GT_expand prithvi300m_upernet 3-fold） |
| 參數分組 | encoder ×0.1 lr／head lr0 | encoder ×0.1 lr／**decoder+head** lr0（`train_p44.param_groups` 只認 encoder+head，P50 wrapper 換成 encoder 對 decoder+head，同樣檢查完整分割） |
| wandb | group P46 | group P50、名稱 P50_UPN_S{seed} |

其餘逐項相同：同一支 trainer（`train_p46.py` 未改）、同 seed 的同一份 H1′ schedule（S42→P46 sched_H1、S43/S44→P48 sched_H1_S43/S44）、同 config（`p50.py configs` 檢查只有上列 architecture 鍵不同）、Tversky 0.3/0.7、batch 16、lr0 5e-5、AMP、每 epoch 4,000 步、最多 40 epoch、patience 8、VAL 37 ROI 步距 256 選 best。
臺灣算子＝P44 凍結算子（27 ROI、步距 128、TTA、fp32），predict 程式照抄 P48。

#### 3. 評估
臺灣 24 張 ROI pooled OilIoU（`evaluate_p45.score`），P50 三 seed 與同 seed H1′ DLV3 配對：全部、扣大林、Landsat（11）、Sentinel-2（13）、大林；19 張／新增5張並列；逐張勝負；P44_best 0.300 一起重算。
描述性。只有「三個 seed 在扣大林與 Landsat 都高於同 seed DLV3」時，才建議拿 UperNet 當 P49 teacher 重跑（仍需使用者決定）。

#### 4. 已知限制
- 三 seed；DLV3 對照三 seed 臺灣 0.311±0.014，扣大林 0.139–0.228，seed 間差已不小。
- UperNet 在 GoM 的 0.4498 為決定性修復前的 legacy 數字，從未在 ROI 資料上訓練過。
- Prithvi 各層都是 1/16 解析度，UperNet 的多尺度是從同一 16×16 網格上下採樣而來，細線改善可能有限。
- 計畫任務基線綁 DeepLabV3，P50 只是研究支線。

#### 5. 程式與執行
`analysis/p50_upernet_h1_20261110/p50.py`（configs／train／predict／evaluate）、`launch.sh`。
smoke（S42，60 步、VAL 2 景）：5.5 it/s、峰值 8.6 GiB，loss 有限。
執行：11:40 `launch.sh` 讓 S42 與 S43 同時訓練（各約 3 it/s，總吞吐與單跑相同）。12:44 使用者要求「停掉43 一個一個跑」：停掉 launch.sh 與 S43（epoch 3 中，移到 `train_UPN_S43_failed_attempt1/`，wandb run 中斷），S42 繼續；`launch_seq.sh` 等 S42 釋放 GPU lock 後依序跑 S43（從頭）→ S44 → 三 seed 預測 → 評分。
產物：`/home/alanyh/oil_dataset/new/_p50_upernet_h1_20261110/`。

#### 6. 結果（2026-11-11 01:57，三 seed；描述性）
訓練：S42 11:40–16:23（best ve 12）、S43 16:23–20:59（best ve 14）、S44 20:59–01:24（best ve 13）。`eval_p50.json` sha b31a49ceea50…；P44_best 重算 0.300。

| 臺灣24 ROI pooled OilIoU | UperNet 三 seed | DLV3 三 seed | UPN−DLV3 配對（S42/S43/S44） |
|---|---|---|---|
| 全部 24 張 | 0.299±0.016（.317/.293/.287） | 0.311±0.014 | −0.000／−0.002／−0.032 |
| 扣大林（22） | 0.172±0.005 | 0.184±0.044 | −0.058／+0.030／−0.008 |
| Landsat（11） | 0.147±0.010 | 0.175±0.038 | −0.067／+0.014／−0.033 |
| Sentinel-2（13） | 0.378±0.010 | 0.378±0.015 | +0.009／+0.006／−0.016 |
| 大林（2） | 0.451±0.018 | 0.493±0.042 | +0.014／−0.081／−0.059 |
| 19 張／新增5張 | 0.311±0.008／0.210±0.066 | 0.320±0.014／0.209±0.048 | |
| P／R／pred-gt | 0.44／0.48／1.09 | 0.40／0.60／1.52 | |
| VAL best | 0.188／0.206／0.210 | 0.198／0.223／0.194 | |

逐張勝同 seed DLV3：14／14／13 張（24 張中）。參考 P44_best：扣大林 0.267、Landsat 0.286。
**判讀**：§3 條件（三 seed 扣大林與 Landsat 都升）不成立 → 不建議換 UperNet 當 P49 teacher。UperNet 的 seed 間差明顯較小、誤判較少（pred/gt 1.09 vs 1.52）但漏判較多，平均與 DLV3 分不開；兩種頭在扣大林／Landsat 都仍遠低於 P44（自然比例）。

## 相關頁面

- [[20261110_P49_LFMDA搬到ROI_三seed_執行紀錄]]
- [[20261108_P48補seed_H1與P44]]
- [[20261107_P46大圖版配比與OOF探勘]]
- [[20261106_P44_ROI重疊滑窗全球排除臺灣設計]]
