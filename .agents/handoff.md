# handoff.md — 工作階段交接檔

用途：任一 harness 接手時（例如 Claude token 用盡、使用者跳到 Codex 繼續），讀本檔即可掌握目前工作階段並繼續做工。
更新紀律：每完成一個工作段落就更新「目前狀態」，不要等 token 快用完才寫（真用完就來不及了）。
必寫事件:process 啟動/結束、fold 完成、共識案收斂。交接前檢查 STATUS_AS_OF 與訓練 log/最近事件的時間差,逾時標 STALE,不得宣稱最新。
同機接手：Claude Code 與 Codex 都在這台 Linux server 上，讀寫同一份 vault，接手不需要 commit/push。只有要讓 Windows 端的 vault checkout 看到最新狀態時才需要 commit + push。

## 目前狀態(每次更新覆蓋本節;本節只是摘要,研究細節唯一正本=institution research/oil_spill_project_status.md)

- STATUS_AS_OF:2026-08-31(更新者:Claude 主 harness,Windows 端 vault checkout;本次更新=**僅 vault 文件層同步**,新增 Prithvi-EO-2.0 論文摘要頁+index+log,並結清債務 #5。**研究狀態未變**,下方「最近事件/執行中程序」仍停在 2026-08-25,若需最新研究進度請以 Linux 端 institution 正本為準)
- 前次全面同步:2026-08-25(主軸換軌後首次,補上 08-12→08-25 的空白)
- 正本路徑不變:`/home/alanyh/.agents/institution/research/oil_spill_project_status.md`
- ⚠ 本節先前停在 2026-08-06,漏記 19 天。以下按時序補齊。

### 主軸已換(2026-08-12)

- 研究主軸從 GT_expand 的「全場景 + 槓桿調校」轉為 **`OIL_PROJECT_MutiBand_Alan` 的 patch-based 海外 source → 台灣 target 轉移**。設計正本=Alan `docs/thesis_research_plan_v0.md`。
- 固定對照組(綁死,不再當變因):Prithvi-EO-2.0-300M encoder + DeepLabV3 ASPP(6,12,18) + Tversky α=0.3/β=0.7;主協定 `source_only_zero_shot`;研究單位=incident,不是 patch。
- GT_expand 全場景線=凍結的 legacy evidence,不是現行主軸。**要跑實驗一律在 Alan repo。**
- 2026-08-18:**11 月 deadline 取消**(計畫本身延續,不是停案)。

### 最近事件

1. **SAPP source event-group holdout 探索實驗完成(2026-08-25)** — Prithvi + SAPP(PPM bins 1/2/3/6)3-fold 全部正常 early-stop。source patch test Oil IoU `0.3840/0.3147/0.5026`,fold-macro mean±SD `0.4004±0.0950`。**尚不能判定 SAPP 優於 ASPP**:paired ASPP 對照 `configs/experiments/alan_patch_source_erm_v0.yaml` 仍是 `draft_not_authorized` / `run: false`,舊 ASPP 數字因 split/protocol 不同不可比。claim 上限=`provisional_event_group_disjoint_source_development`,不得稱 incident-disjoint、不得稱台灣 zero-shot。細節見正本 2026-08-25 條目。
2. **全場景 OOF v1 產物經使用者裁決刪除(2026-08-25)** — 203 MB / 68.6 GPU-h,**從未產出任何可報告指標**(`run_metrics.json` 從未生成),故刪除不推翻任何已成立結論。自足紀錄=Alan `docs/decisions/20260825_fullscene_oof_v1_退役與刪除.md`。連帶作廢兩份相反指示:`archive/fullscene_v1_2/README.md` 的「Do not delete」與 `docs/OUTPUTS.md` §6 遷移計畫(兩處均已加退役 banner)。
3. **文獻**:08-21/22 讀入七篇。Chang et al. 2024(SAR morphological attention U-Net,DOI `10.3390/s24206768`)已完成 Methods 統整,定位為 SAR 相鄰先例,**非 strict zero-shot benchmark**。

### 執行中程序

- **無訓練、無推論在跑,GPU 閒置。**

### 未償債務(接手者請優先處理)

1. **P6 α/β 掃描未結案回寫** — `GT_expand/analysis/p6_alpha_beta_sweep/`(產出 mtime 2026-08-23)在 institution 現況檔**沒有任何條目**,違反「結案當天回寫」。需有人核對產物後補寫,**不得憑印象寫**。
2. **權限層與制度矛盾** — `/mnt/backup/alanyh/.claude/settings.local.json` 的 allow 清單預先放行 `Bash(git push *)`、`Bash(git commit *)`、`Bash(pkill -f "main_runner.py")`、`Bash(python *)`、`Bash(conda run *)`,而 `00_diagnosis.md` C2 與 `20_judgment_rubrics.md` R3 要求這類不可逆動作逐次授權。目前只靠 harness 自律擋。**待使用者裁決是否收緊。**
3. **superpowers 外掛層未納入制度** — 每 session 注入 `EXTREMELY_IMPORTANT` 指令,institution 六份檔案零次提及,違反唯一正本原則。**待使用者裁決。**
4. **制度未定義「探索 vs 確認」兩檔速度** — SAPP 以 `authorized_exploratory` 執行,無 prereg、無共識紀錄。這在探索期未必是錯,但規則面沒有對應條文,落差需明文化。**待使用者裁決。**
5. ~~**vault `log.md` 的 2026-08-25 刪除紀錄尚未 commit**,等使用者授權。~~ **已結(2026-08-31)**:該紀錄已隨 commit `1d765fa`/`e982f07` 進入 master 並推送至 origin。

### 2026-08-25 harness 修繕(本次 session,已完成並驗證)

- `institution/scripts/audit_harness_parity.py`:移除對專案層 `.codex/config.toml` 的判定(Codex 從不讀它,見 `40_harness_continuity.md` §4),改查兩份索引 + 全域信任項;Alan repo 納入 `PROJECTS`。稽核由 FAIL 轉 **PASS agents=4 skills=4 projects=3**,孤兒檔改以 NOTE 提示。
- `~/.codex/config.toml`:補 Alan repo `trust_level = "trusted"`。
- Stop hook `check_writeback_staleness.sh`:原本只盯 `GT_expand/result-seg`,Alan 的 `artifacts/experiments/` 從不觸發;已改為同時掃描兩者(目錄深度不同,分開指定),三情境測試通過。
- Alan `CLAUDE.md` / `AGENTS.md`:移除「保留已刪除之外部 output」的過期指令。
- `30_maintenance.md` §7-3:同步為三專案索引 + 信任項。
- 各檔均留 `.bak-20260825`。

## Harness continuity

- PARITY_AS_OF:2026-08-25(更新者:Claude 主 harness)
- 狀態:Codex failover ready;Claude 不可用時可從共享狀態繼續可逆分析、實作、測試與 vault 工作。
- 驗證:`PARITY AUDIT: PASS (agents=4 skills=4 projects=3)`。另有 2 則 NOTE=GT_expand 與 0422 的專案層 `.codex/config.toml` 為 **inert orphan**(Codex 不讀),處置仍 open,見 `40_harness_continuity.md` §4。
- 已部署:4 個 Codex custom agents、4 個 vault skill symlinks、全域 `AGENTS.md`、三專案索引 + 三專案 Codex 信任項。
- Codex 專案 session 需同時寫 repo 與 vault 時,**唯一有效方法**是命令列傳入:`codex -C <repo> --add-dir /home/alanyh/obsidian-vault --add-dir /home/alanyh/.agents/institution`。專案層 config 無效,不要依賴它。
- 權限邊界:缺席 harness 不得被冒稱同意;重大決策交使用者仲裁;launch/kill/delete/commit/push/publish 仍需使用者明確授權。
- ⚠ Stop hook 是 **Claude 端專屬**基礎設施,Codex 沒有;Codex 接手時要人工自查同一條件。
- ⚠ institution 目錄**仍未納入 git**(`30_maintenance.md` §5 建議但未執行),改動只靠 `.bak-YYYYMMDD` 備份。
- Git:2026-08-25 的 vault 變更已 commit 並推送(`1d765fa`/`e982f07` 等);2026-08-31 的 vault 文件層變更亦已 commit+push(使用者授權)。⚠ institution 目錄不在 git 內,不受此同步涵蓋。

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
