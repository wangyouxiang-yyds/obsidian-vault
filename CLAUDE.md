# CLAUDE.md — 油汙偵測研究 Wiki 維護規則

你是這個研究 Wiki 的維護者。每次對話開始時請先讀取這份文件，了解 Wiki 的結構與規則。
Codex 經 `AGENTS.md` 也讀本檔；規則與 harness 無關，雙方都照做。

> **更新節奏（使用者 2026-11-11 裁決）**：本檔每 14 天檢查並更新一次，與 §4 健檢 lint 同次執行。
> - 最後全面更新：**2026-11-11**　下次檢查：**2026-11-25**
> - 「規則」各節（目錄、記錄格式、Git、啟動）只在制度改變時修改；
>   「§6 目前研究基線快照」每 14 天依近期結論更新，舊內容不在本檔保留（歷史在 wiki 頁面與 log.md）。
> - 實驗結論變動很快，不以本檔為準：最新研究數字以 `/home/alanyh/.agents/institution/research/oil_spill_project_status.md`、
>   來源檔與相關頁面為準；目前操作／process 狀態看 `.agents/handoff.md` 的 STATUS_AS_OF；回答時仍須讀正式頁面並查勘誤表（§3.0、§3.6）。

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
│   ├── experiments/   ← 【事件頁】實驗結果卡（九步驟卡 7–9）與執行紀錄
│   ├── papers/        ← 【事件頁】文獻摘要
│   ├── pipeline/      ← 【知識頁】流程與 SOP
│   ├── models/        ← 【知識頁】模型路線
│   ├── datasets/      ← 【知識頁】資料集說明
│   ├── concepts/      ← 【知識頁】概念、評估口徑、研究線
│   ├── environment/   ← 【知識頁】環境設定
│   └── corrections.md ← 作廢清冊（勘誤表，§3.6）
├── outputs/           ← 回答存檔、階段快照、lint 報告
└── .agents/           ← 雙 AI 協作：consensus.md、handoff.md、decisions/（九步驟卡 1–6、裁決）、agents/、skills/
```

---

## 3. 記錄規則（⚠ 2026-11 曾漂移，此節為校正後正本；設計見 `.agents/decisions/20261111_vault知識庫修復_WP0目標設計.md`）

### 3.0 總原則：index 與 log 只指路，內容在頁面
- `index.md` 是地圖、`log.md` 是時間線；**數字、結論、裁決內容一律寫在頁面**，不寫進 index／log。
- `.agents/handoff.md` 只放交接用的操作狀態（STATUS_AS_OF）與共識紀錄一行摘要（`consensus.md` 規定）；它不是研究證據或永久知識來源。
- 回答問題、引用數字時，**必須讀到頁面本身**才可引用；不可只憑 index 摘要、log 條目或 handoff 作答。
- 檔案分三類：
  - **事件頁**（`wiki/experiments/`、`wiki/papers/`、`.agents/decisions/`）：記一次發生的事，寫完後只做最小修改；`.agents/decisions/` 凍結卡**一律不改**（作廢方式見 §3.6）。
  - **知識頁**（`wiki/models/`、`pipeline/`、`datasets/`、`concepts/`、`environment/`）：記「目前我們知道什麼」，**有新證據就改寫**（§3.5）。
  - **輸出與控制檔**：`outputs/`（回答存檔、快照、lint 報告）建立後不改寫正文，新狀態另開新檔，只允許 §3.6 作廢所需的 frontmatter 與頁首一行；`wiki/corrections.md` 依 §3.6 增改列；`index.md`／`log.md`／`.agents/handoff.md` 依 §3.1、§3.2 與 handoff 慣例。

### 3.1 index.md — 只當目錄
- **每個 vault 內頁面一行**：`- [頁面標題](相對路徑.md) — 一句話摘要`，整行約 150 字以內。
- 只連 **vault 內存在的檔案**。repo 路徑（如 `analysis/...`）不可當連結，需要時寫在頁面內文並用 `code` 格式。
- **裁決、數字、執行狀態不寫進 index**：裁決寫 `.agents/decisions/`（或附加到相關 decisions 檔）＋ log.md；
  數字與狀態寫在對應頁面。頁面內容更新時，index 那一行只在標題或一句話摘要失準時才改。
- 同一頁面只有一行；新結果更新既有行，不另加行。

### 3.2 log.md — 操作紀錄
- 每筆一個標題：`## [YYYY-MM-DD] 類型 | 一句話說明`，下面最多 3 行條列（改了哪些檔、指向哪頁）。
- 類型：`ingest`、`experiment`、`prereg`、`result`、`decision`、`correction`、`writeback`、`lint`、`restore`、`harness`。
- **不要**用 `- YYYY-MM-DD｜…` 條列體記新紀錄（2026-11-01～11-11 曾如此，已不再沿用；舊紀錄不改）。
- 細節（數字、時間線、seed 結果）寫在 wiki 頁面，log 只留指標。只能 append，不改舊紀錄。

