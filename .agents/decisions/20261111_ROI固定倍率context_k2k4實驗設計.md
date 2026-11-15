---
experiment: ROI_fixed_context_k2k4
date: 2026-11-11
status: DESIGN_CONSENSUS_NOT_AUTHORIZED
type: Claude×Codex 協商共識（設計，未執行）
consensus: Codex R3 ACCEPT／BLOCKING=[]（3 回合）
---

# 2026-11-11 ROI 固定倍率 context（GeoAgent 雙分支，k=2/k=4，無 RL）實驗設計

> ⚠ 2026-11-11 融合方式已改為論文 branch aggregation（v5），γ 版作廢，見文末『設計變更』節。
> ⚠ 2026-11-11 v6 取代 seed／資料／主終點，見文末 v6 節。

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


## 2026-11-11 設計變更：融合改回論文 branch aggregation（R4–R5；取代上方 γ 版）
**觸發**：使用者得知 `F_local+γ⊙C` 是 08-31 自家改法、非論文後裁決「先試試論文的吧」。Claude 揭露：R1 背景內嵌了 08-31 γ 公式，Codex 的「獨立提案」可能受錨定；有 K1 對照後「concat 增加容量造成混淆」的理由已被抵消。
**R4**（Codex REVISE，BLOCKING 4：context aux 不得 ≥0.5 硬標籤／aux 總權重 0.4 平均分配／ROI 外標註來源 gate／ASPP BN 與反事實凍結）→ **R5**（Claude 接受 1/2/4；對 3 提出修正並附實測：mask 整景、值 {0,1,255}，有效像素上的 255 佔 16–45%，P44 本身即以 pixel_mapping 把 255 當背景）→ Codex **ACCEPT／BLOCKING=[]**。

**v5 取代 v3 的部分**
- 融合：共用 Prithvi（第 23 層）＋共用 DeepLabV3 trunk（ASPP→256ch 3×3→BN→ReLU），在最終 classifier 前取 f_l、f_c（16×16×256）；local/context encoder 特徵沿 batch 軸合併成 batch 32 一次過 ASPP（避免 BN 依分支順序被更新兩次；記憶體不過不得改順序式／凍結／分支 BN）。f_c 以 v3 同一 grid-sample 對齊（K1 identity）。Aggregation：concat(512)→Conv3×3(512→256, bias=False)→BN→ReLU→Conv1×1(256→2)；3×3 seeded Kaiming、BN w1/b0、classifier 沿 P44 初始化規則。BN/ReLU 為原文未明細之凍結 operationalization。
- Aux：共用 P44 1×1 aux classifier 作用於 f_l 與完整未裁 f_c。`L = L_main + 0.4·(L_aux_local + L_aux_ctx)/2`；主 loss 仍 P44 Tversky（不改 CE）。
- Context aux target：`y=pixel_mapping(raw)`（raw 255→背景，與 local 同語義）；`w=area(image_valid)`；`q=area(1[y=oil]·valid)/w`；w≥0.5 用 soft q，w<0.5 進獨立 ignore mask；不得把 soft target 再送 pixel_mapping；RRC/flip/rot90 與 context image 同幾何（nearest）。
- 推論、VAL 選模、TTA 只用 aggregation 輸出；aux 不投票。
- 移除 γ 及 γ=0 parity；不保留 γ 對照組（若本案否定／不可判定，另立實驗）。
- non-executed：開跑前 isolated backward——L_main 對 aggregation 3×3 local half 與 context half 梯度、∂L_main/∂C_aligned、∂L_aux_local/∂f_l、∂L_aux_ctx/∂f_c 皆 finite 且 RMS>1e-12，三個 loss 皆有非 ignore target；前 200 steps probationary：≥20 個含油 batch 中 ≥18 個 context-half 梯度與 ∂L_main/∂C_aligned 有效、三 loss finite。
- adoption 反事實：同 checkpoint 以 `C_aligned ← f_l`（K1 式 identity），`A_identity=½[mean_oil|Δm|+mean_bg|Δm|]≥0.01`（m=z_oil−z_bg，VAL stride256 無 TTA 拼接後）；context 置零只作補充敏感度。主證據仍為 derangement。
- K1 parity：input identity ≤1e-6；eval 下 f_c=f_l ≤1e-6；對齊後 C_aligned=f_l ≤1e-6；aux logits、aux target/loss identity ≤1e-6；aggregation 對 reference ≤1e-5。
- 新 gate：soft Tversky 在 hard target 上對現行 TverskyLoss（含油／全背景／混 ignore／多樣本 batch fixtures）loss 差 ≤1e-6、grad 相對差 ≤1e-5（近零梯度改 max abs ≤1e-8）；α/β/smooth/aggregation/ignore/reduction 不得改。
- 255 揭露（描述性，不停跑）：TRAIN、VAL 各以 seed 20261111 無放回抽 min(4096,N) 個窗中心，同中心算 local/K2/K4 在 image-valid 原像素中 raw 255 比例；`p_Kk/max(p_local,1e-6)>2` 標註「context aux 受未定義 255 區影響較大」，不調權重、不移除 aux。
- 結論範圍：檢驗的是「GeoAgent-style aggregation＋auxiliary-training package」，context aux 擴大了監督 footprint，列為競爭解釋；不是純推論像素的無條件因果效果。
- 不變：K1/K2/K4、讀取與 area N/D、RRC Δ/k、batch16、Stage A 9 runs＋futility、Stage B 6 runs、主終點與支持/否定門檻、derangement、臺灣規則、VAL 前置拆解、I/O 與記憶體 gates、三項授權獨立。
附件另含 r4_prompt/r4_out/r5_prompt/r5_out。

