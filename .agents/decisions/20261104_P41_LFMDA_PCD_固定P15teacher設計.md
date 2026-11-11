---
experiment: P41
title: LFMDA PCD warmup，teacher 換成固定 P15 Prithvi＋DeepLabV3（單 seed 探索）
date: 2026-11-04
status: FROZEN v1（2026-11-04，launch 前）
type: exploratory, descriptive only（無 H1 判定）
---

# P41：固定 P15 Prithvi＋DLV3 teacher × SegFormer-B2 student 的 PCD（單 seed 探索）

## 1. 問題與動機
P40 T6（DINOv2 teacher）臺灣 24 張 0.337 < C6（同 student、無 teacher）0.463；teacher 本身 GoM val Oil 0.135、臺灣 0.239，比學生弱。使用者 11-04 提組合 3：teacher 改用我們的 baseline 架構 Prithvi-EO-V2-300M＋DeepLabV3，student 維持 SegFormer-B2，做 PCD。與 Codex（session `01a1a4df-5799-7e52-a46c-47a09bde12cc`，兩輪評估）共識：組合 3 優先於「Prithvi 自蒸餾」，teacher 用固定 P15 checkpoint。使用者 11-04 授權：τ=0.9、單 seed、開跑、訓練掛 wandb。

## 2. 設定（相對 P40 T6：teacher 換成固定的 source-trained P15，並因此移除 teacher optimizer 與 teacher seg loss；T6 只作背景參照，非單因子比較）
- **teacher**：P15 GoM-only Original S42 `best.pt`（`11130fe9…`，args `cc320342…`）。依 GoM val mIoU 在 S42/43/44 中選（0.7407／0.7297／0.7301）；S42 也是臺灣最高，但選擇只依 GoM val。全程 eval mode、no grad、不更新（無 teacher optimizer、無 teacher seg loss；訓練前後 state hash 相同）。
- **teacher 輸入**：P15 原生前處理（8 slot、DN/10000、無效值→0、P15 args mean/std Normalize，無 8-bit 量化），與 student 視圖共用同一次幾何增強（source：同一次 albumentations 呼叫的 additional target；target：同一組翻轉抽樣）。
- **PCD 特徵**：Prithvi 第 23 層（terminal norm 後，1024-d、16×16）→ bilinear 到 student 8×8（作者做法）；偽標籤 = teacher softmax（單一 head；作者兩 head 平均的式子保留，兩者相同）。τ=0.9。
- **原型初始化**：作者 init_prototypes_dg 流程，改用 P15 teacher 特徵（342 batch）。
- **student／訓練**：與 P40 T6 相同——SegFormer-B2 6 波段（B04/B03/B02/B8A/B11/B12，C6 前處理）、10,000 步、source batch 8＋target batch 4、CE、AdamW 6e-5、poly、target clamp(max=1)、KD T=0.3、原型 EMA 0.99、S2333。
- **wandb**（init 失敗即在訓練前中止；之後的上傳錯誤只警告）：專案 `oil-gt-expand`，run `P41_T3_S2333`、group `P41_T3`。每 50 步：train seg loss、lr、PCD loss；每 1000 步：GoM val（mIoU／Oil IoU／P／R／F1）＋target 機制量（`mechanism/`：target 高信心比例、高信心油比例、PCD 格數）。train 模式要求線上 wandb（WANDB_MODE=disabled/offline 即中止）。終端機只印 source 值；Claude 不看 wandb 上的 `mechanism/`，直到封存（執行盲化）。

