---
date: 2026-11-11
type: environment
tags: [docker, codex, remote, troubleshooting, persistence]
---

# Docker 升級與 Codex 遠端：已遇問題及處置入口

這是歷史故障與排查順序，不表示現在故障仍存在。正本程序見主線 repo `Docker/PERSISTENCE.md`、`Docker/run.sh`；主機本機化切換見 `/mnt/backup/alanyh/_migration_20261023/README.md`。

| 狀況 | 已確認原因／目前防護 | 先查 |
|---|---|---|
| 重建 container 後 Codex 對話、登入或 vault 看似消失 | image 與 container 私有層不能當持久化。`run.sh` 掛 `STATE_ROOT/codex`、`agents`、`obsidian-vault`，並要求 `.migrated` 標記。 | `STATE_ROOT`、掛載來源、`/root/.codex` 與 vault 是否存在；不要輸出憑證。 |
| Desktop 遠端連不上，CLI 仍存在 | 曾因 NFS 上 `app-server-control` 目錄 owner 不符合 socket 要求。現行 `run.sh` 以 root-owned tmpfs 覆蓋。 | tmpfs 掛載、`docker logs oil_deeplab_v3` 的 daemon 訊息；CLI 與 Desktop app-server 版本。 |
| build 後仍在舊 image | `docker build` 不替換現存 container；`run.sh` 比對 image ID。host wrapper 指向固定名 `oil_deeplab_v3`。 | container 與 tag 的 image ID；按 `PERSISTENCE.md` 切換。 |
| 關掉終端被誤認為服務停止 | `run.sh` 使用背景常駐 container，再 `docker exec` 進去；離開 shell 不等於停止服務。 | `docker ps`、訓練 process 與 log，分開判斷。 |
| Codex 更新後 CLI／Desktop 版本不同 | CLI 更新包在本地 Docker volume；Desktop app-server 另同步，開著的 chat 可能延後更新。 | `codex --version`、`codex app-server daemon version`、repo 文件列的更新日誌。 |
| GPU 突然 NVML Unknown Error | 曾在 host systemd daemon-reload 後失去容器 GPU 裝置權限；`run.sh` 除 `--gpus all` 還明列 NVIDIA 裝置。 | host／容器 `nvidia-smi`、裝置與維護紀錄；這與 Codex session 消失不同。 |
| state 意外指向 NAS／空目錄 | 2026-10-26 起改 host 本機 bind，NAS 留備份；`run.sh` 核對 device:inode 與 `.migrated`。 | `mountpoint`、`findmnt`、`stat`；先保留原 state。 |

升級時保留遠端的關鍵核對：同一 `STATE_ROOT`；新容器掛載 `/root/.codex`、`/home/alanyh/.agents`、`/home/alanyh/obsidian-vault`；`app-server-control` 是 tmpfs；host wrapper 的 container 名仍是 `oil_deeplab_v3`。新 image 先做隔離 smoke；切換前確認訓練與 chat；切換後驗證 Codex、vault、agent 和 GPU。詳細命令以 repo `Docker/PERSISTENCE.md` 現行版為準。

舊 [Claude MCP 修復](Claude_MCP_Docker_Fix.md)與 [Gemini MCP 修復](Gemini_MCP_Docker_Fix.md)是當時的設定紀錄，不表示目前 Dockerfile 仍用相同方式自動建立 MCP；Antigravity 一般顧問現已停用。