### 3.3 實驗紀錄（九步驟卡）
- 1–6（設計，launch 前凍結）→ `.agents/decisions/YYYYMMDD_主題.md`
- 7–9（結果，結果出來才填）→ `wiki/experiments/YYYYMMDD_主題.md`，並連回 decisions 檔
- 欄位與判準：`/home/alanyh/.agents/institution/templates/experiment_card.md`、`20_judgment_rubrics.md` R6
- 7–9 的「知識頁影響」欄列出此結果要改寫的知識頁與要作廢的舊說法；結案時照 §4「收進實驗結果」一併處理。
- 實驗結案、共識收斂、process 啟停、資料清理：**當天**回寫 log.md＋handoff.md（＋ institution 現況檔）

### 3.4 頁面 frontmatter
```yaml
---
date: YYYY-MM-DD
type: experiment | model | paper | concept | dataset | environment | pipeline | summary
tags: [kebab-case 英文]
# 實驗頁另加：
stage: preprocess | training | evaluation | supervised | unsupervised
status: running | completed | failed | abandoned   # D3：只用這四個英文值；結論寫內文，不寫進 status
# 除凍結卡外所有頁面皆可加（缺省視為 current）：
validity: current | superseded
superseded_by: "[[取代它的頁面]]"   # validity 為 superseded 時必填
---
```
- 實驗頁章節：1 背景／2 方法／3 關鍵參數／4 量化結果／5 觀察與結論／6 後續方向／7 檔案位置／相關頁面
- 概念頁：一句話定義／詳細說明／在本專案的應用／相關頁面
- 文獻頁：基本資訊／研究問題／方法／主要結果／與本專案對照／啟發／相關筆記
- 寫 `[[wikilink]]` 前先確認目標存在。

### 3.5 知識頁（常青主題頁）
- 一個主題一頁，**檔名不加日期**（例：`wiki/models/Prithvi_DeepLabV3.md`、`wiki/concepts/評估口徑.md`）。
- 新證據到來就改寫正文，使正文永遠代表「目前的理解」；頁尾 `## 變更紀錄` 一行一筆：`- YYYY-MM-DD｜改了什麼｜依據 [[事件頁]]`。
- 可驗證的事實與結論標來源（事件頁連結或 repo 路徑）；量化結果與效能比較另標資料範圍、版本與評估口徑；未核實處明寫「未核實」。
- 階段性整理（某天的全貌）是快照，放 `outputs/`，不放 wiki 知識資料夾。

### 3.6 作廢與復原（使用者 2026-11-11 裁決 D2）
- **作廢不刪內容，且每一筆作廢都登錄勘誤表**（整頁或單項皆同），勘誤表因此是完整的作廢清冊。
  - 整頁作廢（知識頁、事件頁、outputs）：frontmatter `validity: superseded`＋`superseded_by`，頁首加一行 `> ⚠ 已作廢（YYYY-MM-DD，決定者）：原因；改看 [[取代頁]]`。
  - 頁內單一數字或說法作廢：該處加刪除線與 `（YYYY-MM-DD 作廢：原因）`。
  - **凍結卡（`.agents/decisions/`）不動**：由取代它的新卡或結果卡註明「取代 X」，並在勘誤表登錄；讀取時以勘誤表判定該卡已被取代。
- **勘誤表 `wiki/corrections.md`** 每筆一行：`| 日期 | 作廢項目 | 所在頁 | 原因 | 決定者 | 狀態（作廢中／已恢復 YYYY-MM-DD） |`。
- **讀取規則**：`validity: superseded` 的頁不得當作現行結論引用；引用任何數字前，先查勘誤表是否登錄。
- **復原只由使用者發起**：使用者說「恢復 X」才做，log 記 `## [YYYY-MM-DD] restore | X`：
  - 整頁：`validity` 改回 `current`、刪除 `superseded_by`、頁首作廢行改為 `> 已於 YYYY-MM-DD 恢復（使用者）：原因`。
  - 單一數字或說法：解除刪除線、在該處加 `（YYYY-MM-DD 恢復：原因）`。
  - 凍結卡 A（曾被 B 取代）：新增一張**恢復決策卡** C（`.agents/decisions/YYYYMMDD_恢復_<A>.md`，記「恢復 A、取代 B」及原因）；A 的勘誤列改「已恢復」、為 B 新增一列「作廢中（被 C 取代）」；A、B 原卡都不改。
  - 勘誤表：**更新原本那一列**的狀態為「已恢復 YYYY-MM-DD＋原因」，不新增第二列。
