---
date: 2026-11-11
type: model
tags: [lfmda, segformer, pcd, sam, domain-adaptation]
validity: current
---

# LFMDA 模型路線

## 架構與資料角色

這條線以六波段 SegFormer-B2 student 作 source-only 分割，再測固定 teacher／PCD 偽標籤及 SAM 區域的 LRH 自訓練。P37–P43 的 GT-centric 與 P49 的 ROI 資料、teacher、target 入口不同，沒有單一代表整條線的 checkpoint。來源：[P37 source-only](../experiments/20261101_P37_LFMDA_SegFormerB2_sourceonly_結果.md)、[P49 ROI](../experiments/20261110_P49_LFMDA搬到ROI_三seed_執行紀錄.md)。

## 已有結果

| 範圍與版本 | 結果及判讀 |
|---|---|
| P37 GoM source-only、臺灣 19 張 GT-aware，單 seed | final pooled OilIoU 0.3563；可作該流程後續基準，仍是單次描述。[P37](../experiments/20261101_P37_LFMDA_SegFormerB2_sourceonly_結果.md) |
| P38/P39 DINOv2 teacher、warmup/PCD、SAM/LRH | 兩階段完成；正式卡判 `not improved`，只適用於該版本與口徑。[P38](../experiments/20261103_P38_LFMDA_warmupPCD_結果.md)、[P39](../experiments/20261103_P39_LFMDA_stage2_SAM_LRH_結果.md) |
| P40–P43 六波段、固定 P15 teacher、臺灣 GT-aware | P41 T3 單 seed 24 張 OilIoU 0.4906；同 seed P42 0.4786，P43 補 seed 的 Stage 2 也未建立改善。此處是歷史 GT-aware 口徑。[P41](../experiments/20261104_P41_LFMDA_PCD_固定P15teacher_執行紀錄.md)、[P42](../experiments/20261104_P42_LFMDA_stage2_from_P41_T3_執行紀錄.md)、[P43](../experiments/20261105_P43_LFMDA流程補seed穩定性_執行紀錄.md) |
| P49 P44 ROI 資料、H1′ teacher、三 seed | 臺灣 24 張 GT 衍生 ROI 的 run 內 pooled OilIoU Stage 2 final 為 0.307／0.422／0.336；大林兩張主導全體值，其他 22 張與 Landsat 表現不足，且 Stage 1 有 seed 塌縮。這是 transductive、描述性結果。[P49](../experiments/20261110_P49_LFMDA搬到ROI_三seed_執行紀錄.md) |

GT-aware 與 ROI 的區域、重建與取樣算子不同，分數不能互排；單 seed 差異不能歸因於架構。已測設定下的 SAM/LRH 多有擦除油預測的風險，並不證明其他 SAM 或域適應方法無效。P49 沒有改變 [P44 自然比例基線](Prithvi_DeepLabV3.md) 的地位。

## 變更紀錄

- 2026-11-11｜把 P37–P49 的不同資料制度收回同一模型主題｜依據 [[20261101_P37_LFMDA_SegFormerB2_sourceonly_結果]]、[[20261110_P49_LFMDA搬到ROI_三seed_執行紀錄]]
