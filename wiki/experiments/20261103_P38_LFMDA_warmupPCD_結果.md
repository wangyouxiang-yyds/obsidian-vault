---
date: 2026-11-03
type: experiment
stage: evaluation
status: completed
tags: [p38, lfmda, warmup, pcd, dinov2-teacher, taiwan-19, gt-roi, oil-iou, not-improved, experiment-result]
---

# P38 LFMDA warmup（DINOv2 教師＋PCD）對 P37 source-only 結果

> 本頁為 P38 的結果頁（凍結卡第 7–9 步）。事前凍結卡見 [[20261101_P38_LFMDA_warmupPCD設計]]（`.agents/decisions/`，v7，SHA `700d4883…`，**本頁未修改**）；協商紀錄見 [[20261101_P38_LFMDA_warmupPCD事前卡協商]]；對照組 C 為 [[20261101_P37_LFMDA_SegFormerB2_sourceonly_結果]]。設計與判準不在此重抄。
>
> 臺灣 GT 於 2026-11-03 經使用者授權讀取一次。結案日期 2026-11-03。

## 1. 實驗背景

P38 以 P37 source-only（C）為對照，檢驗本案 GoM-normalized LFMDA warmup（DINOv2 教師＋PCD，學生 T）是否在臺灣原 19 景上優於 C。稱呼：本案 GoM-normalized LFMDA warmup 實作，transductive（看過 19 景 270 窗影像），非 zero-shot。

## 2. 方法架構 / Pipeline 流程

- 評估：`evaluate_p38.py`（repo commit 710eb4c），評估前封存 `eval_seal.json`（SHA `8eae0e97…`）。
- 主判定 H1：學生 final，原 19 景 coverage-union pooled OilIoU。
- 次要：GoM-val best 學生（secondary）。H2：GoM 保留。H4：新增五景已標油 recall（探索）。

## 3. 關鍵參數

| 項目 | 內容 |
|---|---|
| 評估輸出 | `/home/alanyh/oil_dataset/new/_p38_lfmda_warmup_20261102/eval_result.json` |
| 輸出 SHA-256 | `7b18bd53bad7d520d5dc397bb707c6fb333c30e7870c7bf836a1c18aa4c967f3` |
| 封存 SHA（eval_seal.json） | `8eae0e97…` |
| 評估程式 | `evaluate_p38.py`（repo commit 710eb4c） |
| 對照組 / 處理組 | C（P37 source-only）／T（warmup 學生），各三個 run |

## 4. 量化結果

**H1（主判定，學生 final，原 19 景 coverage-union pooled OilIoU）：否定（rejected）**

| | 三個 run | 平均 |
|---|---|---|
| C（P37 source-only） | 0.3563／0.3349／0.2530 | 0.3147 |
| T（warmup 學生） | 0.2538／0.2017／0.1754 | 0.2103 |

- Δ̄ = −0.1044，s_p = 0.0478；T 三個 run 全部低於 C 平均（0/3 高於）；9 個跨組比較中 T>C 只有 1 個（0.111）。
- 失敗旗標：無（T 預測油比例 3.3–4.9%，皆高於 floor 0.1582）。否定來自方向條件（Δ̄≤0 且 3/3 T ≤ C 平均）。
- 逐景：scene-macro −0.103，19 景只有 4 景改善（最多 +0.038，皆天使輪 L9），sign-flip one-sided p = 0.9999。降最多的是 taiwan_file S2 新景（−0.20～−0.37）。
- P/R：C precision ≈0.81、recall 0.27–0.39；T precision 0.77–0.86、recall 0.18–0.27 → 漏判更多。
- 教師 final 自身 19 景 IoU 0.2411／0.2536／0.1398，也低於 C。
- secondary（GoM-val best 學生）：C 0.3224／0.3159／0.2827 vs T 0.2497／0.2772／0.2282，Δ̄ −0.0553，同為 rejected。

**H2（GoM 保留）：未保留**

| | 三個 run（GoM val final OilIoU） | 平均 |
|---|---|---|
| C | 0.3763／0.3906／0.3556 | 0.3742 |
| T | 0.2960／0.3492／0.3284 | 0.3245 |

Δ̄ −0.0497，3/3 T 低於 C 平均。

