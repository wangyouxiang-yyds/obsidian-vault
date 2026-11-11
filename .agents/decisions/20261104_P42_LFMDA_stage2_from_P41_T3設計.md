---
experiment: P42
title: LFMDA stage 2（自訓練＋SAM＋LRH，原 P39 架構）接在 P41 T3 之後（單 seed 探索）
date: 2026-11-04
status: FROZEN v1（2026-11-04，launch 前）
type: exploratory, descriptive only（無 H1 判定）
---

# P42：P41 T3 → LFMDA stage 2（P39 原架構 SAM＋LRH），單 seed 探索

## 1. 問題與動機
P41 只跑了 LFMDA stage 1（PCD warmup，固定 P15 teacher → SegFormer-B2 6 波段），臺灣 24 張 T3 student final 0.491。依使用者「多階段方法先每步走一次」方針，新線還沒走 stage 2。使用者 11-04 裁決：**先照原架構的 SAM（P39 設定）跑一次**，之後再回頭討論 SAM 是否適合本資料（候選：DADS-SAM）。已知風險：P39 的 SAM 在固定 8-bit proxy 上 168/270 窗只有 1 區、221 窗區域蓋滿整窗，LRH 系統性把油擦成背景（P39 recall 0.09–0.22）。本卡**刻意不修** SAM，預期可能重演。

