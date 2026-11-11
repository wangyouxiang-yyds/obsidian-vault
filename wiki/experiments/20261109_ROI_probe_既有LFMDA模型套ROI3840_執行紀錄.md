---
date: 2026-11-09
type: experiment
stage: evaluation
status: 完成
tags: [lfmda, roi, roi3840, probe, descriptive, inference-only, p44, p41, p42, p43, taiwan24, segformer-b2, execution-record]
---

# ROI probe：既有 LFMDA 模型直接套 P44 ROI 3840 推論口徑——只推論不訓練，臺灣 24 張 pooled OilIoU 0.32–0.35（高於 P44 0.300，描述性）

> 本頁為執行紀錄。只做推論與計分，**沒有訓練、沒有新的事前凍結卡**（非預先登記）。被套用的模型來自 [[20261104_P41_LFMDA_PCD_固定P15teacher_執行紀錄]]（C6／T3）、[[20261104_P42_LFMDA_stage2_from_P41_T3_執行紀錄]]、[[20261105_P43_LFMDA流程補seed穩定性_執行紀錄]]（T3／P42 補 seed）；ROI 算子與對照數字見 [[20261106_P44_ROI重疊滑窗全球排除臺灣設計]]、[[20261107_P46大圖版配比與OOF探勘]]、[[20261108_P48補seed_H1與P44]]。

## 1. 實驗背景

使用者 2026-11-09 想把 LFMDA 搬到 P44 的 ROI 3840 流程，但擔心 ROI 內大量背景會讓機制大幅退化。先拿已訓練好的模型直接套 ROI 推論，看分數掉多少，再決定要不要重新設計訓練。使用者同意「先做個推論試試」；推論完成後以「跑評分阿」授權讀臺灣 GT。

同日裁決：之後 LFMDA-ROI 的 target 改用臺灣 24 張（新增 5 張不再只當測試）。該設計尚未建立，也未授權。

## 2. 方法架構 / Pipeline 流程

只推論，不訓練：

1. 取臺灣 ROI 算子（沿用 P44 凍結：27 個 ROI、21,966 窗、步距 128）。
2. 每窗推論做 TTA：原圖＋水平翻轉＋垂直翻轉，logits 平均，fp32。
3. ROI 聯集逐像素計分一次（評分同 `evaluate_p45.score`）。
4. 8 個模型各跑一次；另以同程式重算 P44_best 作核對（0.300，與既有紀錄一致）。

SegFormer 保留原訓練前處理（8-bit 量化、GoM mean/std）。與原推論路徑核對：輸入差 2e-7、機率差 2e-4。

## 3. 關鍵參數

| 項目 | 內容 |
|---|---|
| 模型（8 個） | P15_S42（P41–P43 的 teacher）、C6_S2333 final、T3_S2333／2334／2335 student_final（stage 1）、P42_S2333／2334／2335 student_final（stage 2） |
| ROI 算子 | P44 凍結臺灣 ROI：27 ROI、21,966 窗、步距 128 |
| TTA | 原圖＋水平＋垂直翻轉，logits 平均 |
| 精度 | fp32 |
| 計分 | ROI 聯集逐像素算一次，同 `evaluate_p45.score` |
| 前處理 | 保留 SegFormer 原訓練前處理（8-bit 量化、GoM mean/std） |
| 推論時間 | 2026-11-09 22:03–22:32 |
| seal SHA-256 | `8f5e5c3ddd3768affab00b1f2c340af2e543e82815e9020523b6cdf678ef9e3f` |
| 程式 commit | `predict_roi_lfmda.py` 0d800cf；`evaluate_roi_lfmda.py` 52aea7d |
| P15 checkpoint | 本機路徑已失效，改讀 NAS（SHA 相同） |

## 4. 量化結果

臺灣 24 ROI pooled OilIoU（單 seed 單次；P／R＝precision／recall；19 張／新增 5 張為子集 IoU）：

