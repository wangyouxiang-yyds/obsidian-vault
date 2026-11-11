---
date: 2026-10-15
type: experiment
stage: supervised
status: completed
tags: [dads-sam, gom-only, taiwan-holdout, public-recipe, training-recipe, three-seed, experiment-card, descriptive, below-range]
---

# DADS-SAM B × 公開碼訓練設定（三個 seed）：臺灣像素加總 Oil IoU 0.3043 ± 0.1071，三個 seed 都低於 B 的範圍

> 本頁為「實驗九步驟卡」制度（`.agents/institution/templates/experiment_card.md`）下的結案卡，只填第 7–9 步。第 1–6 步（開跑前凍結：目的、問題、做什麼、開跑前檢查、怎麼報告、限制）正本為兩份已凍結的事前紀錄（vault 根目錄下），本頁不重複抄錄、不修改它們：
> - seed 42：`.agents/decisions/20261013_DADS_SAM_B_公開碼訓練設定_設計.md`
> - seed 43／44（含三個 seed 的報告方式）：`.agents/decisions/20261014_DADS_SAM_B_公開碼訓練設定_seed43_44_設計.md`
>
> seed 42 的單獨結案卡已存在，原樣保留、不修改：[[20261014_DADS_SAM_B_公開碼訓練設定_臺灣低於B三seed範圍]]。本頁補入 seed 43／44，並依事前紀錄的寫法彙整三個 seed。

**實驗編號**：無 P 編號（DADS-SAM B × 公開碼訓練設定）
**凍結日期**：2026-10-13（seed 42 的 1–6）；2026-10-14 11:18（seed 43／44 的 1–6），見上引兩份事前紀錄
**結案日期**：2026-10-15
**事後補寫**：否
**取代哪張卡**：無（seed 42 結案卡原樣保留；本卡補入 seed 43／44 並彙整三個 seed）
**協商狀態**：未經 Claude × Codex 協商

跑的是：B（公開碼架構、六波段輸入、固定整張圖的 box prompt），訓練設定換成公開碼 `train_port.py` 的（0.7×DiceCE＋0.3×Focal、Adam 學習率 1e-4 固定、batch size 2、固定 100 epoch、取最後一個 epoch 權重、無資料增強、無 gradient clipping、fp32；九項同時改，項目見事前紀錄）。三個 seed：42、43、44。事前紀錄不訂判定門檻，結果只作描述。

---

## 前半：開跑前凍結（1-6）

見正本 `.agents/decisions/20261013_DADS_SAM_B_公開碼訓練設定_設計.md`（seed 42）與 `.agents/decisions/20261014_DADS_SAM_B_公開碼訓練設定_seed43_44_設計.md`（seed 43／44），本頁不重複抄錄。repo 內事前紀錄為 `analysis/dads_sam_gomonly_20261005/B_PUBLIC_RECIPE.md`（commit `d2759a3`）與 `analysis/dads_sam_gomonly_20261005/B_PUBLIC_RECIPE_SEEDS.md`（commit `836e33c`）。

---

## 後半：結案才填（7-9）

### 7. Evidence / Result（實際結果）

來源：各 run 目錄內的 `results.csv`、`run_record.json`、`run_metrics.json`，以及臺灣 prediction manifest（路徑見文末）。臺灣每個 run 只評一次。

**流程與完整性**

- seed 42：2026-10-13 21:21 啟動、10-14 09:16 結束（細節見 seed 42 結案卡）。
- seed 43／44：設定檔 `main/experiments_DADS_SAM_gomonly_B_public_recipe_seeds.yaml`，commit `836e33c`，同一個 main_runner 依序 S43 → S44；2026-10-14 11:18 → 10-15 11:12，整個序列 `EXIT=0`。
- 訓練時間：S42 714.5 分鐘、S43 713.1 分鐘、S44 718.8 分鐘；三個 run 都跑滿 100 epoch（不早停）。
- 三個 run 的驗收項目全數通過：`results.csv` 100 列皆為有限值；學習率全程 1e-4；`run_record` status trained；臺灣 9/9 景 prediction manifest complete；臺灣測試用權重是 `last.pt`（第 100 epoch），manifest 記錄的 sha256 為 S42 `bf5ba919…`、S43 `2a06c285…`、S44 `194a995c…`。

**主數字（每個 run 臺灣 9 景像素加總算一次，再跨 seed 取平均 ± 樣本標準差，ddof=1）**

