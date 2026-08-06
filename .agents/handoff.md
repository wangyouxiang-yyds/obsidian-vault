# handoff.md — 工作階段交接檔

用途：任一 harness 接手時（例如 Claude token 用盡、使用者跳到 Codex 繼續），讀本檔即可掌握目前工作階段並繼續做工。
更新紀律：每完成一個工作段落就更新「目前狀態」，不要等 token 快用完才寫（真用完就來不及了）。
必寫事件:process 啟動/結束、fold 完成、共識案收斂。交接前檢查 STATUS_AS_OF 與訓練 log/最近事件的時間差,逾時標 STALE,不得宣稱最新。
同機接手：Claude Code 與 Codex 都在這台 Linux server 上，讀寫同一份 vault，接手不需要 commit/push。只有要讓 Windows 端的 vault checkout 看到最新狀態時才需要 commit + push。

## 目前狀態(每次更新覆蓋本節;本節只是摘要,研究細節唯一正本=institution research/oil_spill_project_status.md)
- STATUS_AS_OF:2026-08-06(更新者:Claude 主 harness,經 obsidian-vault-manager;本次更新=P4 操作點實驗結案,由「進行中」推進到「已結案(實質 null)」)
- 正本路徑不變:/home/alanyh/.agents/institution/research/oil_spill_project_status.md(該正本第 9 條已同步補到 2026-08-06 P4 結案)
- **最近完成事件(三個實驗都已結案,皆為 negative/null)**:
  1. **P2 FTL(focal_tversky,直接指數 q=4/3)判定 null,保留 Tversky**(2026-07-28)。三 arm Contract v1.0 S:baseline 0.3975 / tversky 0.4512 / ftl 0.4524;主對比 FTL vs Tversky ΔS=+0.0011,95% CI [−0.0087,+0.0168],整條 CI 夾在 ±0.02 內＝實質等價。報告 `analysis/p2_focal_tversky/p2_replay_report.md`。loss 分支確認枯竭。
  2. **P3 ASPP-rate grid 適配判定 negative,保留 (6,12,18)**(2026-07-31)。ΔS=−0.0165 CI[−0.0243,−0.0089]、機制層 ΔM_L=−0.0219 CI[−0.0663,−0.0120]＝**反向顯著**,落在 prereg 第三結局「無證據」。機制解讀:幾何圖沒錯(r=12/18 確實補零),但猜錯功能後果——ASPP 的 global-pool 分支＋補零大 dilation 卷積仍提供有用長距 context,縮小 rates＝有效感受野變小＝大圖分層更差;大 receptive field 對大片 diffuse 油膜有用,不是壞掉的。rate 調校這條槓桿關閉,連 prereg 列的 (1,3,5) follow-up 也不做。報告 `analysis/p3_aspp_rate/p3_replay_report.md`。
  3. **P4 操作點(決策閾值)判定「實質 null」,argmax≡τ=0.5 預設維持,閾值槓桿正式關閉**(2026-08-06)。prereg v1.0 四輪收斂凍結(ACCEPT/BLOCKING=[])。**✅ 使用者已裁決:計畫任務以 pooled Oil IoU 評成敗**(解除先前卡住 prereg 凍結的 BLOCKING #8);Δpooled=−0.0011,95% CI[−0.0067,+0.0034](12 事件群 cluster bootstrap,B=9999,seed 20260718),CI 完全落在 ±0.02 採用門檻內。**H2a「模型過度保守」假說被推翻**:GT-positive 像素可救回機率質量(0.1≤p<0.5)僅 2.1%,模型是「有信心地判錯」非保守。**estimand 標籤更正**:招牌數字 0.4399 是「逐折 pooled 再平均」,不是全語料 pooled(=0.4335),兩者是不同估計量,主端點未改。§7 盲化標註稽核證實兩大巨景 GT 品質尚可、Pacific 20211005 一景輸入端無動態範圍(感測器地板)。含兩則撤回聲明(「299/355 景逐位元相同」誤稱、判準量綱教訓,詳見報告)。報告 `analysis/p4_operating_point/p4_report.md`。**loss / rate / 閾值三條槓桿皆已枯竭,精進要換地方找。**
- **重要副產品**:`_dlv3_det`＝Prithvi+DeepLabV3-ASPP 的**第一次決定性(ba25391 後)3-fold**,pooled 0.4945/0.4774/0.3479,**mean 0.4399**。→ 計畫任務基線數字由舊 pre-fix pilot 的 0.4167 **更新為 0.4399**,計畫報告一律引用 0.4399。P4 結案首次為此基線補上可存檔、可重放的逐像素預測(前此完全沒有 pixel-exact 存檔)。
- **執行中程序**:**無訓練、無推論在跑,GPU 閒置**。P4 的背景推論 job(`dump_prob.py --all`)已於結案前跑完並通過所有前置閘;結案診斷過程完整記錄於 [[20260806_P4操作點決策閾值_實質null]]。
- **下一棒接手點(2026-08-06 新增,最優先)**:**剩餘槓桿=方向 C,需重訓 + 新 prereg。** P4 結論把問題從「推論期調校」推回「訓練期」——巨型場景的失敗是機率響應問題,不是決策邊界問題。兩個候選:①**面積加權/上限採樣重訓**(權重錯配已量化:最大 10 景佔 44.4% 油污像素卻只拿 16.7% train patch,約 5× 欠採樣;⚠ codex 警告正比加權會把 pooled 的巨景支配引進訓練、與論文主線 S 衝突,須 stratum-balanced 或設上限);②**多層 tap 餵 ASPP**(head 不變,在計畫紅線內)。**兩者皆須重訓,不再是零成本推論實驗,須另立 prereg。** codex 對優先序有不同意見:不認為應直接跳方向 C,建議先做低自由度的 large/non-large 兩段式操作點作中繼步驟。論文寫作時 pooled 與 Contract S 兩個數字一律並報,不得擇優呈現。
  - 診斷過程與 P4 結案全文已寫入 vault [[20260805_巨型場景診斷與P4操作點實驗]]、[[20260806_P4操作點決策閾值_實質null]]。
- **環境/資料異動**:(a) 2026-08-03 `result-seg/CV_358clean_gt_expand/` 與 `_tversky/` 的舊世代 run 目錄已清除,各只留決定性正本 3 個 run(舊世代 per_scene_iou.csv 一併消失,不能再重算舊世代對照,但「靠 timestamp 分辨世代」的誤抓陷阱同時解除)。**同批清理誤刪了 P3 prereg 要求保留的失敗殘骸**`result-seg/CV_358clean_gt_expand_prithvi300m_dlv3_aspp246/_ABORTED_reboot_20260731/`(2026-07-31 05:06 系統重開機打斷候選 fold3 約 epoch 32 時的殘骸;補跑本身合法且已留 log,但殘骸目錄已於 08-03 09:50 隨同批清理消失,provenance 現只存在於 auto-memory)——**制度教訓:清理 result-seg 前必須先確認沒有 prereg 要求保留的 artifact**。(b) 2026-08-04 `Docker/run.sh` 新掛 `-v /mnt/oil:/mnt/oil`(21T NAS,92% 滿),內含 `IR/OIL_PROJECT_dataset/full_band/` 與**兩個空的 `SAR/`、`UAV/`**,用途未文件化。
- **Git 狀態**:GT_expand HEAD 仍是 `f36e421`(2026-07-27)。**P3 全部產出未 commit**:`main/prithvi_deeplab.py`(ASPP rate 預註冊允許集)、3 個 config yaml、`main/test_prithvi_aspp_rates.py`、整個 `analysis/p3_aspp_rate/`、`analysis/p2_focal_tversky/` 的 replay 腳本與報告,以及 `Docker/run.sh` 的 /mnt/oil 掛載。**P4 全部產出亦未 commit**:`analysis/p4_operating_point/`(`preregistration.md` v1.0 FROZEN、`p4_report.md`、`atoms.json`、`p4_results.json`、`label_audit_report.md`、`verify_parity.py`+`parity_report.md`、`verify_threshold_operator.py`、`dump_prob.py`+`dump_prob.log`、`recon/fold{1,2,3}/gt_aware_recon/prediction_artifacts/` 360 個 NPZ)；`main/recon_gt_aware_module.py` 因 prediction-artifact 共識案被改動(核心 predict/score 五函式與 `f36e421` 逐位元相同,差異僅在輸出序列化,煙霧測試 4/5 景逐位元相同、1 景差 4.5e-06)。**依 08-03 誤刪教訓,P4 產物在此結案狀態下不得清理。**
- **下一步(2026-08-05 使用者裁決後更新;2026-08-06 移除已解決項目)**:
  1. **`/mnt/oil` — 已澄清,不是新研究軸。** 使用者說明:這只是一個 **share folder(放檔案用)**,底下的 `SAR/`、`UAV/` 空目錄純粹是分類佔位。→ 接手的 harness 不要據此推論要開多模態新方向,也不要把它列成待辦。
  2. **Prithvi+UPerNet 決定性 3-fold — 已裁決暫不排跑。** 使用者判斷「大致上不會出太大問題」(即決定性重跑預期不會翻盤結論),且這條線的價值定位是**大圖(全景 / sliding-window 分支 A)的開端**,不是現在要補的數字。→ 本項從「最大未償技術債」降為**已知且已接受的 caveat**:引用 0.4498 時必須註明「pre-fix legacy、單跑」,不得無前綴稱顯著;真要寫進投稿時再補跑。
  3. **(已收斂)** 2026-08-05 09:40 啟動的 mask / prediction artifact JSON RLE 序列化 schema 共識案已收斂,其輸出格式已被 P4 操作點實驗直接採用並跑到結案。
  4. **(已解決,不再阻擋)** P4 prereg 曾卡在 pooled vs S 評成敗的使用者裁決與 codex 8 條 BLOCKING,兩者皆已於 2026-08-06 前處理完畢,prereg v1.0 已凍結並結案,見上「最近完成事件」第 3 項與「下一棒接手點」。
- **2026-08-05 新增 Stop hook(Claude 端 harness 層強制)**:`/root/.claude/hooks/check_writeback_staleness.sh`,於每次 session 結束時比對 `result-seg/*/` 最新 run 目錄 mtime vs institution `research/oil_spill_project_status.md` mtime,有 run 比現況檔新就顯示回寫缺口警告。等同把 `30_maintenance.md` §4 季檢第 4 條自動化。**原因**:回寫規則本來就存在,但寫在「按需引用檔」`30_maintenance.md` 裡,觸發條件被鎖在只有決定要回寫時才會打開的檔案中,所以長期不會被觸發(本次 vault 落後 12 天即為此)。**注意這是 Claude 端專屬基礎設施,Codex 沒有**——Codex 接手時要靠人工自查同一條件。

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
