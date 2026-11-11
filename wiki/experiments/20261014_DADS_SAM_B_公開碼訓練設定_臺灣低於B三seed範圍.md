---
date: 2026-10-14
type: experiment
stage: supervised
status: completed
tags: [dads-sam, gom-only, taiwan-holdout, public-recipe, training-recipe, single-seed, experiment-card, descriptive, below-range]
---

# DADS-SAM B × 公開碼訓練設定（seed 42）：臺灣像素加總 Oil IoU 0.1807，低於 B 三個 seed 的範圍

> 本頁為「實驗九步驟卡」制度（`.agents/institution/templates/experiment_card.md`）下的結案卡，只填第 7–9 步。第 1–6 步（開跑前凍結：目的、問題、做什麼、開跑前檢查、怎麼報告、限制）正本為 `.agents/decisions/20261013_DADS_SAM_B_公開碼訓練設定_設計.md`（vault 根目錄下，1–6 已凍結，本頁不重複抄錄、不修改該卡）。

**實驗編號**：無 P 編號（DADS-SAM B × 公開碼訓練設定）
**凍結日期**：2026-10-13（1–6，見上引設計決議）
**結案日期**：2026-10-14
**事後補寫**：否
**取代哪張卡**：無
**協商狀態**：未經 Claude × Codex 協商

本 run 以正式 B（`MS6_DADS_SAM_B_public6_GoM_R1`，seed 42）的設定為底，只把訓練設定換成公開碼的（九項同時改，項目見事前紀錄）。單一 seed 42；事前紀錄不訂判定門檻，結果只作描述。

---

## 前半：開跑前凍結（1-6）

見正本 `.agents/decisions/20261013_DADS_SAM_B_公開碼訓練設定_設計.md`（vault 根目錄下），本頁不重複抄錄。repo 內事前紀錄為 `analysis/dads_sam_gomonly_20261005/B_PUBLIC_RECIPE.md`（commit `d2759a3`）。

---

## 後半：結案才填（7-9）

### 7. Evidence / Result（實際結果）

來源：run 目錄內的 `results.csv`、`run_record.json`，以及臺灣 prediction manifest（路徑見文末）。臺灣每個 run 只評一次。

**流程與完整性**

- 2026-10-13 21:21 啟動、10-14 09:15 進臺灣測試、09:16 結束；整個序列 `EXIT=0`。
- 跑滿 100 epoch（不早停），訓練 714.5 分鐘。
- `results.csv` 100 列皆為有限值，學習率全程 1e-4。
- `run_record` status trained、error null；launch 時 git_head `d2759a3`、launch gate PASS。
- 臺灣 9/9 景；prediction manifest complete（ok 9／failed 0／skipped 0）。
- 測試用權重是 `last.pt`（第 100 epoch）；manifest 記錄的 model path 與 sha256（`bf5ba919…`）就是 `last.pt`。

**主數字（臺灣 9 景像素加總算一次）**

| Oil IoU | Precision | Recall | F1 | BG IoU | mIoU |
|---:|---:|---:|---:|---:|---:|
| **0.1807** | 0.4274 | 0.2385 | 0.3061 | 0.9150 | 0.5478 |

9 景各算再平均：0.2761。

混淆矩陣（像素）：

| TP | FP | FN | TN |
|---:|---:|---:|---:|
| 72,416 | 97,019 | 231,260 | 3,532,183 |

**對照（同為臺灣 9 景像素加總 Oil IoU）**

| 組 | 各 seed 的 Oil IoU | 平均 |
|---|---|---:|
| 本 run（B × 公開碼訓練設定，單一 seed） | 0.1807（seed 42） | — |
| 正式 B（三個 seed） | 0.4434／0.4695／0.4355 | 0.4495 |
| 基線 P15 Original（三個 seed） | 0.4808／0.4379／0.3704 | 0.4297 |

**GoM 驗證（不用來選權重；最佳 epoch 的權重沒有拿去測臺灣）**

| 項目 | epoch | 驗證 mIoU | 驗證 Oil IoU |
|---|---:|---:|---:|
| 最後一個 epoch（測臺灣用的權重） | 100 | 0.6271 | 0.2964 |
| 最佳 epoch | 43 | 0.6985 | 0.4333 |

