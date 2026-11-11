---
date: 2026-11-03
type: experiment
stage: evaluation
status: completed
tags: [p39, lfmda, self-training, sam, lrh, taiwan-19, gt-roi, oil-iou, not-improved, experiment-result]
---

# P39 LFMDA 第二階段（自訓練＋SAM＋LRH）對 P37 source-only 結果

> 本頁為 P39 的結果頁。事前凍結卡見 [[20261103_P39_LFMDA_stage2_SAM_LRH設計]]（`.agents/decisions/`，FROZEN v4，SHA `f89c9470…`，**本頁未修改**）；承接的 warmup 見 [[20261103_P38_LFMDA_warmupPCD_結果]]（已判 not improved）。設計與判準不在此重抄。
>
> 臺灣 GT 於 2026-11-03 經使用者授權讀取。結案日期 2026-11-03。

## 1. 實驗背景

P39 為 LFMDA 作者流程的第二階段（自訓練＋SAM 區域＋LRH），承接 P38 warmup 產物，依使用者「多階段方法先每步走一次」方針執行。三個 run 為 S2333→S2334→S2335，於 2026-11-03 10:20–12:04 完成（repo commit `81c381b`，各約 33.5 分，EXIT=0）。對照組 C 與 P38 T 的數值取自封存的 P38 `eval_result.json`（`7b18bd53…`），未重算。

## 2. 方法架構 / Pipeline 流程

- 評估器：`evaluate_p39.py`（repo commit `962c5f4`），讀 GT 前封存 `eval_seal.json`，SHA `990c3744babdf35384aaf1a652301fc13b446276c83374ff8aef0be13f5c1bf4`。
- 使用者 2026-11-03 授權讀臺灣 GT，產出 `eval_result.json`，SHA `c03963fa282116fe65ff5a7922bc01a4131b4a9534c93e90cbd39df7fe6fd7d3`。
- 主判定 H1：臺灣 19 景、學生 final。H1b：P39 − P38 T 同 seed。H2：GoM val final 保留。H4：新增五景 micro recall（探索）。

## 3. 關鍵參數

| 項目 | 內容 |
|---|---|
| run | S2333／S2334／S2335，各約 33.5 分，EXIT=0 |
| 訓練程式 commit | `81c381b` |
| 評估程式 | `evaluate_p39.py`（repo commit `962c5f4`） |
| 封存 SHA（eval_seal.json） | `990c3744babdf35384aaf1a652301fc13b446276c83374ff8aef0be13f5c1bf4` |
| 輸出 SHA（eval_result.json） | `c03963fa282116fe65ff5a7922bc01a4131b4a9534c93e90cbd39df7fe6fd7d3` |
| 對照 | C（P37 source-only）與 P38 T，取自 P38 `eval_result.json`（`7b18bd53…`） |

## 4. 量化結果

| 項目 | 判定 | 數字 |
|---|---|---|
| H1（臺灣 19 景、學生 final） | **否定** | C 0.3563／0.3349／0.2530（平均 0.3147）vs P39 0.1759／0.2097／0.0908；Δ̄ −0.156、s_p 0.058；19 景改善 0 景；sign-flip p＝1.0 |
| H1b（P39−P38T 同 seed） | 無正向 change | d＝−0.078／+0.008／−0.085，平均 −0.052 |
| H2（GoM val final） | 未保留 | 0.3287／0.3044／0.2893 vs C 平均 0.3742，Δ̄ −0.067，三個都低於 C |
| H4（新增五景 micro recall，探索） | 無正向訊號 | C 0.2516／0.5287／0.4184 vs P39 0.2114／0.0360／0.0001，ΔR −0.317 |
| **整體** | **not improved／失敗** | |

- run 旗標：S2335 pooled OilIoU 0.0908 ≤ 0.1582（觸發 floor）；預測油比例 3.45%／4.19%／1.81%，皆未觸發近塌縮。只有 1 個 run 觸發旗標，否定是由 (b)「Δ̄≤0 且 ≥2 run ≤ C 平均」成立。
- P/R：precision 0.836／0.828／0.812，recall 0.182／0.219／0.093 → 預測得準但太少，主要是漏判。
- 新增五景預測油比例 0.76%／0.17%／0.00%（S2335 在五景幾乎不預測油）。
- 學生 best（次要）：0.1973／0.2010／0.1756，同樣否定。教師 final 19 景：0.1627／0.1947／0.0886。