**H4（新增五景，已標油 recall，探索）：positive_signal**

- micro recall C 0.252／0.529／0.418 vs T 0.666／0.653／0.203，ΔR +0.108；4/5 景不下降。
- 限制：只有 recall、沒有背景標註；T 預測油比例也較高（3.4／3.2／1.0% vs C 1.3／2.6／1.9%），且兩組各自的 seed 間差距（C 0.25–0.53、T 0.20–0.67）大於組間差，不能當改善證據，也不推翻 H1。

**整體：not improved／失敗**（H1 否定，依卡 H2 不改變此結論）。

## 5. 觀察與結論

- 本結果只說明「本案 GoM-normalized warmup 完整政策」不如 P37 source-only；不得寫成 PCD 單獨無效（T 同時改了 LR 軌跡、無裁剪、混合 batch、target clamp）。
- 與事前列出的機制風險一致：教師在 target 的高信心像素約 91%，其中油僅 1.3–1.4%；PCD 用到的 target 油格約 0.35%，對比訊號幾乎全是背景；作者 1/32 降採樣下 GoM 含油窗只有 518/1,188 留下油格。學生 recall 下降，方向與「背景主導原型把特徵推向背景」相符，但沒有做分離實驗，只能當線索。
- 對照組本身 seed 波動大：C 三 run 0.253–0.356（P37 S2333 是三者最高）。

## 6. 後續改進方向

使用者 2026-11-03 方針：多階段方法先每步走一次再改善。P38 不因否定而停止流程，接著做作者第二階段（SAM 局部區域＋自訓練＋LRH）為 P39，另立事前卡。

- [ ] 起草 P39 事前卡；P39 以 P38 warmup 產物為起點時，須在卡上寫明它承接的是一個已判 not improved 的 warmup

## 7. 實驗檔案位置

- 評估輸出：`/home/alanyh/oil_dataset/new/_p38_lfmda_warmup_20261102/eval_result.json`
- 封存：`/home/alanyh/oil_dataset/new/_p38_lfmda_warmup_20261102/eval_seal.json`
- 事前卡：`/home/alanyh/obsidian-vault/.agents/decisions/20261101_P38_LFMDA_warmupPCD設計.md`
- 協商紀錄：`/home/alanyh/obsidian-vault/.agents/decisions/20261101_P38_LFMDA_warmupPCD事前卡協商.md`
- 結果原稿：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p38_lfmda_warmup_pcd_20261101/P38_RESULTS.md`
- 評估程式：`analysis/p38_lfmda_warmup_pcd_20261101/evaluate_p38.py`（repo commit 710eb4c）

## 事後補充：臺灣 24 張（2026-11-03 規則變更）

使用者 2026-11-03 裁決：臺灣新增五張（11-02 補的 Landsat 8：天使輪 2023-07-24 50QRK／50QRL／51QTF、2023-08-25 51QTF；taiwan_file 2026-05-29 50QRK）**油標註以外視為背景**，取代 2026-11-01「標註外為 unknown、不可合併」規則；**往後新實驗主報臺灣 24 張 pooled OilIoU**，19 張與新增五張並列。五張用 coverage-union、排除六波段無效像素；19 張照原 Lane A 口徑。

本頁原判定（H1 否定、整體 not improved）**不改**；下表只是依新規則的事後補算。

| 組別（學生 final，三 run 平均） | 19 張 | 新增五張 | 24 張 |
|---|---|---|---|
| C（source-only） | 0.3147 | 0.3344 | 0.3156 |
| P38 T | 0.2103 | 0.3933 | 0.2183 |

各 run 24 張 pooled OilIoU：C 0.3514／0.3384／0.2572；P38 T 0.2646／0.2147／0.1757。

純算術事後補算，沒有重讀 GT 或預測檔：程式 repo `GT_expand/analysis/taiwan24_merged_iou_20261103/merge24.py`，結果 `/home/alanyh/oil_dataset/new/_taiwan24_merged_20261103/merged24_result.json`（SHA `96daa0b0fff4bcbbd3288f0e69c9a0c403fc912f7b6f291604772d12f3dc5e58`）。五張只占 36/306 窗，24 張排序與 19 張相同（C 高於 P38 T）。
