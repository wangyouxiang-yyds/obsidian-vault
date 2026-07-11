# 2026-07-11 研究路線重排與 named-incident OOD holdout(consensus v2 第 3 案)

## 議題
使用者交辦:就 TODO/ 研究路線卡與目前進度,與 codex 討論各方面研究可能與改善方法。

## 協商前實查(主持方)
TODO 卡 vs 程式碼一致性:00 卡 C1-C4 宣稱未修,實際全已落地(seed=main_runner.py:52、stats 自動重算、F4 fail-fast、else→raise);01 卡 over-prediction 前提被 07-09 under-seg 診斷推翻;10 總覽基線表停在 07-03。

## 逐輪演變(4 回合收斂)
- R1(codex 盲提案):OBJECT,BLOCKING 5 項(02 升 P0/過時卡回填/12・14 不得再以殺 FP 為目標/v3plus→ignorering→受控 context 順序/B 結果須回 A 驗證)。新貢獻:validation-only threshold calibration、FN 邊界距離分解、假說族 registry、多次接觸 test set 風險。
- R2(候選稿 v1+主持方實查):scene 層級零洩漏(codex 盲點半解消),但事件層級重疊坐實 → codex REVISE,BLOCKING 5 項精修(event-holdout 條件式必要/ledger 與 registry 分開/exploratory 標注/cascade 稱診斷性擱置/quarantine 場景排除 normalization)。
- R3(候選稿 v2+event 可行性稽核):NOAA 三區域集合佔 89%、具名事故僅 9 個/36 scenes → 傳統 leave-event-out 不可行,提出 E1 具名事故 holdout/E2 時空稽核/E3 三層措辭 → codex REVISE,BLOCKING 5 項修文(36≠35/E1 改稱 named-incident OOD holdout 承認 compound domain shift/即刻封存/event-cluster bootstrap/主結果稱 scene-disjoint mixed-corpus CV)。
- R4(候選稿 v3):codex ACCEPT,BLOCKING=[],CONFIDENCE HIGH,附理由/殘餘風險/反證條件。

## blocking 變化
5→5(換內容)→5(換內容)→[]。收斂輪數 4。返工:無(每輪 blocking 均為精修非推翻)。

## 最終決議(已執行)
1. 36 scenes/9 events 封存(analysis/named_incident_holdout/),自即日起選模不得接觸;投稿前 NOAA-only 重訓一次性評估。
2. TODO 重排:00/01/11 關閉、02 升 P0(locked protocol+ledger/registry)、12 診斷性擱置、13 降 P3、14 改寫 threshold 校準卡、新 03/15;10 總覽=唯一決策索引。
3. Roadmap:v3plus 判定→ignorering(C2 待使用者)→threshold+FN 分解→受控 context→B 移植 A(投稿 gate)→條件觸發 Prithvi/MADOS→OOD holdout。
4. 主張三層措辭:scene-disjoint mixed-corpus CV/named-incident OOD/時空稽核 limitation。

## 少數意見
無。

## 殘餘風險(codex 提出)
9 事件 bootstrap 區間寬;NOAA→事故的標註政策差異可能主導 OOD 落差;封存是前瞻非回溯;NOAA 時空近重複若比例高則主 CV 高估泛化;B 過閘門不保證轉移 A。

## 反證條件
ignorering 無效即關閉背景訊號假說不無限調半徑;context 無效才升 Prithvi;時空稽核高重疊→spatial split 升必要件;OOD 改善僅由 1-2 事件驅動→不得宣稱穩健轉移;B 組合 A 不重現→主張限縮;封存後接觸事故結果→holdout 降級 exploratory。

## 後續驗證點
v3plus 完跑判定;E2 時空稽核結果;ignorering 實驗;consensus v2 試行評估(本案為第 3 個樣本,首個 4 回合案例)。
