---
date: 2026-11-15
type: experiment
stage: evaluation
status: completed
tags: [p53, natural-ratio, global, roi3840, stratified-val, macro3, three-seed, taiwan24, collapse, no-decisive-difference]
---

# P53 自然比例全球 baseline＋分層 VAL＋macro-3 選模：結果（no_decisive_difference，2/3 seed 早期塌縮）

> 結果卡（7–9）。P53＝P44 自然比例全球排除臺灣流程，只換 VAL（P52 的 GoM 30 景＋非 GoM 大／中／小各 8 景＝54 景）與選模（macro-3）。設計卡在 repo：`analysis/p53_global_stratval_20261113/P53_CARD.md`（FROZEN v2，GT_expand commit f5b321b；vault 無鏡像）。相關：[[P44_ROI滑窗基線]]、[[20261113_P52_GoM_stratval_macro3_結果]]、[[20261109_P48_H1prime與P44補seed_結果]]。結案時間 2026-11-15 06:06。

## 7. Evidence / Result

2026-11-14 20:33 起訓、2026-11-15 06:06 評估完成，全部 EXIT=0。S42 訓練 20:33–02:48，160,000 step 未早停，best VE37（step 148,000）。**S43、S44 的 VAL oil IoU 自 VE2 起至 VE9 全為 0（整張預測背景）**，patience 8 於 VE9 早停（36,000 step），best checkpoint＝VE1（step 4,000）。

**選模 VAL（54 景 macro-3；GoM 30 景 macro-3 另列；P52 為其 GoM 30 VAL 的 macro-3）**

| seed | P53 54 景 macro-3（best VE） | P53 GoM 30 macro-3 | P52 macro-3 |
|---|---|---|---|
| S42 | .242（VE37） | .330 | .317 |
| S43 | .057（VE1） | .117 | .314 |
| S44 | .107（VE1） | .193 | .314 |

P53 的 54 景 VAL 與 P52 的 GoM 30 VAL 景集不同，只有 GoM 30 子集可與 P52 並列；與 baseline P44 的 37 景 pooled VAL 口徑亦不同，不可互比。

**臺灣 oil IoU（run 內像素加總；S42／S43／S44 → 跨 seed mean±sd）**

| 子集 | P53 | P52 | 舊 baseline（P44 S42／P48 S43,S44） |
|---|---|---|---|
| 大林 2 張 | .297/.045/.235 → .192±.131 | .345/.180/.374 → .300±.105 | .342/.427/.007 → .258±.222 |
| 其他 22 張 | .199/.046/.189 → .145±.086 | .201/.223/.168 → .197±.028 | .267/.280/.089 → .212±.107 |
| 24 張合計（描述性） | .246/.046/.213 → .169±.108 | .281/.200/.285 → .255±.048 | .232±.164 |

**預先登記判讀（Δ≥.03 且 >2·se）**

| 比較 | 大林 Δ（se） | 其他 22 Δ（se） | 判定 |
|---|---|---|---|
| P53 − P52（主要） | −.107（.097） | −.052（.052） | 皆 no_decisive_difference |
| P53 − 舊 baseline（次要） | −.066（.149） | −.067（.079） | 皆 no_decisive_difference |

**塌縮旗標**（預登記：臺灣 pred_to_gt <0.2）：只有 S43 觸發（大林 .072、其他 .152）。S44 未觸發（大林 .316、其他 .584），但它與 S43 同為 VE1 checkpoint、VAL 在 VE2–VE9 全 0，視為同類訓練失敗，在此一併揭露。

**敏感度（非判讀，僅排除被旗標的 S43）**：P53 大林 .266、其他 22 張 .194（n=2）。

**塌縮頻率（自然比例流程，3 seed 中）**：P44/P48 舊 baseline 1/3（S44）；P53 2/3（S43、S44）；P52 GoM-only 0/3。塌縮屬訓練動態（VE2 後所有 epoch VAL 皆 0），不是 checkpoint 選模造成。

來源：`/home/alanyh/oil_dataset/new/_p53_global_stratval_20261113/eval_result.json`、`train_P53_S42/43/44/history.csv`、`train_complete.json`；institution 已回寫（commit dd3101e）。

## 8. Answer / Finding

**沒有可判定差異，且未取代 baseline**：P53 三 seed 平均低於 P52 與舊 baseline，但差距皆在 seed 雜訊內（se .05–.15），預登記判定 no_decisive_difference。

- 平均被塌縮主導：兩個 VE1 seed 的臺灣 oil IoU 皆 .05 上下；唯一完成長訓練的 S42（大林 .297、其他 .199）與 P52 S42（.345、.201）、舊 baseline S42（.342、.267）同量級，且單 seed 僅描述性。
- 自然比例全球流程在此設定下 3 seed 中塌縮 2 次（P44/P48 為 1 次），P52 GoM-only 為 0 次。這支持「自然比例全球 TRAIN 的訓練穩定性差」的描述，但是否因分層 VAL／macro-3 選模加重未核實（VAL 換了、早停以 macro-3 判斷，S43/S44 的 VE1 macro-3 非零所以被選為 best）。
- 不得寫成「分層 VAL 使全球 baseline 變差」；僅 3 seed、大林僅 2 張，且塌縮 seed 的原因未查。

## 9. Decision / Next

- 待使用者決定：如何處理塌縮（例如不計、重跑補 seed、查塌縮機制、改早停或選模規則）。未獲授權前不改卡、不補跑。
- P54（CROMA optical-large＋DLV3，比較對象為 P53）因此暫被擋住，待上述決定。
- 塌縮機制未查：三個 seed 的 VE1 都有 `nonfinite_grad`／`scale_drops` 記錄（S42 亦同，但 S42 未塌縮），不能據此歸因。
- GPU 閒置。下一張卡：無。

**知識頁影響**
- [[P44_ROI滑窗基線]]：已知限制補入 P53 結果與自然比例流程塌縮頻率 1/3 → P53 2/3，並更正「P51 結果未核實」過時句。
- [[研究問題與未解事項]]：基線穩定性列補入 P53；GoM-only 列補對照。
- [[評估口徑]]：未改（口徑未變；塌縮揭露規則已存在）。
- 被推翻的舊說法：無，勘誤表不變。
