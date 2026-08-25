# 2026-07-27 Claude Opus 5 正式共識路由升級

## 議題

Anthropic 推出 Opus 5。使用者明確要求因應此發布，將 `.agents/consensus.md` 的正式共識模型路由由 `claude-opus-4-8 --effort high` 升級為 `claude-opus-5 --effort high`。Claude Code 與 Codex 已就此完成三輪同 session 正式協商並收斂；effective date 2026-07-27。本案僅是模型路由升級，不涉及研究方法論、實驗設計或會寫進論文的結論。

## 協商前實查

- 主 session：requested/actual model 均為 `claude-opus-5`；effort=high；Claude CLI=2.1.220；session_id=`8ad72622-0327-4b21-afe2-fdcc49fc3c3f`；provider=firstParty；本案三輪全程無 fallback、無降級事件。
- effort 負對照 session：`26cfd45e-4d6b-469b-8504-994c5746321b`，requested/actual model 同為 `claude-opus-5`，呼叫點指定 `--effort low`；JSONL 紀錄之 actual effort=low、CLI=2.1.220、成本 USD 0.068335。與主 session 的 actual effort=high 對照，證明呼叫點 `--effort` flag 確實生效、非全域設定覆寫假象。
- 外部查證註記：Anthropic 公開搜尋結果與本機實際供應狀態一度不一致（搜尋結果未必即時反映本機 CLI/後端的實際可用模型），因此本案以 Claude CLI runtime 回報的 `canonicalModel` 與 JSONL usage 紀錄作為上線證據，不採信單純的網路搜尋結果。

## 逐輪候選稿演變

- **R1 — REVISE**：提出正式共識模型由 Opus 4.8 升級 Opus 5，effort 沿用 high。Blocking：B1 canonical model 字串是否確為 `claude-opus-5`（非別名或未來版號猜測）、B2 `--effort high` 在 Opus 5 上是否確實生效（沿用舊模型的假設未必成立）、B3 升級是否引入未經檢視的成本門檻變化。
- **R2 — REVISE**：以 CLI 2.1.220 runtime 驗證 canonical model 字串，解消 B1；接受 Codex 不臆造固定美元或三回合門檻、沿用既有 5/10 回合停止準則並逐案記錄 usage/cost 的處理，解消 B3；B2（effort 生效）尚待獨立負對照驗證，故仍列 blocking。
- **R3 — ACCEPT / BLOCKING=[] / CONFIDENCE=HIGH**：以 session `26cfd45e-...` 的 `--effort low` 負對照完成，JSONL actual effort=low 與主 session actual effort=high 形成對照，證明呼叫點 flag 生效，B2 解消。雙方 ACCEPT，RESTATE 準確複述對方最強論點。

## Blocking 變化

- R1：B1、B2、B3。
- R2：B1 由 runtime canonical model 驗證解消；B3 由撤回臆造的固定美元／三回合門檻、沿用既有 5/10 回合停止準則與逐案 usage/cost 紀錄解消；B2 續留。
- R3：B2 由 `--effort low` 負對照 JSONL 證據解消；`BLOCKING: []`。

## 停止原因

第 3 回合雙方收斂：`STANCE: ACCEPT`、`BLOCKING: []`、RESTATE 準確；未觸發 5 回合檢查點、10 回合硬上限或停滯偵測。正式三輪（R1–R3）成本：R1 USD 0.217255、R2 USD 0.225526、R3 USD 0.106028，合計 USD 0.548809。

## 最終決議

1. 一般跨 harness 諮詢維持 `claude-sonnet-5`，不在呼叫點覆寫 effort，實際繼承全域 `effortLevel=high`；不因本次升級變動。
2. 正式共識模型升級為 `claude-opus-5 --effort high`；後續 `--resume` 同一 session 時須明確保留與首輪相同的模型與 `--effort high`，不得中途漂移。
3. Fallback 鏈明示為 `claude-opus-5 --effort high` → `claude-opus-4-8 --effort high` → `claude-sonnet-5 --effort high`；不得靜默降級，每次降級須寫入 decision record。降到 Sonnet 5 產生的共識標記 `PROVISIONAL`，不得單獨授權不可逆動作。Fable 5 仍不在 fallback 鏈、不是制度依賴。
4. 制度化呼叫一律使用完整模型 ID（如 `claude-opus-5`、`claude-sonnet-5`），不使用別名；互動式全域設定 `model=opus` 維持不動，僅影響非 headless 的互動 session。
5. 每案逐一記錄 requested/actual model、effort、CLI version、session id、usage/cost、fallback（若有）於 decision record，並在 `handoff.md` 共識紀錄加一行。
6. 停止準則沿用既有 5 回合檢查點、10 回合硬上限與停滯偵測；不新增臆造的美元上限或三回合門檻，成本數字僅作為本案佐證，不轉為制度性護欄。
7. 不追溯修改任何歷史 decision record、`handoff.md` 舊行或實驗 provenance；2026-07-18 CLI 2.1.214 / Opus 4.8 三輪驗證原樣保留為歷史紀錄。

## 少數意見

無。R1 的 B1–B3 均由 R2、R3 的實測證據解消，雙方最終 ACCEPT。

## 殘餘風險

- Canonical model 字串比對可證明「當下」路由到 Opus 5，但不保證能偵測未來後端模型漂移（例如同一字串被 Anthropic 後端靜默替換底層權重）；仍需定期以 runtime 驗證覆核。
- 一般諮詢目前 `effort` 標示為「不覆寫」，但實際生效值取決於全域設定，目前為 `high`；若使用者未來調整全域 `effortLevel`，一般諮詢的實際 effort 會隨之變動，需留意此為隱性耦合。
- 負對照驗證僅覆蓋 effort flag 是否生效，未覆蓋 Opus 5 於長任務/多輪 resume 下是否有未預期的行為差異；此類差異仍需在實際協商案例中持續觀察。

## 反證條件

- Claude CLI 升級後 canonical model 字串或 `--effort` 語意改變，導致本案驗證方式失效。
- 發現呼叫點指定的模型/effort 與 JSONL 實際回報不一致（即 requested≠actual 且非預期降級）。
- Opus 5 在實際協商案例中出現 Opus 4.8 未見的品質或穩定性問題，需要重新評估升級決策。

任一成立即暫停本次升級的預設使用，回退至 fallback 鏈的下一級並上報使用者。

## 後續驗證點

- [x] 主 session 三輪同 session resume（`8ad72622-...`），CLI=2.1.220、actual model=claude-opus-5、effort=high。
- [x] effort 負對照 session（`26cfd45e-...`），`--effort low`，JSONL actual effort=low，證明呼叫點 flag 生效。
- [x] 正式三輪成本核算（R1 0.217255、R2 0.225526、R3 0.106028 USD，合計 0.548809 USD）。
- [x] `consensus.md` 路由段落、decision record 欄位、`handoff.md` 共識紀錄同步更新。
- [ ] 下次 Claude CLI 升級時，重跑 Opus 5 canonical model 與 `--effort` 語意驗證。
