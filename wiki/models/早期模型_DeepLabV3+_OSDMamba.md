---
date: 2026-11-11
type: model
tags: [deeplabv3-plus, osdmamba, historical-model, sliding-window]
validity: current
---

# 早期 DeepLabV3+ 與 OSDMamba 工程路線

本頁整理 0420／0422 時期的模型接線與環境經驗，**不作現行六波段 P44 配方或成績依據**。細節來源：[OSDMamba 移植事件](../experiments/20260429_OSDMamba整合至0420.md)、[訓練效能優化](../experiments/20260430_訓練效能優化.md)、[0422 流程](../pipeline/OIL_PROJECT_VRT_0422.md)；舊工程頁保留作歷史快照。

| 路線 | 當時設定 | 現行解讀 |
|---|---|---|
| ResNet50＋DeepLabV3+ | 八波段 B01/B02/B03/B04/B08/B8A/B11/B12；FocalLoss、256 視窗、stride 192、TTA。 | 與 [Prithvi＋DeepLabV3](Prithvi_DeepLabV3.md) 的 backbone、六波段輸入和 ROI 取樣都不同。舊頁的 `main/deeplab_adapter.py` 配置是否仍對應現行程式未核實。 |
| SwinUMambaD／OSDMamba | 早期 11 波段、Dice＋Focal、EMA、sliding-window；當時 RTX 5090 上的 mamba-ssm 編譯與記憶體調校。 | 這是舊管線工程，不表示目前 P44 已支援 11 波段模型；配置是否仍可重跑未核實。論文架構見 [OSDMamba 摘要](../papers/OSDMamba_摘要.md)。 |

早期記錄的 EMA 優劣、類別權重與效能觀察都受舊資料版本、舊 checkpoint 和硬體設定限制，不能移用到 P44。需要重建時，先核原 repo commit、VRT 波段、mask pixel mapping 與當時結果檔；本頁不提供未核實的現行命令。

## 變更紀錄

- 2026-11-11｜把早期兩模型的可重用工程脈絡與現行基線切開｜依據 [[20260429_OSDMamba整合至0420]]、[[20260430_訓練效能優化]]
