---
date: 2026-11-11
type: summary
tags: [gt-centric, stage-summary, oil-segmentation, roi-transition]
---

# GT-centric 階段結果快照（轉向 ROI 3840 前）

本頁封存 GT 置中取窗階段的主要結果與限制，作為轉向 3,840×3,840 ROI 自然比例流程時的歷史對照。數字是各實驗當時的口徑；**不同測試名冊、GT 覆蓋與聚合方式不可直接排成模型排行榜**。最新研究狀態以 `/home/alanyh/.agents/institution/research/oil_spill_project_status.md` 為準。

## 資料與評估口徑

- GT-centric 由標註油區定位 256×256 窗；小連通區取中心，大區用 256 格鋪磚；訓練另混入純背景窗。既有 `3_fold_stratified_v2` 與後續 P12/P14/P15 的 source split 並非同一個科學估計量。流程見 [GT_expand pipeline](../wiki/pipeline/GT_expand_pipeline.md)。
- 舊臺灣 9 景、後續 19/24 張名冊及「同一 3,840 ROI 重新推論」是三種不同範圍。舊 GT-aware 窗口的 OilIoU 不能與 P44 ROI 大量背景範圍的 OilIoU 直接相減來宣稱模型改善。
- 原始 GT 0=oil；背景與未標註區的評估慣例依各凍結實驗卡。新頁應同時標明 `_kind`、名冊、覆蓋、checkpoint、seed 與聚合方式。

## 代表性結果

| 系列 | 訓練／模型 | 當時測試口徑 | 結果及判讀 |
|---|---|---|---|
| P12 | 全球排除臺灣；Prithvi-EO-2.0-300M＋DeepLabV3，Tversky 0.3/0.7 | 臺灣舊 9 景，逐景 OilIoU 等權平均，三有效 seed | 0.3552／0.4052／0.3660，平均 0.3755±0.0263；原單次 0.4052 是三次最高值。[結果來源](../wiki/experiments/20261106_P44_ROI自然比例基線_結果.md)；完整三 seed 帳在 repo `analysis/p12_taiwan_deploy/SEED_EXTENSION.md`。 |
| P14 | GoM-only 與 NonGoM-only full source；Prithvi＋DeepLabV3 | 臺灣舊 9 景，單 seed；主端點事前指定 event-macro | 描述性 pooled OilIoU 0.4187／0.2037；scene-macro 0.3657／0.2942。**正式 event-macro 主端點仍未計，不能據此填入卡 7–9 或主張來源因果**。[P14 卡](../wiki/experiments/20260915_P14_GoM_vs_NonGoM_full_ablation_主端點未計.md)。 |
| P15／P24 | GoM-only 原配方／GoM 增曝光；全球＋GoM 增曝光 | 臺灣舊 9 景，逐景等權平均，三 seed | P15 Original 0.393±0.026、P15 Rep4496 0.354±0.028、P24 0.401±0.018；與 P12 0.376±0.026 差距約一個 seed SD。P24 切分與 patience 不同，**不能據並列值判定增曝光有效**。來源：研究現況檔 2026-10-11 P12 seed 擴充。 |
| DADS-SAM | 六波段公開碼適配＋Tversky；全球排除臺灣 | 臺灣 19 張 GT-aware，單 run、像素加總 OilIoU | 0.5815；與 P12 舊 9 景三 seed 或後續 ROI 值不同口徑，不能當受控架構勝負。[結案頁](../wiki/experiments/20261026_DADS_SAM_全資料排除臺灣_臺灣19張.md)。 |
| LFMDA P37–P43 | SegFormer-B2 source-only、PCD 與 SAM/LRH 兩階段，GT 定位窗口 | P37 臺灣 19 張；P41/P42 臺灣 24 張，均屬 GT-aware 覆蓋 | P37 final 0.3563（單 seed，支持作後續基準）；P41 T3 0.4906→P42 0.4786（同 seed，stage 2 未改善）；P43 補 seed 亦未顯示 stage 2 增益。見 [P37](../wiki/experiments/20261101_P37_LFMDA_SegFormerB2_sourceonly_結果.md)、[P42](../wiki/experiments/20261104_P42_LFMDA_stage2_from_P41_T3_執行紀錄.md)、[P43](../wiki/experiments/20261105_P43_LFMDA流程補seed穩定性_執行紀錄.md)。 |

## 轉向 ROI 的原因與新基線

GT 置中窗口只看標註附近，無法充分描述周圍海面大量背景、遠距誤判和整景漏判。P44 改用 GT 衍生錨點的 3,840×3,840 ROI，在 ROI 內以 256 窗、步距 128 訓練／推論，不挑背景，已由使用者定為**ROI 階段自然比例 baseline**。同一 ROI 評估器下，P12 三個既有權重為 0.209／0.217／0.223；P44 S42 為 0.300，但 P44 三 seed 包含 S44 塌縮，應完整揭露。[P44 結果](../wiki/experiments/20261106_P44_ROI自然比例基線_結果.md)、[P48 補 seed](../wiki/experiments/20261109_P48_H1prime與P44補seed_結果.md)。ROI 仍由 GT 定位，不是全景盲偵測。現在的報告以**大林兩張**與**其他 22 張**分列。

本頁是歷史快照，不會因後續 ROI 新結果而改寫舊實驗判定；新結果放 `wiki/experiments/`。
