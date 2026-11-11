---
date: 2026-11-10
type: experiment
stage: evaluation
status: 完成
tags: [lfmda, roi, roi3840, p49, three-seed, pcd, sam, lrh, descriptive, transductive, taiwan24, segformer-b2, h1-prime-teacher, execution-record]
---

# P49：LFMDA 整條流程搬到 P44 ROI 資料（三 seed）——臺灣 24 張 S2 final 0.355±0.060（高於 P44 0.300，但增益全靠大林兩張，描述性）

> 本頁為執行紀錄，內含卡片全文副本（原檔 `analysis/p49_lfmda_roi_20261110/P49_CARD.md`，repo commit 9c965fe）。承接 [[20261109_ROI_probe_既有LFMDA模型套ROI3840_執行紀錄]]（既有 LFMDA 模型直接套 ROI 口徑，T3 0.341±0.009）；對照基準見 [[20261106_P44_ROI重疊滑窗全球排除臺灣設計]]、[[20261108_P48補seed_H1與P44]]、[[20261107_P46大圖版配比與OOF探勘]]；LFMDA 流程前身見 [[20261105_P43_LFMDA流程補seed穩定性_執行紀錄]]。**描述性、transductive、未經 Codex、無預先登記門檻**。

## 1. 實驗背景

ROI probe（11-09）顯示 T3 在 ROI 3840 口徑沒崩（0.341），但 teacher（P15）本身在該口徑誤判嚴重（0.178）。使用者 2026-11-10 授權把整條 LFMDA（stage 1 PCD＋stage 2 自訓練／SAM／LRH）改用 ROI 資料訓練（source＝H1′ 取樣、teacher＝H1′ S43、target＝臺灣 24 張 ROI 窗），先跑三個 seed，再做臺灣推論與評估。使用者原話：「直接幫我安排並進行實驗吧……跑一整個完整的流程，先幫我跑三次 seed，之後在幫我進行到台灣的推論與評估」、「接下來都自行執行不用我授權我要去睡了」（11-10 00:00 前）。

## 2. 方法架構 / Pipeline 流程

見下方「卡片全文」第 2 節；摘要如下：

1. prep：target 快取、teacher 固定偽標籤（只算一次）、stage 2 子集（teacher 油機率 >0.5 達 64 px 的窗，至多 1,000，＋1.5 倍其他窗）、SAM ViT-H 只對子集算。
2. stage 1：SegFormer-B2 6 波段、H1′ 串流 source（batch 8）＋臺灣 24 張 ROI 窗 target（batch 4）、PCD、10,000 步，VAL 37 張 ROI 窗選 best。
3. stage 2：自訓練＋SAM＋LRH，6,000 步，source 接續 H1′ 串流。
4. 對照 C6-ROI（S2333 一次）：同 student、同串流、無 teacher／target／PCD。
5. 臺灣 24 ROI 推論（P44 凍結算子、翻轉 TTA）＋`evaluate_p45.score` 計分，P44_best 同程式重算核對。

## 3. 關鍵參數

| 項目 | 內容 |
|---|---|
| Student | SegFormer-B2、6 波段（B04/B03/B02/B8A/B11/B12，8-bit，GoM mean/std） |
| Teacher | H1′ S43 best.pt（固定不更新） |
| Source | P44 TRAIN ROI 窗，H1′ 串流（seed 2333→H1′ S42、2334→S43、2335→S44 的批次順序） |
| Target | 臺灣 24 張 ROI 窗（21,039 窗，transductive） |
| Stage 1／2 步數 | 10,000／6,000 |
| 三個 seed | 2333／2334／2335 |
| 程式 commit | 第一輪 7815100；第二輪 baf527f（`run_rest.sh`）；卡片 9c965fe |
| 第二輪修正 | VAL batch 64→16＋`empty_cache` |
| seal | `probe_seal.json` sha-256 `d04460831c4b…` |
| 評分結果 | `eval_p49.json` sha-256 `1bc478442120…`；P44_best 重算 0.3002 |

## 4. 量化結果

臺灣 24 ROI pooled OilIoU（描述性；平均±SD 為三 seed）：

| 臺灣24 ROI pooled OilIoU | S2333 | S2334 | S2335 | 平均±SD |
|---|---|---|---|---|
| S1 final | 0.297 | 0.001（塌成全背景） | 0.235 | 0.178±0.156 |
| S1 best | 0.297 | 0.111 | 0.229 | 0.212±0.094 |
| **S2 final** | 0.307 | 0.422 | 0.336 | **0.355±0.060** |
| S2 best | 0.296 | 0.422 | 0.336 | 0.351±0.064 |
| C6-ROI | 0.097 | — | — | — |

