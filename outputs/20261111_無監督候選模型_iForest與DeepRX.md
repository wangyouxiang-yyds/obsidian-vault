---
date: 2026-11-11
type: model
tags: [unsupervised, isolation-forest, deep-rx, vae, model-history]
validity: superseded
superseded_by: "[[無監督_iForest_DeepRX]]"
---

> ⚠ 已作廢（2026-11-11，使用者 WP3 裁決）：日期快照已併入常青模型頁；改看 [[無監督_iForest_DeepRX]]

# 早期無監督候選模型：iForest 與 DeepRX

這兩條是**人工標記候選提示**路線，評估的是候選 patch，與目前監督式像素分割 OilIoU 不同，不能同表排名。

| 路線 | 核心做法 | 當時全場景結果 |
|---|---|---|
| iForest＋Local Contrast | B04/B08/B11 鄰域差分先挑候選，再 PCA＋Isolation Forest 排序。 | micro recall 0.0225、precision 0.0043、F1 0.0072；漏掉大部分油 patch，當時判難以作候選提示器。[實驗頁](../wiki/experiments/20260613_iForest_LocalContrast_B4B8B11_全場景實驗總結.md)。 |
| DeepRX＋EM v3 | 每景 VAE latent 上算 Mahalanobis/RX，配 Coast Buffer、Local Contrast 和 EM 門檻。 | micro recall 0.3712、precision 0.0041、F1 0.0081；召回改善但約 1,698 TP／414,162 FP，遠不足以作自動偵測。[實驗頁](../wiki/experiments/20260614_DeepRX_EM_v3_全場景實驗總結.md)。 |

方法細節與失效條件見 [iForest 概念](../wiki/concepts/iforest_架構與運作機制.md)及 [DeepRX/VAE 概念](../wiki/concepts/deeprx_vae_架構與運作機制.md)。這些結果屬 2026-06 的資料與候選定義，不是現行 ROI 3840 的對照組。