**逐景差（P39 三 run 平均 − C 三 run 平均，學生 final）**

19 景全部為負，範圍 −0.074（大林 50QRK）～ −0.661（taiwan_file 20261004 50RPN）；scene-macro −0.300。

## 5. 觀察與結論

機制紀錄（不讀 GT；三 run 封存後揭露）：

- 偽標籤：教師在 270 窗上的油機率 >0.5 比例，step 0 為 3.5%／5.0%／2.1%，step 3000 降到 2.0%／2.6%／1.2%。
- 選取後油像素只占 target 像素 1–2.5%；LRH 幾乎單向：每 1,000 步「油→背景」5 萬～106 萬 px，「背景→油」僅數百～一千多 px；「忽略→背景」遠多於「忽略→油」。
- SAM ViT-H 在固定 8-bit RGB proxy 上：270 窗中 168 窗只有 1 個區域（2／3／4 個區域：56／41／5 窗），221 窗區域覆蓋整窗 → LRH 等於整窗多數決，實際作用是把少量的油抹成背景。
- GoM val best step＝3000／1／1：兩個 run 在第 1 步之後 GoM 就沒變好。

解讀（不推翻上述判定）：

- 接線正確但 SAM 區域幾乎不分割、LRH 系統性擦除油 → 依卡屬「有效的科學結果」，非工程無效。
- P39 不可歸因於 SAM／LRH 單項（transductive continuation package）。結果與「偽標籤稀少＋LRH 整窗多數決 → 自訓練往背景收斂 → 漏判加重」一致，但未做消融，屬推測。

## 6. 後續改進方向

依使用者「每步先走一次」方針，LFMDA 作者流程（warmup＋PCD → 自訓練＋SAM＋LRH）已全部走完一次；兩階段在本案都 not improved。回頭改善的候選（未授權、需另立卡）：

- [ ] SAM 輸入改拉伸或其他 proxy
- [ ] 面積門檻依 256 窗尺度縮放
- [ ] 關 LRH 對照
- [ ] 偽標籤門檻

## 7. 實驗檔案位置

- 結果原稿：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p39_lfmda_stage2_sam_lrh_20261103/P39_RESULTS.md`
- 評估程式：`analysis/p39_lfmda_stage2_sam_lrh_20261103/evaluate_p39.py`（repo commit `962c5f4`）
- 事前卡：`/home/alanyh/obsidian-vault/.agents/decisions/20261103_P39_LFMDA_stage2_SAM_LRH設計.md`
- 訓練輸出資料夾（log）：`/home/alanyh/oil_dataset/new/_p39_lfmda_stage2_20261103/`

## 事後補充：臺灣 24 張（2026-11-03 規則變更）

使用者 2026-11-03 裁決：臺灣新增五張（11-02 補的 Landsat 8：天使輪 2023-07-24 50QRK／50QRL／51QTF、2023-08-25 51QTF；taiwan_file 2026-05-29 50QRK）**油標註以外視為背景**，取代 2026-11-01「標註外為 unknown、不可合併」規則；**往後新實驗主報臺灣 24 張 pooled OilIoU**，19 張與新增五張並列。五張用 coverage-union、排除六波段無效像素；19 張照原 Lane A 口徑。

本頁原判定（H1 否定、整體 not improved）**不改**；下表只是依新規則的事後補算。

| 組別（學生 final，三 run 平均） | 19 張 | 新增五張 | 24 張 |
|---|---|---|---|
| C（source-only） | 0.3147 | 0.3344 | 0.3156 |
| P38 T | 0.2103 | 0.3933 | 0.2183 |
| P39 | 0.1588 | 0.0813 | 0.1561 |

各 run 24 張 pooled OilIoU：C 0.3514／0.3384／0.2572；P38 T 0.2646／0.2147／0.1757；P39 0.1770／0.2037／0.0876。

純算術事後補算，沒有重讀 GT 或預測檔：程式 repo `GT_expand/analysis/taiwan24_merged_iou_20261103/merge24.py`，結果 `/home/alanyh/oil_dataset/new/_taiwan24_merged_20261103/merged24_result.json`（SHA `96daa0b0fff4bcbbd3288f0e69c9a0c403fc912f7b6f291604772d12f3dc5e58`）。五張只占 36/306 窗，24 張排序與 19 張相同（C > P38 T > P39）。