## 3. 前置閘門（僅 GoM val 與臺灣無標籤影像，不讀 GT）
- `gate41.py`：teacher 以 LFMDA confusion 重算 GoM val = Oil IoU 0.514、mIoU 0.7407（與主線紀錄一致 → 與 C6 數字同口徑）；8-bit 量化輸入降為 0.437 → 用原生輸入。
- τ 規則（事前寫定 precision ≥0.80、coverage ≥0.20）：往下格（0.9–0.5）precision 0.67–0.71、往上格（0.95/0.97/0.99）0.72–0.74，皆不過；PCD 格層級（32×32、75%）precision 0.64–0.67。P15 的錯誤多為高信心錯（Tversky 校準；與已知線寬偏胖一致）。**使用者 11-04 裁決用作者 0.9**（0.80 線無依據；改門檻幾乎不改善）。臺灣無標籤 coverage 檢查因規則停在 τ 選擇而未跑。
- 輸出：`_p41_prithvi_teacher_20261104/gate_v1_downward/`、`gate/`。

## 4. 評估（描述性）
- 主報臺灣 24 張 pooled OilIoU（使用者 11-04：只看 24 張）；student final 主、best 次；final 對 final、best 對 best。
- 對照（取自封存 P40 `eval_result.json` `be9ea4e2…`）：C6 final 0.463／best 0.480；T6 0.337；REF P15 S42/43/44 0.489/0.407/0.393（平均 0.430；T3 teacher = REF_S42）。
- 執行盲化：封存前不看臺灣預測、不開 GT、不看 mechanism log／wandb `mechanism/`；讀 GT 需使用者另授權。

## 5. 已知限制
- 單 seed；C 組 seed 間 24 張範圍約 0.1。
- teacher 偽標籤 GoM val 油類 precision 約 0.71（約三成錯），且臺灣上 teacher 與 C6 同級（0.489 vs 0.463），不是「強老師教弱學生」。
- student 初始化與 source 批次順序的 RNG 消耗與 C6、T6 不同（teacher 不再於 student 之前隨機初始化；P38/P40 已有同類差異）。
- 若 T3 > C6，仍需「只對 source 做 PCD」的 ablation 才能區分 target adaptation 與 P15→SegFormer 的 source 蒸餾。
- 固定 teacher、移除 teacher seg loss：屬 LFMDA-inspired，不是原樣 LFMDA。

## 6. 程式與 smoke（不讀 GT）
- `teacher41.py`（固定 teacher、原生輸入）、`data41.py`（雙視圖 source/target）、`warmup41.py`（P40 warmup6 改寫＋wandb）、`predict41.py`、`evaluate_p41.py`（沿用 P40 evaluator 與 PINNED）、`gate41.py`、`test_p41.py`、`launch.sh`。
- `test_p41`：student 視圖與 `RGBSource` 前 48 筆逐位元相同、teacher 視圖幾何對齊（由 teacher 視圖重建 student 張量 atol 1e-4）；teacher 輸出 = P15 `model(x)['out']`、特徵 1×1024×16×16、恆為 eval、無可訓練參數——通過。
- smoke（wandb 關）：2 步，seg 20.27、PCD 0.636，PCD 梯度到達 mapper，teacher 無梯度且 state 不變，峰值顯存 4.6 GB，init 原型 342 batch、cells 28,201／653,991。
- 成本：約 1 小時（P40 T6 約 70 分；teacher 不需反傳）。

## 7. 共識（Codex gpt-5.6-sol／high，session `01a1a4df-5799-7e52-a46c-47a09bde12cc`）
- E1/E2：方向評估（組合 1／2，再組合 3）→ 組合 3 優先、固定 P15 teacher、原生輸入、τ 先過 GoM val 閘門。
- R1 REVISE：blocking「predict41 `[0,0]` 取到 tuple」——查證不成立（eval 回傳 softmax，與 P40 predict6 同），補 `test_student_eval_output_is_probability`，R2 撤銷。non-blocking 採：wandb init 失敗中止、manifest 封存 gate sha 與 τ 人工裁決、evaluator 檢查 schema/threshold/TTA、卡片措辭。
- R2 REVISE：blocking「WANDB_MODE=disabled 時 train 會靜默無 wandb」→ 已改為 train 模式遇 disabled/offline 即中止（照 Codex 指定修法，未再送 R3）。
