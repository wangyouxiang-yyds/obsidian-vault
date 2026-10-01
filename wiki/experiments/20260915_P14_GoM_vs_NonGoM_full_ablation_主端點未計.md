---
date: 2026-09-15
type: experiment
stage: training
status: in-progress
tags: [p14, gom-vs-nongom, full-source-ablation, taiwan-holdout, gt-expand, oil-detection, prithvi, deeplabv3, tversky, experiment-card, post-hoc, primary-endpoint-pending, event-macro]
---

# P14 GoM vs NonGoM Full-Source Ablation：主端點（event-macro Oil IoU）尚未計算

> 本卡為「實驗九步驟卡」制度（2026-09-17 新增，`.agents/institution/templates/experiment_card.md` + `20_judgment_rubrics.md` R6）下的事後補寫案例之一（另一張見 [[20260913_P13海域來源敏感度_只部分回答]]）。第 1–6 步重排自 `.agents/decisions/20260914_P14_GoM_full-source_ablation設計.md`，該檔為正本，本卡只做重排濃縮，不取代、不修改原檔內容。第 7–9 步因主端點尚未計算而留白待補。

**實驗編號**：P14
**凍結日期**：2026-09-14
**事後補寫**：是（補寫於 2026-09-18；1-6 重排自 `.agents/decisions/20260914_P14_GoM_full-source_ablation設計.md`，該檔為正本，本卡不取代它）
**取代哪張卡**：無

---

## 前半：開跑前凍結（1-6，來源：20260914 設計決議）

### 1. Objective（目的）—— 對應論文 Background

原始設計未再進一步以「對應論文 Background」的措辭說明這如何推進更上層研究目的；設計文件開頭的議題陳述是最接近此欄的內容：

> 「延續 P13 source-group sensitivity 的提問，但改用『每個海域分組的全部可用 source scenes』（不做 P13 式的 budget-matched 配額），比較 GoM-only 與 NonGoM-only 兩個 full-source 訓練臂在同一台灣 holdout 上的表現差異，並以描述性門檻 δ=0.02 判定差異是否達到值得記錄的量級。」（設計文件第1段）

**使用者 2026-09-18 補述原文**（涵蓋 P14 與 P15 兩步，本卡對應第 1 點）：

> 1. 將墨西哥灣自 G3 中分離，建立 GoM-only 與 Non GoM-only(全資料 - GoM)，可以觀察單只有墨西哥灣海域V.S.全資料扣除墨西哥灣的
> 2. GoM-only做data augmentation觀察提高資料的曝光度是否有增加在各項指標表現

第 2 點對應 P15，九步驟卡見 `.agents/decisions/20260918_P15_GoM曝光量倍增設計_部分事前凍結.md`。

### 2. Problem（問題）—— 對應 Motivation / Related Work

設計文件第4節明確記載相對 P13 方法論的限制與改進動機：

> 「與 P13 不同，P14 兩臂各自擁有獨立的 source-domain Val 集（GoM-Val 312 oil/390 BG；NonGoM-Val 132 oil/310 BG），checkpoint 依各臂在自身 source-Val 上的最佳表現選取，不使用 Taiwan 資料做模型選擇，避免 Taiwan-side 洩漏進入 checkpoint 選取。此為相對 P13 train-proxy 選模的方法論改進，但兩臂 Val 規模不同（312/390 vs 132/310），選模穩定性仍可能因 Val 集大小差異而不對稱，不視為完全消除的差異來源。」

### 3. Research Question（研究問題）—— 對應 Problem Definition

> 「描述『在這一個 seed、凍結協定、固定台灣 holdout 下，GoM-only 與 NonGoM-only 兩個 full-source 訓練臂的 Taiwan Oil IoU 差值是否達到 δ=0.02 描述性門檻』。」（設計文件第6節「可以」條列）

**已否定路線檢查(R6-1,必填)**：（原始設計未明確記載，補寫時無法回推。）

### 4. Hypothesis（假說）—— 對應 Experiment 初稿

（原始設計未明確記載，補寫時無法回推。）設計文件未登記開跑前預期 GoM-only 或 NonGoM-only 何者表現較佳，也未給理由；只登記了對稱的判定門檻（見第6步），未登記方向性預期。

### 5. Method（方法）—— 對應 Proposed Method

做法：兩臂 full-source ablation，各自使用該海域分組全部可用 scenes，未做跨臂配額配平。

| Arm | Train scenes | Train oil patches | Train BG patches | Val oil | Val BG |
|---|---:|---:|---:|---:|---:|
| GoM-only | 156 | 1,188 | 1,550 | 312 | 390 |
| NonGoM-only | 120 | 584 | 1,180 | 132 | 310 |

