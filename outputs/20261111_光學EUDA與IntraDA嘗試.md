---
date: 2026-11-11
type: model
tags: [euda, intrada, domain-adaptation, model-history]
validity: superseded
superseded_by: "[[EUDA_IntraDA]]"
---

> ⚠ 已作廢（2026-11-11，使用者 WP3 裁決）：日期快照已併入常青模型頁；改看 [[EUDA_IntraDA]]

# 光學 EUDA／IntraDA 來源對抗與自訓練嘗試

這組路線把來源域分割模型搬到臺灣 target 影像：Stage I 嘗試對抗式來源對齊，Stage II 使用熵或其他候選可靠度，Stage III 嘗試 EMA teacher／偽標籤。各 P 編號的資料入口和估計量不同，不能概括為某篇公開方法已完整重現。

- **P29–P30**：P30 的 target-only EMA 原型完成 5,000 次更新後全背景塌縮；臺灣19景 GT-aware 歷史主指標 scene-macro OilIoU 由起點 0.4709 降至 0，按預先登記規則否定**此配置**。模型 checkpoint 後由使用者授權清理，保留結果與診斷。[P30](../wiki/experiments/20261028_P30_StageIII結果與診斷.md)。
- **P31–P33**：改用 GT 位置輔助的 target 270 窗探索 Stage I／III，屬另一種目標資料制度。設計與變更見 `.agents/decisions/` 的 P31–P33 卡及 [P30 診斷](../wiki/experiments/20261028_P30_StageIII結果與診斷.md)。
- **P36**：判別器 GoM 範本只保留含油窗，三 seed 預測油量比 R 平均 0.577，低於事前 0.60 門檻，判 `not_improved`。此判定未讀臺灣 GT，**不是臺灣準確率**。[P36 結果](../wiki/experiments/20261030_P36_結果.md)。

這是實際試過的光學域適應原型紀錄。已見失效不等於所有 EUDA／IntraDA 或物理先驗方法都無效；如需換資料角色或方法須另立實驗卡。來源程式在主線 repo `analysis/euda_stage3_20261028/`、`analysis/euda_gtroi_pipeline_20261029/` 等各卡所列路徑。