| 模型 | 24 張 IoU | P／R | pred/gt | 19 張／新增 5 張 | 備註 |
|---|---|---|---|---|---|
| P15_S42（teacher） | 0.178 | 0.199／0.628 | 3.16 | 0.185／0.103 | FP 3.69M |
| C6_S2333 | 0.251 | 0.534／0.321 | 0.60 | 0.252／0.240 | 3 張 IoU=0 |
| T3_S2333 | 0.340 | 0.519／0.496 | 0.96 | 0.345／0.267 | |
| T3_S2334 | 0.351 | 0.549／0.493 | 0.90 | 0.361／0.246 | |
| T3_S2335 | 0.333 | 0.445／0.568 | 1.28 | 0.370／0.120 | |
| **T3 三 seed 平均** | **0.341 ± 0.009** | | | | |
| P42_S2333 | 0.328 | 0.462／0.530 | 1.15 | 0.335／0.241 | stage 2 |
| P42_S2334 | 0.338 | 0.502／0.509 | 1.01 | 0.345／0.246 | stage 2 |
| P42_S2335 | 0.321 | 0.450／0.529 | 1.18 | 0.337／0.183 | stage 2 |
| **P42 三 seed 平均** | **0.329 ± 0.009** | | | | |
| P44_best（同程式重算，核對） | 0.300 | 0.426／0.503 | 1.18 | 0.304／0.252 | FP 0.99M |

對照（既有紀錄，非本次重算）：P12 三 seed 0.216、H1′ 三 seed 0.311 ± 0.014。

逐張：T3 三 seed 平均勝 P44 的只有 11／24 張。Landsat：T3 0.29–0.36、P44 0.286。

P42 的 P／R／pred/gt 與子集欄位取自 `eval_probe.json`；表中平均 ± SD 為 ddof=1。

## 5. 觀察與結論

（描述性）

- GT 置中訓練的 LFMDA student 搬到大量背景的 ROI 口徑**沒有崩**，24 張 IoU 0.32–0.35，還高於 P44（0.300）與 H1′（0.311）。
- P15 teacher 本身在 ROI 口徑遠處誤判嚴重（pred/gt 3.16、FP 3.69M）；student 的 precision（0.45–0.55）遠高於 teacher（0.199）。
- stage 2（SAM＋LRH）仍比 stage 1 低約 0.012（0.329 vs 0.341），方向與 P43 一致。
- C6（source-only）0.251，低於 T3／P42，且 3 張 IoU=0。
- 逐張只有 11／24 張 T3 勝 P44，平均差距主要不是全面提升。

**限制（不可越界解讀）**：

- T3／P42 的 PCD 看過臺灣 19 張 270 窗的影像（transductive）；P44／H1′ 完全沒看過臺灣；LFMDA source 只有 GoM。兩邊資訊量不對等。
- 非預先登記，臺灣是重用基準。
- 因此**不能寫成「LFMDA 勝過 ROI 取樣」**，只能寫成「既有 LFMDA 模型套 ROI 口徑沒有崩」。
- 單次推論，T3／P42 各只有 student seed 變異（S2333／2334／2335）。

## 6. 後續改進方向

- [x] 8 個既有模型套 ROI 推論＋計分（2026-11-09）
- [ ] LFMDA-ROI 設計：teacher 換 H1′、source 用 H1′ 清單、target 用臺灣 24 張 ROI 窗——**尚未授權、未建卡**

## 7. 實驗檔案位置

- 程式：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/roi_lfmda_probe_20261109/predict_roi_lfmda.py`（0d800cf）、`.../evaluate_roi_lfmda.py`（52aea7d）
- 輸出資料夾：`/home/alanyh/oil_dataset/new/_roi_lfmda_probe_20261109/`
- 結果：`/home/alanyh/oil_dataset/new/_roi_lfmda_probe_20261109/eval_probe.json`（SHA-256 開頭 `af44a79f439da06e`）
- 逐景：`/home/alanyh/oil_dataset/new/_roi_lfmda_probe_20261109/eval_probe_per_scene.csv`
- seal：`/home/alanyh/oil_dataset/new/_roi_lfmda_probe_20261109/probe_seal.json`（seal SHA-256 `8f5e5c3d…`）

## 相關頁面

- [[20261104_P41_LFMDA_PCD_固定P15teacher_執行紀錄]]
- [[20261104_P42_LFMDA_stage2_from_P41_T3_執行紀錄]]
- [[20261105_P43_LFMDA流程補seed穩定性_執行紀錄]]
- [[20261104_P40_LFMDA_6波段_執行紀錄]]
- [[20261106_P44_ROI重疊滑窗全球排除臺灣設計]]
- [[20261107_P46大圖版配比與OOF探勘]]
- [[20261108_P48補seed_H1與P44]]
- [[20261108_P47_SpectralGPT_ROI3840]]
