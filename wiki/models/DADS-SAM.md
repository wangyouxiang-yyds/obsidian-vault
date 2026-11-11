---
date: 2026-11-11
type: model
tags: [dads-sam, sam, prithvi, segmentation]
validity: current
---

# DADS-SAM 與 Prithvi＋SAM decoder

## 兩條不同模型線

公開 DADS-SAM 六波段適配（B）以 SAM encoder 作分割；Prithvi encoder 接 SAM decoder 的 hybrid（C）是另一條線，不能把 B/C 的差異歸因於單一模組。模型來源與接線見主線 repo `analysis/dads_sam_gomonly_20261005/` 及 [GoM-only 正式結果](../experiments/20261011_DADS_SAM_GoM_only_正式C與B_皆不可判定.md)。公開碼封存位置：`/mnt/backup/alanyh/model_sources/DADS-SAM-7005e82`。

## 已知證據

- GoM-only A（Prithvi＋DeepLabV3）、B、C 的舊臺灣 9 張 GT-aware 測試：B 逐景等權 OilIoU 0.3732、C 0.3596，相對 A 設定四次平均 0.3862；依當時事前 ±0.05 門檻，兩者皆不可判定。資料範圍、版本及**逐景等權**口徑以 [正式結果卡](../experiments/20261011_DADS_SAM_GoM_only_正式C與B_皆不可判定.md) 為準，不可與 ROI 24 張 pooled 指標直接比。
- B 六波段＋Tversky 在全球排除臺灣資料上訓練、測臺灣 19 張，單 run 像素加總 OilIoU 0.5815；只屬該 19 張名冊、該模型與 checkpoint 的描述性結果，不能作跨名冊架構排序。來源：[全資料測試卡](../experiments/20261026_DADS_SAM_全資料排除臺灣_臺灣19張.md)。
- 全資料三折公開碼 loss 與 Tversky 比較屬另一資料制度；設定與結果應讀其 [三折結果卡](../experiments/20261024_DADS_SAM_Tversky_全資料3fold.md)，不得將不同 loss、學習率與架構混為單因子。

## 工程邊界

Prithvi 權重不能直接載入 SAM encoder；hybrid 須適配特徵、位置與提示接口。LFMDA 使用 SAM 區域做 LRH，和此處 B/C 監督式分割用途不同。這些實驗沒有取代 [P44 自然比例基線](Prithvi_DeepLabV3.md)；各實驗的跨 seed 穩定性與未見地點泛化仍須按原卡判讀。

## 變更紀錄

- 2026-11-11｜整理 B/C 架構區別及不可跨口徑比較的結果｜依據 [[20261011_DADS_SAM_GoM_only_正式C與B_皆不可判定]]、[[20261026_DADS_SAM_全資料排除臺灣_臺灣19張]]
