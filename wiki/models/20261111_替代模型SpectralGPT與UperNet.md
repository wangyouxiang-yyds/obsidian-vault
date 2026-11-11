---
date: 2026-11-11
type: model
tags: [spectralgpt, upernet, prithvi, roi, model-history]
---

# ROI 階段替代 encoder／head：SpectralGPT+ 與 UperNet

## SpectralGPT+＋DeepLabV3（P47）

在 P44 ROI 工作流試 SpectralGPT+ encoder 接 DeepLabV3。除模型外，波段佔位、正規化與 token 格也不同，**不是純 encoder 消融**。單 seed 臺灣24 ROI pooled OilIoU best 0.315、P44 S42 0.300；依事前規則判 indistinguishable。P47 只有4/24景勝 P44、9景 IoU=0，推論／訓練速度慢約五倍；沒有成為基線。[P47 結果](../experiments/20261109_P47_SpectralGPT_ROI_結果.md)。程式在主線 repo `analysis/spectralgpt_dlv3_draft_20261106/` 與 P47 卡所列路徑。

## Prithvi＋UperNet（P50）

固定 H1′ ROI 資料與訓練管線，把 DeepLabV3 第23層 ASPP 頭換為 UperNet 第5/11/17/23層特徵融合。三 seed 臺灣24 ROI pooled OilIoU UperNet 0.299±0.016、同 seed DLV3 0.311±0.014；大林和 Landsat 的配對改善條件未成立，按 P50 記錄**不建議換作 P49 teacher**。UperNet 漏判較多；此為描述性結果，不是所有 UperNet 配方的否定。[P50 執行與結果](../experiments/20261110_P50_H1prime_UperNet_三seed_執行紀錄.md)。實作在主線 repo `main/prithvi_upernet.py`。

兩條都須與自然比例 [P44 baseline](../experiments/20261106_P44_ROI自然比例基線_結果.md)分開標註資料配方與 seed。
