---
date: 2026-11-11
type: summary
tags: [harness, codex, claude-code, vault, snapshot]
---

# AI 協作 Harness 現況快照（2026-11-11）

這是 2026-11-11 的架構快照；舊 [2026-07-11 快照](20260711_AI協作Harness現況快照.md)僅供歷史比較。即時研究狀態以 `/home/alanyh/.agents/institution/research/oil_spill_project_status.md` 為準，工作交接讀 [handoff](../.agents/handoff.md) 的最新 `STATUS_AS_OF`。

| 層級 | 正本／用途 |
|---|---|
| 共同政策 | `/home/alanyh/.agents/institution/README.md` 與 00／10／20；研究現況另讀其 `research/oil_spill_project_status.md`。 |
| 主要入口 | Claude Code、Codex 為對等 primary harness；當下入口主持。另一方不可用時依 institution `40_harness_continuity.md` 接手，不虛構雙方共識。 |
| 研究程式 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand` 為現行主線；實驗卡、設定、指標原件及大型產物留在 repo／資料目錄。 |
| 研究知識 | 本 vault：`wiki/` 是整理後的知識和結果，`outputs/` 存階段快照與分析產物，`.agents/decisions/` 存凍結設計和裁決，`index.md` 導覽，`log.md` 記寫入。 |
| 寫入代理 | Claude/Codex 的 `obsidian-vault-manager` 負責 vault 搜尋、寫入及檢查；主代理負責分派和驗收。若環境限制使代理無法寫入，依本 vault `CLAUDE.md` §4 例外並在 `log.md` 記原因。 |

**Agent 狀態**：`code-reviewer` 只在明確要求審查時使用；`paper-extractor` 做事實萃取；Antigravity 一般顧問入口依 2026-09-17 使用者決定停用，不派遣。舊快照中其角色與模型名稱不是現行調度規則。

**容器與遠端**：主線 Docker image 含 Codex CLI 基線版本；`Docker/run.sh` 將 Codex 設定、登入和 sessions 的持久化目錄掛到 `/root/.codex`，Codex Desktop app-server control socket 使用 tmpfs，容器名稱固定 `oil_deeplab_v3` 供 host 端 wrapper 連入。實際機制與更換容器步驟見 [Docker 環境頁](../wiki/environment/Docker_環境設定.md)及 repo `Docker/PERSISTENCE.md`。此快照不表示當下 process 或遠端連線狀態。

**權限與回寫**：新的實驗設計 1–6 在 launch 前凍結；結果 7–9 入 `wiki/experiments/`；結案與重要裁決當天回寫 vault 和 institution。啟停 process、刪除、commit/push 等依本 vault 規則取得使用者明確授權。不同文件有差異時以使用者當次指示及本 vault／institution 正本為準。
