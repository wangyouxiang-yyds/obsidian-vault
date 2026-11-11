---
date: 2026-11-11
type: model
tags: [dads-sam, sam, prithvi, model-history]
validity: superseded
superseded_by: "[[DADS-SAM]]"
---

> ⚠ 已作廢（2026-11-11，使用者 WP3 裁決）：日期快照已併入常青模型頁；改看 [[DADS-SAM]]

# DADS-SAM 六波段與 Prithvi＋SAM decoder 嘗試

## 一句話定義

研究中試過兩種不同方案：**B** 是公開 DADS-SAM 的 SAM encoder 六波段適配；**C** 是 Prithvi encoder 接 SAM decoder 的 hybrid。C 不是原版 DADS-SAM，也不能把 B/C 差異歸因於單一模組。

## 試驗與判讀

- GoM-only 的 A（Prithvi＋DeepLabV3）、B、C 正式對照採舊臺灣9景 GT-aware 口徑。B 逐景等權 OilIoU 0.3732、C 0.3596；相對 A 設定四次平均 0.3862，兩者依事前 ±0.05 門檻**皆不可判定**。來源：[正式卡](../wiki/experiments/20261011_DADS_SAM_GoM_only_正式C與B_皆不可判定.md)。
- B 六波段＋Tversky 曾用全球排除臺灣資料測 19 張，單 run 像素加總 OilIoU 0.5815；該口徑及名冊不能與 GoM-only 9景或 ROI 24景直接相比。來源：[全資料結果](../wiki/experiments/20261026_DADS_SAM_全資料排除臺灣_臺灣19張.md)。
- 三折公開碼 loss 與 Tversky、GoM 正式 C/B 的結果另見 `wiki/experiments/` 對應卡；每卡的 source、loss、seed、test 名冊不同。

## 工程與限制

公開碼封存於 `/mnt/backup/alanyh/model_sources/DADS-SAM-7005e82`；Prithvi 權重不能直接載入 SAM encoder，hybrid 需要特徵／位置／提示接口適配。後續 LFMDA 使用封存 SAM 區域做 LRH，與 B/C 監督式分割是不同用途。完整設定以主線 repo `analysis/dads_sam_gomonly_20261005/` 和凍結卡為準。
