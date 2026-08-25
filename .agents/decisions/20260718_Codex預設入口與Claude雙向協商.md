# 2026-07-18 Codex 預設入口與 Claude 雙向協商

## 議題

使用者未來無法使用 Claude Fable 5，預計主要從 Codex 下令，但要求 Claude Code 與 Codex 保持對等且能真正多回合雙向往返。本案修正主持角色、Claude 模型路由、session resume、model/harness failover 與 parity gate；不是研究方法論決策。

## 協商前實查

- Claude CLI：2.1.214。
- Claude 全域設定原為損壞字串 `claude-fable-5[1m]`；現行 Codex → Claude 指令是未釘模型、未保存 session 的裸 `claude -p`。
- Claude CLI 支援 `--model`、`--effort`、`--output-format json`、`--resume`。
- 本案 requested/actual model：`claude-opus-4-8`；effort：high。
- Claude session id：`1a00669c-ceda-4eeb-b5e0-7aec42391db4`；R1 → R2 → R3 均沿用同一 session。
- 實作前 parity audit：`PARITY AUDIT: PASS (agents=4 skills=4 projects=2)`。
- 實作後未指定 `--model` 的 low-effort smoke test 回傳 `DEFAULT_MODEL_OK`，actual model=`claude-sonnet-5`。

## 逐輪候選稿演變

- **R1 Claude 盲提案 — REVISE**：提出全域預設改 Sonnet、正式共識釘 Opus 4.8/high、JSON session resume、L0/L1/L2、深度 1 與 VERDICT 機器解析。Blocking：B1 成本上限、B2 resume 未驗證、B3 Codex 能否擷取 session id。
- **R2 Codex 單一候選稿 — Claude REVISE**：Fable 完全移出依賴/fallback；一般諮詢 Sonnet、正式共識 Opus；成本沿用 5/10 回合與停滯護欄，不武斷設美元上限；本輪實證 resume 與 JSON 擷取；補 README/10 中 Claude 主持與執行階級的不對稱。B1–B3 撤回；新增 B4：主持 harness 的執行重新分配必須先通過 parity audit。
- **R3 — ACCEPT**：Codex 接受 B4，實跑 audit PASS，明訂主持權不得代替 capability parity；一般 Sonnet 不在呼叫點覆寫 effort（目前實際繼承全域 high），故障探針只在實際診斷時使用。Claude 最終 `ACCEPT / BLOCKING: [] / CONFIDENCE: HIGH`。

## Blocking 變化

- R1：B1、B2、B3。
- R2：B1–B3 解消；B4。
- R3：B4 由 audit PASS、adapter gate 與本案 vault-manager 路由解消；`BLOCKING: []`。

## 停止原因

第 3 回合雙方收斂：Claude 明確 ACCEPT、blocking 清空、RESTATE 準確；未觸發 5 回合檢查點、10 回合上限或停滯偵測。

## 最終決議

1. Claude Code 與 Codex 維持對等 primary；使用者當下入口主持，並記錄使用者預期以 Codex 為常態入口。
2. Claude 全域預設改 `claude-sonnet-5`。一般跨 harness 諮詢用 Sonnet 且不在呼叫點覆寫 effort（目前繼承全域 `effortLevel=high`）；正式共識明確使用 `claude-opus-4-8 --effort high`。
3. Fable 5 完全移出制度依賴與 fallback。Opus 不可用時降 Sonnet並記錄；這是單一模型 L0，不是 Claude harness L1。
4. Codex → Claude 首輪以 JSON 取得 session id，後續 `--resume` 同一 session；Claude → Codex 沿用 `codex exec resume`。每案記 requested/actual model、CLI version、session id 與降級事件。
5. 只有面對使用者的主持方可推進回合；headless 對手不得 fan-out 或反向再喚起另一 primary，最大深度 1。
6. 主持 harness 用自己的 adapter 執行前，parity audit 必須 PASS；未過就走已驗證 adapter 或延後。vault 寫入永不得 inline 繞過 vault-manager。
7. 成本護欄沿用第 5 回合回報、第 10 回合硬上限與停滯偵測；使用者未指定前不虛構美元/token cap。

## 少數意見

無。Claude R1 的三項 blocking 與 R2 的 B4 均以實測或制度條款解消。

## 殘餘風險

- parity audit 證明存在性、source/runtime 未漂移與設定合法，不等於完整行為語意測試。
- JSON schema 綁 Claude CLI 2.1.214；CLI 升級後須重跑 resume 驗收。
- resume 的同 session 持續性以 R2/R3 行為、actual model 與 usage metadata 佐證，仍屬外部行為證據。
- 本 session 的 active writable roots 不含 vault；vault-manager adapter 完成唯讀盤點後正確拒絕寫入。因 bubblewrap userns 失敗，主持方在使用者明確批准下以外部 `apply_patch` transport 完成窄 patch，記為 break-glass；未 commit/push。
- Fresh-context audit 曾指出一般 Sonnet 的 effort 文字與既有全域 `effortLevel=high` 不一致；本案保留使用者既有全域 high 設定，並改文為明確繼承。

## 反證條件

- parity audit 轉 FAIL，或發現 Codex adapter 行為語意分歧（尤其 vault-manager inline 寫入）。
- 真實共識無法完成至少兩輪同 session resume。
- Claude CLI 升級後 JSON `session_id` 擷取或 `--resume` 失效。
- 任一入口仍必須依賴 Fable 5 才能協商。

任一成立即停用多回合路徑，回退為顯式釘 Opus 的單發 `claude -p` 並上報使用者；全域設定不得退回損壞的 Fable 字串。

## 後續驗證點

- [x] Claude R1 → R2 → R3 同 session resume。
- [x] JSON 擷取 session id、actual model 與 usage。
- [x] `scripts/audit_harness_parity.py` 通過。
- [x] 實作完成後 read-back、JSON 合法性與 dirty-diff 保護。
- [x] Claude 全域預設動態解析為 `claude-sonnet-5`。
- [ ] Claude CLI 升級時重跑雙向 resume smoke test。
