# CLAUDE.md — 油汙偵測研究 Wiki 維護規則

你是這個研究 Wiki 的維護者。每次對話開始時請先讀取這份文件，了解 Wiki 的結構與規則。

---

## 專案背景

**研究目標**：利用多光譜衛星影像（Sentinel-2 / Landsat 8/9）進行海面油汙自動偵測，最終訓練 Deeplabv3+ 或 OSDmamba 語義分割模型。

**資料來源**：Sen2Like 處理後的多波段 TIF 檔案，統一為 11 個波段（443/492/560/665/704/740/783/833/865/1614/2202 nm），解析度對齊至 10m（10980×10980）。

**Pipeline 全貌**：
```
原始衛星影像 (S2/L8/L9)
    ↓ Sen2Like 處理（大氣校正、光譜一致化、Fusion）
    ↓ stack_multiband_tif.py（11 波段疊合）
    ↓ patch_from_stacked_tif.py（切 256×256 patch）
    ↓ Hard_Negative_Mining（HNM 四步驟）
    ↓ Deeplabv3+ or OSDmamba 訓練
    ↓ 油汙偵測結果
```

---

## Wiki 目錄結構

```
obsidian-vault/
├── CLAUDE.md                  ← 本文件（規則文件，勿修改內容格式）
├── index.md                   ← 所有頁面的目錄（每次新增頁面後更新）
├── log.md                     ← 操作紀錄（append-only，每次操作後追加）
├── raw/                       ← 原始資料（只讀，不修改）
│   ├── papers/                ← 論文 PDF 或摘要
│   └── assets/                ← 影像、圖表
├── wiki/
│   ├── experiments/           ← 實驗紀錄
│   ├── pipeline/              ← Pipeline 各步驟說明
│   ├── models/                ← 模型架構頁面
│   ├── datasets/              ← 資料集說明
│   ├── concepts/              ← 技術概念頁面
│   └── papers/                ← 文獻摘要頁面
└── outputs/                   ← 分析結果、比較表、圖表
```

---

## 頁面格式規範

### 實驗紀錄（wiki/experiments/）
檔名格式：`YYYYMMDD_實驗簡述.md`

```markdown
---
date: YYYY-MM-DD
type: experiment
stage: [preprocess | patching | HNM | training | evaluation]
status: [running | completed | failed]
tags: [相關標籤]
---

# 實驗名稱

## 目標
本次實驗想驗證什麼？

## 設定
- 腳本：`xxx.py`
- 關鍵參數：
  - PATCH_SIZE: 256
  - OVERLAP: 64
  - OIL_TO_BG_RATIO: 1.5
  - NUM_WORKERS: 2

## 結果
- Oil patches: 
- BG patches: 
- 實際比例: 

## 觀察與結論

## 下一步
```

### 概念頁面（wiki/concepts/）
```markdown
---
type: concept
tags: []
related: []
---

# 概念名稱

## 一句話定義

## 詳細說明

## 在本專案的應用

## 相關頁面
```

### 文獻摘要（wiki/papers/）
```markdown
---
type: paper
title: 
authors: 
year: 
tags: []
---

# 論文標題

## 核心貢獻

## 與本專案的關聯

## 關鍵方法

## 重要數據/結果
```

---

## 操作規則

### 新增論文時（Ingest）
1. 閱讀論文，與使用者討論關鍵重點
2. 在 `wiki/papers/` 建立摘要頁面
3. 更新 `wiki/concepts/` 中相關概念頁面
4. 更新 `index.md`
5. 在 `log.md` 追加一筆紀錄

### 記錄實驗時
1. 在 `wiki/experiments/` 建立新頁面
2. 填入設定參數與結果
3. 更新 `index.md`
4. 在 `log.md` 追加一筆紀錄

### 回答問題時
1. 先讀取 `index.md` 找相關頁面
2. 讀取相關頁面合成答案
3. 若答案有價值，存成新頁面至 `outputs/`

### log.md 格式
每筆紀錄以 `## [YYYY-MM-DD] 操作類型 | 說明` 開頭，例如：
```
## [2026-05-04] ingest | 論文：SegFormer oil spill detection
## [2026-05-04] experiment | patch_from_stacked_tif_0421 測試集切割
## [2026-05-04] lint | 檢查孤立頁面與過時內容
```

---

## 本專案關鍵技術知識

### 波段順序（11 bands，升序）
| Index | 波長(nm) | 對應 S2 波段 | 解析度 |
|-------|----------|-------------|--------|
| 1 | 443 | B01 | 60m→10m |
| 2 | 492 | B02 | 10m |
| 3 | 560 | B03 | 10m |
| 4 | 665 | B04 | 10m |
| 5 | 704 | B05 | 20m→10m |
| 6 | 740 | B06 | 20m→10m |
| 7 | 783 | B07 | 20m→10m |
| 8 | 833 | B8A | 10m |
| 9 | 865 | B8A/B05 | 20m→10m |
| 10 | 1614 | B11 | 20m→10m(Fused) |
| 11 | 2202 | B12 | 20m→10m(Fused) |

### Mask 像素值定義
- `128` = Oil（油汙，正樣本）
- `64` = Background（已標記背景）
- `192` = Others（船隻等次要類別）
- `0` = 未標記區域

### Patching 關鍵參數
- PATCH_SIZE: 256
- OVERLAP: 64（油汙 patch 使用，背景不重疊）
- OIL_TO_BG_RATIO: 1.5（balanced 模式，每個油汙 patch 對應 1.5 個背景）
- 斷點續傳：`.scene_status/` 目錄下的 JSON 標記檔

### HNM 四步驟
- Step 2：100% 正樣本 + 10% 隨機背景 → V1 訓練集
- Step 3：訓練 Probe 模型 V1（支援 Resume）
- Step 4：掃描純背景，conf > threshold 的誤判 → Hard Negatives
- Step 5：Golden Dataset 組裝（嚴格防止 data leakage）

---

## Git 同步規則

每次 Claude 完成一批 Wiki 更新後，執行：
```bash
cd /home/alanyh/obsidian-vault
git add .
git commit -m "wiki: [說明更新內容]"
git push
```

---

## 注意事項

- `raw/` 目錄下的檔案**只讀**，不可修改
- `log.md` 只能 append，不可修改舊紀錄
- 每次新增頁面後必須更新 `index.md`
- 不確定的技術細節請詢問使用者後再寫入，不要猜測
- 重大決策須與 Codex 協商取得共識，規則見 `.agents/consensus.md`。
- 工作段落結束時更新 `.agents/handoff.md`；接手他方工作前先讀該檔。


## 啟動規則

每次 Claude Code 在以下目錄啟動時，自動執行：
1. 讀取本文件（CLAUDE.md）
2. 讀取 index.md 了解目前 wiki 狀態
3. 等待使用者指令，不要主動修改任何檔案