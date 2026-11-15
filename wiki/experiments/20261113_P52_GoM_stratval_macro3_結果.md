---
date: 2026-11-13
type: experiment
stage: evaluation
status: completed
tags: [p52, gom-only, roi3840, stratified-val, macro3, three-seed, taiwan24, no-decisive-difference]
---

# P52 GoM-only 分層 VAL＋macro-3 選模：結果（no_decisive_difference）

> 結果卡（7–9）。P52＝P51 GoM-only 自然比例 ROI 流程，只換 VAL（ctx k2k4 v6 共用切分）與選模（macro-3）。設計卡在 repo：`analysis/p52_gom_stratval_20261111/P52_CARD.md`（FROZEN v3，code commit 47c5fd2；vault 無鏡像）。相關設計：[[20261111_P51_GoM_only_ROI資料制度_設計]]；VAL 切分來源見 [[20261112_ctx_k2k4_v6_pilot_不可判定_結果]]。結案時間 2026-11-13 05:57。

## 7. Evidence / Result

設定：GoM 195 景，VAL 30 景（大／中／小各 10）、TRAIN 165 景；選模＝三個 size-bin VAL IoU 的平均（macro-3）；seed 42/43/44。2026-11-12 18:55 起訓（ctx k2k4 K1/K2/K4 之後取得 GPU 鎖），2026-11-13 05:57 全部完成，EXIT=0，無 collapse。Best VE 27／12／15；VAL macro-3 .317／.314／.314（與 P51 的 GoM25 pooled VAL、baseline P44 的 37 景 pooled VAL 口徑不同，不可互比）。

**臺灣 oil IoU（run 內像素加總；S42／S43／S44 → 跨 seed mean±sd）**

| 子集 | P52 | 自然比例 baseline（P44 S42／P48 S43,S44） | P51 |
|---|---|---|---|
| 大林 2 張 | .345/.180/.374 → .300±.105 | .342/.427/.007 → .258±.222 | .209/.304/.169 → .228±.069 |
| 其他 22 張 | .201/.223/.168 → .197±.028 | .267/.280/.089 → .212±.107 | .172/.187/.269 → .209±.052 |

**預先登記判讀（Δ≥.03 且 >2·se）**

| 比較 | 大林 Δ（se） | 其他 22 Δ（se） |
|---|---|---|
| P52 − baseline（主要） | +.041（.142） | −.015（.064） |
| P52 − P51（次要） | +.072（.072） | −.012（.034） |

四項皆 no_decisive_difference。

**敏感度（非判讀）**：排除塌縮的 baseline S44 後，baseline 大林 .384、其他 22 張 .274，P52 皆低於之。全 24 張描述性：P52 .281／.200／.285。

來源：`/home/alanyh/oil_dataset/new/_p52_gom_stratval_20261111/eval_result.json`；institution commit cb19e35。

## 8. Answer / Finding

**沒有可判定差異**：分層 VAL＋macro-3 沒有讓臺灣結果明確優於自然比例 baseline 或 P51。

- 描述性：其他 22 張的 seed 間 sd .028，低於 P51 的 .052（baseline .107），GoM-only 的 seed 變異在此設定下較小；但平均未優於 baseline 或 P51。
- 大林 2 張 P52 平均 .300 高於 P51 .228，差距在 seed 雜訊內（se .072）。
- 不得寫成「P52 較佳」或「分層 VAL 有效」；僅 3 seed、大林僅 2 張。

## 9. Decision / Next

- GoM-only 路線（P51、P52）至此：兩個 VAL／選模設定皆未明顯優於全球自然比例 baseline；P52 只降低 seed 變異。後續是否繼續 GoM-only 待使用者決定。
- 未排除的替代解釋：baseline 的 S44 塌縮拉低其平均與放大其 sd；TRAIN／VAL／曝光量一起改，不能單獨歸因 VAL 或選模。
- 下一張卡：無。GPU 鎖現由 ctx k2k4 補 seed launcher（pid 2322067）持有，訓練 K1_S43。

**知識頁影響**
- [[研究問題與未解事項]]：「GoM-only 的資料制度有利臺灣嗎？」列補入 P52 結果（no_decisive_difference；seed 變異較小）。
- [[P44_ROI滑窗基線]]：未改（baseline 數字未變；其「P51 結果未核實」句屬 P51 結案欄，P51 結果卡尚未建立，另行處理）。
- 被推翻的舊說法：無，勘誤表不變。
