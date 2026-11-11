---
date: 2026-10-24
type: experiment
stage: preprocessing
status: completed
tags: [p27, wakashio, optical-ratio, rajendran2021, auroc, leave-date-out, gom-source, stage2-pseudo-label, experiment-card, single-feature, descriptive, rejected]
---

# P27 Wakashio 可見光比值來源端診斷：否定其作為 Stage 2 單一優先候選

> 本頁為「實驗九步驟卡」制度（`.agents/institution/templates/experiment_card.md`）下的結案卡，只填第 7–9 步。第 1–6 步（開跑前凍結：Objective、Problem、Research Question、Hypothesis、Method、Expected Evidence／事前數字門檻）正本為 `.agents/decisions/20261024_P27_Wakashio比值來源端初步診斷設計.md`（vault 根目錄下，1–6 已凍結，本頁不重複抄錄、不修改該卡）。
>
> **範圍澄清（與第 1–6 步一致，本頁重申以避免誤讀）**：本輪比較的是「原始 GT=0 已標油」vs「原始 GT=1（LabelMe `background` polygon）」，**GT=1 尚未逐一確認為海水**，可能含雲、眩光或其他背景；本輪不是油／海水辨識實驗，不是 Stage 2 品質實驗，也不是 segmentation 實驗。任何數字都不能外推成這三者的結論。
>
> **協商狀態（如實記錄）**：草稿階段 Claude Code（Sonnet 5）提供了兩輪方法諮詢（優先用原始 GT=1 而非 SCL=6／GT=255 當負類；建議逐日期呈現並嚴格限制外推），但 Claude 未獨立驗證所有場景、產品 lineage 或結果數字。**本案未經正式 Claude×Codex 協商或雙 harness verdict 紀錄，不構成制度上的正式共識**；使用者為最終裁決者並已授權啟動與結案判定。

**實驗編號**：P27
**凍結日期**：2026-10-24（1–6，見上引設計決議）
**結案日期**：2026-10-24

---

## 前半：開跑前凍結（1-6）

見正本 `.agents/decisions/20261024_P27_Wakashio比值來源端初步診斷設計.md`（vault 根目錄下），本頁不重複抄錄。

---

## 後半：結案才填（7-9）

### 7. Evidence / Result（實際結果）

範圍：P14 GoM-only R1 `train_oil.txt`、取得日期 ≤2021 的 S2 有油 patch，`_kind=patch`。比較 raw 儲存值比值 `(B03+B04)/B02` 的 GT=0（油）vs GT=1（原始 `background` polygon）leave-date-out directional AUROC；B02 單波段同像素、同流程作描述性對照。不訓練模型、不選分類門檻。

| 指標 | Wakashio 比值 | B02 描述性對照 |
| --- | ---: | ---: |
| 有效日期／場景 | 14／14 | 14／14 |
| 日期等權 directional AUROC（主 estimand） | **0.623** | 0.601 |
| 日期 AUROC≥0.60 的比例 | **7/14（50%）** | 7/14（50%） |
| AUROC<0.50 的日期數 | 1/14 | 6/14 |

- 比值日期 bootstrap 95% 區間（描述性，日期 resample 10,000 次，seed=20261024）：**[0.523, 0.715]**。
- 唯一方向反轉日期：2020-02-12（該日期油低方向 AUROC=0.192）。
- 去重後有效像素：已標油 60,399、已標背景 358,734；**這些是像素數，不是獨立日期樣本**，不得當作 14 以外的證據量。
- B02 與比值平均差約 0.022，**無預先設計的 paired superiority test**，不能主張比值優於單波段。