## 2026-11-11 前置 gate 1 結果

- 授權：使用者授權第 1 項（「先做第一步吧」）。P44 S42、P48 S43 各取 best；VAL 37 景；stride 128＋TTA fp32。
- VAL IoU .188／.168（S42／S43）；recall .246／.209。
- >20px：GT 1,087,999 px、3 事件群，FN share .654／.640，recall .103／.078；6–20px recall .406／.348。
- ≥80% density：700,409 px、2 事件群，FN share .452／.439，recall .036／.017。
- 判定：兩 seed 皆過可評估＋相關性 → 依凍結規則定位 **targeted**。
- 注意集中度：VAL 油像素 51% 為單一景單一元件 NOAA_Gulf_of_Mexico_High_Confidence_20190925_S2_16RCS（1,008,035 px，recall .06／.04）；≥80% bin 692k／700k 來自 GoM 事件；千里達 20240303 20PQT 137k px 元件整塊漏。機制證據實質由 1–2 景支撐，主終點 VAL pooled IoU 也受該景主導。規則未改，只作揭露。
- 產物：`/home/alanyh/oil_dataset/new/_ctx_k2k4_prereq_20261111/{S42,S43}/breakdown.json`；程式 GT_expand commit 5818075；institution commit 58bb913／1c42289。
- 第 2、3 項仍未授權。

## 2026-11-11 設計變更 v6（使用者三項變更；Codex R6 REVISE → R7 ACCEPT，BLOCKING=[]）
附件：`attachments/20261111_ROI固定倍率context_k2k4實驗設計/r6_prompt.txt`、`r6_out.txt`、`r7_prompt.txt`、`r7_out.txt`。

**使用者原話**：「我認為這個部分先幫我改成個跑一次就好」；「我想要你幫我改成GoM-only好了，val組成的部份大中小油污幫我平均分配」；「我沒有要先用3 fold的機制 是val的資料集我覺得是要挑一些大中小油污進去」；同意 macro-3 與單 seed 版（「OK 我了解了 同意」）。

**Seed**：K1／K2／K4 各一次 S42，共 3 runs。Stage A／B、五 seed、4/5、median、futility、legacy |K1−P44| 規則全移除。結果為單 seed pilot，不稱支持／否定。

**資料**：GoM-only，195 景池（GoM High Confidence 且 splits_sv2 有值），沿用 GoM 大中小 v1 分級（每景 8 鄰接最大油區；q1=4091.33、q2=18171），不用 v1 三折成員。VAL 30＝大中小各 10，TRAIN 約 165（55/55/55）。duplicate_group_id（157 組）不拆；group ID 穩定排序 → seed 20261111 打亂 → exact DP（state ⊂ (0..10)^3、lexicographic、首次抵達 predecessor）；不可達 fail-closed，不換 seed、不拆組、不放寬。驗證並輸出 manifest／hash；揭露 12 個 P44 舊 drop_leak 景分到哪側、舊 split_leak_pairs 跨側數、bbox overlap 實際跨側數；結論限「已見 GoM 地點附近的新場景」。12 景以 p44_roi.scene_rois 同規則補 ROI。

**主指標**：macro-3＝大中小各自 pooled oil IoU 平均；每 VE 選 best 也用 macro-3（stride256 無 TTA；tie 取早 VE；patience 8）。並列 pooled IoU、各 bin IoU／P／R／pred-gt；小 bin 不設門檻，揭露各 bin GT px、最大單景佔比、各景 IoU median／range。

**Pilot 分類**（collapse 優先：best checkpoint VAL recall≤.01 或 pred/gt≤.01；K1 collapse → 兩比較皆不可判定；Kk collapse → 該倍率不可判定）：
- Δmacro-3（selection metric）≥+0.05 且 Δrecall_>20px（stride128＋TTA 重跑 val_fn_prereq 方法）≥+0.10 →「值得補 seed」（非追加訓練授權）。
- Δmacro-3 ≤−0.05 →「pilot 負向」。
- macro-3 ≥+.05 但 interior 未達 →「不可判定—非預期效益訊號」。
- 其餘「不可判定」。

