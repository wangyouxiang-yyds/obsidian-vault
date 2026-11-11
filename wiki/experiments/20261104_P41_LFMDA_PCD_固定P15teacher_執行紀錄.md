---
date: 2026-11-04
type: experiment
stage: evaluation
status: completed
tags: [p41, lfmda, six-band, segformer-b2, prithvi-teacher, deeplabv3, warmup-pcd, execution-record, wandb]
---

# P41 LFMDA PCD（固定 P15 Prithvi＋DeepLabV3 teacher）執行紀錄：已完成（單 seed，描述性）

> 本頁為執行紀錄。事前凍結卡見 [[20261104_P41_LFMDA_PCD_固定P15teacher設計]]（`.agents/decisions/`，FROZEN v1，SHA-256 `2194e3bc…`，**本頁未修改該卡**）；前一輪 6 波段結果見 [[20261104_P40_LFMDA_6波段_執行紀錄]]。設計、判準與評估規則不在此重抄。訓練已完成並經使用者授權讀臺灣 GT，臺灣 24 張結果見 §4.1（來源：repo `analysis/p41_prithvi_teacher_pcd_20261104/P41_RESULTS.md`）。

## 1. 實驗背景

P40 T6（DINOv2 teacher）臺灣 24 張 pooled OilIoU 0.337，低於同 student、無 teacher 的 C6 0.463；teacher 本身比學生弱（GoM val Oil 0.135、臺灣 0.239）。使用者 2026-11-04 提出組合 3：teacher 改用 baseline 架構（Prithvi＋DeepLabV3），student 維持 SegFormer-B2，做 PCD。

經 Codex 兩輪評估達成共識：組合 3 優先於「Prithvi 自蒸餾」；teacher 用固定的 P15 GoM-only Original S42 checkpoint（依 GoM val 選，不依臺灣）。

## 2. 方法架構 / Pipeline 流程

固定 P15 teacher（eval mode、不更新）＋SegFormer-B2 6 波段 student，沿用 P40 T6 的 PCD warmup 流程，teacher optimizer 與 teacher seg loss 移除；teacher 輸入用 P15 原生前處理。細節見凍結卡 §2。

## 3. 關鍵參數

| 項目 | 內容 |
|---|---|
| teacher | P15 GoM-only Original S42 `best.pt`，GoM val Oil IoU 0.514、mIoU 0.7407 |
| teacher 輸入 | 原生輸入（8-bit 量化輸入會掉到 0.437，故不用） |
| 偽標籤門檻 τ | 0.9（作者值） |
| student／訓練 | SegFormer-B2 6 波段、10,000 步、單 seed S2333 |
| 授權 launch | 2026-11-04 14:11，使用者授權；tmux `p41`；repo commit `8399df2` |
| 實際完成 | 2026-11-04 14:55（14:11–14:55，訓練 43 分，LAUNCH_EXIT=0） |
| wandb | run `P41_T3_S2333`：https://wandb.ai/tree8454-national-central-university/oil-gt-expand/runs/j829vhg3 |

## 4. 量化結果

### 4.0 前置閘門（僅 GoM val 與臺灣無標籤影像，不讀 GT）

- teacher 以 LFMDA confusion 重算 GoM val：Oil IoU 0.514、mIoU 0.7407；8-bit 量化輸入降為 0.437 → 改用原生輸入。
- 偽標籤門檻閘門（事前規則：油 precision ≥ 0.80）在 τ 0.5–0.99 全部不過（precision 0.67–0.74）。使用者裁決改用作者 τ=0.9，理由為 0.80 線無依據、改門檻幾乎不改善。

### 4.1 臺灣 24 張結果

2026-11-04 14:11–14:55 跑完（commit `8399df2`，`LAUNCH_EXIT=0`，訓練 43 分）；封存 `eval_seal.json` SHA-256 `2d923489…`；使用者授權讀 GT 後執行 `evaluate_p41.py evaluate`，得 `eval_result.json` `ffda8d4e…`。主報臺灣 24 張 pooled OilIoU；單 seed，不判定（C 組 seed 間範圍約 0.1）。

| 組／checkpoint | OilIoU | Precision | Recall |
|---|---|---|---|
| **T3 student final** | **0.4906** | 0.695 | 0.625 |
| T3 student best（step 6000） | 0.4747 | 0.676 | 0.615 |
| C6 final／best（P40，無 teacher） | 0.4628／0.4799 | 0.760／0.687 | 0.542／0.615 |
| T6 student final（P40，DINOv2 teacher） | 0.3369 | 0.800 | 0.368 |
| REF P15 S42（＝T3 teacher）／S43／S44 | 0.4888／0.4074／0.3931（平均 0.4298） | | |

- T3 − C6：final **+0.028**，best **−0.005**。

### 4.2 GoM val OilIoU（來源域）

- T3 final 0.353（峰 0.400@6000；前 5000 步起伏大，step 5000 為 0.158）；C6 final 0.376（峰 0.450）；T6 0.189。

### 4.3 機制（封存與評估後才讀）

- target 像素 97.6% 過 τ=0.9，其中約 14.7% 為油；每 1000 步 target 油格約 2.1 萬、背景格約 19.7 萬（source 油格約 0.8 萬）→ PCD 有實質油類訊號，不只對齊背景。PCD loss 0.40（step 1000）→ 0.04（step 10000）。

## 5. 觀察與結論

（單 seed，僅描述）

- 把 DINOv2 換成固定 P15 teacher 後，PCD 不再扣分：T3 0.491 vs T6 0.337；T3 final 與其 teacher（P15 S42 0.489）同級，高於 REF 三 seed 平均 0.430。
- 相對無 teacher 的 C6：final +0.028、best −0.005，都在 seed 雜訊內 → 目前看不出 PCD 本身有增益，只能說「好 teacher 下 PCD 不傷」。T3 相對 C6 是 recall 上升（0.54→0.63）、precision 下降（0.76→0.70），方向接近 teacher 的特性。
- 若要主張增益，需補 seed（T3 與 C6 同 seed 配對），並做「只對 source 做 PCD」ablation 以區分 target adaptation 與 P15→SegFormer 的 source 蒸餾（卡 §5）；補 seed 與 ablation 是否進行待使用者決定。

備註：LFMDA 線首次掛 wandb（P37–P40 皆未接），P41 為第一個有線上曲線的 LFMDA run；Claude 在封存前不看 wandb 上的 `mechanism/` 欄位（執行盲化）。

## 6. 後續改進方向

- [x] 訓練跑完並封存預測
- [x] 使用者授權後讀臺灣 GT，跑 `evaluate_p41.py`（24 張主報）
- [ ] 補 seed（T3 與 C6 同 seed 配對），待使用者決定
- [ ] 「只對 source 做 PCD」ablation，才能區分 target adaptation 與 P15→SegFormer 的 source 蒸餾，待使用者決定

## 7. 實驗檔案位置

- 程式與凍結卡草稿：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p41_prithvi_teacher_pcd_20261104/`
- 結果彙整：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p41_prithvi_teacher_pcd_20261104/P41_RESULTS.md`
- 輸出資料夾：`/home/alanyh/oil_dataset/new/_p41_prithvi_teacher_20261104/`（閘門輸出在其下 `gate_v1_downward/`、`gate/`）
- 凍結卡：`/home/alanyh/obsidian-vault/.agents/decisions/20261104_P41_LFMDA_PCD_固定P15teacher設計.md`

## 相關頁面

- [[20261104_P41_LFMDA_PCD_固定P15teacher設計]]
- [[20261104_P40_LFMDA_6波段_執行紀錄]]