來源（正本）：
- `/home/alanyh/oil_dataset/new/_euda_optical_metadata_20261017/wakashio_ratio_pilot_20261024/labeled_auc_summary.json`
- 同目錄 `per_scene_labeled_auc.csv`、`source_label_audit.csv`、`label_audit_summary.json`、`per_date_auc.png`
- 報告：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/wakashio_ratio_pilot_20261024/RESULT.md`
- P14 `train_oil.txt` SHA256：`4a5e1b9c8674a1f74be70cf5a131a161b14550ff6ec96abcfba32773116d19fb`

### 8. Answer / Finding（結論）

**否定**——但範圍限定於第 6 步事前訂的具體問題：**否定「此比值作為目前光學 Stage 2 來源端優先候選」**，不是否定所有光學特徵、不是對油／海水可分性下結論、不是對 Stage 2 偽標籤品質或 segmentation 下結論。

- 與第 6 步門檻對照：事前規則為「mean≤0.55 **或** 至多 50% 日期 AUROC≥0.60 → 否定」。本輪 50% 日期達 AUROC≥0.60（剛好踩在「至多 50%」這一側），觸發否定門檻；若只看 mean=0.623，單獨不會觸發否定（未達 0.70 支持門檻，但也高於 0.55）。**觸發否定的是日期穩定性未達標，不是均值太低。**
- 14/14 日期達最低有效日期要求（≥10），不是「不可判定」。
- 描述性觀察（不構成另一判定）：14 日期中 7 日期 AUROC>0.60、均值高於隨機（0.5），顯示此比值**不是完全沒有光譜訊號**，只是跨日期不夠穩定，且已標背景（GT=1）的水體純度尚未核實，不能排除訊號部分來自雲／非海水背景。
- B02 單波段同像素同流程的描述性 AUROC（0.601、7/14≥0.60）與比值相近，呼應第 6 步「若 B02 同等或更好，限制結論為比值未證明優於單波段」的預告限制——此處比值 mean 略高（0.623 vs 0.601）但未經 paired test，**不能宣稱比值優於 B02**。

依第 8 步模板規則，選「否定」須同時回寫現況檔③已否定路線——本頁僅記錄此條目，實際回寫見下方第 9 步與檔案位置。

### 9. Decision / Next（下一步）

- **這條路線**：**收線**——收束的是「直接用 `(B03+B04)/B02` 單一比值決定 EUDA-PLR Stage 2 strong／weak patch」這個具體做法，**不是收束整個光學特徵方向**。
- **若研究目的仍需光學物理線索，下一步的前置工作**（任一項皆尚未做，需新卡才能動）：
  1. 不看比值圖，盲審原始 RGB／annotation，把 GT=1 中的真實海水子集獨立出來（本輪的 GT=1 混有雲／眩光等風險未排除）；
  2. 核查每景 Sen2Like 反射率轉換與雲處理 lineage（本輪只測了儲存值排序，未核實物理量）；
  3. 若要評估其他文獻特徵（如 FOCI、NIR Rayleigh-corrected reflectance 等），須與本輪 B02 對照和現有 entropy 排序方法同資料比較，且仍先在 source 定規則、之後才能碰 target（台灣）GT 終點。
- **未被此實驗排除的替代解釋**：
  - 14 景樣本量仍小，日期間變異本身可能主要是雜訊而非真實物理差異；
  - GT=1 水體純度未知，若剔除雲污染日期，穩定性可能改變方向；
  - 只測了 raw 儲存值，未做反射率校正，比值的物理意義未核實。
- **下一張卡**：無（待使用者裁決是否開啟上述前置工作的新設計卡）。

---

## 實驗檔案位置

| 類型 | 絕對路徑 |
|---|---|
| 專案 repo | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand` |
| 設計決議（1-6 正本） | `.agents/decisions/20261024_P27_Wakashio比值來源端初步診斷設計.md`（vault 根目錄下） |
| 結果報告 | `analysis/wakashio_ratio_pilot_20261024/RESULT.md`（repo 內） |
| 數值正本 | `/home/alanyh/oil_dataset/new/_euda_optical_metadata_20261017/wakashio_ratio_pilot_20261024/labeled_auc_summary.json`、`per_scene_labeled_auc.csv`、`source_label_audit.csv`、`label_audit_summary.json`、`per_date_auc.png` |
| 重現腳本 | `analysis/wakashio_ratio_pilot_20261024/build_review_gallery.py`、`measure_labeled_separability.py --measure`、`plot_results.py`（repo 內） |

## 相關頁面

- [[20261011_P25_HNM背景替換在臺灣終點無效_GoM否定_全球不可判定]] — 同屬「九步驟卡」否定判定的格式參照（非主題相關，僅格式慣例）
