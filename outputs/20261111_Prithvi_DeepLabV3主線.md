---
date: 2026-11-11
type: model
tags: [prithvi, deeplabv3, gt-centric, roi, baseline]
validity: superseded
superseded_by: "[[Prithvi_DeepLabV3]]"
---

> ⚠ 已作廢（2026-11-11，使用者 WP3 裁決）：日期快照已併入常青模型頁；改看 [[Prithvi_DeepLabV3]]

# Prithvi-EO-2.0-300M＋DeepLabV3 主線

## 一句話定義

以 Prithvi-EO-2.0-300M 多光譜預訓練 encoder 的第 23 層特徵，接 DeepLabV3 ASPP(6,12,18) 二類分割頭；是 GT-centric 晚期與 P44 ROI 基線的共同模型家族。**不同階段的取樣、清單和訓練設定不能只憑模型名視為同一實驗。**

## 工程設定與演進

- 現行輸入六波段 B02/B03/B04/B8A/B11/B12；原始八槽 VRT 仍保留 B01/B08。P44 用 `/10000` 後接 Prithvi 官方 mean/std；光譜指數另須注意原始 DN offset。詳 [資料集](20261111_資料集現況與物理標記清冊.md)及 vault `CLAUDE.md` §6。
- P12／P14／P15：GT 衍生 256 窗，Tversky α0.3／β0.7；P12 全球排除臺灣，P14 比 GoM/NonGoM，P15 試 GoM 曝光。詳 [GT-centric 快照](20261111_GT-centric階段結果快照.md)。
- P44：同一模型家族改 3,840 ROI 的 256 窗、步距128；自然比例、不挑背景。P44 S42 臺灣 ROI pooled OilIoU 0.300，三 seed 有塌縮，完整結果見 [P44](../wiki/experiments/20261106_P44_ROI自然比例基線_結果.md)與 [P48](../wiki/experiments/20261109_P48_H1prime與P44補seed_結果.md)。

## 已知限制

GT-derived ROI 不是部署時的盲選區；大林兩張主導 pooled 指標，應分大林與其他22張。P44 的單 seed best/last 差異大，任何對照都要報 seed 與 checkpoint。程式正本在主線 repo `main/`、P44 封存卡在 `analysis/p44_roi_sliding_global_20261106/`；本頁不代替凍結設定。
