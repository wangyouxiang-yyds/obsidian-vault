# handoff.md — 工作階段交接檔

用途：任一 harness 接手時（例如 Claude token 用盡、使用者跳到 Codex 繼續），讀本檔即可掌握目前工作階段並繼續做工。
更新紀律：每完成一個工作段落就更新「目前狀態」，不要等 token 快用完才寫（真用完就來不及了）。
必寫事件:process 啟動/結束、fold 完成、共識案收斂。交接前檢查 STATUS_AS_OF 與訓練 log/最近事件的時間差,逾時標 STALE,不得宣稱最新。
同機接手：Claude Code 與 Codex 都在這台 Linux server 上，讀寫同一份 vault，接手不需要 commit/push。只有要讓 Windows 端的 vault checkout 看到最新狀態時才需要 commit + push。

## 目前狀態(每次更新覆蓋本節;本節只是摘要,研究細節唯一正本=institution research/oil_spill_project_status.md)
- STATUS_AS_OF:2026-08-05(更新者:Claude 主 harness,經 obsidian-vault-manager;舊 STATUS_AS_OF 停在 07-24 已 stale 12 天,本次補記 07-24~08-05 缺口)
- 正本路徑不變:/home/alanyh/.agents/institution/research/oil_spill_project_status.md(該正本同一批次已補到 2026-07-31 P3 條目)
- **最近完成事件(兩個實驗都已結案,皆為 negative/null)**:
  1. **P2 FTL(focal_tversky,直接指數 q=4/3)判定 null,保留 Tversky**(2026-07-28)。三 arm Contract v1.0 S:baseline 0.3975 / tversky 0.4512 / ftl 0.4524;主對比 FTL vs Tversky ΔS=+0.0011,95% CI [−0.0087,+0.0168],整條 CI 夾在 ±0.02 內＝實質等價。報告 `analysis/p2_focal_tversky/p2_replay_report.md`。loss 分支確認枯竭。
  2. **P3 ASPP-rate grid 適配判定 negative,保留 (6,12,18)**(2026-07-31)。ΔS=−0.0165 CI[−0.0243,−0.0089]、機制層 ΔM_L=−0.0219 CI[−0.0663,−0.0120]＝**反向顯著**,落在 prereg 第三結局「無證據」。機制解讀:幾何圖沒錯(r=12/18 確實補零),但猜錯功能後果——ASPP 的 global-pool 分支＋補零大 dilation 卷積仍提供有用長距 context,縮小 rates＝有效感受野變小＝大圖分層更差;大 receptive field 對大片 diffuse 油膜有用,不是壞掉的。rate 調校這條槓桿關閉,連 prereg 列的 (1,3,5) follow-up 也不做。報告 `analysis/p3_aspp_rate/p3_replay_report.md`。**loss 與 rate 兩條槓桿皆已枯竭,精進要換地方找**(多層 tap 餵 ASPP／操作點與 TTA／合成資料)。
