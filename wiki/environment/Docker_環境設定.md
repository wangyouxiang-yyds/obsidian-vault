# Docker 環境與 MCP 設定紀錄

本文件紀錄了 /mnt/backup/alanyh/OSDMamba/Docker/Dockerfile 中定義的目前專案開發環境設定，確保在不同容器實例中能維持一致的開發體驗。

---

## 1. 基礎映像檔 (Base Image)
- **Image**: ghcr.io/wangyouxiang-yyds/osdmamba_env:v4
- **目的**: 提供基於 PyTorch 的 OSDMamba 核心開發環境。

---

## 2. 系統工具與 Node.js 環境
- **系統工具**: curl, build-essential, ninja-build (用於編譯 CUDA Kernel)。
- **Node.js**: v20.x
- **全域套件**:
  - @google/gemini-cli (Gemini CLI 工具)
  - @anthropic-ai/claude-code (Claude Code CLI 工具)
  - mcp-obsidian (Obsidian Model Context Protocol 伺服器)

---

## 3. 深度學習套件 (Deep Learning stack)
- **CUDA 核心算子**: causal-conv1d, mamba-ssm (針對 RTX 5090 編譯)。
- **關鍵影像處理庫**: rasterio, albumentations, opencv-python-headless, tifffile, segmentation-models-pytorch。
- **科學計算**: numpy, pandas, scikit-learn, scikit-image。

---

## 4. MCP 伺服器設定 (自動化)
為了確保容器重啟後 Obsidian 依然能與 AI 連線，Dockerfile 中已包含以下自動設定指令：

### Claude Code 設定
```bash
claude mcp add obsidian npx mcp-obsidian /home/alanyh/oil_dataset/new/obsidian-vault
```

### Gemini CLI 設定
```bash
gemini mcp add obsidian npx -y mcp-obsidian /home/alanyh/oil_dataset/new/obsidian-vault
```

---

## 5. 環境變數與 Shell 別名
- **終端機顯示**: 啟用 256 色與強制顏色顯示。
- **Git 設定**:
  - User: wangyouxiang-yyds
  - Email: tree8454@gmail.com
  - 安全目錄: 已將 /home/alanyh/oil_dataset/new/obsidian-vault 加入 safe.directory。
- **Bash 別名**:
  - nv: watch -n 5 nvidia-smi (監控 GPU)
  - ll: ls -lah
  - python: python3
- **自動啟動**: 登入 Shell 後會自動執行 conda activate mamba_env。

---

## 6. 掛載路徑
- **Vault 路徑**: /home/alanyh/oil_dataset/new/obsidian-vault
- **工作目錄**: /mnt/backups/alanyh/

---
*最後更新日期: 2026-05-04 (Gemini CLI 自動彙整)*
