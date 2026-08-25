---
type: paper
status: preliminary
title: "MS3OSD: A Novel Deep Learning Approach for Oil Spills Detection Using Optical Satellite Multisensor Spatial-Spectral Fusion Images"
authors: Kai Du, Yi Ma, Zhongwei Li, Rongjie Liu, Zongchen Jiang, Junfang Yang
year: 2025
tags: [optical, multisensor-fusion, HY-1C, HY-1D, UV, NIR, transformer, sun-glint, oil-spill, reading-queue]
---

# MS3OSD（Du et al., 2025）

> 來源：https://doi.org/10.1109/JSTARS.2025.3550421
> 發表：IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing 18, 8617–8629
> 收錄狀態：第一輪紀錄；主方法與結果已核對，exact scene split 仍待補。

---

## 核心貢獻

把 HY-1C/D 的 CZI、UVI、COCTS 融合成 50 m、10-band UV–VIS–NIR 影像，再用 CNN spatial branch 與 Transformer spectral branch 分類油污。論文涵蓋四個海域與強／弱 sunglint，但沒有可核實的 leave-one-sea-out 或 source-only target-blind protocol。

## 關鍵方法

- MS3OSD-F：convolutional autoencoder，以低解析度多感測器資料重建 50 m 10-band 影像。
- MS3OSD-C：中心像素光譜 CNN branch，加周圍空間 ViT branch。
- 最佳 patch size 為 17。
- 增強：rotation、flipping、random noise。

## 資料、波段與切分

| 項目 | 內容 |
|---|---|
| 感測器 | HY-1C/D CZI + UVI + COCTS |
| 融合產品 | 50 m、10-band UV–VIS–NIR |
| 代表波長 | 355、385、443、490、520、670、750、865 nm 等 |
| IR 類型 | **NIR；沒有 SWIR 或 TIR** |
| 海域 | 黃海、南海、安達曼海、Karimata Strait |
| Domain | 弱 sunglint：海水／油膜／乳化油；強 sunglint：海水／正對比油污 |
| 切分 | 強／弱 sunglint 分開訓測；是否同景抽 patch、是否事件隔離待確認 |
| Zero-shot 判定 | **未證明**；多海域不等於 leave-one-region-out |

## 重要數據／結果

- 主分類 F1：弱 sunglint 油膜 95.24%、乳化油 93.04%；強 sunglint 正對比油污 90.06%。
- 三感測器融合消融 F1 約為 95.73%、91.41%、90.46%，高於只用 CZI。
- Fusion：MAPE 2.31%、SAM 0.23°、runtime 1.85 s。

## 與本專案的關聯

可借用的是把 sunglint 當作 domain factor、融合中心光譜與周圍空間資訊，以及多感測器共同表徵的概念。限制是 HY-1 的 UV/VNIR 組成與 Sen2Like 的 VIS/NIR/SWIR 不同；若未進行 leave-one-sea-out，無法當成海外→台灣 zero-shot 前例。

## 研究缺口與假設

- 僅被動光學；作者把 TIR、LiDAR、SAR 與 cross-satellite HY-3A 列為未來工作。
- exact train/test scene IDs 與事件分組不夠透明。
- 多感測器 fusion 依賴同平台／同時相資料，與 S2/Landsat harmonization 問題不同。
- 隱含假設是四海域的 patch 分布足以代表部署域，尚未用 sealed region 驗證。

## 九步分析摘要

1. **領域地景**：光學多感測器超解析融合與 spatial-spectral classification。
2. **矛盾偵測**：多海域實驗不等於跨海域 generalization。
3. **引用鏈**：上游為多感測器融合、CNN spectral branch 與 ViT spatial modeling。
4. **研究缺口**：缺 region-held-out、跨衛星及 SWIR/TIR 驗證。
5. **方法審核**：融合與分類消融完整，但 split independence 是主要未知。
6. **假設殺手**：若 patch 來自同一影像再隨機切分，高 F1 不能外推陌生海域。
7. **知識地圖**：連到 sunglint domain shift 與 multisensor harmonization。
8. **文獻綜合**：證明 UV–VNIR fusion 有效，但不回答台灣 target-blind transfer。
9. **So What**：可借 domain-factor 分層，不宜直接照搬感測器融合架構。

## 相關頁面

- [[跨海域_source-only_zero-shot油污偵測]]
- [[acolite_vs_sen2like]]