**描述性分析**：derangement（donor 跨 duplicate_group_id、coverage decile）與 adoption 對非 collapse K2／K4 做、純描述。臺灣在 VAL 與 pilot 分類凍結後才讀，只報大林 2／其他 22。

**gate 1**：改標「historically targeted」（舊全球 VAL＋全球模型）；K1 完成後在新 VAL 做描述性拆解，標 replicated／not replicated／dominated by one scene，不改門檻或 split。

**不變**：v5 架構（shared trunk、batch32 ASPP、concat 聚合頭、aux 0.4 soft occupancy）、context 讀取／area N/D／RRC Δ/k、preflight gates 2–4、200-step exposure、schedule（batch16、4000 steps/VE、max 40 VE）。255 揭露統計在新 GoM manifest 上重算。

**狀態**：institution commit 6632a5f。第 2 項（建 split／ROI、寫程式、preflight、凍結卡）與第 3 項（GPU launch）皆未授權。

## 2026-11-11 v6 實作與凍結（使用者授權第 2 項；Codex R8 REVISE → R9 REVISE → R10 REVISE → R11 ACCEPT，BLOCKING=[]）

**使用者原話**：「好，就進行下一步吧跑實驗前我會跟你說我要另外加的實驗」（授權第 2 項；**launch 未授權**；使用者會先告知要另外加的實驗）。

**程式與凍結**：GT_expand `analysis/ctx_k2k4_20261111/`；實驗卡 `CTX_CARD.md` FROZEN。首次凍結 commit 15a5958，卡片日期更正後最終凍結 8e5be4d，紀錄 `FREEZE_RECORD.md`（47c5fd2），FROZEN.sha256 32 項（SHA d9c7f00c…e695）。

**Split**：TRAIN 165 景／187 ROI／153,853 窗（含油窗 4,749）；VAL 30 景（大中小各 10）／33 ROI／7,425 窗；22 個 duplicate group 入 VAL。揭露：跨側油區 bbox 相交 943 對、ROI 相交 1,592 對；舊 P44 drop_leak 12 景 → TRAIN 10／VAL 2；VAL 大 bin 最大單景佔 39%（20210609 15RVM）。

**Codex 接受的實作偏離**：
- (A) TRAIN 讀同解析度無損 ZSTD 快取（源 TIF 為 LZW 整列 strip，K4 讀檔原 2.3 batch/s）；以凍結 builder 由 VRT 重建 330 檔 SHA 全同；strict 不 fallback；manifest SHA 釘在 config。
- (B) 只讀模型 6 槽，B01/B08 槽恆 0（VRT 對 L8 B08 nearest 取樣隨讀窗起點變；使用中 6 槽經 G6/G7 驗證不受影響）。
- (C) area N/D 整數累加後一次除（誤差 ≤6e-8）。
- 增強自寫以同步（同分布、非位元一致）。GDAL_CACHEMAX 128 MB。

**Preflight**（正式 preflight.json SHA 2d5f6678…c800）G1–G8 全過：G1 幾何誤差 0；G2 K1 identity 0；G3 FP32 五 fixture 0；G4 12.68 GiB；G5 9.30 vs 6.99 batch/s（1.33×，16 workers）；G6 165 景、L8+S2、825 讀取 0 不符、出快取窗 0；G7 TRAIN 64 窗與 VAL 33 ROI 三組 local 輸入逐位元相同；G8 main／context-aux 梯度皆通。D1（揭露）raw-255 TRAIN ≈.64、VAL ≈.69，context 不高於 local。

**Codex 審查**：R8 五項 BLOCKING（跨 K local parity、cache manifest／coverage／strict、FP32 fixtures＋隔離梯度、D1 4096 樣本、重跑並綁 FROZEN）、R9 四項（launch 完整性 check、FROZEN 涵蓋全部 list、快取血緣、卡片狀態）、R10 一項有效（clean-tree 須拒絕 staged；另一項為 Codex 看到舊版而撤回）全數處理；R11 ACCEPT。附件：`attachments/20261111_ROI固定倍率context_k2k4實驗設計/r8`–`r11` 的 `_prompt.txt`／`_out.txt`（r8_prompt 約 90 KB，含程式碼）。

**Smoke**：K4 ~5.5 it/s，每 VE 約 13–15 min；預估每 run 6–12 h。

**待使用者**：launch 授權（第 3 項）；使用者要先說明另外要加的實驗；可刪除的暫存：`cache_train/`（59 GB 舊 npy 快取）、`cache_rebuild_check/`（19 GB 驗證重建）。

institution commit c1bf894（＋日期更正）。

2026-11-11 23:43 launch（使用者授權第 3 項）；P52 排隊於後；暫存快取經同意刪除。
