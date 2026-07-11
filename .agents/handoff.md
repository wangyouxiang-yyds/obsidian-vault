# handoff.md — 工作階段交接檔

用途：任一 harness 接手時（例如 Claude token 用盡、使用者跳到 Codex 繼續），讀本檔即可掌握目前工作階段並繼續做工。
更新紀律：每完成一個工作段落就更新「目前狀態」，不要等 token 快用完才寫（真用完就來不及了）。
同機接手：Claude Code 與 Codex 都在這台 Linux server 上，讀寫同一份 vault，接手不需要 commit/push。只有要讓 Windows 端的 vault checkout 看到最新狀態時才需要 commit + push。

## 目前狀態（每次更新覆蓋本節）
- 更新時間：2026-07-10
- 更新者：Claude Code（Linux server）
- 當前任務：建立 Claude × Codex 共識協商與交接制度（本檔與 consensus.md 即本輪產出）
- 已完成：雙 harness 文件按機器分列修正；Claude→Codex 與 Codex→Claude 喚起指令驗證（後者僅限沙箱外）
- 進行中：consensus.md / handoff.md 草稿，待使用者核可
- 下一步：使用者過目 → commit + push；Windows 側驗證 codex→claude、回填 model/effort
- 未提交變更：AGENTS.md、.agents/README.md、.agents/consensus.md、.agents/handoff.md、.claude/commands/ingest-paper.md（使用者自改，勿動）
- 共識紀錄：（尚無）