對照：正式 B 三個 seed 在各自最佳 epoch 的驗證 Oil IoU 為 0.508／0.523／0.482。

**訓練過程**

- 訓練 loss：前 10 個 epoch 平均 0.40、最後 10 個 epoch 平均 0.16。
- 驗證 Oil IoU：第 41–50 epoch 平均 0.368、最後 10 個 epoch 平均 0.334（範圍 0.296–0.368）。

**逐景 Oil IoU／Precision／Recall**

| 事件 | 景 | Oil IoU | Precision | Recall |
|---|---|---:|---:|---:|
| 天使輪 | 0801_L9_50QRK | 0.081 | 0.119 | 0.204 |
| 天使輪 | 0801_L9_50QRL | 0.067 | 0.117 | 0.134 |
| 天使輪 | 0912_S2_50QRL | 0.535 | 0.660 | 0.738 |
| 天使輪 | 0918_L9_50QRK | 0.521 | 0.534 | 0.957 |
| 天使輪 | 0918_L9_50QRL | 0.574 | 0.602 | 0.924 |
| 天使輪 | 0922_S2_50QRK | 0.268 | 0.273 | 0.931 |
| 天使輪 | 0922_S2_50QRL | 0.260 | 0.263 | 0.958 |
| 05-22 | 0522_L8_51QTD | 0.109 | 0.443 | 0.127 |
| 05-22 | 0522_L8_51QUD | 0.071 | 0.779 | 0.072 |

### 8. Answer / Finding（結論）

**低於 B 三個 seed 的範圍（0.4355–0.4695）**；單一 seed，描述性。本 run 的 0.1807 也低於基線 P15 Original 三個 seed 的範圍（0.3704–0.4808）。依事前紀錄的寫法，不訂判定門檻，只作描述。10-10 凍結卡對 B 的判定不因本 run 更改。

**觀察（描述性）**
- 05-22 兩景約占全部油污像素的 80%，這兩景的召回率只有 0.127／0.072，是像素加總偏低的主因。
- 0801 兩景也低（正式 B 三個 seed 在這兩景為 0.18–0.41）。
- 0912、0918、0922 五景與正式 B 相近或略高。

**不可主張**
- 差異來自 loss、batch size、optimizer、資料增強、取最後權重中的哪一項：九項訓練設定同時改，無法歸因。
- 「公開碼的方法本身比較差」：prompt 與公開碼不同（本 run 用固定整張圖的 box，公開碼用標註外接框）、資料是 GoM 衛星影像而非公開碼的無人機 RGB、且只有一個 seed。

**缺什麼 / 限制**
- 單一 seed。
- B 的臺灣結果事前已看過，不是盲測。
- 臺灣只有 9 景、兩個事件。
- 未經 Claude × Codex 協商。

### 9. Decision / Next（下一步）

- **這條路線**（續跑／收線／改設計）：未填（待使用者）。
- **是否補 seed 43／44**：未填（待使用者）。
- **未被此實驗排除的替代解釋**：
  - 單一 seed 的雜訊。
  - 九項訓練設定同時改，無法歸因。
  - prompt 與資料和公開碼不同（固定整張圖的 box 對標註外接框；GoM 衛星影像對無人機 RGB）。
- **下一張卡**：未填（待使用者）。
- **排程狀態（非判定內容）**：GPU 已釋放，沒有 main_runner 在跑。

---

## 實驗檔案位置

| 類型 | 絕對路徑 |
|---|---|
| 專案 repo | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand` |
| 設計決議（1-6 正本） | `.agents/decisions/20261013_DADS_SAM_B_公開碼訓練設定_設計.md`（vault 根目錄下） |
| repo 內事前紀錄 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/dads_sam_gomonly_20261005/B_PUBLIC_RECIPE.md`（commit `d2759a3`） |
| 設定檔 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/main/experiments_DADS_SAM_gomonly_B_public_recipe.yaml` |
| run 目錄 | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_public_recipe/20261013-212124_MS6_DADS_SAM_B_public6_GoM_PubRecipe_S42/` |
| 序列 log（結束時末行 `EXIT=`） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_public_recipe_main_runner.log` |

## 相關頁面

- [[20261011_DADS_SAM_GoM_only_正式C與B_皆不可判定]] — 正式 B（seed 42）與 C 的 10-10 凍結卡結案；本 run 以其 B 設定為底
