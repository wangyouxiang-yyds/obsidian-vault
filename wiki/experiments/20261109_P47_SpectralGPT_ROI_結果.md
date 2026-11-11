---
date: 2026-11-09
type: experiment
tags: [p47, spectralgpt, roi]
stage: evaluation
status: completed
---

# P47 SpectralGPT+ 接 P44 ROI：結果

設計與原始結果：[P47 卡](../../.agents/decisions/20261108_P47_SpectralGPT_ROI3840.md)。此組在 P44 ROI 流程只換模型，但波段佔位、正規化與 token 格也與 Prithvi 不同，不能當純 encoder 消融。

## 7. Evidence / Result

單 seed 臺灣 24 張 ROI pooled OilIoU：P47 best 0.315、P44 S42 best 0.300，差 +0.015；P47 last 0.145。只有 4／24 景勝過 P44；P47 有 9 景 IoU＝0。訓練約 2.0 it/s，P44 約 10.6 it/s。完整逐景觀察及產物位置見原卡 §4–7。

## 8. Answer / Finding

按原卡事前規則，單 seed |Δ|＜0.09，判 `indistinguishable`。pooled 的小幅優勢主要由少數大油區撐起；中小景多漏判，且速度慢約五倍。

## 9. Decision / Next

此結果不支持直接替換 P44 baseline。若再比較需明確控制輸入與評估差異，並補 seed；目前沒有後續結果。
