# Gemini CLI MCP 在 Docker 容器內的設定與持久化

## 問題分析

Gemini CLI 與 Claude Code 的 MCP 設定存儲方式不同：
- **Claude Code**: 存放在 `~/.claude.json` (單一 JSON 檔案)。
- **Gemini CLI**: 存放在 `~/.gemini/settings.json` (目錄下的 JSON 檔案) 或者專案目錄下的 `.gemini/settings.json`。

當你在 Docker 內執行 `gemini mcp add` 時：
1. 如果你在一個專案目錄下（例如 `/mnt/backup/alanyh`），它會優先寫入該目錄下的 `.gemini/settings.json`。
2. 因為你掛載了 `-v /mnt/backup/alanyh/:/mnt/backup/alanyh/`，這個設定會直接寫回你的 **Host 主機**。
3. 如果 Host 主機上的這個檔案路徑不正確（例如之前手動改過或路徑不對），Gemini CLI 會優先讀取它，導致 MCP 無法連接。

## 解決方案

### 1. 修正專案層級設定 (Project Settings)

目前你的專案路徑 `/mnt/backup/alanyh/.gemini/settings.json` 已被修正為正確的容器內路徑：
```json
{
  "mcpServers": {
    "obsidian": {
      "command": "/usr/bin/mcp-obsidian",
      "args": ["/home/alanyh/obsidian-vault"]
    }
  }
}
```

### 2. 修改 Dockerfile

確保在建置時正確設定全域 MCP（作為備援）：

```dockerfile
# 確保目錄存在
RUN mkdir -p /root/.gemini

# 設定 Gemini MCP (全域)
RUN gemini mcp add obsidian /usr/bin/mcp-obsidian /home/alanyh/obsidian-vault || true
```

### 3. 修改 docker run 腳本 (持久化全域設定)

如果你希望 Gemini 的全域設定（如登入資訊、全域 MCP）也能在容器間持久化，請掛載 `.gemini` 目錄：

```bash
docker run -it --rm \
  ... \
  -v /home/alanyh/.gemini:/root/.gemini \
  ...
```

## 常見陷阱

1. **優先權問題**: 如果專案目錄下有 `.gemini/settings.json`，它會覆蓋 `/root/.gemini/settings.json` 的設定。如果你在 Dockerfile 裡寫的是全域設定，但掛載的專案目錄裡有舊設定，那「重啟/重蓋 Docker」也沒用，因為專案設定是從 Host 掛進來的。
2. **路徑一致性**: 確保 MCP server 的參數路徑（如 Obsidian Vault）在容器內是存在的且正確掛載的。
3. **Binary 路徑**: 建議使用絕對路徑（如 `/usr/bin/mcp-obsidian`）而不是 `npx`，這樣啟動速度更快且更穩定。

## 檢查指令

在容器內執行：
```bash
gemini mcp list
```
應顯示 `✓ obsidian: ... - Connected`。
