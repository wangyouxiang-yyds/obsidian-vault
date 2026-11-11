---
date: 2026-11-11
type: model
tags: [euda, intrada, domain-adaptation, self-training]
validity: current
---

# 光學 EUDA／IntraDA 原型

## 研究定位

目的是把來源域分割模型調整到臺灣 target 影像：Stage I 試對抗式對齊，Stage II 試候選可靠度或熵排序，Stage III 試 EMA teacher 與偽標籤。各 P 編號的 target 窗、GT 位置先驗與估計量不同；目前紀錄的是**光學改寫原型**，不是公開 EUDA-PLR／IntraDA 方法的完整重現。來源：[P30 結果與診斷](../experiments/20261028_P30_StageIII結果與診斷.md)、[P36 結果](../experiments/20261030_P36_結果.md)；程式在主線 repo `analysis/euda_stage3_20261028/`、`analysis/euda_gtroi_pipeline_20261029/`。

## 已知失效與限制

- P30 target-only EMA Stage III 於 5000 次更新後全背景塌縮。其**歷史主指標**為臺灣 19 張 GT-aware 270 窗的 scene-macro OilIoU，起點 0.4709、終點 0；依當時事前規則否定此配置。這不是現行 run 內 pooled 指標，也不能推論全部 EUDA 方法無效。[P30](../experiments/20261028_P30_StageIII結果與診斷.md)
- P31–P33 改用 GT 位置輔助 target 窗，與 P30 全網格 target 不是同一資料制度。P32/P33 的結果、凍結設計與不同評估口徑須讀各自卡；現行路線判讀不得只用模型名稱合併。來源：institution `research/oil_spill_project_status.md` 中 P31–P33 各節；具體結果頁在本批未核實。
- P36 的 GoM 判別器範本只保留含油窗，三 seed 預測油量比 R 平均 0.577，未達預先門檻 0.60，判 `not_improved`。此指標未讀臺灣 GT，不是臺灣準確率。[P36](../experiments/20261030_P36_結果.md)

未核實：跨 P 編號共同因果機制、使用不同 target 取樣後可否穩定改善臺灣。若改資料角色或方法需新實驗卡；本頁只表述已驗原型的邊界。

## 變更紀錄

- 2026-11-11｜整理 P30–P36 光學改寫、歷史口徑與已知失效｜依據 [[20261028_P30_StageIII結果與診斷]]、[[20261030_P36_結果]]
