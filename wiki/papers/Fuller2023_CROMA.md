---
date: 2026-11-13
type: paper
title: "CROMA: Remote Sensing Representations with Contrastive Radar-Optical Masked Autoencoders"
authors: Anthony Fuller, Koreen Millard, James R. Green
year: 2023
tags: [foundation-model, self-supervised, sentinel-2, sentinel-1, contrastive, mae, vit, p54]
validity: current
---

# CROMA（Fuller, Millard, Green, NeurIPS 2023）

> 來源：arXiv 2311.00566；NeurIPS 2023。原檔 `raw/papers/CROMA.pdf`。
> 收錄狀態：只依使用者交辦的事實摘錄，未全文逐節精讀；表內數字出自論文 §3.1 Table 3 與 Table 6 的轉述。

## 基本資訊與研究問題

遙測自監督預訓練：如何同時利用配對的 Sentinel-1（雷達）與 Sentinel-2（光學），學到可單獨下游使用的表徵。

## 方法

- 預訓練＝雷達-光學跨模態對比學習＋MAE 重建（S1＋S2 配對，遮罩比例 75%）。
- 光學 encoder 可單獨用於下游。Table 6 的「optical-only」僅指 MAE 重建目標只有光學，**預訓練仍含雷達**。
- ViT-L 光學 encoder：24 層、patch 8、2D-ALiBi 位置偏置、303M 參數；輸入 12 個 S2 波段（B01–B12，含 B8A、B09）。

## 主要結果（§3.1 Table 3，光學單模態、凍結 encoder＋linear probe 分割 mIoU）

| 資料集 | CROMA-B | CROMA-L |
|---|---|---|
| DFC2020 | 46.67 | 49.78 |
| DW-Expert | 58.55 | 58.71 |
| MARIDA | 65.56 | 63.32 |

評估使用 96×96 裁切，尺度遠小於本專案的滑窗／ROI 輸入；數字不可直接比較。

## 與本專案對照

- PANGAEA 以 CROMA-L 光學 encoder 凍結＋UPerNet 評估 MADOS：15 類 mIoU 67.55，**沒有 Oil 單類 IoU**；MADOS 缺 B09，於正規化後補 0。因此現有文獻證據不能直接說明 CROMA 對油汙 IoU 的效果。
- P54 以 CROMA optical-large（官方 CROMA_large.pt）取代 P53 全球流程的 encoder，並採 PANGAEA 式輸入處理（TRAIN 8 槽 mean/std z-score、B05/B06/B07/B09 正規化後補 0）。實驗卡在 GT_expand `analysis/p54_croma_global_20261113/P54_CARD.md`（DRAFT r2，未 launch；操作狀態見 `.agents/handoff.md`）。
- 概念背景：[[self_supervised_pretraining_遙測]]；同類 foundation model 對照：[[Wang2023_SSL4EO-S12]]、[[Szwarcman2026_PrithviEO2]]。
