# .agents/ — Claude Code × Codex 共用制度正本

本目錄是「Claude Code」與「Codex CLI」兩個 harness 共用規則的 single source of truth。
兩者地位對等。**Gemini CLI 不是對等 harness**，只接雜工任務（見 `dispatch_to_gemini.md`），不讀寫本目錄。

## 兩個 harness 索引怎麼接到這裡
- Claude Code → 讀 `CLAUDE.md`（vault 完整規則，這次不動）。跨 harness 制度另外查看本目錄。
- Codex → 讀 `AGENTS.md`（薄索引）→ 指向 `CLAUDE.md`（wiki 規則正本）+ 本目錄（跨 harness 制度正本）。

## Harness 互叫
Claude Code 主對話可用已驗證指令非互動喚起 codex：
`codex exec --skip-git-repo-check -s read-only -c tools.web_search=true "<提示詞>"`
（2026-07-10 驗證，web search 須用 config 覆寫，exec 不吃頂層 --search）。要寫檔則改 `-s workspace-write`。
Codex → Claude：`claude -p "<提示詞>"`（2026-07-10 直接執行驗證可用；但 Linux server 核心未開 unprivileged userns，codex exec 沙箱內執行會失敗，互動模式需經核准跳出沙箱）。

## 本目錄結構
- `skills/` — 兩個 harness 共用的 skill 規格（目前：ingest-paper, critical-read, lit-corpus-analysis, paper-triage）
  - Claude Code 實際讀取位置：`.claude/skills/<name>/SKILL.md`，僅 critical-read / lit-corpus-analysis / paper-triage 三個（2026-07-10 Linux server 核對：與此處內容逐字相同）；`ingest-paper` 不在其中，repo 層只有 `.claude/commands/ingest-paper.md`（slash command 版，非 skill），與 `skills/ingest-paper` 正本是不同東西。
  - `~/.claude/skills/ingest-paper/` 與 `~/.agents/skills/ingest-paper/`（個人層級，不隨 repo 走）目前只在 Windows 機器上存在同一份 ingest-paper，2026-07-10 核對三者逐字相同；Linux server 上這兩個個人層級目錄都不存在（2026-07-10 核對），不要在 Linux 上白找。
    但沒有機制強制同步，之後若改 skill 內容，**先改這裡，再手動同步其他處**。
  - Codex 目前沒有確認原生支援 skills 目錄；透過 `AGENTS.md` 引導 Codex 閱讀本目錄取得規則。
- `agents/` — 跨 harness 子代理（subagent）規格（目前：paper-extractor）
  - Claude Code 實際檔案：`.claude/agents/paper-extractor.md`
  - Codex 實際檔案：`.codex/agents/paper-extractor.toml`
  - 兩邊格式不同（frontmatter+Markdown vs TOML），正文需手動保持語意一致；同步狀態記在 `agents/paper-extractor.md` 底部。
- `dispatch_to_gemini.md` — Gemini CLI 的雜工任務範圍與交辦格式，兩個 harness 都適用。
- `consensus.md` — Claude Code × Codex 重大決策共識協商制度（觸發門檻、回合流程、防錨定、仲裁規則）。
- `handoff.md` — 工作階段交接檔；harness 之間接手工作的 single source of truth，更新紀律見該檔。

## 維護規則
1. 任何跨 harness 規則只改這裡一份，不要各自在 `CLAUDE.md` / `AGENTS.md` 裡重複寫。
2. `skills/` 與 `agents/` 底下正本改動後，必須手動同步到 `.claude/` 與 `.codex/` 對應檔案，同步完在該檔「同步檢查」段落記一筆日期。
3. 本目錄常載內容（不含各 skill 自己的長內容）合計不超過幾百行；超過就精簡或拆成按需引用檔。
4. 若發現 `~/.agents/` 或 `~/.claude/skills/` 底下內容跟這裡不一致，代表已經漂移：以這裡的內容為準覆蓋回去，不要反向覆蓋。

## 已知待辦（本輪只做骨架收斂，未展開）
- 型號／effort 調度守則、任務交辦 prompt 範本、判斷力外化 rubric（原任務書 C/D/E 項）尚未寫。
- `.claude/commands/ingest-paper.md` 是第三份 ingest-paper 實作（slash command 格式，非 skill，內容較短），
  與 `skills/ingest-paper` 的分工尚未釐清，建議之後決定合併或保留各自用途。
- Codex 預設 model／reasoning effort：Linux server 側已於 2026-07-10 實測回填至 `AGENTS.md`（`gpt-5.6-sol` / `high`）；Windows 側 `~/.codex/config.toml` 仍未查到明確設定值，待實測回填。
- 研究現況按需引用檔（research status）尚未建立；目前 wiki 本身（index.md + wiki/experiments 等）就是現況來源。