- **重要副產品**:`_dlv3_det`＝Prithvi+DeepLabV3-ASPP 的**第一次決定性(ba25391 後)3-fold**,pooled 0.4945/0.4774/0.3479,**mean 0.4399**。→ 計畫任務基線數字由舊 pre-fix pilot 的 0.4167 **更新為 0.4399**,計畫報告一律引用 0.4399。
- **執行中程序**:**無訓練在跑**,GPU 閒置(RTX5090,137 MiB / 0%)。唯一在跑的是 2026-08-05 09:40 啟動的 headless `claude -p`(Opus 5/high)Claude×codex 共識案,題目＝**mask / prediction artifact 的 JSON RLE 序列化 schema 跨兩 repo 重構**(codec 放哪、checksum/run manifest、GT JSON parity、dual-write 分階段遷移、10980² mask 的 RAM 問題),尚未收斂。
- **環境/資料異動**:(a) 2026-08-03 `result-seg/CV_358clean_gt_expand/` 與 `_tversky/` 的舊世代 run 目錄已清除,各只留決定性正本 3 個 run(舊世代 per_scene_iou.csv 一併消失,不能再重算舊世代對照,但「靠 timestamp 分辨世代」的誤抓陷阱同時解除)。**同批清理誤刪了 P3 prereg 要求保留的失敗殘骸**`result-seg/CV_358clean_gt_expand_prithvi300m_dlv3_aspp246/_ABORTED_reboot_20260731/`(2026-07-31 05:06 系統重開機打斷候選 fold3 約 epoch 32 時的殘骸;補跑本身合法且已留 log,但殘骸目錄已於 08-03 09:50 隨同批清理消失,provenance 現只存在於 auto-memory)——**制度教訓:清理 result-seg 前必須先確認沒有 prereg 要求保留的 artifact**。(b) 2026-08-04 `Docker/run.sh` 新掛 `-v /mnt/oil:/mnt/oil`(21T NAS,92% 滿),內含 `IR/OIL_PROJECT_dataset/full_band/` 與**兩個空的 `SAR/`、`UAV/`**,用途未文件化。
- **Git 狀態**:GT_expand HEAD 仍是 `f36e421`(2026-07-27)。**P3 全部產出未 commit**:`main/prithvi_deeplab.py`(ASPP rate 預註冊允許集)、3 個 config yaml、`main/test_prithvi_aspp_rates.py`、整個 `analysis/p3_aspp_rate/`、`analysis/p2_focal_tversky/` 的 replay 腳本與報告,以及 `Docker/run.sh` 的 /mnt/oil 掛載。
- **下一步待使用者裁決**:①`/mnt/oil` 的 SAR/UAV 是否要開新軸;②序列化共識案跑完後的裁決;③**Prithvi+UPerNet(現行主線)至今仍未做決定性 3-fold**——主線那個 mean 0.4498 依然是 ba25391 修復前的 legacy 數字,這是目前最大的一塊未償技術債。

## Harness continuity
- PARITY_AS_OF:2026-07-12 12:28 UTC(更新者:Codex)
- 狀態:Codex failover ready;Claude 不可用時可從共享狀態繼續可逆分析、實作、測試與 vault 工作。
- 已部署:4 個 Codex custom agents、4 個 vault skill symlinks、全域/兩專案 `AGENTS.md`、兩專案 workspace-write + vault/institution writable roots。
- 制度正本:`/home/alanyh/.agents/institution/40_harness_continuity.md`;稽核:`scripts/audit_harness_parity.py`。
- 驗證:`PARITY AUDIT: PASS (agents=4 skills=4 projects=2)`;Codex CLI prompt-input 已顯示 workspace-write 且載入 project/global AGENTS 與四個 writable roots。
- 權限邊界:缺席 harness 不得被冒稱同意;重大決策交使用者仲裁;launch/kill/delete/commit/push/publish 仍需使用者明確授權。
- Git:本次變更尚未 commit/push。

