# AGENTS.md — Codex 專屬薄索引（勿在此複製長內容，正本見下方連結）

你是 Codex，在這個油汙偵測研究 Wiki repo 裡工作。你和 Claude Code 地位對等，都是這個研究的主要協作
harness；Gemini CLI 不是對等 harness，只做雜工（見 `.agents/dispatch_to_gemini.md`）。

## 先讀這些檔案，不要用記憶猜規則
1. `CLAUDE.md` — Wiki 結構、頁面格式、操作規則、波段/Mask/Patching/HNM 技術知識的完整正本。
   內容雖然是寫給 Claude 看的，但規則本身跟 harness 無關，Codex 一樣要照做，不要因為抬頭寫 Claude 就跳過。
2. `index.md` — 目前 wiki 有哪些頁面。
3. `.agents/README.md` — Claude Code 與 Codex 共用的跨 harness 制度正本（skills、subagent 對照、Gemini 派工規則）。

## Codex 專屬事項（其餘 harness 中立規則一律去上面三份找，不要在這裡重複寫）
- 子代理：`.codex/agents/paper-extractor.toml`（規格正本見 `.agents/agents/paper-extractor.md`；改動先改正本再同步這份）
- 已啟用外掛（見 `~/.codex/config.toml`）：documents / pdf / spreadsheets / presentations / visualize / browser
- 本專案 trust_level = trusted（`~/.codex/config.toml` 的 `[projects.'d:\lab\obsidian-vault']`）
- Model／reasoning effort：未在 `~/.codex/config.toml` 查到明確預設值，不要憑印象假設，需要時到 Codex
  介面實測後回填這裡。

## Git 同步
```bash
git add .
git commit -m "wiki: [說明更新內容]"
git push
```
不要用舊的 `cd /home/alanyh/obsidian-vault`（先前 Linux server 路徑，已不適用），直接在目前工作目錄操作即可。

## 啟動規則
每次 Codex 在此目錄啟動時：
1. 讀 `CLAUDE.md` → `index.md` → `.agents/README.md`
2. 等待使用者指令，不要主動修改任何檔案
