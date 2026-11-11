---
experiment: ROI_fixed_context_k2k4
date: 2026-11-11
status: DESIGN_CONSENSUS_NOT_AUTHORIZED
type: Claude×Codex 協商共識（設計，未執行）
consensus: Codex R3 ACCEPT／BLOCKING=[]（3 回合）
---

# 2026-11-11 ROI 固定倍率 context（GeoAgent 雙分支，k=2/k=4，無 RL）實驗設計

議題：在 P44 ROI 滑窗基線上，測試 GeoAgent（Liu et al., ICCV 2023）式 patch 外 context 是否有用；使用者指定固定倍率 k=2（讀 512）與 k=4（讀 1024），先不導入 RL／尺度選擇器。
主持方：Claude Code（Opus 5.5，claude-opus-5-5）。對手方：Codex（codex-cli 0.167.1，gpt-5.6-sol，reasoning high，session `01a1c89b-ad85-7930-85ce-f6b14c36cc69`；全文內嵌、未依賴讀檔）。
回合數：3。收斂：R3 雙方 `ACCEPT`、`BLOCKING=[]`。無降級事件。
**執行狀態：僅設計共識；未寫碼、未跑 VAL 推論、未凍結 prereg、未 launch。三項授權（VAL 推論／prereg 凍結 commit／GPU launch）彼此獨立，皆待使用者。**

## 觸發
- 使用者想重新試 GeoAgent：P11（GT-centric）attempt1 判 non-executed、attempt2 結果查無紀錄＝context 在本專案從未被有效測過；ROI 3840 改版後提供連續滑窗 episode。
- Claude 11-10 唯讀診斷（P44_best S42 臺灣）：256 方框油佔比 ≥80% 的像素 recall .14、佔 FN 43%；離油邊 >20px recall .36、佔 FN 65%；**≥80% 那格 FN 100% 來自大林兩張**（中心方形漏判）。其他 22 張 FN 以小碎片整塊漏為主（<1k px 元件 44% 全漏）。屬臺灣 GT 事後描述，依規則不得反推設計 → 設前置 VAL 拆解。

## 逐輪
| 回合 | 內容 | Codex |
|---|---|---|
| R1 盲提案 | Codex 獨立提出：K1 同碼對照＋K2＋K4、5 配對 seed（15 runs）、共用 encoder、zero-init channel γ residual、grid-sample 對齊、RRC Δ/k、batch16 不得梯度累積替代、derangement、non-executed/null 區分、VAL FN 前置 | ACCEPT（BLOCKING 3 項：VAL phenotype、parity/對齊測試、記憶體） |
| R2 | Claude 提 M1 兩階段 seed、M2 derangement 只對候選組、M3 I/O preflight、M4 VAL/TTA 帶 context、M5 legacy 只當 sanity，Q1–Q3 | ACCEPT，收緊 M1 停止條件、定 rasterio average 等價規範、Q1 不一致降級、Q3 改 class-balanced margin |
| R3 | Claude 全收；BLOCKING 2/3 重分類為 fail-closed launch gates | ACCEPT，BLOCKING=[]；200-step exposure 標為 launch 後 probationary gate |

首輪實質返工：15 runs → 9＋6 兩階段；adoption 定義改 class-balanced oil-logit margin；rasterio out_shape 降為待驗證實作。