## 共識紀錄
- 2026-07-11｜Codex 前處理審查 F1-F10 處置方案｜一回合全條 AGREE：F2 降 Medium（空 tile=context-negative 保留）、F4 維持 High fail-fast、三批執行順序、ignorering 用凍結資料先跑與 v2 解耦｜雙方立場各經程式碼實查
- 2026-07-11｜DeepLabV3+ 架構切換實作方案｜二回合收斂：smp DeepLabV3Plus + model_family 切換鍵 + M1 預訓公平性 + rates(6,12,18) + dropout 維持出廠 0.5（Claude OBJECT Codex 的 0.1 成立）+ EMA 移除｜歷史 V3 checkpoint 逐位元回歸通過
- 2026-07-11｜協商制度升級 v2（收斂驅動：VERDICT 區塊、5 回合檢查點+10 回合硬上限、停滯偵測、反方終審限不可逆決策）｜兩回合收斂，雙方 ACCEPT｜詳見 decisions/20260711_收斂驅動協商制度升級.md
- 2026-07-11｜harness 結構改革(入口路由三方化/subagent 定義版本化/狀態單一正本+handoff 瘦身/稽核 checklist/agy 型號 unverified 標註/v2 試行評估附錄)｜三回合收斂:codex 首輪 OBJECT BLOCKING[1,2,3]→候選稿 v1 REVISE BLOCKING[P1-2a push 授權]→主持方以 vault-manager 定義第19行(常設 auto commit+push)證據解消→ACCEPT BLOCKING[]｜詳見 decisions/20260711_harness結構改革.md
- 2026-07-11｜研究路線重排+named-incident OOD holdout(36 scenes/9 events 封存、TODO 00/01/11 關、02 升 P0、12 擱置、14 改寫、新 03/15、B 移植 A=投稿 gate)｜四回合收斂:OBJECT→REVISE→REVISE→ACCEPT[]｜詳見 decisions/20260711_研究路線重排與OOD_holdout.md
- 2026-07-13 Prithvi-EO-2.0 接入方案(第 4 案):兩輪收斂 ACCEPT/BLOCKING=[]。vendor 官方 encoder 不裝 terratorch、300M、6-band [1,2,3,5,6,7]、UPerNet head(codex 論點勝出)、pilot 預註冊 fold1/Δ≥+0.030/Wilcoxon。設計正本=GT_expand repo TODO/15;decision record=decisions/20260713_prithvi_eo2_接入方案.md。僅設計未動工(觸發條件=context 實驗無效)。
- 2026-07-13｜Claude token 中斷 failover 實測｜Codex 從未提交 diff/mtime/handoff 還原 Prithvi 進度，完成靜態審查與 CPU Phase 0；發現並修正 batch=1 PPM BatchNorm、AMP 未轉發、checkpoint 未固定官方 SHA256 三項問題；8/8 PASS，C6/下載/GPU/訓練仍未執行。
- 2026-07-13｜Prithvi 實作互審輪（第 4 案追加）｜二回合收斂 REVISE→ACCEPT/BLOCKING=[]：Codex 三修正經 Claude 獨立查證屬實（SHA 對 HF API、Phase 0 重跑）；共識修正 F1=collate 縮水 batch 守門+accumulation 計數證明+三案例回歸、F2=AMP 向後影響寫入 status 檔；殘餘條件=C6 未過不得啟動、batch2 仍 OOM 即停、n_skipped_small 系統性偏高須查資料完整性
- 2026-07-18｜Codex 常態入口與 Claude 雙向協商 v2.1｜三回合同 session 收斂：R1 REVISE(B1–B3)→R2 REVISE(B4)→R3 ACCEPT/BLOCKING=[]；一般 Claude 諮詢=Sonnet 5、正式共識=Opus 4.8/high、Fable 5 非依賴、JSON session resume、深度 1、model/harness 分級 fallback、parity gate｜詳見 decisions/20260718_Codex預設入口與Claude雙向協商.md
- 2026-07-24｜Focal Tversky 小圖實驗可行性｜兩回合同 session 收斂：R1 conditional worth、BLOCKING[plain Tversky 先完成/FTL 必須直接對 plain Tversky]→Codex 撤除未驗證的 soft-TI attenuation 推算→R2 雙方 ACCEPT/BLOCKING=[]；FTL 僅在 canonical Tversky 前置 gate 通過後取得一張 q=0.75 單變因 coupon，fold1 失敗即停、不掃 q｜詳見 decisions/20260724_Focal_Tversky小圖實驗可行性.md
- 2026-07-24｜Codex 獨立審查 vault + 決定性重跑優先序｜STANCE=CONDITIONAL GO（vault AMBER、研究方向 GREEN、里程碑 AMBER）：三個最優先動作（①安全跑完/等價重跑 0.3/0.7 三折 ②用決定性正本重做 Contract/LOGO、舊 grouped 全降 legacy single-run ③修 freshness+全 vault 撤回舊主張，之後只排固定 0.3/0.7 的 Prithvi+UPerNet deterministic 3-fold，P2/0.5-sweep/Focal-Tversky 延後）+ 四個盲點（grouped 須在決定性正本上整組重做、grouped 不修洩漏、僅 ~12 事件群需 per-event+LOGO、Prithvi+UPerNet 舊優勢同受 augmentation bug 污染）+ 制度規則建議（單一 checkout 為 vault main 唯一 writer、push 前先 fetch、main 只收 fast-forward、validity-changing finding 當天寫 stub）｜詳見 decisions/20260724_Codex審查與決定性重跑優先序.md
- 2026-07-27｜Claude Opus 5 正式共識路由升級｜三回合同 session 收斂：R1 REVISE(B1–B3)→R2 REVISE(B2)→R3 ACCEPT/BLOCKING=[]；一般諮詢維持 Sonnet 5、正式共識升級 Opus 5/high、fallback=Opus 5→Opus 4.8→Sonnet 5（Sonnet 結論 PROVISIONAL）、歷史 provenance 不回寫｜詳見 decisions/20260727_Claude_Opus_5正式共識路由升級.md
