---
date: 2026-11-11
type: model
tags: [spectralgpt, upernet, prithvi, roi]
validity: current
---

# ROI 替代模型：SpectralGPT+ 與 UperNet

## SpectralGPT+ encoder＋DeepLabV3（P47）

P47 在 P44 ROI 工作流接 SpectralGPT+ encoder；波段佔位、正規化與 token 格也變了，並非純 encoder 消融。單 seed 臺灣 24 張 GT 衍生 ROI、run 內像素加總 OilIoU：best 0.315，P44 S42 0.300，依 P47 事前規則為 `indistinguishable`。P47 4/24 景勝 P44、9 景 IoU=0，運算約慢五倍；單 seed 只屬描述性，不足以支持取代基線。來源：[P47 結果](../experiments/20261109_P47_SpectralGPT_ROI_結果.md)；工程草案在主線 repo `analysis/spectralgpt_dlv3_draft_20261106/`。

## Prithvi＋UperNet head（P50）

P50 固定 H1′ ROI 資料與訓練串流，以 Prithvi 第 5/11/17/23 層做 UperNet 融合，對照同 seed 的第 23 層 DeepLabV3。臺灣 24 張 GT 衍生 ROI 的 run 內像素加總 OilIoU，再跨 S42/S43/S44 平均：UperNet 0.299±0.016、DeepLabV3 0.311±0.014；依 P50 記錄，換作 P49 teacher 的條件未成立。若依目前呈現規則分開看，大林 2 張為 0.451±0.018 對 0.493±0.042，其他 22 張為 0.172±0.005 對 0.184±0.044。這是 H1′ 配方內的描述性比較，不能概括全部 UperNet。來源：[P50 記錄](../experiments/20261110_P50_H1prime_UperNet_三seed_執行紀錄.md)；程式主線 repo `main/prithvi_upernet.py`。

P47 的單 seed、P50 的 H1′ 三 seed 與 [P44 自然比例基線](Prithvi_DeepLabV3.md)不同；報告時同時寫出資料配方、測試範圍、版本和算子。

## 變更紀錄

- 2026-11-11｜合併 ROI 替代 encoder 與 head 的受限證據｜依據 [[20261109_P47_SpectralGPT_ROI_結果]]、[[20261110_P50_H1prime_UperNet_三seed_執行紀錄]]
