---
experiment: P47
title: SpectralGPT+（ViT-B）＋DeepLabV3 接 P44 ROI 3840 流程（只換模型，單 seed）→ 臺灣 24 景
date: 2026-11-08
status: RESULT_INDISTINGUISHABLE
type: 單 seed 探索（預先登記判讀：|Δ|<0.09 與 P44 無法區分）
repo_card: /mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p47_spectralgpt_roi_20261108/P47_CARD.md
gt_expand_commit: 6461c34（launch commit 3942b79）
consensus: 無（未經 Codex）
---

# P47：SpectralGPT+＋DeepLabV3（P44 ROI 流程只換模型）

> 對照基準見 [[20261106_P44_ROI重疊滑窗全球排除臺灣設計]]；同期補 seed 見 [[20261108_P48補seed_H1與P44]]。

## 1. 背景與時程

- 問題：同一套 P44 資料／取樣／訓練／評估下，把 Prithvi-EO-2.0 300M 換成 SpectralGPT+（ViT-B，patch 8×8），臺灣 24 張 ROI 口徑 oil IoU 是否比 P44 好。
- 2026-11-08 準備（卡 P47_CARD.md，未經 Codex）；2026-11-09 09:22 launch；20:35 預測完成；21:15 讀臺灣 GT 計分。
- P48 補 seed 持有 GPU 鎖，P47 排隊至其後。

## 2. 方法（唯一改動是模型）

- 模型：SpectralGPT+（官方 `SpectralGPT+.pth`）＋DeepLabV3。
- 輸入：8 波段放入 12 波段槽，B05／B06／B07／B09 填 0.5；每視窗逐波段 min–max。
- 其餘沿用 P44：ROI 3,840、ROI 內 256 視窗 stride 128、自然背景、Tversky（0.3／0.7）、seed 42；以 VAL ROI pooled oil IoU 選 best。

## 3. 訓練

- 速度 2.0 it/s（P44 Prithvi 10.6 it/s，慢約 5 倍，因 4,096 token）。
- VE15 早停；best VAL oil IoU 0.182＠VE7（P44 0.176＠VE17）。
- wandb run：P47_S42（`k6tjn6kg`）。

## 4. 結果（臺灣 24 張 ROI pooled oil IoU）

**判讀（卡預先登記）：單 seed |Δ|<0.09 → 與 P44 無法區分（indistinguishable）。**

| 組別 | 臺灣 24 張 | 臺灣 19 張 | 新增 5 張 |
|---|---|---|---|
| P47_best | 0.315（P 0.474／R 0.485） | 0.326 | 0.121 |
| P44_best S42 | 0.300 | 0.304 | 0.252 |
| P47_last | 0.145 | 0.146 | 0.115 |

- Δ（P47_best − P44_best S42）= +0.015。
- P44 三 seed 臺灣 24 張：0.300／0.351／0.045（見 [[20261108_P48補seed_H1與P44]]）。
- 感測器分組：S2 0.354 vs P44 0.308；Landsat 0.248 vs P44 0.286。

## 5. 逐景觀察

- 只有 4／24 張勝過 P44_best。
- IoU=0 的景：P47_best 9 張（逐景 CSV 計數；P44_best 僅 1 張，`taiwan_file_20260924_S2_51RTK`）。
- pooled 主要被中油大林 2 號兩景（GT 約占 58%）與馬祖撐高；中小景多半漏判。

## 6. 限制

- 單 seed；P44 本身單 seed 也曾差到 0.09 以上（三 seed 0.300／0.351／0.045）。
- 8→12 波段有 4 槽為常數 0.5 佔位；每視窗 min–max 丟失絕對反射率；token 格 32×32（Prithvi 16×16）、模型 92M vs 300M，非純 encoder 消融。
- 臺灣 24 張重用、非盲。
- 後續（補 seed 與否）待使用者決定。

## 7. 檔案位置

- 程式與卡：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p47_spectralgpt_roi_20261108/`（commit 6461c34、launch commit 3942b79）
- 產物：`/home/alanyh/oil_dataset/new/_p47_spectralgpt_roi_20261108/`（`eval_P47.json`、`eval_P47_per_scene.csv`、`train.log`、`predictions/`）
- wandb：P47_S42（`k6tjn6kg`）