- S2−S1 final 配對：+0.010／+0.422／+0.102（S2334 的 +0.422 來自 stage 1 塌縮）。
- S2 final：19 張 0.359±0.059、新增 5 張 0.295±0.064。
- 參考：ROI probe T3 0.341±0.009、P42 0.329、P15 0.178、C6(GoM) 0.251；H1′ 0.311±0.014；P44 0.300；P12 0.216。

**分解（S2 final 三 seed）**：

| 子集 | P49 S2 final | P44 |
|---|---|---|
| 大林兩張（GT 約 85 萬 px，約占 24 張 GT 的 59%） | IoU 0.52–0.62 | 0.37／0.31 |
| 扣掉大林 pooled | 0.109–0.152 | 0.267 |
| Landsat 11 張 | 0.120–0.174 | 0.286 |
| S2 衛星 13 張 | 0.378–0.504 | 0.308 |
| 逐張勝 P44 | 7／7／8 張（三 seed） | — |

TP 有 80–85% 來自大林。

## 5. 觀察與結論

（描述性）

- 主數字（S2 final 0.355±0.060）高於 P44 0.300 與 H1′ 0.311±0.014，與 ROI probe T3 0.341±0.009 相近。
- 但增益全靠大林兩張；其餘 22 張普遍低於 P44，Landsat 掉分明顯（0.120–0.174 vs 0.286）。**不能說 LFMDA-ROI 整體改善。**
- Stage 1 在 S2334 塌成全背景（final 0.001），stage 2 救回（0.422）；seed 間差異大（S2 final SD 0.060），與 P43（SD 0.002）的穩定性形成對比。
- C6-ROI 單次 0.097，遠低於各 S1／S2，但只有一次、不作結論。

**限制**：

- Target 24 張即測試 24 張（transductive），新增 5 張不再是 test-only。
- Stage 2 只在子集上自訓練（算力限制），子集由 teacher 預測挑（不讀 GT）。
- 三個 seed 只換 student seed 與 source 批次順序；teacher、SAM 區域、子集固定。
- 未經 Codex 審、無預先登記門檻；臺灣是重用基準。
- 與 T3 同時改了 source、teacher、target、選模，只能比整包。

## 6. 後續改進方向

- [x] 三 seed stage 1＋stage 2 完成、臺灣 24 ROI 推論與評估（2026-11-10 10:12）
- [ ] 是否處理「大林主導」問題（例如分層報告、扣大林的主指標）——待使用者決定
- [ ] 是否處理 Landsat 掉分（0.120–0.174 vs P44 0.286）——待使用者決定
- [ ] 是否補 seed／穩定 stage 1 塌縮（S2334）——待使用者決定

## 7. 實驗檔案位置

