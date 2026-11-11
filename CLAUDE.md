# CLAUDE.md — 油汙偵測研究 Wiki 維護規則

你是這個研究 Wiki 的維護者。每次對話開始時請先讀取這份文件，了解 Wiki 的結構與規則。
Codex 經 `AGENTS.md` 也讀本檔；規則與 harness 無關，雙方都照做。

> **更新節奏（使用者 2026-11-11 裁決）**：本檔每半個月檢查並更新一次。
> - 最後全面更新：**2026-11-11**　下次檢查：**2026-11-25**
> - 「規則」各節（目錄、記錄格式、Git、啟動）只在制度改變時修改；
>   「§6 目前研究基線快照」每半月依近期結論更新，舊內容不在本檔保留（歷史在 wiki 頁面與 log.md）。
> - 實驗結論變動很快：任何「最新數字／目前做到哪」以 `.agents/handoff.md` 與
>   `/home/alanyh/.agents/institution/research/oil_spill_project_status.md` 為準，不以本檔為準。

---

## 1. 專案背景

**研究目標**：用多光譜衛星影像（Sentinel-2 / Landsat 8/9，經 Sen2Like 處理）做海面油汙語義分割。
主要測試場景為臺灣；訓練資料以全球（排除臺灣）為主，其中墨西哥灣（GoM）佔大宗。

**程式碼位置**（vault 只放知識與紀錄，不放程式）：
- 主線 repo：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand`
- 舊版 0420／0422（11 波段、sliding-window）已非主線，說明保留在 `wiki/pipeline/` 對應頁面，不再在本檔維護。

---

## 2. Wiki 目錄結構

```
obsidian-vault/
├── CLAUDE.md          ← 本文件（規則正本；Codex 經 AGENTS.md 讀本檔）
├── AGENTS.md          ← Codex 薄索引
├── index.md           ← 頁面目錄（只放連結，格式見 §3.1）
├── log.md             ← 操作紀錄（append-only，格式見 §3.2）
├── raw/               ← 原始資料（只讀）：papers/、assets/
├── wiki/
│   ├── experiments/   ← 實驗結果卡（九步驟卡 7–9）與執行紀錄
│   ├── pipeline/      ← 流程與 SOP
│   ├── models/        ← 模型工程紀錄
│   ├── datasets/      ← 資料集說明
│   ├── concepts/      ← 概念頁
│   ├── papers/        ← 文獻摘要
│   └── environment/   ← 環境設定
├── outputs/           ← 分析結果、比較表、快照
└── .agents/           ← 雙 AI 協作：consensus.md、handoff.md、decisions/（九步驟卡 1–6、裁決）、agents/、skills/
```

---

## 3. 記錄規則（⚠ 2026-11 曾漂移，此節為校正後正本）

### 3.1 index.md — 只當目錄
- **每個 vault 內頁面一行**：`- [頁面標題](相對路徑.md) — 一句話摘要`，整行約 150 字以內。
- 只連 **vault 內存在的檔案**。repo 路徑（如 `analysis/...`）不可當連結，需要時寫在頁面內文並用 `code` 格式。
- **裁決、數字、執行狀態不寫進 index**：裁決寫 `.agents/decisions/`（或附加到相關 decisions 檔）＋ log.md；
  數字與狀態寫在對應頁面。頁面內容更新時，index 那一行只在標題或一句話摘要失準時才改。
- 同一頁面只有一行；新結果更新既有行，不另加行。

### 3.2 log.md — 操作紀錄
- 每筆一個標題：`## [YYYY-MM-DD] 類型 | 一句話說明`，下面最多 3 行條列（改了哪些檔、指向哪頁）。
- 類型：`ingest`、`experiment`、`prereg`、`result`、`decision`、`correction`、`writeback`、`lint`、`harness`。
- **不要**用 `- YYYY-MM-DD｜…` 條列體記新紀錄（2026-11-01～11-11 曾如此，已不再沿用；舊紀錄不改）。
- 細節（數字、時間線、seed 結果）寫在 wiki 頁面，log 只留指標。只能 append，不改舊紀錄。

### 3.3 實驗紀錄（九步驟卡）
- 1–6（設計，launch 前凍結）→ `.agents/decisions/YYYYMMDD_主題.md`
- 7–9（結果，結果出來才填）→ `wiki/experiments/YYYYMMDD_主題.md`，並連回 decisions 檔
- 欄位與判準：`/home/alanyh/.agents/institution/templates/experiment_card.md`、`20_judgment_rubrics.md` R6
- 實驗結案、共識收斂、process 啟停、資料清理：**當天**回寫 log.md＋handoff.md（＋ institution 現況檔）