模型：Prithvi-EO-2.0-300M + DeepLabV3+（與 P13 相同）。Loss：Tversky `alpha=0.3` / `beta=0.7`（與 P13 相同）。batch size 16，最多 300 epochs，patience 50。Checkpoint selection：best source-Val（不使用 Taiwan 資料做模型選擇）。seed = 42，單次 run／每臂。Taiwan test：137 patches／9 scenes／2 events（與 P13 完全相同的固定 holdout）。

**對照組與 estimand(R6-2,必填)**
- 對照組是誰：（原始設計未明確記載到 run 名稱或設定 hash 層級，補寫時無法回推。）設計文件僅以「GoM-only」「NonGoM-only」兩臂並列比較，未指定何者為 control。
- 主指標：Taiwan holdout 的 Oil IoU（設計文件第5節）。`_kind`：（原始設計未明確記載，補寫時無法回推。）
- 聚合方式：**event-macro**（兩事件等權）——來源：`.agents/institution/research/oil_spill_project_status.md` 第23行「主端點為兩事件等權 event-macro Oil IoU」。此句落於任務指定的第11–26行範圍內；設計決議文件本身未寫出「event-macro」四字，聚合方式定義是由現況檔補齊，非設計文件原文。
- 兩組間唯一不同的是什麼：非單一變因。設計文件第9節明載：「GoM／NonGoM 兩臂 scenes／oil／BG 數量皆不對等（full-source、非配平設計），無法排除資料量或場景多樣性混淆」，故兩臂同時在 source 海域分組與資料規模上不同，非受控實驗。

### 6. Expected Evidence（預期證據）—— 逐字照抄，來源：現況檔第23行 + 設計文件第5節

- 若 Hypothesis 成立，預期看到：（原始設計未記載方向性 Hypothesis，故無對應描述；僅登記了下列對稱門檻。）
- **算支持的門檻（逐字，來源：現況檔第23行）**：「只有兩事件 ΔOil-IoU 同方向且各 `|Δ|>=0.02` 才稱在固定測試上有一致且具實務幅度的較高表現」。設計文件第5節的簡化版：「`|Δ| >= 0.02` 時記錄為『觀察到量級差異』」。
- **算否定/不足以區分的門檻（逐字，來源：現況檔第23行）**：「否則『不足以區分』，不得稱顯著、robust、generalizes 或純海域因果」。設計文件第5節的簡化版：「`|Δ| < 0.02` 時記錄為『未觀察到值得記錄的量級差異』」。
- 落在兩者之間 → 判為「不可判定」，見第8步。
- δ=0.02 明文為**描述性**分類門檻，不是統計檢定顯著性水準，不附信賴區間或 p-value（設計文件第5節）。

---

## 後半：結案才填（7-9）

### 7. Evidence / Result（實際結果）—— 對應 Experiment 定稿

⏳ 待計算——正式主端點 event-macro Oil IoU 尚未計算。現有 scene/pooled 單點（來源：現況檔第 13 行）不得直接取代主端點。

參考數字（**非主端點，僅供對照**；來源：`.agents/institution/research/oil_spill_project_status.md` 第13行）：

| Arm | Early Stop Epoch | best Epoch | best source-Val mIoU | Val Oil IoU | Taiwan pooled Oil IoU | Taiwan scene-macro Oil IoU | Taiwan scene-macro mIoU |
|---|---:|---:|---:|---:|---:|---:|---:|
| GoM-only | 96 | 46 | 0.730333 | 0.493401 | 0.4187 | 0.3657 | 0.6498 |
| NonGoM-only | 114 | 64 | 0.653613 | 0.341589 | 0.2037 | 0.2942 | 0.6149 |

### 8. Answer / Finding（結論）—— 對應 Experiment 定稿

⏳ 待第7步（正式主端點 event-macro Oil IoU）計算完成後才能填寫，不先寫結論。

### 9. Decision / Next（下一步）—— 對應 Contribution / Experiment / RQ

⏳ 待第7、8步完成後才能填寫，不先寫結論。

---

## 實驗檔案位置

| 類型 | 絕對路徑 |
|---|---|
| 專案 repo | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand` |
| 設計決議（1-6 正本） | `.agents/decisions/20260914_P14_GoM_full-source_ablation設計.md`（vault 根目錄下） |
| 現況檔對應條目（執行狀態與參考數字） | `.agents/institution/research/oil_spill_project_status.md` 第11–26行、第332–341行 |

## 相關頁面

- [[20260913_P13海域來源敏感度_只部分回答]] — 前一項 source-group sensitivity 設計（budget-matched，非 full-source），P14 承接其提問但改用 full-source 兩臂設計
