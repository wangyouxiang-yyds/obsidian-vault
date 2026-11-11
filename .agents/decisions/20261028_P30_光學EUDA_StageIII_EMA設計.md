# P30 光學 EUDA Stage III：EMA 線上 argmax 基準設計

**實驗編號**：P30　**凍結日期**：2026-10-28 20:12 Asia/Taipei　**狀態**：FROZEN；第 1–6 步已於正式訓練前凍結（開跑前凍結，非事後補寫）。
**事後補寫?**：否；目前沒有 Stage III 訓練或 target 評估結果。
**取代哪張卡**：無。
**授權與協商**：方法共識見 repo `analysis/euda_stage3_20261028/claude_round3_response.md`（Claude Opus 5／high 與 Codex 三輪收斂，R3 ACCEPT、BLOCKING=[]）；程式核對見 `claude_code_check_round2_response.md`（ACCEPT）；正式 launch review 見 `claude_launch_review_response.md`（ACCEPT，無 blocking）。使用者已明示要求掛 tmux 開始本次 Stage III 實驗；本卡凍結時正式訓練尚未啟動。
**程式與設定正本**：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/euda_stage3_20261028/`（`CARD.md`、`README.md`、`build_inputs.py`、`main/euda_stage3.py`）；正式設定 `main/experiments_EUDA_stage3_20261028_run.yaml`。本卡第 1–6 步取自源卡 `CARD.md` 的「1. Objective」至「6. Expected evidence」，逐項翻譯但不改動方法、estimand 或任何數字；源卡首行原為「Freeze status: PRELAUNCH CANDIDATE; must be copied and frozen in vault before launch」，**本卡即為該凍結動作**，非事後補寫。精確的最終 YAML SHA-256 與訓練時 repo Git commit，由啟動後的 launch receipt 另行記錄，本卡不猜測、不代填。後續啟用 run 旗標、改變卡片路徑或 launch 狀態，不得更動本卡第 1–6 步鎖定的科學設定；如需變動方法、資料或第 6 步門檻，須另立新版本卡並標記取代關係。

## 1. Objective

檢驗已實作完成的 target-only Stage III 調適套件，能否改善固定臺灣歷史重建終點相對其 S42 起點的表現。

## 2. Problem

光學強／弱知識規則尚未建立。目前已有一份凍結的 IntraDA entropy 排序與一個已完成的 Stage I 模型。本實驗是探索性工程／科學基準，**不是**完整重現 Cui et al. 的 persistent PLR 方法。

## 3. Research Question

在 S42 基礎上加入固定預算的 target-only EMA easy CE／hard U 對抗調適，是否能在同一 19 張臺灣影像上改善歷史 per-image macro Oil IoU（相對 frozen S42）？

**已否定路線檢查**：全域 256-window entropy 排序中，easy=18,482 個視窗僅含 6 個模型預測為油的視窗；hard=9,104 個視窗含 2,426 個。低 entropy 不等於已驗證的油偽標籤可靠性。不得在看過 target GT 後回頭改選取規則。P27（Wakashio 比值）、P28（B08-dip）兩個光學物理 proxy 皆未獲支持，本案不重用這兩個 proxy；本案是完整 EMA CE／對抗套件的第一次訓練測試，不是重跑這兩個 proxy。

## 4. Hypothesis

Easy CE 與 easy-to-hard 不確定度對齊可能改善固定終點；背景崩潰（background collapse）是可能的失敗模式。結果可能不可判定。單一 S42 無法建立跨 seed 穩健性或未見事件的泛化。

## 5. Method／對照／estimand

**初始化／對照**：Stage I `MS6_EUDA_S1_uncertainty_adv_S42/best.pt`，SHA-256 `140f1d86e1783c084ae55427e42d31c02b9f92b789b27f93d25295fe20c1bdf3`。

**架構**：既有 Prithvi EO V2 300M + DeepLabV3，六個活躍波段，凍結 Stage I 的數值解碼 `DN/10000` 與 mean/std。Student／teacher 為同一模型。

**Stage II（沿用既有、不重建）**：既有全域 entropy 排序，256 視窗單位，λ=.67；`target_split.tsv` SHA-256 `23e1f23f46536ad6ab28548ee65a8c6b477e5dc103b593d8cc8442fc79a48c07`。

**Stage III**：
- target-only easy：線上 teacher argmax 監督 CE（無效像素 ignore=255）；hard：U 對抗權重 .001。無 source loss、無 tau、無 class balancing、無 oil pool。
- Teacher 參數 EMA alpha=.99；**只更新參數，buffers 固定在 S42 初始值**（不隨訓練改變）。
- D：全新單通道五層 FC 判別器，FP32。不自行發明未知的 persistent PLR 記憶規則。
- **Batch：easy 16＋hard 16**，不做梯度累積，序列前向（sequential forward）。
- 正式預算：**5,000 次成功 student 更新**，每 100 step 一個記錄週期；easy 累計接受 80,000 次抽樣、hard 累計接受 80,000 次抽樣，另外單獨記錄 skip 次數。Seed 42；群組採「均勻、可取代」抽樣。Optimizer：AdamW，encoder 5e-6／head 5e-5，weight decay .001，cosine，eta_min=5e-8；D 用 Adam 1e-4，betas(.9,.99)。每 500 次成功 step 存一次完整狀態；最終選用的 checkpoint 是預算打滿時的 **last student**。不得用臺灣 GT 或 source val 做停止或 checkpoint／分組選擇。
- 非有限 loss 或非 AMP 梯度算工程失敗，不算科學上的否定結果。AMP overflow 時 scaler 退讓、不推進 student／D／EMA／scheduler，並還原 student buffers；forward 已消耗的 RNG 不還原。空監督的 attempt 在 forward 前就 skip。連續被 skip 的 attempt 上限 100 次、總 attempt 上限為預算的 10 倍；未跑滿預算時回報為不完整，不得當作已完成的實驗計分。
- 訓練入口：`main/main_runner.py`，走共用 GPU lock 與 clean Git gate；tmux session 名稱 `euda_stage3_s42_20261028`。沿用既有 W&B 與三分頁 workbook。

**固定終點（只在訓練完成後執行）**：
- 固定 **19 張影像、270 個既有 GT-aware 視窗**複製為 `taiwan19_test.txt`；名單須與 image-only 調適時所用的名冊一致。確切清單的 SHA 記在 YAML `_endpoint.test_txt_sha256`。訓練或預檢期間都未開啟任何 target mask。
- 以 frozen S42 與固定後的 last student 分別跑**同一套**既有重建流程：FP32 三視角 mean-logit TTA、argmax、256px、last-write-wins、相同覆蓋範圍。產出逐像素精確預測；缺漏或跳過影像視為終點失敗。
- 主指標：`_kind=recon(gt_aware)`，**19 張的 per-image macro Oil IoU**，沿用歷史「raw255→背景」慣例。次要：pooled Oil IoU／P／R／F1、逐影像配對差值；另列同覆蓋範圍下只用 raw 0/1/2 已標像素的 known-label-only 指標、預測油面積、未知 255 區域的預測油面積、已標油覆蓋比例。
- 歷史口徑的 IoU／precision 因 GT255 屬未知而**不是真正的全景精度**；known-label precision 也只限已標註覆蓋範圍。這是**非盲、transductive** 評估：這些 target 影像在調適階段已被使用過（無標籤）。
- 主對照改動的是**整套 Stage III 套件加上訓練預算**；沒有 source-continuation control 或隨機選取 control，因此結果**無法**把 EMA、對抗學習、entropy 選取或額外更新步數的效果單獨歸因於某一個機制。也不宣稱重現 Cui et al. 的完整 persistent PLR 方法。

## 6. 預期證據與數值判準

以下是本研究專屬的探索性續做邊界，不是統計顯著性檢定：

- **支持續做**：primary per-image macro Oil IoU 差值 `≥+0.03`，**且**固定覆蓋範圍內已標油 recall 差值 `≥−0.02`。即使此門檻通過，仍須回報 precision／面積與全部 19 張配對差值；通過不代表已建立泛化。
- **否定本原型於此終點的優先性**：primary 差值 `≤−0.03`。
- **其餘情況**：不可判定。primary 上升但 recall guard 未過時算不可判定，不算支持。終點影像缺漏或訓練未跑滿預算＝invalid／incomplete，不計入判定。

## 凍結與 GPU 預檢證據（不屬於事後結果）

- 真實 GPU batch16 預檢（`GPU_PREFLIGHT_BATCH16.json`）：100 次成功更新／102 次 attempt、2 次初期 AMP overflow、最後 20 次 attempt 無 overflow；GPU peak reserved 12,320,768,000 bytes（約 12.32 GB）／device total 33,668,857,856 bytes，佔 36.6%；teacher parameter EMA、teacher buffers frozen、student/D 確實變動、checkpoint 全組件可回載，四項皆 PASS；預檢期間 `target_gt_access=none`。**正式訓練一律用全新 S42 初始化，不沿用此預檢所留下的任何權重狀態。**
- 最終選用的 checkpoint 固定為預算打滿時的 **last student**（非 best，因無可用於選模的 target 標註）。
- 第 7–9 步只在結案時填寫；第 1–6 步此後不隨執行結果改寫，若需更動方法、資料或第 6 步門檻，須另立新版本卡並標記取代關係。

## 7–9. 結果／發現／下一步

尚無結果；本卡凍結時正式訓練尚未啟動。待訓練完成與固定終點重建跑完後再補寫。
