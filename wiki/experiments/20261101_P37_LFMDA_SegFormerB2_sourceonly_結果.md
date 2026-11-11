---
date: 2026-11-01
type: experiment
stage: evaluation
status: completed
tags: [p37, lfmda, segformer-b2, source-only, taiwan-19, lane-a, gt-roi, oil-iou, baseline-adequacy, experiment-result]
---

# P37 LFMDA SegFormer-B2 source-only 結果（Lane A：臺灣原 19 景 GT 評估）

> 本頁為 P37 基準的結果頁。事前凍結卡見 [[20261101_P37_LFMDA_SegFormerB2_sourceonly設計_v2]]（`.agents/decisions/`，sha-256 `9e5de5b1…`）；Lane B（新增五景已標油 recall）見 [[20261101_P37_臺灣24景補充_LaneB結果]]；24 景補充推論卡見 [[20261101_P37_臺灣24景補充推論設計]]（sha-256 `619e1dad…`）；編碼勘誤見 [[20261101_P37_臺灣預測圖編碼勘誤]]。**兩張凍結卡本頁皆未修改，設計與判準不在此重抄。**
>
> 使用者已明確授權 Lane A。評估使用勘誤後 0=oil 預測。

## 1. 實驗背景

P37 為 LFMDA 的 source-only 基準：SegFormer-B2 僅用 GoM 來源訓練，Lane A 於臺灣原 19 景（270 窗）以 GT 評估，作為 LFMDA 後續方法的對照基線。Lane B 已先完成（新五景，描述性）。

## 2. 方法架構 / Pipeline 流程

入口 `run_corrected_target_eval.py --encoding-erratum-sha256 4a48c199d629b8c878d4d667a0501b7eb841c935ed9bd99fa2aa81c5945955a8 --allow-target-gt`，用勘誤後 0=oil 預測。口徑：_kind patch，19 景／270 窗 coverage-union pooled，後列覆蓋，raw0=oil、其餘=BG（raw255 計背景）。

## 3. 關鍵參數

| 項目 | 內容 |
|---|---|
| 頭條 | final checkpoint，pooled OilIoU |
| 次報 | GoM-val-best checkpoint |
| 輸出 | `/home/alanyh/oil_dataset/new/_lfmda_sourceonly_20261101/target_eval_final_best.json` |
| 輸出 SHA-256 | `ff36cb9d3ad427a76662a897b584288e577f7bfe8505b0982720aa55b8469080` |
| 驗證 | 獨立重算 TP/FN/FP 完全一致 |
| repo commit | GT_expand 7030a77（本機未 push） |

## 4. 量化結果

| 指標 | final（頭條） | GoM-val-best（次報） |
|---|---|---|
| pooled OilIoU | 0.3563 | 0.3224 |
| precision | 0.814 | 0.820 |
| recall | 0.388 | 0.347 |
| F1 | 0.525 | 0.488 |
| TP | 546,136 | 488,513 |
| FN | 862,159 | 919,782 |
| FP | 124,526 | 107,167 |
| IoU=0 景數 | 0 | — |

判定量：同覆蓋 all-oil floor 0.1582；支持門檻 max(0.10, floor+0.05)=0.2082；GoM source-val final OilIoU 0.3763 ≥ 0.10、十筆 val 有限。**判定：support baseline adequacy**（卡上三分法）。

逐景 final IoU：

| 事件 | 景 | IoU |
|---|---|---|
| 大林 0624 S2 | 50QRK／51QTE | 0.392／0.382（兩景占覆蓋油像素約 60%） |
| 天使輪 0801 L9 | 50QRK／50QRL | 0.051／0.029 |
| 天使輪 0912 S2 | 50QRL | 0.251 |
| 天使輪 0918 L9 | 50QRK／50QRL | 0.524／0.308 |
| 天使輪 0922 S2 | 50QRK／50QRL | 0.348／0.369 |
| taiwan_file 0522 L8 | 51QTD／51QUD | 0.121／0.103 |
| taiwan_file 0924 S2 | 51RTK | 0.499 |
| taiwan_file 1001 S2 | 50QPM／50RPN／50RQN | 0.254／0.313／0.538 |
| taiwan_file 1004 S2 | 50RPN／50RQP／50RRP | 0.674／0.618／0.604 |
| 馬祖 0622 S2 | 51RTK | 0.432 |

## 5. 觀察與結論

- **Finding**：誤差以漏判為主（recall 0.39、precision 0.81）。
- **判定**：support baseline adequacy；僅代表可當 LFMDA 後續對照基線，**不是跨域方法效益**。
- 限制：單 seed；GT-derived ROI 非全景盲測；raw255 計背景；未與其他模型同口徑配對比較。與 Lane B（新五景已標油 recall 0.2516）**不合併為 24 景 IoU**。
- **2026-11-03 規則變更（附加，上列原文保留）**：使用者裁決新增五張油標註以外視為背景，取代「不合併為 24 張 IoU」的 11-01 規則；往後新實驗主報臺灣 24 張 pooled OilIoU，19 張與新增五張並列。本頁判定（support baseline adequacy）不改；C 的 24 張數字見 [[20261103_P38_LFMDA_warmupPCD_結果]] 末節。

## 6. 後續改進方向（Decision / Next，待使用者決定）

- [ ] 以 P37 final 為對照，設計完整 LFMDA 下一步（DINOv2 教師／PCD／偽標籤）新卡；開跑前凍結 1–6 並與 Codex 協商
- [ ] 查 0801 L9 與 0529 L8 低分景的影像原因（不需讀 GT）

## 7. 實驗檔案位置

- 評估輸出：`/home/alanyh/oil_dataset/new/_lfmda_sourceonly_20261101/target_eval_final_best.json`
- 事前卡：`/home/alanyh/obsidian-vault/.agents/decisions/20261101_P37_LFMDA_SegFormerB2_sourceonly設計_v2.md`
- 程式：GT_expand commit 7030a77（本機未 push）