## 2. 設定（相對 P39 `stage2.py`：只換起點與 teacher，其餘照 P39 凍結卡 v4）
- **起點**：P41 `T3_S2333/student_final.pth`（`fd690481…`）＋`warmup_prototypes.pth`（`c99a0e74…`），strict 載入、核對 P41 `train_complete.json`。optimizer 與 LR 排程重新初始化（同 P39／作者）。
- **teacher**：與 P41 相同的固定 P15 S42 Prithvi-EO-V2-300M＋DeepLabV3（`11130fe9…`），eval、no grad、不更新、原生 8 slot 輸入。因此**沒有 teacher optimizer、沒有 teacher seg loss**（P39 的 DINOv2 teacher 會訓練 decoder）。label_refine 讀它的 logits（eval 態，P39 為 train 態）與第 23 層特徵（bilinear 到 1/32 = 8×8，同 P41 原型的空間）。
- **偽標籤**：step 0 與 3000 以 `pre_slide(tile 256, TTA×4)` 產生，取 teacher softmax（`ProbTeacher`），float32 存檔、讀回逐列核對 window key 後才建 loader。teacher 固定 → 兩次內容相同；仍照作者流程在 3000 重建 loader。
- **SAM 區域**：直接用 P39 封存的 ViT-H 區域（manifest `05fa132b…`，270 窗，固定 8-bit B04/B03/B02 proxy、面積 ≥1,024 px），不重跑、不改。
- **其餘照 P39**：6,000 步、warmup 1,500 → poly 分母 6,300、AdamW 6e-5（head ×10）、WD 0.01、source batch 8＋target batch 4、label_refine（mode all、temp 2.0）、pseudo_selection（0.8／0.6）、LRH 0.95、原型 EMA 0.99、target HFlip/VFlip/Rot90 各 0.5（四個 tensor 同步）、clamp(max=1)、loss＝學生 CE(source GT)＋CE(target LRH 偽標籤)；step 1 與每 1,000 步 GoM val，best＝mIoU 最大（`>=`）。
- **student 輸入**：P40 C6 6 波段前處理（同 P41）。target＝Taiwan19 270 窗（只用影像，transductive）；新增五張不進訓練。
- **seed**：S2333 單一。
- **wandb**：`oil-gt-expand`，run `P42_S2333`、group `P42`；train 模式要求線上（disabled/offline 即中止）。每 50 步 train/seg_s_stu、train/lr、mechanism/seg_t_stu；每 1,000 步 val/*＋mechanism/（LRH 前後各類像素數與轉換數）。終端機只印 source 值；Claude 封存前不看 `mechanism/`。

## 3. 評估（描述性）
- 主報臺灣 24 張 pooled OilIoU（P40 evaluator 函式與 PINNED），student final 主、best 次。
- 對照取自封存 P41 `eval_result.json`（`ffda8d4e…`）：**T3 final 0.4906（stage 2 起點）**、C6 final 0.4628／best 0.4799、T6 0.3369、REF P15 S42/43/44 0.489/0.407/0.393。報 P42 − T3 final、P42 − C6。
- 機制（封存後揭露）：偽標籤油機率 >0.5 比例、LRH 油→背景／背景→油像素數、P/R。
- 執行盲化：封存前不看臺灣預測、mechanism log、wandb `mechanism/`；讀 GT 需使用者另授權。

## 4. 已知限制
- 單 seed；C 組 seed 間 24 張範圍約 0.1。
- SAM／LRH 照 P39 原設定，已知在本資料上區域幾乎不分割；P42 若變差，只能說「與已知 SAM/LRH failure mode 相容」（無 source-only、no-LRH、no-SAM 對照，不能拆因果）。
- 固定 teacher（無 teacher 訓練）屬 LFMDA-inspired，不是原樣 LFMDA；與 P39 不是單因子比較。
- 沒有「T3 再做 6,000 步 source-only」對照，也沒有 no-LRH 對照 → 結論只針對整個 stage-2 continuation package。

## 5. 程式與 smoke（不讀 GT）
- `stage2_42.py`（P39 stage2 改寫：固定 P15 teacher、四 tensor 同步 target dataset、wandb；`homogenize`、常數、`CallCounter` 直接 import P39 `stage2.py`）、`predict42.py`、`evaluate_p42.py`（P41 evaluator 改寫，pin P40／P41 result）、`test_p42.py`、`launch.sh`。
- `test_p42`（4 項通過）：四個 tensor 幾何同步（200 次、8 種二面體結果全出現）；`ProbTeacher` = P15 `model(x)['out'].softmax`；target 路徑只開 270 窗 VRT（學生＋teacher 各 270 次）、未開 mask 與新增五張；lr 公式同 P39。
- smoke（wandb 關，2 步）：P41 起點 SHA 相符、pre_slide 恰 1 tile×4 TTA、window key 抽查 0/1/134/268/269 相符、label_refine 和與 1 最大差 5.4e-7、選取／LRH 值域 {−1,0,1}、teacher 無梯度且 state 不變、學生 347 個參數有梯度、loss 有限、峰值顯存 5.0 GB。
- 成本：估約 35–45 分（P39 單 run 33.5 分含 teacher 反傳）＋預測約 2 分。

## 6. 共識（Codex gpt-5.6-sol／high，session `01a1a6ab-daa4-7f93-80ef-0afc004b39c9`，全文內嵌）
- R1 REVISE：blocking＝起點與 SAM manifest 只做自洽驗證、P39 被 import 的程式未 pin。處理：`stage2_42.py` 硬 pin `START_SHA`（student_final `fd690481…`、warmup_prototypes `c99a0e74…`，且須同時等於 P41 `train_complete.json`）與 `SAM_MANIFEST_SHA`（`05fa132b…`，manifest 本身與 receipt 都要相符）；P39 `stage2.py`、`sam_regions.py` 列入 P42 `FROZEN_CODE.sha256`，launch 逐一核對。non-blocking 採：wandb `mode='online'`；launch 在預測後自動跑 image-only `seal`；§4 措辭改為「與已知 SAM/LRH failure mode 相容」。未採：pin 的負向測試（pin 為單行等式比對）。修正後 test_p42 4/4、smoke 重跑通過（舊 smoke 目錄更名 `smoke_P42_S2333_preR1`）。
- R2（resume）：**ACCEPT／BLOCKING=[]**。non-blocking：FROZEN_CODE 落盤並記 SHA → 已做。Codex RESTATE：「P42 只能描述單一 seed 下，從硬 pin 的 P41 T3 起點接續固定 P15 teacher 與原 P39 SAM＋LRH 整包後的 Taiwan24 final／best 表現及機制現象，不能拆解 stage 2、SAM、LRH 或 teacher 的個別因果效果，也不能宣稱是原樣 LFMDA reproduction。」
- 使用者 2026-11-04 授權：照原架構 SAM 跑一次（「先照原架構的SAM下去跑跑看」）。讀臺灣 GT 另需授權。
