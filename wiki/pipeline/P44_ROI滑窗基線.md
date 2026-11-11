---
date: 2026-11-11
type: pipeline
tags: [p44, roi, sliding-window, baseline]
validity: current
---

# P44 自然比例 ROI 滑窗基線

## 定位與適用範圍

2026-11-10 使用者指定 P44 的**自然比例、不挑背景**流程為後續基線。這是 GT 衍生錨點的 ROI 研究流程，臺灣測試先有 ROI 位置，不能宣稱全景盲掃。設定正本：[P44 凍結卡](../../.agents/decisions/20261106_P44_ROI重疊滑窗全球排除臺灣設計.md)及主線 repo `analysis/p44_roi_sliding_global_20261106/P44_CARD_DRAFT.md`；結果見 [P44](../experiments/20261106_P44_ROI自然比例基線_結果.md)、[P48 補 seed](../experiments/20261109_P48_H1prime與P44補seed_結果.md)。

## 可重用流程

1. 從已確認的 mask 油連通區建立外框，依凍結規則分組、對齊 256 起點，產生每個 3840×3840 ROI；靠影像邊界時按規則推回。ROI 錨點由 GT 衍生。程式與名單以 P44 卡及 repo `analysis/p44_roi_sliding_global_20261106/` 為準。
2. 全球資料排除臺灣，按 P44 凍結的 TRAIN／VAL 清單取窗。TRAIN 在 ROI 內用 256×256 窗、步距 128，**保留自然背景比例**；沒有 HNM、SI 篩選或每 batch 固定含油窗。背景挑選實驗另見 [P45](../experiments/20261107_P45_固定含油窗數_結果.md)、[P46](../experiments/20261108_P46_HNM_SI大圖版_結果.md)。
3. 六波段 B02/B03/B04/B8A/B11/B12 輸入 [Prithvi＋DeepLabV3](../models/Prithvi_DeepLabV3.md)，Tversky α=0.3、β=0.7；特定版本的 seed、選模及 checkpoint 以卡和封存輸出為準。
4. 臺灣在 GT 錨定 ROI 內用步距 128、翻轉 TTA 重組，再依各 run 的有效測試像素加總計 OilIoU；跨 seed 才取平均。以大林 2 張與其他 22 張各自報告。計分範圍與重複覆蓋處理見 [評估口徑](../concepts/評估口徑.md)，不得與舊 GT-centric 256 窗分數直接相減。

## 已知限制

P44 S42/S43/S44 的臺灣 24 張 ROI pooled OilIoU 為 0.300／0.351／0.045；S44 塌縮，三 seed 平均 0.232±0.164。這是 P44/P48 版本、GT 錨定 ROI、run 內像素加總後跨 seed 平均；使用作對照時須列全部 seed。不同資料制度（如 P51 GoM-only）會同時改 TRAIN、VAL、曝光量和正負比例，不能把差異單獨歸因來源海域。來源：[P48](../experiments/20261109_P48_H1prime與P44補seed_結果.md)、[P51 設計](../../.agents/decisions/20261111_P51_GoM_only_ROI資料制度_設計.md)。P51 結果截至本頁寫入未核實。

## 變更紀錄

- 2026-11-11｜建立自然比例 ROI 流程及與 GT-centric 不可比的邊界｜依據 [[20261106_P44_ROI自然比例基線_結果]]、[[20261109_P48_H1prime與P44補seed_結果]]
