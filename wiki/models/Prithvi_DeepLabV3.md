---
date: 2026-11-11
type: model
tags: [prithvi, deeplabv3, roi, baseline]
validity: current
---

# Prithvi＋DeepLabV3

## 現行定位

Prithvi-EO-2.0-300M 第 23 層特徵接 DeepLabV3 ASPP(6,12,18) 二類分割頭，是 P44 **自然比例 ROI 滑窗**基線的模型。基線是模型、資料清單、取樣與評估算子的組合，不能把早期同架構的 GT-centric 成績視為同一基線。依據：[P44 結果](../experiments/20261106_P44_ROI自然比例基線_結果.md)、[P48 補 seed](../experiments/20261109_P48_H1prime與P44補seed_結果.md)、[P44 凍結設計](../../.agents/decisions/20261106_P44_ROI重疊滑窗全球排除臺灣設計.md)。

## 輸入與訓練

- VRT 保留八槽 B01/B02/B03/B04/B08/B8A/B11/B12；模型取 B02/B03/B04/B8A/B11/B12 六波段，除以 10000 後以 Prithvi 官方 mean/std 標準化。原始 DN 有 +1000 offset，計算光譜指數前須處理。來源：主線 repo `analysis/p44_roi_sliding_global_20261106/P44_CARD_DRAFT.md`、vault [資料頁](../datasets/資料集與物理標記.md)。
- P44 以每景 3840×3840 GT 衍生 ROI、256 窗、步距 128 訓練，背景不篩選；Tversky α=0.3、β=0.7。臺灣測試在 ROI 內步距 128，加翻轉 TTA。此為已知位置輔助測試，並非全景盲掃。來源：[P44 凍結設計](../../.agents/decisions/20261106_P44_ROI重疊滑窗全球排除臺灣設計.md)。

## 結果與限制

P44 S42 與 P48 補的 S43/S44 在臺灣 24 張 ROI 測試的 run 內像素加總 OilIoU 分別為 0.300／0.351／0.045，三 seed 平均 0.232±0.164；S44 塌縮，不能在比較時省略。這是 P44/P48 封存版本、臺灣 GT 衍生 ROI、24 張同一測試名冊的結果；跨 seed 先各自算 pooled OilIoU 再平均。依 2026-11-11 呈現規則，同一批測試後續分報大林 2 張及其他 22 張；舊卡的原判定不追改。來源：[P44 結果](../experiments/20261106_P44_ROI自然比例基線_結果.md)、[P48 補 seed](../experiments/20261109_P48_H1prime與P44補seed_結果.md)、institution `research/oil_spill_project_status.md`「臺灣結果呈現改為大林 vs 其他 22 張」。

GT-centric P12/P14/P15 屬同一模型家族、不同取窗及測試口徑；歷史結果見 [GT_expand 管線](../pipeline/GT_expand_pipeline.md)。P44 的單 seed best 與 last 差距大，且臺灣資料曾重複用於研究判讀；泛化到未見地點的程度未核實。

## 變更紀錄

- 2026-11-11｜由日期快照整理現行模型、基線與三 seed 限制｜依據 [[20261106_P44_ROI自然比例基線_結果]]、[[20261109_P48_H1prime與P44補seed_結果]]
- 2026-11-11｜導覽句由已作廢快照改指現行知識頁｜依據 log.md 2026-11-11 harness WP3 收尾