## 最終設計（候選稿 v3）
**組別**：K1-control（context=同一 256，雙 forward、同 γ）／K2（512→256）／K4（1024→256）。P44 S42／P48 S43／S44 只作 legacy sanity（|K1−legacy|>0.05 只標註，不判定、不 non-executed）。不做 random-k、context-only。
**Seed 兩階段**：Stage A＝S42/S43/S44×3 組＝9 runs（Latin 順序：S42 K1→K2→K4；S43 K2→K4→K1；S44 K4→K1→K2）。Stage A 只准 futility 停止：K2、K4 **皆**「3 個 paired Δ 全 ≤0 且 mean Δ ≤ −0.010」→ futility denial 結案；否則三組整組進 Stage B（S45/S46，6 runs）。Stage A 不讀臺灣、不得改任何設定或只延伸某倍率。
**架構**：共用 Prithvi（第 23 層 16×16×1024）；`F = F_local + γ⊙C_aligned`，γ [1,1024,1,1] fp32、zero-init、no WD、head LR 5e-5；無 projection／concat／norm；ASPP、head、optimizer 不動；γ=0 時須等價 P44 local path。對齊：context 完整 16×16 特徵以 grid-sample bilinear、align_corners=False，取樣位置 `g_j=(((2j+1)/16)−1)/k`；K1 退化為 identity。
**讀取**：local 中心固定；K2 由 (x−128,y−128) 讀 512，K4 由 (x−384,y−384) 讀 1024；可讀 ROI 外、不得跨 VRT／split；area 降採樣，`N=avg(image×valid)`、`D=avg(valid)`、output=N/D，D=0 用既有 fill；不加 validity channel；loss 只看 local。記錄 context valid-fraction 分布並分層。
**增強**：flip/rot90 三者同步；RRC 在 local 抽 (Δx,Δy,w,h)，context 用中心 Δ/k、同 w,h；Brightness/Contrast 同參數；GaussNoise 同強度同 realization。context 專用 RNG 獨立 stream，保 paired-seed。
**訓練**：完全沿用 P44（ROI、stride128、自然比例、batch16、Tversky batch 加總、AdamW、Cosine、40 VE、patience 8）。記憶體優先對 Prithvi transformer blocks 做 activation checkpointing（保 RNG），ASPP 不 checkpoint。禁止 batch8×累積2；僅允許驗證過的 exact-gradient replay（loss 誤差 ≤1e-6、grad/update 相對誤差 ≤1e-5）。
**VAL/測試**：訓練 VAL（stride256、無 TTA）與測試（stride128＋orig/hflip/vflip）都走完整 context path，flip 同步、logit 先逆轉再平均拼接；K1 也雙 forward。
**主終點**：VAL ROI pooled 油 IoU，paired Δ_{k,s}=IoU(Kk,s)−IoU(K1,s)。「支持」須同時：五 seed mean Δ ≥+0.020；≥4/5 Δ>0；median ≥+0.010；treatment collapse 數 ≤K1；derangement 後 IoU 降 ≥0.010 且 ≥該倍率增益 50%。「否定」：K2、K4 mean Δ 皆 ≤0 且各至多 2/5 勝。其餘「不可判定」。collapse（VAL recall ≤0.01 或 pred/gt ≤0.01）照算不剔除，是科學結果。
**derangement**：只對五 seed 通過條件 1–4 的倍率；主 VAL operator；依送入 encoder 前 context valid fraction 每倍率十分位；permutation seed 20261111；context 與其 validity 一起置換、跨具名事件群；不可配對 bin 由低到高掃描與異事件候選最多的相鄰 bin 合併（同數取低編號、邊界只向內）；mapping 存檔、所有 checkpoint 共用。
**次終點**：VAL P/R/pred-gt、per-event IoU、>20px 與 ≥80% density bins recall/FN share、小元件 recall、coverage 分層；γ=0／derangement／full 三反事實。
**臺灣**：VAL 判定全部凍結後才讀；只報大林 2 張與其他 22 張（run 內像素加總、跨 seed 平均、paired 差）。Corroboration：大林相對 K1 mean ≥+0.05 且 ≥4/5 改善，其他 22 平均差不低於 −0.02；不取代 VAL。大林內部 recall（≥80% density、>20px，門檻 +0.10）只作「由臺灣事後發現導出」的描述性機制診斷，不得選模／改 k／調 γ／翻主結論。
**non-executed**（只限 treatment 未施加）：讀取倍率/中心/對齊/split 測試失敗、γ=0 parity 失敗、context 誤複製/梯度斷線/載入錯誤、OOM 後改 batch/loss/optimizer、主終點不可還原；梯度 exposure：前 200 steps ≥20 個含油 batch，其中 ≥18 個 γ grad finite 且 RMS>1e-12（launch 後 probationary gate，未過即停、判 non-executed）。
**adoption**（未達＝scientific null，非 non-executed）：同 checkpoint full vs γ=0、主 VAL operator、拼接後 oil margin m=z_oil−z_bg，`A_logit=½[mean_oil|Δm|+mean_bg|Δm|]`（非 ignore 有效像素）≥0.01；另 median_window RMS(γ⊙C)/RMS(F_local) ≥0.01。

## 前置與 launch gates（fail-closed，不得放寬、改 batch／算子、刪 K4 或推定授權）
1. **VAL FN 拆解**（P51 結束後；P44 S42、P48 S43；stride128＋TTA；bins 沿用既定）：可評估＝>20px 或 ≥80% bin ≥10,000 GT px 且 ≥2 具名事件群；相關性＝median interior FN share（>20px ≥30% 或 ≥80% ≥20%）且 interior recall 比 6–20px 低 ≥0.15。兩 seed 都過→targeted；否則（含一過一不過）→ general fixed-context exploratory，不得平均後過線。
2. synthetic coordinate-ramp 中心誤差 ≤0.5 px；K1 encoder input identity ≤1e-6；γ=0 logits parity ≤1e-5。
3. batch16 peak memory ≤30.5 GiB。
4. K4 I/O：400-step data-only throughput 須高於 GPU 消耗率約 20%；可加 workers（不改 sampler/RNG）；rasterio average out_shape 須過等價（post-normalization max abs ≤1e-6、validity 完全一致、K1 identity 仍過），否則讀全解析度後 reference area；不得 bilinear／不同解析度快取。
5. CUBLAS_WORKSPACE_CONFIG 於 import torch 前設定、main_runner.set_global_seed、DataLoader worker 與 augmentation RNG 固定；不宣稱 bitwise。
6. 使用者個別授權：VAL 推論、prereg 凍結/commit、GPU launch。
7. 訓練一個一個跑，P51 未結束不得並行。

## 算力
P44 約 3.5–5.5 GPU h/run；雙 forward＋checkpointing 保守 11–17 h/run。Stage A 9 runs ≈ 4–6 天，Stage B 再 6 runs ≈ 3–4 天。以 preflight 實測重估，不得因此改組別或 seed。

## 競爭解釋與殘餘風險
雙 forward／額外梯度（K1 控制）；invalid border／scene extent 捷徑（coverage-matched derangement）；低頻光譜平滑而非地理關係；單一大事件驅動（per-event＋大林/其他 22）；seed collapse（5 配對 seed＋不可判定區）；大片均質油面超出 1024 仍看不到邊界；標註／光譜域偏移。

## 少數意見
無。

## 後續驗證點
VAL FN 拆解結果與 targeted/exploratory 定位；preflight 各 gate；Stage A futility 判定；Stage B 五 seed 判定與 derangement。


附件（協商各輪 prompt／output 原樣）：`.agents/decisions/attachments/20261111_ROI固定倍率context_k2k4實驗設計/`