| 指標 | S42 | S43 | S44 | 平均 ± 標準差 |
|---|---:|---:|---:|---:|
| **Oil IoU** | 0.1807 | 0.3709 | 0.3614 | **0.3043 ± 0.1071** |
| Precision | 0.4274 | 0.7388 | 0.6175 | 0.5946 ± 0.1570 |
| Recall | 0.2385 | 0.4268 | 0.4656 | 0.3770 ± 0.1215 |
| F1 | 0.3061 | 0.5411 | 0.5309 | 0.4594 ± 0.1328 |
| BG IoU | 0.9150 | 0.9422 | 0.9341 | 0.9304 ± 0.0140 |
| mIoU | 0.5478 | 0.6565 | 0.6477 | 0.6174 ± 0.0604 |
| 9 景各算再平均 Oil IoU | 0.2761 | 0.4052 | 0.3872 | 0.3562 ± 0.0699 |

混淆矩陣（像素）：

| seed | TP | FP | FN | TN |
|---|---:|---:|---:|---:|
| S42 | 72,416 | 97,019 | 231,260 | 3,532,183 |
| S43 | 129,614 | 45,821 | 174,062 | 3,583,381 |
| S44 | 141,399 | 87,592 | 162,277 | 3,541,610 |

**對照（同為臺灣 9 景像素加總，同一算法）**

| 組 | 各 seed 的 Oil IoU | 平均 ± 標準差 | Precision | Recall | F1 | BG IoU | mIoU |
|---|---|---:|---:|---:|---:|---:|---:|
| 本卡（B × 公開碼訓練設定） | 0.1807／0.3709／0.3614（seed 42／43／44） | **0.3043 ± 0.1071** | 0.5946 | 0.3770 | 0.4594 | 0.9304 | 0.6174 |
| 正式 B（我們的訓練設定，與基線相同：Tversky、AdamW、batch 16、早停、取驗證最佳） | 0.4434／0.4695／0.4355 | 0.4495 ± 0.0178 | 0.5695 | 0.6825 | 0.6201 | 0.9319 | 0.6907 |
| 基線 P15 Original（Prithvi＋DeepLabV3，GoM-only） | 0.4808／0.4379／0.3704 | 0.4297 ± 0.0557 | 0.5473 | 0.6718 | 0.5997 | 0.9278 | 0.6788 |

**GoM 驗證（不用來選權重；最佳 epoch 的權重沒有拿去測臺灣）**

| seed | 最後一個 epoch（測臺灣用的權重）驗證 mIoU | 最後一個 epoch 驗證 Oil IoU | 最佳 epoch | 最佳 epoch 驗證 Oil IoU |
|---|---:|---:|---:|---:|
| S42 | 0.6271 | 0.2964 | 43 | 0.4333 |
| S43 | 0.6510 | 0.3391 | 27 | 0.4350 |
| S44 | 0.6866 | 0.4100 | 12 | 0.4237 |

對照：正式 B 三個 seed 在各自最佳 epoch 的驗證 Oil IoU 為 0.508／0.523／0.482。

**逐景 Oil IoU（臺灣 9 景；S42 取自 seed 42 結案卡）**

| 事件 | 景 | S42 | S43 | S44 |
|---|---|---:|---:|---:|
| 天使輪 | 0801_L9_50QRK | 0.081 | 0.260 | 0.335 |
| 天使輪 | 0801_L9_50QRL | 0.067 | 0.173 | 0.163 |
| 天使輪 | 0912_S2_50QRL | 0.535 | 0.385 | 0.425 |
| 天使輪 | 0918_L9_50QRK | 0.521 | 0.622 | 0.610 |
| 天使輪 | 0918_L9_50QRL | 0.574 | 0.653 | 0.655 |
| 天使輪 | 0922_S2_50QRK | 0.268 | 0.455 | 0.328 |
| 天使輪 | 0922_S2_50QRL | 0.260 | 0.438 | 0.290 |
| 05-22 | 0522_L8_51QTD | 0.109 | 0.331 | 0.336 |
| 05-22 | 0522_L8_51QUD | 0.071 | 0.330 | 0.343 |

**wandb 紀錄**

- S43：run `lkh4hy25`；S44：run `7lgnkgb0`；專案 `oil-gt-expand`。兩個 run 都已 finished，history 各 100 列，與 `results.csv` 相同，summary 內有臺灣 recon 指標。seed 42 沒有 wandb 紀錄（當時尚未導入）。
- 已知小問題（未修，待使用者決定）：訓練期間 wandb 曲線比 `results.csv` 慢一個 epoch，原因是 `main/wandb_logging.py` 的 `run.log(row, step=epoch)` 沒給 `commit=True`；最後一列在 run 結束時補寫，沒有資料遺失。

### 8. Answer / Finding（結論）

**三個 seed 都低於 B 的範圍**（B 三個 seed 的 Oil IoU 範圍 0.4355–0.4695；本卡三個 seed 為 0.1807／0.3709／0.3614，平均 0.3043 ± 0.1071）。依事前紀錄的寫法，不訂判定門檻，只作描述。10-10 凍結卡對 B 的判定不因本卡更改。

