---
date: 2026-11-11
type: environment
tags: [docker, codex, claude-code, persistence, remote]
---

# 現行 Docker 環境與 Codex 遠端持久化

本頁依主線 repo `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/Docker/{Dockerfile,run.sh,PERSISTENCE.md}` 於 2026-11-11 核對。舊頁的 OSDMamba v4、Node 20、舊 vault 路徑及「Dockerfile 自動建立 Obsidian MCP」描述已過時。這是配置文件快照；本次未 build image、替換 container 或驗證即時遠端連線。

| 部分 | repo 中的設定 |
|---|---|
| Image | GT_expand `Docker/Dockerfile` 以 `pytorch/pytorch:2.7.1-cuda12.8-cudnn9-runtime` 為 base，安裝 Python 依賴、Node 22、tmux、Claude Code、Codex CLI、mcp-obsidian。`Docker/run.sh` 預設 tag `oil_deeplab:v5`；tag 不是目前 container image ID 的實測。 |
| Codex 程式 | Dockerfile 的 `CODEX_VERSION=0.160.0` 是 build 基線，裝到 `/usr/local/bin/codex`；runtime 可經 launcher 更新，實際版本在容器內查。`/opt/codex` 是 image 安裝區。 |
| Codex 狀態 | `CODEX_HOME=/root/.codex`，由 `STATE_ROOT/codex` 掛入，保存 config、登入、sessions、rules、skills。更新後的可執行檔另在 Docker volume `codex_sqlite_local`；憑證不得進 image 或公開 Git。 |
| Desktop 遠端 | host wrapper 以固定 container 名 `oil_deeplab_v3` 執行 `docker exec ... codex`。`/root/.codex/app-server-control` 掛 root-owned tmpfs 供 app-server socket。 |
| 其他狀態 | `STATE_ROOT/agents`→`/home/alanyh/.agents`；`claude` 和 `claude.json`；`wandb`；`obsidian-vault`→`/home/alanyh/obsidian-vault`。`gemini` 保存歷史狀態，Antigravity 一般顧問入口已停用。 |
| Host 儲存 | `STATE_ROOT` 預設 `/mnt/backup/alanyh/.container_state/oil_deeplab`；2026-10-26 起 `/mnt/backup/alanyh` 是 host 本機 `/home/alanyh/oil_project` 的 bind，NAS 是備份。`run.sh` 檢查 bind 身分與 `.migrated`。 |

**分工**：Dockerfile 定義軟體，`run.sh` 定義掛載與啟動，`PERSISTENCE.md` 定義遷移和驗證。Codex 個人設定應在持久化目錄，**不是烘進 Dockerfile**。只改 Dockerfile 不會更新已存在 container；替換時要保留 `STATE_ROOT` 和固定名稱的遠端連入約定。

## 唯讀檢查順序

1. Docker host 查 `docker ps -a --filter name=oil_deeplab_v3`，核對要進的 container。`run.sh` 會拒絕把舊 container 當新 image 使用。
2. 在既有 container 查 `codex --version`、`test -f /root/.codex/config.toml`、`test -d /home/alanyh/obsidian-vault/.agents`。**不要輸出 auth.json 或 token**。
3. 核對 `CODEX_HOME`、`/root/.codex/app-server-control` tmpfs 與 host `STATE_ROOT/codex`／vault 掛載。遠端失聯先看 `docker logs oil_deeplab_v3` 的 app-server 訊息及 [故障紀錄](20261111_Docker升級與Codex遠端故障紀錄.md)。
4. 升級按 repo `Docker/PERSISTENCE.md` 的新 image、隔離 smoke、切換、驗證順序。stop/start/replace 會改變使用中的環境，須先確認授權與 process 狀態。

研究訓練目前可由主線 `/opt/conda/bin/python` 運行；舊 [VRT 訓練頁](../pipeline/VRT_pipeline_02_模型訓練.md)的 OSDMamba Docker v6 屬歷史分支。Dockerfile／run.sh 若要實際修改，應在主線 repo 另行實作與驗證。
