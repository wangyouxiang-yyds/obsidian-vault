# handoff.md — 工作階段交接檔

用途：任一 harness 接手時（例如 Claude token 用盡、使用者跳到 Codex 繼續），讀本檔即可掌握目前工作階段並繼續做工。
更新紀律：每完成一個工作段落就更新「目前狀態」，不要等 token 快用完才寫（真用完就來不及了）。
必寫事件:process 啟動/結束、fold 完成、共識案收斂。交接前檢查 STATUS_AS_OF 與訓練 log/最近事件的時間差,逾時標 STALE,不得宣稱最新。
同機接手：Claude Code 與 Codex 都在這台 Linux server 上，讀寫同一份 vault，接手不需要 commit/push。只有要讓 Windows 端的 vault checkout 看到最新狀態時才需要 commit + push。

## 目前狀態(每次更新覆蓋本節;本節只是摘要,研究細節唯一正本=institution research/oil_spill_project_status.md)
- STATUS_AS_OF:2026-07-24(更新者:Claude/obsidian-vault-manager;依 codex review 要求校正 freshness——舊 STATUS_AS_OF 停在 07-13,但本節下方共識紀錄早已有 07-24 條目)
- 正本路徑:/home/alanyh/.agents/institution/research/oil_spill_project_status.md(⚠️ 該正本抬頭仍停在 2026-07-05,由使用者另行處理,不在本次校正範圍)
- 最近完成事件:**訓練不可重現 bug 已修復並通過 bit-identical 驗收**——2026-07-21~22 定位根因為 Albumentations 2.0.8 `A.Compose` 內部 RNG 未鎖(torch/np/random seed 全數失效);commit `ba25391` 修復(train/val Compose 傳 seed + `set_augmentation_seed` + `_seed_worker` 三處);獨立雙跑 reproC/D 驗收全過(370 tensor 逐 bit 相同、`best.pt` SHA256 相同、119/119 場景預測逐像素一致)。連帶地,2026-07-16~18 完成的事件層級溯源稽核(12 事件群、350/355 姊妹景近重複、external-84 假設作廢)與 Evaluation Contract v1.0(M1/M2 + 12 事件群 cluster bootstrap)已取代舊 pooled+355-Wilcoxon 雙 gate。**此前所有 grid/screen 掃描與 grouped replay 數字全數降為 legacy single-run,不構成 confirmatory**。詳見 vault `wiki/experiments/20260716_資料溯源洩漏與評估合約v1.md`、`wiki/experiments/20260718_訓練不可重現根因與決定性修復.md`。
- 決定性正本重跑(2026-07-23 凍結,`analysis/p1_tversky_sweep/canonical_rerun_prereg.md`):baseline(FocalLoss)3-fold **已完成**(avg pooled_oil_iou=0.3447);**Tversky α0.3/β0.7 3-fold 訓練中**(唯一變因=loss,其餘逐項凍結,commit `ba25391` 在 HEAD);設看門狗於 **2026-07-25 07:45** 對整個 fold 發 SIGKILL 乾淨停止(非 mid-fold resume,防明早 08:00 停電),決定性下 from-scratch 重跑可證明與被中斷前的假設性完跑結果等價;`0.5/0.5`(Dice)正式比較延後,日後補跑需加 10 paired seeds 才能下 superiority 結論。
- 使用者資料計畫:5 個 2026 Atlantic 場景已完成 mask/JSON/VRT,刻意未加入 split,待使用者決策。
- 下一步(依 2026-07-24 codex review 共識,`decisions/20260724_Codex審查與決定性重跑優先序.md`):①確認 Tversky 三折安全跑完/停電後可證明等價重跑;②用決定性正本 predictions 重做 Evaluation Contract v1.0 grouped replay + LOGO,舊 grouped 數字(Tversky +0.053、UPerNet +0.037/+0.039)全降 legacy single-run,不可無前綴寫「顯著」;③全 vault 就地標記失效主張(本輪已完成,見 log.md);之後只排一個固定 0.3/0.7 的 Prithvi+UPerNet deterministic 3-fold,P2/0.5-sweep/Focal-Tversky 全部延後。
- 執行中程序:Tversky canonical rerun 訓練中(`train_log/nohup_canonical_tversky_run2.log`);看門狗 PGID=787,target=2026-07-25 07:45(`train_log/watchdog_tversky_deadline.log`);Prithvi 無程序、未占 GPU。

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