- 卡片與程式：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p49_lfmda_roi_20261110/`（`P49_CARD.md`、`run_all.sh`、`run_rest.sh`、`prep49.py`、`stage1_49.py`、`stage2_49.py`、`predict49.py`、`evaluate49.py`、`p49_common.py`）
- 輸出資料夾：`/home/alanyh/oil_dataset/new/_p49_lfmda_roi_20261110/`
- 結果：`/home/alanyh/oil_dataset/new/_p49_lfmda_roi_20261110/eval_p49.json`（sha-256 開頭 `1bc478442120`）
- 逐景：`/home/alanyh/oil_dataset/new/_p49_lfmda_roi_20261110/eval_p49_per_scene.csv`
- seal：`/home/alanyh/oil_dataset/new/_p49_lfmda_roi_20261110/probe_seal.json`（sha-256 開頭 `d04460831c4b`）
- 記錄：`.../run_all.log`（第一輪）、`.../run_rest.stdout`、`.../r2_*.log`（第二輪）
- 第一輪封存：`.../round1_eval_p49.json`、`.../round1_eval_p49_per_scene.csv`、`.../round1_probe_seal.json`、`.../round1_failed/`（S1_S2333、S1_S2335）

## 執行時間線

| 時間（2026-11-10，Asia/Taipei） | 事件 |
|---|---|
| 00:00 前 | 使用者授權無人值守：「接下來都自行執行不用我授權我要去睡了」 |
| 00:16 | 第一輪 launch（`run_all.sh`，commit 7815100） |
| 第一輪中 | S2333／S2335 stage 1 在 step 1000 的 VAL batch 64 發生 GPU OOM（3 訓練＋2 SAM 同卡 32GB） |
| 04:03 | 第一輪結束，只有 S2334 完成（S1 final 0.001 塌縮、S2 final 0.422、C6-ROI 0.097） |
| 07:51 | 第二輪 `run_rest.sh` launch（VAL batch 16＋`empty_cache`，commit baf527f） |
| 08:45 | stage 1 完成，無塌縮（S2333／S2335） |
| 09:4x | stage 2 完成 |
| 10:10 | 重新封存（`probe_seal.json` sha d04460831c4b…） |
| 10:12 | 評分完成（`eval_p49.json` sha 1bc478442120…） |

## 卡片全文（P49_CARD.md，commit 9c965fe 副本）

### P49：LFMDA 整條流程（stage 1 PCD＋stage 2 自訓練／SAM／LRH）搬到 P44 ROI 資料，三 seed

狀態：**使用者授權後直接執行（2026-11-10 00:00 前後）**。使用者原話：「直接幫我安排並進行實驗吧……跑一整個完整的流程，先幫我跑三次 seed，之後在幫我進行到台灣的推論與評估」、「接下來都自行執行不用我授權我要去睡了」。
未經 Codex 審（使用者要求當晚無人值守完成）；讀臺灣 GT 已含在授權內。描述性實驗，無預先登記門檻。

#### 1. 問題
ROI probe（11-09）顯示：GT 置中訓練、PCD 看過臺灣 270 窗的 LFMDA student（T3）在 ROI 3840 口徑上是 0.341±0.009，高於 P44 0.300／H1′ 0.311；但 P15 teacher 只有 0.178（誤判多）。
P49 問：把整條 LFMDA 換成 ROI 資料（source＝H1′ 取樣、teacher＝ROI 訓練的 H1′、target＝臺灣 24 張 ROI 窗），三個 seed 在臺灣 24 張 ROI 口徑上落在哪裡？

#### 2. 設定（相對 P41 T3 / P42 的差異）
| 項目 | P41/P42 | P49 |
|---|---|---|
| Source | GoM GT 置中 2,738 patch | P44 TRAIN ROI 窗，H1′ 串流（油：隨機背景：OOF HNM+SI＝1:1:0.5；直接重用 H1′ 已凍結的 schedule：seed 2333→H1′ S42、2334→S43、2335→S44 的批次順序，切成每批 8） |
| 無效像素 | — | P44 規則，loss 忽略（−1） |
| Teacher | P15 GoM-only S42 | **H1′ S43** best.pt（P48 三 seed 中 VAL 0.223 最高，只依 VAL 選），固定不更新，P44 輸入 |
| 原型初始化 | 一個 source epoch | 一個 H1′ list-epoch（2,225 批） |
| Target | 臺灣 19 張 270 窗 | **臺灣 24 張 ROI 窗**（P44 rois_taiwan，步距 128，無效 >10% 剔除 → 21,039 窗；使用者 11-09 裁決用 24 張） |
| 選 best | GoM val mIoU | P44 VAL 37 張 ROI 窗（步距 256）pooled oil IoU |
| Stage 2 target | 270 窗全部 | **子集**：SAM ViT-H 每窗 2.7 秒，21k 窗要 16 小時 → teacher TTA 油機率 >0.5 達 64 px 的窗（至多 1,000）＋1.5 倍其他窗，seed 20261110；SAM 只對子集算 |
| Stage 2 偽標籤 | step 0 與 3000 各產生一次 | teacher 固定 → 兩次相同，只算一次（float16 油機率） |
| Source（stage 2） | GoM | H1′ 串流接續（第 80,000–127,999 個位置） |

其餘照 P41/P42：SegFormer-B2 6 波段（B04/B03/B02/B8A/B11/B12，8-bit，GoM mean/std）、stage 1 10,000 步、source batch 8＋target batch 4、AdamW 6e-5、poly 15,000／warmup 1,500、PCD 溫度 0.3、原型 EMA 0.99、τ 0.9、target clamp、作者翻轉；stage 2 6,000 步、poly 6,300、label_refine（temp 2）、pseudo_selection 0.8/0.6、LRH 0.95、SAM ViT-H（points 32、iou 0.90、stability 0.95、crop 1、面積 ≥1,024）。

**對照**：C6-ROI（S2333 一次）＝同 student、同 H1′ 串流、同排程與選模，無 teacher、無 target、無 PCD。

#### 3. 評估
臺灣 24 張 ROI pooled OilIoU（P44 凍結算子：27 ROI、步距 128、TTA logits 平均；`evaluate_p45.score`；P44_best 一起重算核對 0.300）。報 S1／S2 final（主）與 best、三 seed 平均±SD、S2−S1 配對差；19 張／新增5張並列。
參考：ROI probe T3 0.341±0.009、P42 0.329、P15 0.178、C6(GoM) 0.251；H1′ 0.311±0.014；P44 0.300；P12 0.216。

#### 4. 已知限制
- Target 24 張即測試 24 張（transductive），新增5張不再是 test-only。
- Stage 2 只在子集上自訓練（算力限制），子集由 teacher 預測挑（不讀 GT）。
- 三個 seed 只換 student seed 與 source 批次順序；teacher、SAM 區域、子集固定。
- 未經 Codex 審、無預先登記門檻；臺灣是重用基準。
- 與 T3（GoM source、P15 teacher、270 窗 target）同時改了 source、teacher、target、選模 → 只能比整包。

#### 5. 程式（`analysis/p49_lfmda_roi_20261110/`，commit 7815100）
p49_common.py（快取、views、H1′ 串流、VAL）、prep49.py（快取／偽標籤／子集／SAM）、stage1_49.py、stage2_49.py、predict49.py（沿用 ROI probe 推論）、evaluate49.py、run_all.sh。
核對：target 快取與 VRT 逐像素相同；teacher 補 0 的 B01/B08 槽對輸出零影響；student 前處理與 infer_target 差 2e-7；stage 1 smoke（PCD 版與 C6 版）loss 有限、teacher 無梯度、student 347 參數有梯度。
產物：`/home/alanyh/oil_dataset/new/_p49_lfmda_roi_20261110/`（run_all.log、各步 log、eval_p49.json）。

#### 6. 結果（2026-11-10 10:12，三 seed；描述性）
第一輪（commit 7815100）S2333/S2335 stage 1 在 VAL batch 64 時 GPU OOM，只有 S2334 完成；第二輪（`run_rest.sh`，commit baf527f，VAL batch 16）補跑兩個 seed，之後重新封存全部 artifact。
seal `probe_seal.json` sha d04460831c4b…；`eval_p49.json` sha 1bc478442120…；P44_best 重算 0.3002。

| 臺灣24 ROI pooled OilIoU | S2333 | S2334 | S2335 | 平均±SD |
|---|---|---|---|---|
| S1 final | 0.297 | 0.001（塌成全背景） | 0.235 | 0.178±0.156 |
| S1 best | 0.297 | 0.111 | 0.229 | 0.212±0.094 |
| **S2 final** | 0.307 | 0.422 | 0.336 | **0.355±0.060** |
| S2 best | 0.296 | 0.422 | 0.336 | 0.351±0.064 |
| C6-ROI | 0.097 | — | — | — |

S2−S1 final 配對：+0.010／+0.422／+0.102。S2 final 19 張 0.359±0.059、新增5張 0.295±0.064。

**分解（S2 final 三 seed）**：大林兩張（GT 85 萬 px，約占 24 張 GT 的 59%）IoU 0.52–0.62（P44 0.37/0.31），TP 有 80–85% 來自大林；扣掉大林 pooled 0.109–0.152（P44 0.267）；Landsat 11 張 0.120–0.174（P44 0.286），S2 衛星 13 張 0.378–0.504（P44 0.308）；逐張勝過 P44 的只有 7／7／8 張。
**判讀**：主數字高於 P44 0.300、H1′ 0.311±0.014，與 ROI probe T3 0.341±0.009 相近，但增益全靠大林兩張，其餘 22 張普遍低於 P44 → 不能說 LFMDA-ROI 整體改善。

## 相關頁面

- [[20261109_ROI_probe_既有LFMDA模型套ROI3840_執行紀錄]]
- [[20261105_P43_LFMDA流程補seed穩定性_執行紀錄]]
- [[20261106_P44_ROI重疊滑窗全球排除臺灣設計]]
- [[20261107_P46大圖版配比與OOF探勘]]
- [[20261108_P48補seed_H1與P44]]
- [[20261104_P41_LFMDA_PCD_固定P15teacher_執行紀錄]]
- [[20261104_P42_LFMDA_stage2_from_P41_T3_執行紀錄]]
