# AGENTS.md — Codex 專屬薄索引（勿在此複製長內容，正本見下方連結）

你是 Codex，在這個油汙偵測研究 Wiki repo 裡工作。你和 Claude Code 地位對等，都是主要協作
harness；agy/Antigravity 提供搜尋、初篩與第二意見，不參與正式共識。

## 先讀這些檔案，不要用記憶猜規則
1. `CLAUDE.md` — Wiki 結構、頁面格式、操作規則、波段/Mask/Patching/HNM 技術知識的完整正本。
   內容雖然是寫給 Claude 看的，但規則本身跟 harness 無關，Codex 一樣要照做，不要因為抬頭寫 Claude 就跳過。
2. `index.md` — 目前 wiki 有哪些頁面。
3. `.agents/README.md` — vault 內協商、agent 規格、skills 與 handoff 索引。
4. `.agents/handoff.md` — 目前工作階段交接檔；接手或繼續工作前必讀。

## Codex 專屬事項
- Custom agents：`.codex/agents/` 四份 TOML；行為正本見 `.agents/agents/`。
- Shared skills：`.agents/skills/`，Linux runtime 以 `~/.codex/skills/<name>` symlink 載入。
- 政策正本：`/home/alanyh/.agents/institution/README.md`；Claude 不可用時讀 `40_harness_continuity.md`。
- 重大決策見 `.agents/consensus.md`；對手不可用時不得假冒共識，由使用者仲裁。

## Git 同步
NaN
NaN

## 啟動規則
NaN
NaN
NaN
NaN