### 3.4 頁面 frontmatter
```yaml
---
date: YYYY-MM-DD
type: experiment | model | paper | concept | dataset | environment | pipeline | summary
tags: [kebab-case 英文]
# 實驗頁另加：
stage: preprocess | training | evaluation | supervised | unsupervised
status: running | completed | failed | abandoned   # 用英文，不寫「完成」
---
```
- 實驗頁章節：1 背景／2 方法／3 關鍵參數／4 量化結果／5 觀察與結論／6 後續方向／7 檔案位置／相關頁面
- 概念頁：一句話定義／詳細說明／在本專案的應用／相關頁面
- 文獻頁：基本資訊／研究問題／方法／主要結果／與本專案對照／啟發／相關筆記
- 寫 `[[wikilink]]` 前先確認目標存在。

---

## 4. 操作流程

- **寫入權限**：Claude 與 Codex 對 vault 的任何寫入一律經各自的 `obsidian-vault-manager` agent；
  緊急繞過（break-glass）須在 log.md 記原因。
- **新增論文（ingest）**：讀論文並與使用者討論重點 → `wiki/papers/` 建頁 → 更新相關 concepts → index 一行 → log 一筆
- **記錄實驗**：依 §3.3 → index 一行 → log 一筆
- **回答問題**：先讀 index.md 找頁 → 讀相關頁面合成答案 → 有保存價值才存到 `outputs/`
- **內容維護不是只更新目錄**：每段研究工作先判斷是否要同步 `wiki/datasets/`（新景、標記、來源與物理欄）、`wiki/models/`（已實試的架構與限制）、`wiki/pipeline/`（可重用的處理路由）、`wiki/environment/`（環境或遠端故障）、`wiki/experiments/`（實驗結果）。階段收束或制度快照放 `outputs/`；原始檔仍留原資料位置／`raw/`。沒有新證據時不為了湊齊資料夾而寫頁。
- **驗收**：新結果或資料變更應能從 `index.md` 找到相應內容頁；內容頁須列來源、版本／日期、評估口徑及未核實處，`log.md` 只記本次寫入指標。主代理要核對這些頁是否存在、鏈結是否可用，不能把 `handoff.md` 或 `log.md` 當作唯一保存位置。
- 不確定的技術細節先問使用者，不要猜。重大決策與 Codex 協商（`.agents/consensus.md`），工作段落結束更新 `.agents/handoff.md`。

---

## 5. Git 規則

- 寫入完成後回報 `git status`／diff 摘要；**commit 與 push 都要使用者明確授權**，不自動 `git add . && push`。
- 只 `git add` 明列的檔案；commit 訊息格式 `[claude|codex|user] 範圍: 說明`。
- ⚠ 本 vault 的 GitHub repo 是**公開**的：不寫帳密、token、私人資料。

---

## 6. 目前研究基線快照（每半月更新；本版 2026-11-11）

> 只放「寫筆記時需要對齊的口徑與設定」，不放最新實驗數字。

**資料與標籤**
- VRT 8 槽：B01、B02、B03、B04、B08、B8A、B11、B12；DN＝反射率×10000＋1000（算光譜指數前要先扣 1000）。
- 模型輸入 6 波段：B02／B03／B04／B8A／B11／B12（/10000 後用 Prithvi 官方 mean/std）。
- 本地 mask：原始值 0＝油；1、2、255＝背景；影像無效像素在 loss 與計分時為 ignore（255）。
- 舊版 mask 定義（128 油／64 背景／192 其他）屬 0420/0422 舊 pipeline，勿套用到現行資料。

**基線流程（P44，2026-11-10 裁決為 baseline：自然比例、不挑背景）**
- 取樣：每景以 3,840×3,840 ROI 框住油區；TRAIN 在 ROI 內 256 窗、步距 128 滑窗；全球排除臺灣。
- 模型：Prithvi-EO-2.0-300M ＋ DeepLabV3 ASPP(6,12,18)，第 23 層；Tversky α0.3／β0.7。
- 臺灣測試：ROI 內步距 128＋翻轉 TTA。
- 卡片：repo `analysis/p44_roi_sliding_global_20261106/P44_CARD_DRAFT.md`；vault `.agents/decisions/20261106_P44_ROI重疊滑窗全球排除臺灣設計.md`

**評估口徑**
- 每個 run 內像素加總算一次 OilIoU，再跨 seed／fold 取平均（2026-10-11 起；舊卡原判定不回頭改）。
- 臺灣結果分兩欄呈現：**大林（2 張）**與**其他 22 張**（2026-11-11 起；不再列 24／19／5）。
- 單 seed 結果標「描述性」；seed 間差距可達 ±0.1 級，比較時須報全部 seed 並揭露塌縮的 seed。

**用語**：並列訓練設定稱「組」，單次執行稱 run／次（不用「臂」）。

---

## 7. 注意事項

- `raw/` 只讀；`log.md` 只 append；新增頁面後更新 index（§3.1）。
- 每次對話開始：讀本檔 → index.md → `.agents/handoff.md` 最近一段；等待使用者指令，不要主動修改任何檔案。
- 發現本檔「下次檢查」日期已過：提醒使用者本檔該更新了。
