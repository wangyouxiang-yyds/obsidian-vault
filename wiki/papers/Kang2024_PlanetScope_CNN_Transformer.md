---
type: paper
status: preliminary
title: "Detection of Marine Oil Spill from PlanetScope Images Using CNN and Transformer Models"
authors: Jonggu Kang, Chansu Yang, Jonghyuk Yi, Yangwon Lee
year: 2024
tags: [optical, PlanetScope, NIR, DeepLabV3Plus, UPerNet, Mask2Former, transformer, augmentation, oil-spill, reading-queue]
---

# Detection of Marine Oil Spill from PlanetScope Images Using CNN and Transformer Models（Kang et al., 2024）

> 來源：https://doi.org/10.3390/jmse12112095
> 發表：Journal of Marine Science and Engineering 12(11), 2095
> 收錄狀態：第一輪紀錄；出版社全文可讀。

---

## 核心貢獻

以 16 張 PlanetScope 四波段影像、八次油污事件，比較 CNN 與 Transformer 分割模型。Swin-UPerNet 的平均 mIoU 與 Oil IoU 均高於 DeepLabV3+ 與 Mask2Former；但 5-fold 是對 patch 隨機切分，沒有按事件或區域隔離，因此不能視為跨海域 zero-shot 證據。

## 關鍵方法

- PlanetScope 影像切成 260 個 `256×256` patches，再增強十倍至 2,600。
- 模型：DeepLabV3+/ResNet101、Swin-B+UPerNet、Swin-B+Mask2Former。
- 損失：binary cross-entropy；optimizer 使用 Adam／AdamW。
- 增強：90° rotation、水平／垂直翻轉、optical/grid distortion、RGB shift、brightness/contrast。

## 資料、波段與切分

| 項目 | 內容 |
|---|---|
| 感測器 | PlanetScope，約 3–4.1 m |
| 波段 | Blue 455–515、Green 500–590、Red 590–670、NIR 780–860 nm |
| IR 類型 | **NIR；沒有 SWIR 或 TIR** |
| 地區 | 八次事件；含 Persian Gulf、Texas、California、Venezuela、Syria、Louisiana |
| 時間 | 2017–2021 |
| 切分 | 260 patches 隨機分成 5 folds；每輪 train/val/test = 3/1/1 folds |
| Zero-shot 判定 | **否**；未按 incident、scene 或 region 分組 |

## 重要數據／結果

| 模型 | mIoU | Oil IoU | F1 |
|---|---:|---:|---:|
| DeepLabV3+ | 0.740 | 0.608 | 0.843 |
| Swin-UPerNet | **0.840** | **0.758** | **0.911** |
| Mask2Former | 0.804 | 0.704 | 0.887 |

## 與本專案的關聯

它支持「UPerNet 值得納入架構 baseline」以及 Transformer 對多尺度油膜分割可能有優勢，也提供了多地區 NIR 光學資料與較豐富 augmentation 的做法。但 PlanetScope 沒有 SWIR，且 random patch CV 可能把同一事件的相似 patch 分散至 train/test；因此模型排序不能直接外推到台灣 target-blind 測試。

## 研究缺口與假設

- 原始 patch 只有 260 個，且增強不會創造新的事件或海域多樣性。
- 隨機 patch CV 可能產生空間／事件洩漏。
- 雲、油色與厚度多樣性仍有限。
- 隱含假設是 patch-level IID；這正是本專案要拒絕的假設。

## 九步分析摘要

1. **領域地景**：高解析度四波段光學油污 segmentation 架構比較。
2. **矛盾偵測**：UPerNet 的高分可能同時反映架構與較寬鬆切分。
3. **引用鏈**：DeepLabV3+、Swin Transformer、UPerNet、Mask2Former。
4. **研究缺口**：缺 incident-held-out、region-held-out 與 SWIR。
5. **方法審核**：模型比較完整，但獨立樣本單位不是事件。
6. **假設殺手**：若改成 leave-one-incident-out，模型排序與分數可能改變。
7. **知識地圖**：連到 UPerNet、augmentation 與事件層切分。
8. **文獻綜合**：它是架構與 augmentation 參考，不是跨海域 transfer 先例。
9. **So What**：可把 UPerNet 列 baseline，但主實驗必須使用凍結事件／台灣 target。

## 相關頁面

- [[跨海域_source-only_zero-shot油污偵測]]
- [[dataset_split_strategy]]
- [[DeepLabV3+]]

