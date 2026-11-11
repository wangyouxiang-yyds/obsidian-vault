---
date: 2026-11-11
type: model
tags: [unsupervised, isolation-forest, deep-rx, vae]
validity: current
---

# 無監督候選：iForest 與 DeepRX

這兩條是 2026-06 **人工標記候選 patch 提示**，不是現行監督式像素分割；候選層 micro 指標不可與 P44 OilIoU 排名。

| 路線 | 當時方法與候選層結果 |
|---|---|
| iForest＋Local Contrast | B04/B08/B11 鄰域差分先找候選，再 PCA＋Isolation Forest 排序。當時全場景 micro recall 0.0225、precision 0.0043、F1 0.0072；漏掉多數油 patch，難以充當候選提示器。[實驗總結](../experiments/20260613_iForest_LocalContrast_B4B8B11_全場景實驗總結.md) |
| DeepRX＋EM v3 | 每景 VAE latent 用 Mahalanobis/RX，加入 Coast Buffer、Local Contrast 與 EM 門檻。當時全場景 micro recall 0.3712、precision 0.0041、F1 0.0081；約 1698 TP 對 414162 FP，當時不適合自動偵測。[實驗總結](../experiments/20260614_DeepRX_EM_v3_全場景實驗總結.md) |

數字只適用於兩張 2026-06 總結卡定義的資料、候選 patch 與 v3/當時版本；並非 P44 的 3840 ROI 或 run 內 pooled 油像素指標。概念和失效條件另見 [iForest 架構](../concepts/iforest_架構與運作機制.md)、[DeepRX/VAE 架構](../concepts/deeprx_vae_架構與運作機制.md)。

## 變更紀錄

- 2026-11-11｜將早期候選提示模型與現行分割口徑分開｜依據 [[20260613_iForest_LocalContrast_B4B8B11_全場景實驗總結]]、[[20260614_DeepRX_EM_v3_全場景實驗總結]]