- AI **不得自行復原**；新證據看似支持已作廢的結果時，只能提醒使用者「與已作廢的 X 相關」。

---

## 4. 操作流程

- **寫入權限**：Claude 與 Codex 對 vault 的任何寫入一律經各自的 `obsidian-vault-manager` agent；
  緊急繞過（break-glass）須在 log.md 記原因。
- **新增論文（ingest）**：讀論文並與使用者討論重點 → `wiki/papers/` 建頁 → 更新相關 concepts → index 一行 → log 一筆
- **收進實驗結果（D4）**：一個實驗結案時，依序
  1. `wiki/experiments/` 寫結果卡（7–9，含「知識頁影響」欄）；
  2. **改寫「知識頁影響」列出的每一頁**（§3.5，頁尾記變更紀錄並連回結果卡）；沒有列出、但主題明顯相關的知識頁也要檢查；
  3. 被推翻的舊說法依 §3.6 標作廢並登錄勘誤表；
  4. index 一行（只在新增頁面或摘要失準時）→ log 一筆 `result`（§3.2 三行內，列改了哪些頁的路徑）；
  5. handoff「目前狀態」最上方加一行 STATUS_AS_OF；
  6. institution `research/oil_spill_project_status.md` 當天回寫（不論結論是支持、否定或不可判定）。此檔不在 vault，由主對話寫入；vault-manager 在回報中提醒。
  只寫結果卡＋index＋log、沒有動到任何知識頁＝未完成（除非卡上寫明「無知識頁影響」及理由）。
- **回答問題（D0）**：index 找頁 → 讀頁面本身 → 查勘誤表 → 附出處回答；問「最近發生什麼」才看 log 尾端，再循指標讀頁。有保存價值的回答整理成 `outputs/` 新頁（並依 §3.1 加 index 一行）。
- **健檢 lint（D5）**：自 2026-11-25 起**每 14 天**一次（11-25、12-09、12-23…），與本檔 §6 更新同次執行；「下次檢查」日期見檔頭。檢查：頁面間矛盾、被新結果推翻但未標作廢的說法、孤兒頁（index 沒列、沒人連）、反覆出現卻沒有自己頁面的概念、斷鏈、index／log 是否又塞了內容、handoff 是否塞入超出 STATUS_AS_OF／共識一行摘要的研究細節、知識頁是否落後於最近的結果卡。報告存 `outputs/lint_YYYYMMDD.md`（附勘誤表狀態為「作廢中」的**列數**，並加 index 一行），修正後 log 記 `lint` 一筆。lint 由「最近兩週沒有主寫 vault 的一方」執行為佳；無法判定或只有一個 harness 可用時由可用的一方執行，不因此延後。
- **內容維護不是只更新目錄**：datasets（新景、標記、來源）、models（試過的架構與限制）、pipeline（可重用流程）、environment（環境或故障）都依 §3.5 維護。沒有新證據時不為了湊齊資料夾而寫頁。
- **驗收**：新結果應能從 index 找到相應頁；內容頁依 §3.5 標來源與日期（量化結果與效能比較另標資料範圍、版本與評估口徑），未核實處明寫；主代理核對頁面存在、鏈結可用。
- 不確定的技術細節先問使用者，不要猜。重大決策與 Codex 協商（`.agents/consensus.md`），工作段落結束更新 `.agents/handoff.md`。

---

## 5. Git 規則

- 寫入完成後回報 `git status`／diff 摘要；**commit 與 push 都要使用者明確授權**，不自動 `git add . && push`。
- 只 `git add` 明列的檔案；commit 訊息格式 `[claude|codex|user] 範圍: 說明`。
- ⚠ 本 vault 的 GitHub repo 是**公開**的：不寫帳密、token、私人資料。

---

## 6. 目前研究基線快照（每 14 天更新；本版 2026-11-11）

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
