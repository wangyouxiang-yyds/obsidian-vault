# AGENTS.md — Codex 入口

這是油汙偵測研究 Obsidian vault。Codex 與 Claude Code 是對等的主要協作入口；Antigravity 一般顧問入口依使用者 2026-09-17 決定停用。

## 開場讀取

1. 讀本 vault 的 `CLAUDE.md`：目錄、筆記格式、寫入與 Git 規則。
2. 讀 `index.md` 找頁面；接手工作時讀 `.agents/handoff.md` 的最新狀態。
3. 讀 `/home/alanyh/.agents/institution/README.md` 與 `00_diagnosis.md`、`10_model_dispatch.md`、`20_judgment_rubrics.md`。
4. 研究或實驗工作另讀 `research/oil_spill_project_status.md`；任一主要 harness 不可用時讀 `40_harness_continuity.md`。
5. 寫入前檢查本 vault 的 `git status` 與最近提交，保留他人的未提交修改。

## 權威來源與寫入

- 研究即時狀態以 institution 的 `research/oil_spill_project_status.md` 為準；`.agents/handoff.md` 是交接摘要，`index.md` 只是頁面目錄。
- 實驗設計 1–6 存 `.agents/decisions/`；結果 7–9 存 `wiki/experiments/`。已凍結的設計卡不可因後續結果改寫。
- vault 寫入按 `CLAUDE.md` §4 經 `obsidian-vault-manager`；若環境使該路由無法使用，例外寫入須在 `log.md` 記錄原因。
- kill／launch、刪除、commit／push、出版及不可逆資料或方法變更須使用者明確授權。本 vault 的 `CLAUDE.md` §5 另要求 commit／push 先取得授權。

## 協作文件

- `.agents/README.md`：agent、skill 與協商文件索引。
- `.agents/consensus.md` 與 `.agents/decisions/`：協商和逐案決定。
- `.codex/agents/`：Codex adapter；角色正本在 `.agents/agents/`。