與基線 P15 Original 的範圍（0.3704–0.4808）相比：只有 S43（0.3709）落在範圍內，僅高於基線最低值 0.3704 約 0.0005；S42（0.1807）、S44（0.3614）低於基線最低值。

**觀察（描述性）**
- 三個 seed 的召回率都低於 B 的平均（0.24／0.43／0.47 對 0.68）；S43、S44 的 precision 高於 B 的平均（0.74／0.62 對 0.57）。
- seed 間差異大：Oil IoU 標準差 0.107，正式 B 為 0.018、基線為 0.056。
- seed 間差異主要在 05-22 兩景：S42 召回率 0.13／0.07，S43／S44 為 0.35–0.42；這兩景的 Oil IoU 由 S42 的 0.109／0.071 變成 S43／S44 的 0.33–0.34。

**不可主張**
- 差異來自 loss、batch size、optimizer、資料增強、取最後權重中的哪一項：九項訓練設定同時改，無法歸因。
- 「公開碼的方法本身比較差」：prompt 與公開碼不同（本卡用固定整張圖的 box；公開碼用標註外接框，那在測試時等於把答案給模型，不能用）、資料是 GoM 衛星影像而非公開碼的無人機 RGB。

**缺什麼 / 限制**
- 不是盲測：seed 42 的結果（0.1807）在 seed 43／44 開跑前已看過。
- 臺灣只有 9 景、兩個事件。
- 三個 seed 的標準差本身不穩（n=3），平均 ± 標準差只是描述。
- 未經 Claude × Codex 協商。

### 9. Decision / Next（下一步）

- **這條路線**（DADS-SAM 線續跑或收線）：未填（待使用者）。
- **投影片寫法**：未填（待使用者）。下列是建議句，不是已決定：「照公開程式碼的訓練設定（只改輸入與輸出）時，三個 seed 平均 Oil IoU 0.30 ± 0.11，三個都低於改用 baseline 訓練設定的 DADS-SAM（0.45 ± 0.02）；因此本研究的 DADS-SAM 採用與 baseline 相同的訓練設定。prompt 與公開碼不同。」
- **wandb 曲線慢一個 epoch**：是否修 `main/wandb_logging.py`：未填（待使用者）。
- **未被此實驗排除的替代解釋**：
  - 九項訓練設定同時改，無法歸因。
  - prompt 與資料和公開碼不同（固定整張圖的 box 對標註外接框；GoM 衛星影像對無人機 RGB）。
  - seed 間差異大（尤其 05-22 兩景），n=3 對這個變異量的刻畫有限。
- **下一張卡**：未填（待使用者）。
- **排程狀態（非判定內容）**：GPU 已交給排隊中的馬祖推論（Codex），P24 Original S44 排在其後；DADS session 已不佔 GPU。

---

## 實驗檔案位置

| 類型 | 絕對路徑 |
|---|---|
| 專案 repo | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand` |
| 設計決議（seed 42 的 1-6 正本） | `.agents/decisions/20261013_DADS_SAM_B_公開碼訓練設定_設計.md`（vault 根目錄下） |
| 設計決議（seed 43／44 的 1-6 正本） | `.agents/decisions/20261014_DADS_SAM_B_公開碼訓練設定_seed43_44_設計.md`（vault 根目錄下） |
| repo 內事前紀錄 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/dads_sam_gomonly_20261005/B_PUBLIC_RECIPE.md`（commit `d2759a3`）、`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/dads_sam_gomonly_20261005/B_PUBLIC_RECIPE_SEEDS.md`（commit `836e33c`） |
| 設定檔（seed 42） | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/main/experiments_DADS_SAM_gomonly_B_public_recipe.yaml` |
| 設定檔（seed 43／44） | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/main/experiments_DADS_SAM_gomonly_B_public_recipe_seeds.yaml` |
| 結果目錄（S42） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_public_recipe/` |
| 結果目錄（S43、S44） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_public_recipe_seeds/` |
| run 目錄（S43） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_public_recipe_seeds/20261014-111842_MS6_DADS_SAM_B_public6_GoM_PubRecipe_S43/` |
| run 目錄（S44） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_public_recipe_seeds/20261014-231231_MS6_DADS_SAM_B_public6_GoM_PubRecipe_S44/` |
| 序列 log（S43／S44，結束時末行 `EXIT=`） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_public_recipe_seeds_main_runner.log` |
| institution 現況檔回寫 | institution commit `18630f5`（S43 完成）、`ed8eb66`（三個 seed 完成） |

## 相關頁面

- [[20261014_DADS_SAM_B_公開碼訓練設定_臺灣低於B三seed範圍]] — seed 42 的單獨結案卡（本卡補入 seed 43／44）
- [[20261011_DADS_SAM_GoM_only_正式C與B_皆不可判定]] — 正式 B（seed 42）與 C 的 10-10 凍結卡結案；本卡以其 B 設定為底
