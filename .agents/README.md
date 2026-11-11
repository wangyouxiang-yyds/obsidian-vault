# .agents/ — 協作與交接入口

政策正本在 `/home/alanyh/.agents/institution/`；本目錄保存 vault 內的逐案設計、協商紀錄、agent 角色和工作交接。研究即時數字以 institution 的 `research/oil_spill_project_status.md` 為準。

## 先看哪裡

- `handoff.md`：最近工作與接手狀態。讀最上面的 `STATUS_AS_OF` 並核對時間；下方舊條目是歷史，不代表目前仍在執行。
- `consensus.md`：協商紀錄入口；逐案決議及實驗設計見 `decisions/`。
- `../index.md`：vault 內可閱讀頁面的目錄；實驗結果見 `../wiki/experiments/`。
- `../CLAUDE.md`：vault 筆記格式、寫入、Git 和研究基線快照。

## Agent 與 skill

- `agents/obsidian-vault-manager.md`：vault 搜尋、寫入與驗證。
- `agents/code-reviewer.md`：使用者明確要求的唯讀程式審查。
- `agents/paper-extractor.md`：論文事實規格擷取。
- `agents/antigravity-consultant.md`：歷史檔；一般派送已停用。
- `skills/`：論文初篩、收錄、深讀與文獻整合的共用指引。

## 維護原則

- 設計卡凍結前寫九步驟 1–6，結果 7–9 寫實驗頁；不要事後改寫凍結判準。
- 新增內容頁時更新 `../index.md` 一行摘要，操作紀錄只追加到 `../log.md`。
- 回寫時檢查任務是否需要 `../outputs/` 快照或 `../wiki/` 下的 dataset、environment、model、pipeline、experiment 內容頁；不能只在 index/log/handoff 留下摘要。
- Claude Code 與 Codex 均經各自的 vault-manager 寫入；例外路由須在 log 說明。
- 不以舊的 handoff 或 index 摘要推斷 process 現況；需對照最新狀態與實際紀錄。
