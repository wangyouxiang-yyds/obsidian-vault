# Claude Code MCP 在 Docker 容器內無法掛載的解決方案

## 問題描述

進入 Docker 容器後執行 `claude mcp list`，顯示：
```
No MCP servers configured. Use `claude mcp add` to add a server.
```
即使 Dockerfile 裡有寫入設定也無效。

---

## 根本原因

Claude Code 的 MCP 設定**不是**存在以下這些地方：
- ❌ `~/.claude/settings.json`
- ❌ `~/.claude/.mcp.json`
- ❌ `~/.claude/settings.local.json`

**實際上存在：**
```
~/.claude.json   ← 這個檔案（home 目錄下的單一 JSON 檔）
```

而且是按**工作目錄（project）**分組儲存，格式如下：
```json
{
  "projects": {
    "/mnt/backup/alanyh": {
      "mcpServers": {
        "obsidian": {
          "type": "stdio",
          "command": "/usr/bin/mcp-obsidian",
          "args": ["/home/alanyh/obsidian-vault"],
          "env": {}
        }
      }
    }
  }
}
```

---

## 解決步驟

### 步驟 1：確認容器內的 binary 路徑

在容器內執行：
```bash
which mcp-obsidian
```
> 注意：npm 全域安裝的 binary 在不同環境路徑不同，可能是 `/usr/bin/` 或 `/usr/local/bin/`，不能假設。

### 步驟 2：修改 docker run 腳本，掛載 `.claude.json`

在 `docker run` 指令加入兩個 mount：
```bash
-v /home/alanyh/.claude:/root/.claude \
-v /home/alanyh/.claude.json:/root/.claude.json \
```

> **重要：** `/root/.claude.json` 這個路徑在容器內必須預先存在（否則 Docker 會把它建成目錄）。
> 在 Dockerfile 加上 `RUN touch /root/.claude.json` 解決。

### 步驟 3：Dockerfile 加入預建空檔

```dockerfile
# 預建空檔，讓 docker -v 能正確掛載 .claude.json（而不是建成目錄）
RUN touch /root/.claude.json
```

### 步驟 4：進容器後執行一次 MCP 設定（只需做一次）

```bash
claude mcp add obsidian /usr/bin/mcp-obsidian /home/alanyh/obsidian-vault
```

確認成功：
```bash
claude mcp list
# 應顯示：obsidian: /usr/bin/mcp-obsidian /home/alanyh/obsidian-vault - ✓ Connected
```

---

## 為什麼只需做一次？

因為 `-v /home/alanyh/.claude.json:/root/.claude.json` 是 bind mount，容器內對 `~/.claude.json` 的修改會直接寫回 host 的檔案。下次重新進入容器，設定已經存在於 host 的 `.claude.json`，掛載進來後直接生效。

---

## 完整的 docker run 範例

```bash
docker run -it --rm \
  --name osdmamba_env \
  --gpus all \
  --shm-size=32g \
  -w /mnt/backup/alanyh/ \
  -v /mnt/backup/alanyh/:/mnt/backup/alanyh/ \
  -v /home/alanyh/obsidian-vault:/home/alanyh/obsidian-vault \
  -v /home/alanyh/.ssh:/root/.ssh:ro \
  -v /home/alanyh/.claude:/root/.claude \
  -v /home/alanyh/.claude.json:/root/.claude.json \
  -e ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" \
  your-image:tag /bin/bash
```

---

## 常見錯誤

| 錯誤訊息 | 原因 | 解法 |
|----------|------|------|
| `No MCP servers configured` | `.claude.json` 沒有掛載，或工作目錄路徑不符 | 加上 `-v ~/.claude.json:/root/.claude.json` |
| `✗ Failed to connect` | binary 路徑錯誤 | 用 `which mcp-obsidian` 確認實際路徑再 `claude mcp add` |
| Docker 把 `.claude.json` 建成目錄 | 容器內目標路徑不存在 | Dockerfile 加 `RUN touch /root/.claude.json` |
