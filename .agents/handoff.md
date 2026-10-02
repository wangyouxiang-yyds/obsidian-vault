# handoff.md — 工作階段交接檔

用途：任一 harness 接手時（例如 Claude token 用盡、使用者跳到 Codex 繼續），讀本檔即可掌握目前工作階段並繼續做工。
更新紀律：每完成一個工作段落就更新「目前狀態」，不要等 token 快用完才寫（真用完就來不及了）。
必寫事件:process 啟動/結束、fold 完成、共識案收斂。交接前檢查 STATUS_AS_OF 與訓練 log/最近事件的時間差,逾時標 STALE,不得宣稱最新。
同機接手：Claude Code 與 Codex 都在這台 Linux server 上，讀寫同一份 vault，接手不需要 commit/push。只有要讓 Windows 端的 vault checkout 看到最新狀態時才需要 commit + push。

## 目前狀態(每次更新覆蓋本節;本節只是摘要,研究細節唯一正本=institution research/oil_spill_project_status.md)

- STATUS_AS_OF:2026-10-02（更新者:Claude 主 harness 經 obsidian-vault-manager：**P21（tva02 組）訓練進行中**，預估約 2026-10-03 00:30 出 `P21_RESULTS.json`。**P22（GoM 大／中小分組五 run）已凍結並排隊**：Claude×Codex（session `01a0fbcb-b99e-7bf2-9066-c2e8a9ce14f4`）R1 REVISE_THEN_AGREE→R2 AGREE／BLOCKING NONE 收斂；正本 `analysis/p22_gom_size_split/DESIGN.md`（GT_expand commit `bce534b`）；五個 run（SM_raw/L_raw/SM_fixed10/L_fixed10/L_aligned）排 tmux `p22_queue`，等 P21 結束後自動開跑，約 2026-10-03 01:00 開始，最壞約 57h。詳見 `.agents/decisions/20261002_P22_GoM大中小分組五run設計.md`。⚠ **P22 跑完前不要 commit 對 main 核心碼／P22 腳本的修改**——`launch.sh` 每段開跑前會重新核對 manifest，若與凍結內容不符即中止整個排隊。⚠ **本次未刷新其他實驗進度，沿用舊條目者視為 STALE**。）
- STATUS_AS_OF:2026-10-01（更新者:Claude 主 harness 經 obsidian-vault-manager：**P20 由使用者裁決停止**（判定「沒有必要跑了」，kill `launch.sh` 與 `main_runner`）。已完成 4/6 組訓練：`rho19_S43`（best epoch46、val mIoU 0.6996）、`rho21_S43`（best epoch61、val mIoU 0.7114）、`rho23_S43`（best epoch60、val mIoU 0.7088）、`rho23_S44`（best epoch39、val mIoU 0.7011）；`rho21_S44` 停在 epoch42；`rho19_S44` 未跑。台灣推論與 `build_report` 皆未執行，**無 P20 結論**，比例維持 1:23。結果目錄 `/home/alanyh/oil_dataset/new/_p20_runs/P20_taiwan_rho_seeds/` 原地保留。詳見 `.agents/decisions/20260927_P20_P18_seed擴充設計.md` §「2026-10-01 使用者停止」。
  **main_runner 已接上 launch_gate 與共用 GPU flock**（GT_expand commit `7416743`）：鎖檔 `/home/alanyh/oil_dataset/new/.gpu_main_runner.lock`，`main_runner` 全程持有、非阻塞取得，拿不到即 exit 2；實測鎖競爭 exit 2、持有者退出後鎖可被取得、`test_launch_gate` 5 項 PASS。從此 `main_runner` 啟動需要 tracked 工作區乾淨＋config 已被 git 追蹤。
  **P21 已 launch**：GoM val-matched patch × Tversky，兩組（`tv03` 對照＝α0.3/β0.7 現行碼重跑、`tva02`＝α0.2/β0.8），seed 42，各約 14.5h，依序跑完後出報告。tmux `p21_run`，repo log `logs/p21_launch_20261001.log`，GT_expand HEAD `990d50f`。使用者授權偏離：`launch.sh` 移除「等待 P20 完成＋`P20_RESULTS.json` COMPLETE 標記」前置條件，因 P20 已停、永不產生該檔。launch 前各閘全 PASS（`launch_files.sha256` 29 檔、`--verify-frozen`、P17 `--parity` event-macro `0.45575147698056634`、val 全量 442/39 景/None=0、`launch_gate` PASS）。詳見 `.agents/decisions/20260930_P21_GoM_Tversky_a02設計.md` §「2026-10-01 launch」。
  下一步：約 29h 後（估 2026-10-02 深夜～10-03）出 `P21_RESULTS.json`；P22（GoM 大／中小分組）尚未設計。⚠ **本次未刷新其他實驗進度，沿用舊條目者視為 STALE**。）
- STATUS_AS_OF:2026-09-30（更新者:Claude 主 harness 經 obsidian-vault-manager：**P21（GoM val-matched patch × Tversky α=0.2/β=0.8 單 seed pilot）設計已凍結，尚未 launch**。Claude×Codex 3 回合同一 Codex session `01a0f04a-45ba-7b41-acbe-874029ebcc87` 收斂（R1 REVISE[5]→R2 REVISE[3]→R3 ACCEPT/BLOCKING=[]）。使用者定案：單 seed 42、編號 P21（覆寫卡 16「勿預占 P21」，GoM 大／中小分組實驗順延為 **P22**）、對照組在 P21 內以現行碼重跑 0.3/0.7（`tv03`）。判定規則：Δ_macro≥±0.04 且兩事件 Δ 皆同向 >±0.005 才算 PROMISING/HARMFUL，其餘 NULL_PILOT；P/R 與 TP/FP/FN 只描述不判定。已實跑檢查全 PASS（P17 parity 逐位元、val 442 筆全讀 None=0、`--verify-frozen`、選模核對）。**launch 前必須依序完成 DESIGN.md §8**：等 P20 COMPLETE → 把 launch_gate 與共用 GPU flock 接進 `main_runner.py` 並 commit → 驗證 flock 競爭行為（另一 main_runner 搶鎖須 exit 2）→ 建 `launch_files.sha256` → 使用者授權 `run:true` → 再 commit 一次 → tmux 執行 `launch.sh`；任何一步不得跳過或視為已自動完成。正本 `analysis/p21_gom_tversky_a02/DESIGN.md`；decision record=`.agents/decisions/20260930_P21_GoM_Tversky_a02設計.md`。⚠ **本次未刷新 P20 訓練進度，沿用舊條目者視為 STALE**。）
- STATUS_AS_OF:2026-09-29（更新者:Claude 主 harness 經 obsidian-vault-manager：**Git 版控制度 ACTIVE**。本條只記制度導入，未刷新任何實驗進度，其他條目沿用者視為 STALE。）
  - 制度正本：`/home/alanyh/.agents/institution/50_git_control.md`（ACTIVE；同日 Codex read-only 複述驗證 4/4 一致）。要點：已驗證工作段落 local commit=義務不需授權；只 `git add` 明列路徑（禁 `add -A`／`add .`／`commit -a`）；訊息第一行 `[claude|codex|user] <範圍>: <說明>`，由 `.githooks/commit-msg` 強制；push／改寫歷史／刪分支／merge 需使用者明確授權；交接記 HEAD hash 與未認領改動。
  - institution 目錄已 `git init`（本機、無 remote）：HEAD=`176645e`。README／20 R3／40 §2§4 已加指標。
  - GT_expand：HEAD=`2a783f5`（本機 main 領先 origin/main 3 個 commit，未 push：77449d2 hook、5d64de7 launch_gate 模組、2a783f5 索引指標）。
  - ⏳ launch 硬閘門 `main/launch_gate.py` 已完成測試但**尚未接進 main_runner.py**：P20 `launch_files.sha256:28` 鎖定 main_runner.py，接線延到 P20 結束後（規格見 50 號文件 §2「接線狀態」）。過渡期人工遵守：launch 前 `git status --untracked-files=no` 須為空、config 已追蹤；P20 啟動時 yaml run 翻轉須先 commit。
  - 未認領改動（待使用者決定）：GT_expand `analysis/size_tier_check/`、`analysis/size_tier_gallery/`（untracked）；`p11-geoagent-attempt2` 分支未 merge／未 push；root global safe.directory 是否加入兩 repo。
- STATUS_AS_OF:2026-09-21（更新者:Claude 主 harness 經 obsidian-vault-manager：降 token 案（雙 harness 降低 token 消耗方案）R3 收斂（雙方 ACCEPT／BLOCKING=[]）；Phase 0 完成；Phase 1（改 institution／vault 正本）未授權，未動任何制度檔。決議全文見 `.agents/decisions/20260921_雙harness降token消耗方案.md`。⚠ **本次未刷新 P16／其他實驗進度，沿用舊條目者視為 STALE，不得宣稱最新**。）
- STATUS_AS_OF:2026-09-20 21:15 Asia/Taipei（更新者:Claude 主 harness 經 obsidian-vault-manager：P16 首次送跑已中斷並刪除、經授權重跑，推翻下方 20:30 條目（其 PID／tmux／log／run 目錄皆已失效）。本次全部經唯讀實查覆核：20:17:47 啟動的 GoM 臂（main PID `23909`、tmux `p16_matched_s42`）於 2026-09-20 21:07 左右，在 `torch.save()` 寫 checkpoint 當下收到 `KeyboardInterrupt`（SIGINT）而死，停在 **Epoch 15/300**；非 OOM、非程式錯誤、非磁碟問題。NonGoM 臂從未啟動。中斷後 GPU 回到 idle（214 MiB／0%），tmux `p16_matched_s42` 已不存在。⚠ **成因未完全確證**：觀察到 tmux `claudeXcodex_review` 建立於 `21:07:59`，與死亡時間吻合到秒，**強烈暗示**該 tmux session 被關閉／取代導致 SIGINT 傳給前景程序群；但無直接證據（無 tmux server log 佐證），**不得寫成已確認的因果，亦不點名任何人為責任**。使用者於 2026-09-20 21:10 明確授權「刪掉重跑」（與 P14 2026-09-14 中斷後的裁決一致）。刪除前已重驗凍結性：`launch_files.sha256` 12 檔全 `OK`、`build_design.py --verify-frozen` `PASS`——中斷未污染任何凍結輸入或設計。已刪除 `20260920-201818_MS6_P16_GoM_matched1764_S42`（7.9 GB，含 `last.tmp` 3.46 GB 寫一半的殘骸），**不可復原**。刪除前已將 `args.yaml`／`data.yaml`／`results.csv`（15 epochs）與完整 log 另存於 `/home/alanyh/oil_dataset/new/_p16_runs/P16_gom_matched_patch/_interrupted_20260920-2107_record/`，log 改名 `P16_seed42_20260920_INTERRUPTED_ep15.log`（本次實查該目錄內容與此一致）。中斷 run 的 15 epochs 最佳為 Epoch 13 `Val mIoU 0.699347／Oil IoU 0.436575`——⚠ **僅供中斷存證，非可報告指標，不得引用、不得寫成任何 P16 結果**。21:12:40 已由 epoch 0 重跑：main PID `28418`（本次實查存活，elapsed 02:55），**detached** tmux `p16_restart_2111`（刻意不綁終端機，降低再次被 SIGINT 的風險，本次實查該 session 存在，建立於 21:12:09），同一 `launch.sh`、同一凍結 config，seed42 已鎖。新 GoM run 目錄=`/home/alanyh/oil_dataset/new/_p16_runs/P16_gom_matched_patch/20260920-211240_MS6_P16_GoM_matched1764_S42`，log=`/home/alanyh/oil_dataset/new/_p16_runs/P16_gom_matched_patch/P16_seed42_restart_20260920-2111.log`。兩臂串行設定不變。21:15 實查 log：Epoch 1/300 訓練中（52/110 steps），GPU 使用中（9972 MiB／11%），程序存活。log 中的 `fatal: detected dubious ownership in repository` 為既有現象（中斷前那份 log 同樣出現 1 次），非本次新增問題。事前卡 `.agents/decisions/20260920_P16_GoM等patch單seed設計.md` 狀態仍為 `FROZEN_prelaunch`、SHA256 實查=`fe5410196e177946bb03cc2a203da16345cc904bbe5808ff567746964628efcb`，凍結後不回改；設計／資料／端點全部沿用，無任何變更。完成時間重估：兩臂約 28 小時，估 2026-09-22 凌晨前後（推估，非保證）。institution 正本已於 21:12 寫入 CURRENT 條目記載本次中斷與重跑，本次核對兩層內容一致。）
- STATUS_AS_OF:2026-09-20 20:30 Asia/Taipei（更新者:Claude 主 harness 經 obsidian-vault-manager：補上 P16 已啟動證據，推翻下方 20:17 條目「等待主線 snapshot 與 launch；本次回寫尚無啟動證據」——launch 實際發生於同日 20:17:47，比該次回寫晚 16 秒。本次全部經唯讀實查覆核：獨立 tmux session=`p16_matched_s42`（建立於 2026-09-20 20:17:47），main PID=`23909`（實查時已運行 11:27，process 存活）；兩臂串行 `MS6_P16_GoM_matched1764_S42`→`MS6_P16_NonGoM_matched1764_S42`，config 內兩臂皆 `run: true`。config=`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/main/experiments_P16_gom_matched_patch.yaml`（SHA256 實查=`5c8fdde49bd4140dc156699143b67d51a86f708f04455b52456d9f3a63ce290a`，與 institution 正本一致）。結果根目錄=`/home/alanyh/oil_dataset/new/_p16_runs/P16_gom_matched_patch/`；log=`P16_seed42_20260920.log`；GoM run 目錄=`20260920-201818_MS6_P16_GoM_matched1764_S42`。訓練預算：300 epochs、patience301（實質不早停）、110 updates/epoch=33,000 updates/臂，seed42。進度（2026-09-20 20:28～20:29 實查 log）：GoM 臂已完成 Epoch 1–3、Epoch 4 進行中；Epoch 2 記錄 V-mIoU 0.5635、Target(Oil) IoU 0.2187（Epoch 3 已進一步到 V-mIoU 0.6038／Oil 0.2684，僅供進度參考，**非可報告最終指標**）。依觀察到的每 epoch 約 2.8 分推估，單臂約 14 小時、兩臂約 28 小時，粗估 2026-09-22 凌晨前後才有完整結果——此為推估非已完成，本次不宣稱、不預測任何 P16 最終結果。事前卡`.agents/decisions/20260920_P16_GoM等patch單seed設計.md`狀態仍為`FROZEN_prelaunch`、SHA256 實查=`fe5410196e177946bb03cc2a203da16345cc904bbe5808ff567746964628efcb`，凍結後不回改；launch 前 12 檔 `launch_files.sha256` snapshot 與 `build_design.py --verify-frozen` 檢查均已通過（沿用凍結後既有紀錄，本次未重跑）。institution 正本已於 20:18:32 寫入 CURRENT 條目記載此次 launch，本次核對兩層內容一致。）⚠ **已於同日 21:15 被上方新 STATUS_AS_OF 推翻**：該 run 已於同日 21:07 中斷並刪除，PID `23909`／tmux `p16_matched_s42`／log `P16_seed42_20260920.log`／run 目錄 `20260920-201818_MS6_P16_GoM_matched1764_S42` 皆已失效，本行原樣保留僅供追溯。
- STATUS_AS_OF:2026-09-20 20:17 Asia/Taipei（Codex 主 harness 經 vault-manager：P16 事前卡已凍結，獨立 runtime gates 已 PASS，等待主線 snapshot 與 launch；本次回寫尚無啟動證據。以下舊實驗進度非本次全面刷新，不得視為即時狀態。）⚠ **已於同日 20:30 被上方新 STATUS_AS_OF 推翻**：launch 實際發生於同日 20:17:47（比本次回寫晚 16 秒），「尚無啟動證據」一句已不成立，本行原樣保留僅供追溯。
- P16 事前卡：`.agents/decisions/20260920_P16_GoM等patch單seed設計.md`，`FROZEN_prelaunch`；每臂 584 GT＋1,180 BG，GoM 固定一次分層無放回下採樣，NonGoM 全保留；seed42、各 33,000 updates、best source-Val mIoU 選模。R3 正式協商 ACCEPT／BLOCKING=[]，input/config audit PASS；P16 新 report runtime parity 與 snapshot 校驗仍為 launch gates。config=`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/main/experiments_P16_gom_matched_patch.yaml`；results=`/home/alanyh/oil_dataset/new/_p16_runs/P16_gom_matched_patch`（已建目錄，不代表已啟動）。
- 凍結後 runtime gates 更新：主線回報 P16 新 report 對 P12 endpoint exact parity PASS、P14 兩臂 checkpoint provenance PASS（GoM epoch46／updates7866；NonGoM epoch64／updates7040）、獨立 `--verify-frozen` PASS。卡片不回改，SHA256=`fe5410196e177946bb03cc2a203da16345cc904bbe5808ff567746964628efcb`；等待主線留存 snapshot 並送跑。
- STATUS_AS_OF:2026-09-18(更新者:Claude 主 harness 經 obsidian-vault-manager:回補 P15 GoM-only 曝光量倍增實驗——此實驗在 2026-09-16 已由使用者授權、Codex 啟動，但截至 2026-09-18 之前於 institution 與 vault 兩層皆無記載，違反 `40_harness_continuity.md` §2 共用狀態要求，本次補建九步驟卡與 handoff 條目。)
- STATUS_AS_OF(舊):2026-09-15 11:35 Asia/Taipei(更新者:Codex 主 harness 經 obsidian-vault-manager:P14 GoM-only/full 與 NonGoM-only/full 兩臂均已 early stop、完成固定 Taiwan test 與 Excel 彙整；tmux session 自然退出、GPU idle，屬正常完成而非中斷。)
- 前次全面同步:2026-08-31((a) Windows 端 vault checkout:新增 Prithvi-EO-2.0 論文摘要頁+index+log,並結清債務 #5;(b) Linux 端:背景 patch 配額三部曲 P7→P8→P9 週結回寫)
- 正本路徑不變:`/home/alanyh/.agents/institution/research/oil_spill_project_status.md`

### P15 GoM-only 曝光量倍增（2026-09-16 授權、Codex 啟動，2026-09-18 執行中，回補記載）

- ⚠ **回補說明**：P15 在 2026-09-18 之前於 institution 現況檔與本 handoff 兩層共用狀態都是零記載，`.agents/decisions/` 也沒有設計檔，違反 `40_harness_continuity.md` §2（failover 靠共用狀態）。本次僅回補記載，不變更任何 process 或 config。
- 實驗內容：GoM-only 曝光量倍增（Original vs Rep4496）。unique 訓練資料完全固定（oil 1188 / bg 1550），只把每 epoch 的 augmented 曝光量提高到 4496 virtual patches（oil 1951 + bg 2545）。
- seeds 42/43/44 同 seed 配對，共 6 臂（`MS6_P15_GoM_Original_S42/S43/S44` + `MS6_P15_GoM_Rep4496_S42/S43/S44`）。截至 2026-09-18 10:14，已完成 5 臂，第 6 臂 `MS6_P15_GoM_Rep4496_S44` 仍在跑（epoch 100、未改善 36/50）。
- config：`main/experiments_P15_gom_rep4502.yaml`（於 `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/`）；結果根目錄：`/home/alanyh/oil_dataset/new/_p15_runs/P15_gom_rep4496/`。
- launch 授權：使用者 2026-09-16（記於 YAML `_warning`），由 Codex 啟動；main PID `2830515`。
- 九步驟卡（1-6 部分事前凍結，7-9 待結果）：`.agents/decisions/20260918_P15_GoM曝光量倍增設計_部分事前凍結.md`。

### P14 GoM full-source ablation（2026-09-14～15，兩臂已完成）

- 23:55 Asia/Taipei，使用者明確授權刪除中斷 run 並由 epoch 0 重跑。舊 4.7 GB 目錄 `20260914-154828_MS6_P14_GoM_only_full_R1` 與空白 `P14_gom_full_ablation_20260914.log` 已刪除且不可復原。
- 23:56 以獨立 tmux session=`p14_gom_full_restart` 重啟同一兩臂 runner；main PID=`2052764`，完整 log=`/home/alanyh/oil_dataset/new/_p14_runs/P14_gom_full_ablation/P14_gom_full_ablation_restart_20260914-2355.log`，新 GoM run=`20260914-235613_MS6_P14_GoM_only_full_R1`。
- GoM-only/full 已完成：Early stop 於 Epoch 96；best Epoch 46，source-Val mIoU=`0.730333`、Val Oil IoU=`0.493401`，共 `16416` optimizer updates、耗時 `373.23 min`。固定 Taiwan test 已用 `best.pt` 完成 9 scenes：pooled Oil IoU=`0.4187`、scene-macro Oil IoU=`0.3657`、scene-macro mIoU=`0.6498`。
- NonGoM run=`20260915-061741_MS6_P14_NonGoM_only_full_R1` 已完成：Early stop 於 Epoch 114；best Epoch 64，source-Val mIoU=`0.653613`、Val Oil IoU=`0.341589`。固定 Taiwan test 已用 `best.pt` 完成 9 scenes：pooled Oil IoU=`0.2037`、scene-macro Oil IoU=`0.2942`、scene-macro mIoU=`0.6149`。
- runner 已正常完成全部任務；log 末尾確認「所有任務執行完畢」且 Excel 成功產出。tmux session 自然退出、GPU idle，並非再次中斷。彙整檔=`/home/alanyh/oil_dataset/new/_p14_runs/P14_gom_full_ablation/result_P14_GoM_full_ablation_restructured.xlsx`。

### P13 patch-balanced 台灣 source sensitivity（2026-09-11，已定案、未啟動）

- 與 pixel-balanced 平行；8 arms=`A/B×G1/G2/G3/Mixed`，單 seed42。A=164 oil+180 BG=344；B=1,658+1,820=3,478；Mixed 配額為 A oil 55/55/54、BG 60/60/60，B oil 553/553/552、BG 607/607/606。
- 300 epochs/patience50/best/batch16；以 source-matched non-independent train proxy 選模，固定台灣137 patches/9 scenes/2 events；runner 已加 `epoch_to_best`/`updates_to_best`/`stop_epoch`。
- 限制：單 seed、非獨立 validation、B oil validation exposure 不同；不得作海域因果、顯著性或跨 seed 穩健性聲稱。8 arms 全部 `run:false`；任何 launch 需使用者新授權。
- Claude×Codex 第4回合 `ACCEPT/BLOCKING=[]`；詳見 `.agents/decisions/20260911_P13_patch-balanced台灣source_sensitivity實驗設計.md`。

### P11 GeoAgent／A2C 尺度配置設計與建碼(2026-09-02,程式就緒、draft fail-closed)

- 使用者明示要多方嘗試 RL。Codex×Claude 正式方法協商六輪於 Round 6 `ACCEPT/BLOCKING=[]`；程式初審 `REVISE` 後修正，Claude 聚焦複審最終 `VERDICT: ACCEPT/BLOCKING: NONE`。
- 全部程式只存在隔離 worktree：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_geoagent_dev`；原 repo 與 P10 未碰。Track A=fixed local/3×/5× + supervised atom regressor/Dinkelbach；Track B=hard-budget A2C + fixed/random/supervised/oracle 同成本比較。
- development 僅 NOAA319；named-event 36 保持 sealed。兩份 config 均 `DRAFT_NOT_AUTHORIZED`、`run:false`；CPU audit 與 launch 各自 fail closed。
- CPU synthetic selftest **16/16 PASS**，`py_compile` 通過；pytest-style modules 已寫，但環境沒有 pytest，故 suite 未執行。未讀真資料、未載 Prithvi、未用 GPU、未訓練，亦未 commit/push。
- 下一棒：等 P10 完成後，仍須使用者分別明確授權 (1) 真實資料 CPU data/geometry audit、(2) prereg freeze/commit、(3) GPU launch；三項不可相互推定。詳見 `.agents/decisions/20260902_GeoAgent_A2C尺度配置實驗設計與建碼.md`。

### P10 油背像素比例六點格實驗結案(2026-09-01 授權,09-01~03 訓練,09-04 分析,09-05 圖表,09-06 vault 回填)

- **已結案,判定 `RETAIN_23.0187_protocol_continuity`**。使用者於 2026-09-01 明示要求在 P9 的 15~23 bracket 內加測 ρ=17/19/21,細化為 6 點格 ρ∈{15,17,19,21,23.0187,30}(15/23.0187/30 沿用 P9 不重跑);主持 harness 已完整告知功效不足(頂點增益上限僅 +0.0020,比單一對比 sim CI 半寬 ±0.0139 小一個量級)後使用者仍重申決定執行,故另立決議 `.agents/decisions/20260901_P10油背比例細格研究設計.md`,不沿用 P9 決議。
- rho17/19/21 三臂共 9 個 run 全數完成(2026-09-01 09:28 → 09-03 23:00),七道建構閘、parity gate、regression vs P9 gate 全過。
- **與 P9 不同:峰位「有」被辨識**——E1 峰位(內點五臂)ρ\* = 19.751,bootstrap CI [18.873, 21.091] 排除正典 23.0187;E3 選擇機率 rho19 = 0.9255(`peak_identified = True`)。但 **E2 峰增益 G = +0.0064,同時 CI [−0.00946, +0.02222] 跨 0**,未穩定超過 +0.02 實質門檻。預註冊換值規則(三條須同時成立)六臂全不合格,故**維持正典 ρ=23.0187**,理由記為 protocol continuity、非經驗最優。
- exposure audit 判定 NOT_CENSORED(18 折皆乾淨早停),但 steps-to-best 跨臂差 2.19 倍、與 S 的 Spearman ρ=+0.829,estimand 為完整政策包敏感度而非純 ρ 因果效應,已列入 Limitations。
- 事前功效分析(ρ≈20.54/+0.0020)與事後配適(ρ\*=20.565/+0.0023)幾乎完全命中,是「先算功效再花 GPU」的正面方法論案例。
- 決議 §10.2 原訂「E3 max < 0.5 則 ρ 軸正式結案為已耗盡的槓桿」的自動觸發條件**本次未成立**(E3 max = 0.9255 ≥ 0.5)。**ρ 軸是否正式結案為已耗盡的槓桿,需使用者另行裁決,不可視為已自動判定**——這是本次 P10 唯一的下一棒待裁決事項。
- 詳見 vault `wiki/experiments/20260904_P10油背像素比例六點格_峰位辨識但增益不足.md` 與決議檔 §11 結案回填。
- ⚠ GT_expand repo `analysis/p10_rho_grid/` 下 11 個結果產物(primary_results.json、exposure_audit.md/csv、四張圖、xlsx、三支腳本)**仍 untracked 未進版控**,待 commit。

### P9 油背像素比例 bracket 敏感度實驗結案(2026-08-31)

- **已結案,判定 inconclusive**。ρ∈{15, 23.0187, 30} 不等距 bracket,29.6 GPU-h,rc=0。S₃₀−S₂₃=−0.0201(simultaneous CI[−0.0328,−0.0074] 排除 0 但跨 −0.02 實質門檻);曲率 C=+0.0174(CI[+0.0065,+0.0284] 排除 0 但跨 +0.02),反應面呈凹形。現行 ρ=23.0187 三個受測水位中 S 最高(0.4606),**因 protocol continuity 暫留,不得稱經驗最佳或全域 optimum**。exposure audit 判定 NOT_CENSORED(九折皆乾淨早停),故曲率/等價詮釋未被禁止,但 rho15 只拿到 rho23 的 58% optimizer 更新,估計混有訓練預算效應。詳見 vault [[20260831_P9油背像素比例bracket_inconclusive與凹性發現]]。
- 連帶:P7(背景配額分佈)已於 08-26 判定不予採用(ΔS=−0.0197 CI 全負);P8(擴充候選池)08-28 因 container 磁碟配額故障於 fold3 中止,使用者裁決**全線砍除**(訓練結果已刪除),僅保留其凍結候選池 `bg_coords.tsv` 供 P9 上臂沿用。三者合稱週結,詳見 vault [[20260831_週結_背景patch配額三部曲P7P8P9]]。

### P9 待辦(接手者請留意)

1. **論文等價段落須改寫為 inconclusive 版本** — 決議 `.agents/decisions/20260828_油背像素比例敏感度研究設計.md` §論文措辭 的等價版前提(all pairwise differences 落在 ±0.02 內、無實質偏離 log-linear)兩者皆不成立,**該段落不得使用**。
2. **不得事後追加水位、seed、單臂延長或重跑** — 除非使用者另行明確授權;決議 §9 明文禁止,不得沿用本決議自行延伸。

### 研究主軸現況(2026-09-17 更新)

- 研究主軸=`OIL_PROJECT_MutiBand_GT_expand`,唯一跑實驗的 repo。
- 2026-08-12 曾將主軸定位轉往 `OIL_PROJECT_MutiBand_Alan` 的 patch-based 海外 source → 台灣 target 轉移;該定位已於 2026-08-25 裁決撤銷,2026-09-17 使用者裁決連同 Alan 整個目錄(14 GB)一併實體刪除(已驗證不存在)。**不再是任何形式的待辦或參考。**
- 固定對照組仍綁死(不再當變因):Prithvi-EO-2.0-300M encoder + DeepLabV3 ASPP(6,12,18) + Tversky α=0.3/β=0.7。
- 台灣 transfer / `source_only_zero_shot` 這條軸隨 2026-08-25 裁決擱置。歷史設計正本(已擱置,僅供追溯)=`docs/thesis_research_plan_v0.md`,現存於 `/home/alanyh/.agents/institution/archive/Alan_textual_record_20260917.tar.gz`。

### 最近事件

1. **SAPP source event-group holdout 探索實驗完成(2026-08-25)** — Prithvi + SAPP(PPM bins 1/2/3/6)3-fold 全部正常 early-stop。source patch test Oil IoU `0.3840/0.3147/0.5026`,fold-macro mean±SD `0.4004±0.0950`。**尚不能判定 SAPP 優於 ASPP**:paired ASPP 對照 `configs/experiments/alan_patch_source_erm_v0.yaml` 仍是 `draft_not_authorized` / `run: false`,舊 ASPP 數字因 split/protocol 不同不可比。claim 上限=`provisional_event_group_disjoint_source_development`,不得稱 incident-disjoint、不得稱台灣 zero-shot。細節見正本 2026-08-25 條目。
2. **全場景 OOF v1 產物經使用者裁決刪除(2026-08-25)** — 203 MB / 68.6 GPU-h,**從未產出任何可報告指標**(`run_metrics.json` 從未生成),故刪除不推翻任何已成立結論。自足紀錄=tar 內 `docs/decisions/20260825_fullscene_oof_v1_退役與刪除.md`(`/home/alanyh/.agents/institution/archive/Alan_textual_record_20260917.tar.gz`)。連帶作廢兩份相反指示:`archive/fullscene_v1_2/README.md` 的「Do not delete」與 `docs/OUTPUTS.md` §6 遷移計畫(兩處均已加退役 banner)。
3. **文獻**:08-21/22 讀入七篇。Chang et al. 2024(SAR morphological attention U-Net,DOI `10.3390/s24206768`)已完成 Methods 統整,定位為 SAR 相鄰先例,**非 strict zero-shot benchmark**。
4. **2026-09-29 P20 launch 後在閘 C 中止（exact loss 相等設計錯誤，P8 已證無 GPU bitwise 決定性）；待改閘 C＋刪 P20_parity 後重跑。**
5. **2026-09-29 P20 閘 C/B 改容差（使用者授權、未經 Codex）後第三次 launch 成功，tmux p20_run3 正式訓練中（約 3 天）。**

### 執行中程序

- **目前有執行中的 P16 程序（重跑；首次送跑已中斷並刪除，詳見上方 21:15 STATUS_AS_OF）**：detached tmux session=`p16_restart_2111`（2026-09-20 21:12:09 建立），main PID=`28418`，兩臂串行 GoM→NonGoM，seed42、300 epochs／patience301／33,000 updates/臂。config=`main/experiments_P16_gom_matched_patch.yaml`；log=`/home/alanyh/oil_dataset/new/_p16_runs/P16_gom_matched_patch/P16_seed42_restart_20260920-2111.log`；GoM run 目錄=`/home/alanyh/oil_dataset/new/_p16_runs/P16_gom_matched_patch/20260920-211240_MS6_P16_GoM_matched1764_S42`。2026-09-20 21:15 實查：GoM 臂 Epoch 1/300 訓練中（52/110 steps），程序存活，GPU 使用中（9972 MiB／11%），僅為進度數字非最終結果；粗估兩臂共需時約 28 小時（約 2026-09-22 凌晨前後）。首次送跑（PID `23909`、tmux `p16_matched_s42`）已於同日 21:07 遭 SIGINT 中斷（Epoch 15/300），不可復原地刪除，於 21:10 獲使用者授權重跑。
- P14 兩臂 runner 已於 2026-09-15 11:35 前正常完成；tmux session=`p14_gom_full_restart` 自然退出、GPU idle。log=`/home/alanyh/oil_dataset/new/_p14_runs/P14_gom_full_ablation/P14_gom_full_ablation_restart_20260914-2355.log`；Excel=`/home/alanyh/oil_dataset/new/_p14_runs/P14_gom_full_ablation/result_P14_GoM_full_ablation_restructured.xlsx`。

### 未償債務(接手者請優先處理)

1. **P6 α/β 掃描未結案回寫** — `GT_expand/analysis/p6_alpha_beta_sweep/`(產出 mtime 2026-08-23)在 institution 現況檔**沒有任何條目**,違反「結案當天回寫」。需有人核對產物後補寫,**不得憑印象寫**。
2. **權限層與制度矛盾** — `/mnt/backup/alanyh/.claude/settings.local.json` 的 allow 清單預先放行 `Bash(git push *)`、`Bash(git commit *)`、`Bash(pkill -f "main_runner.py")`、`Bash(python *)`、`Bash(conda run *)`,而 `00_diagnosis.md` C2 與 `20_judgment_rubrics.md` R3 要求這類不可逆動作逐次授權。目前只靠 harness 自律擋。**待使用者裁決是否收緊。**
3. **superpowers 外掛層未納入制度** — 每 session 注入 `EXTREMELY_IMPORTANT` 指令,institution 六份檔案零次提及,違反唯一正本原則。**待使用者裁決。**
4. **制度未定義「探索 vs 確認」兩檔速度** — SAPP 以 `authorized_exploratory` 執行,無 prereg、無共識紀錄。這在探索期未必是錯,但規則面沒有對應條文,落差需明文化。**待使用者裁決。**
5. ~~**vault `log.md` 的 2026-08-25 刪除紀錄尚未 commit**,等使用者授權。~~ **已結(2026-08-31)**:該紀錄已隨 commit `1d765fa`/`e982f07` 進入 master 並推送至 origin。

### 2026-08-25 harness 修繕(本次 session,已完成並驗證)

⚠ 2026-09-17 supersede:因 Alan 目錄實體刪除,以下三項已失效並反向調整——`audit_harness_parity.py` 的 PROJECTS 已移除 Alan、`~/.codex/config.toml` 的 Alan trust_level 區塊已移除、Stop hook 的 ALAN_EXP 分支已移除。以下內文為當時紀錄,原樣保留。

- `institution/scripts/audit_harness_parity.py`:移除對專案層 `.codex/config.toml` 的判定(Codex 從不讀它,見 `40_harness_continuity.md` §4),改查兩份索引 + 全域信任項;Alan repo 納入 `PROJECTS`。稽核由 FAIL 轉 **PASS agents=4 skills=4 projects=3**,孤兒檔改以 NOTE 提示。
- `~/.codex/config.toml`:補 Alan repo `trust_level = "trusted"`。
- Stop hook `check_writeback_staleness.sh`:原本只盯 `GT_expand/result-seg`,Alan 的 `artifacts/experiments/` 從不觸發;已改為同時掃描兩者(目錄深度不同,分開指定),三情境測試通過。
- Alan `CLAUDE.md` / `AGENTS.md`:移除「保留已刪除之外部 output」的過期指令。
- `30_maintenance.md` §7-3:同步為三專案索引 + 信任項。
- 各檔均留 `.bak-20260825`。

## Harness continuity

- PARITY_AS_OF:2026-09-17(更新者:Claude 主 harness)
- 狀態:Codex failover ready;Claude 不可用時可從共享狀態繼續可逆分析、實作、測試與 vault 工作。
- 驗證:`PARITY AUDIT: PASS (agents=3 skills=4 projects=2)`。另有 2 則 NOTE=GT_expand 與 0422 的專案層 `.codex/config.toml` 為 **inert orphan**(Codex 不讀),處置仍 open,見 `40_harness_continuity.md` §4。
- 已部署:3 個 Codex custom agents（agy bridge 已於 2026-09-17 停用）、4 個 vault skill symlinks、全域 `AGENTS.md`、兩專案索引 + 兩專案 Codex 信任項（GT_expand、0422）。
- ⚠ agy/Antigravity 已於 2026-09-17 使用者裁決停用,不得分派任何任務;此為可逆停用非永久廢除;復活程序見 institution `10_model_dispatch.md`「agy 停用」節。
- Codex 專案 session 需同時寫 repo 與 vault 時,**唯一有效方法**是命令列傳入:`codex -C <repo> --add-dir /home/alanyh/obsidian-vault --add-dir /home/alanyh/.agents/institution`。專案層 config 無效,不要依賴它。
- 權限邊界:缺席 harness 不得被冒稱同意;重大決策交使用者仲裁;launch/kill/delete/commit/push/publish 仍需使用者明確授權。
- ⚠ Stop hook 是 **Claude 端專屬**基礎設施,Codex 沒有;Codex 接手時要人工自查同一條件。
- ⚠ institution 目錄**仍未納入 git**(`30_maintenance.md` §5 建議但未執行),改動只靠 `.bak-YYYYMMDD` 備份。
- Git:2026-08-25 的 vault 變更已 commit 並推送(`1d765fa`/`e982f07` 等);2026-08-31 的 vault 文件層變更亦已 commit+push(使用者授權)。⚠ institution 目錄不在 git 內,不受此同步涵蓋。

## 共識紀錄
- 2026-10-01｜GoM→台灣遷移優勢的機制分析計畫骨架＋DG survey 方向｜Claude×Codex 3 回合同一 Codex session `01a0f50c-fe5d-7430-b6bb-7d75c002d19b`（gpt-5.6-sol/high，CLI 0.159.2）收斂：R1 盲提案雙方同向（研究問題改寫為「解釋 05-22 特定 recall 優勢、天使輪無改善」）→R2 REVISE［①primary 距離未操作化 ②2c 獨立單位 ③2b 介入參照］→R3 雙方 ACCEPT/BLOCKING=[]。決議：Phase 1＝retrospective preregistered audit（單一 primary＝pristine Prithvi class-conditional MMD，D=½D_oil+½D_water；H1 方向不符即拒絕）；Phase 2 全需授權、2c 預期不可行（P5 算術）；survey 分四欄（正式 DG／相鄰 TTA·UDA／油污跨區域實證／成像物理）；outline 分 operational vs oracle-patch 評估。⚠ codex 0.159.2 沙箱無法讀檔（landlock 修法失效），本案全文內嵌不受影響。僅 survey＋分析骨架，未授權任何執行｜詳見 decisions/20261001_GoM台灣遷移機制分析與DG_survey方向.md
- 2026-09-30｜P21 GoM val-matched patch × Tversky α=0.2/β=0.8 單 seed pilot 設計｜Claude×Codex 3 回合同一 Codex session `01a0f04a-45ba-7b41-acbe-874029ebcc87` 收斂：R1 REVISE[5 項：門檻高於推論漂移＋收窄 RQ／P17 舊碼(ca0036…)不等於現行(801fa5…)遞移等價不足／build_design 以 P17 config sha 為權威核 test=137 與 loader／report fail-closed／launch 需 manifest＋鎖＋P20 COMPLETE 標記]→R2 REVISE[B1 三核心 sha 加同一乾淨 commit／B2 val 442 筆全讀 None=0／B3 GPU flock 放進正式入口]→R3 雙方 ACCEPT/BLOCKING=[]（Codex 保留兩點執行期要求：接線後實測鎖競爭 exit 2、run:true 後再 commit）。使用者定案：單 seed42、編號 P21（覆寫卡 16「勿預占 P21」，GoM 大／中小順延為 P22）、對照組 P21 內現行碼重跑 0.3/0.7。判定 Δ_macro≥±0.04＋兩事件同向>±0.005；R2/R3 header 顯示 resume 未釘死模型、實際以 `gpt-6.1-sol` 執行（非使用者主動切換）。凍結、未接 launch_gate、未 launch｜詳見 decisions/20260930_P21_GoM_Tversky_a02設計.md
- 2026-09-29 GoM 大／中小組 BG 配置＋訓練控制 D2：像素 1:23（使用者裁決）、BG 只取本組景（中小 250／大 2,103）、中小 P8 輪替池 2,103、共用 val 312+505 BG、21,900 steps、last.pt 主判定；Codex R1 主張共用 donor/兩 val panel，R2 接受本組景，R3 ACCEPT｜詳見 decisions/20260929_GoM大中小組BG配置與訓練控制.md
- 2026-09-29 GoM 大／中小分組規則＋切點：patch 交疊最多之整景 8 連通 GT 區塊面積 ≥14,959 px（排除天使輪 348 景上三分位）＝大；Claude 提案、Codex R1 主張寬度後於 R3 ACCEPT；BG 另案｜詳見 decisions/20260929_GoM大中小分組規則與切點.md
- 2026-09-27｜P20 P18 seed 擴充（ρ19/21/23 × seeds 43、44）設計｜Claude×Codex 4 回合同一 Codex session `01a0e2d5-29a7-7501-9c5b-3eeaff23dc71` 收斂：R1 ACCEPT→R2 REVISE[1,2]→R3 REVISE[1,2,3]→R4 雙方 ACCEPT/BLOCKING=[]；判定 M：best 下三 seed 平均 event-macro Δ≥0.04＋兩事件平均 Δ>0＋S43/S44 macro Δ>0，改協定需 best SUPPORT 且 last≠REVERSE，無候選→維持 23；經功效模擬取代 R1 嚴格規則（+0.04 時 8.1%→約 53%）；新增閘 B（test-mode 推論 parity）、閘 C（第 1 epoch 訓練 parity，因共用核心 09-26 被改）；FROZEN_prelaunch、全 run:false、未 launch，launch 需使用者授權且自動排在 P19 之後｜詳見 decisions/20260927_P20_P18_seed擴充設計.md
- 2026-09-24｜P19 GoM 形態分組獨立訓練（中小油污組／大油污組）設計｜Claude×Codex 3 回合同一 Codex session `01a0d344-d683-7293-80b3-1ac2c9f26fec` 收斂：R1 REVISE[1,2]→R2 REVISE[B1,B2,B3]→R3 雙方 ACCEPT/BLOCKING=[]；三組 MS-538／Broad-538／Mixed-538（新混合對照，P17 降為歷史參考），皆 538 oil＋630 BG、21,900 updates、維持既有 loader 與 protocol_sha256（estimand＝資料政策 bundle）；三組共用 GT oil-pixel-balanced val（中小 206＋broad k≈36、BG 310）；依事件分開門檻（方案 A 3 seeds：天使輪 0.03／taiwan_file 0.06／P5 0.04 且 3/3 同向；方案 B 單 seed：0.06／0.15／0.08）；台灣 inference 延後至全部 run 完成、報表程式首 run 前凍結；GT 路由組合僅作非決策框。**尚待使用者選方案 A/B（及確認維持 protocol）才凍結**｜詳見 decisions/20260924_P19_GoM形態分組獨立訓練設計.md｜**2026-09-24 更新：使用者選方案 A、維持 loader；已凍結（FROZEN_prelaunch）並備妥 analysis/p19_gom_morph_split（parity/selfcheck/7 tests PASS，27 experiments 全 run:false），等 P18 完成且使用者授權才 launch**
- 2026-09-29 10:16 P19 使用者裁決停止（6/9 完成、無台灣結果；P20 launch.sh 依賴 P19_RESULTS 須改）
- 2026-09-23｜P18 台灣holdout×背景配額ρ19/21/23 設計｜Claude×Codex 3 回合收斂（R1 ACCEPT→R2 REVISE[B1]→R3 雙方 ACCEPT/BLOCKING=[]）：固定 33,000 optimizer updates 消解 P9 更新量混淆；checkpoint 採 epoch-end source-Val best（strict `>`，選模機會125/114/105次）＋33,000-step last.pt 事前指定敏感度（不改共用訓練核心，best/last 兩套皆報）；主端點沿用 P5 event-macro Oil IoU、兩事件各 ±0.02 判準；581 張零油 patch 全排除、B19⊂B21⊂B23 巢狀；曝光比例閘改用靜態清單ρ＋動態528,000-sample 閘（B1 修正機率性敘述）。少數意見：Codex 原偏好共同 update-grid validation，已以 last.pt 敏感度處理。設計收斂、未建清單、未啟動，launch 需使用者另外授權｜詳見 decisions/20260923_P18_台灣holdout背景配額rho19_21_23設計.md
- 2026-09-21｜雙 harness 降低 token 消耗方案（C1–C10）｜Claude×Codex 三回合同一 Codex session `01a0c185-aba8-7b03-8e37-802888cccbad` 收斂：R1 REVISE(BLOCKING=[1,2]，未編號屬格式錯誤)→R2 REVISE(BLOCKING=[C1,C3,C5,C7,C9])→R3 雙方 ACCEPT/BLOCKING=[]；結論=協商收斂、Phase 0 進行中、Phase 1（改 institution／vault 正本）待使用者授權，尚未動任何制度檔。Claude 立場：實測主因是「長 session×每步重讀累積 context×工具迴圈×高 effort」而非協商呼叫本身（兩邊 headless 皆 <10%），並自行驗證後接受 Codex 對計量口徑的三項修正；Codex 立場：挑戰「輪」定義／fork 基線／codex_exec 視窗，最終全條 ACCEPT，指出最大風險為內容搬走後新 harness 找不到（→retrieval test＋failover 演練）｜詳見 decisions/20260921_雙harness降token消耗方案.md
- 2026-09-14｜P14 GoM full-source ablation 設計與啟動｜延續 P13 提問，改用兩臂 GoM-only／NonGoM-only full-source（GoM Train156 scenes/1188 oil/1550 BG、Val39/312/390；NonGoM Train120/584/1180、Val32/132/310），同 Prithvi300M+DLV3、Tversky.3/.7、batch16、max300、patience50、seed42、Taiwan137 patches/9 scenes/2 events；checkpoint 改為 best source-Val（優於 P13 train-proxy）；以 δ=0.02 描述性門檻判定差異量級，不得宣稱海域因果或統計顯著；P12 僅歷史參考非證據基礎。同一 Claude session `cdb9a6bd-f24f-4197-bf0c-eb8e6dc7ea41`（claude-opus-5／effort high／CLI 2.1.270）三回合收斂：R1 REVISE(BLOCKING=[B1–B7])→R2 BLOCKING=[B7]→R3 雙方 ACCEPT/BLOCKING=[]。首個 15:48 run 於 Epoch 97 後中斷；使用者於 23:55 明確授權刪除中斷 run 並從 epoch 0 重跑，舊 4.7 GB run 與空白 log 已刪除且不可復原。23:56 由獨立 tmux `p14_gom_full_restart` 重啟（main PID=`2052764`；新 GoM run=`20260914-235613_MS6_P14_GoM_only_full_R1`），同 runner 後續自動接 `MS6_P14_NonGoM_only_full_R1` 與 Taiwan post-tests｜詳見 decisions/20260914_P14_GoM_full-source_ablation設計.md
- 2026-09-13｜P13 海域與資料量最終結論｜A/B 8 arms 結果已出：P13 只部分回答 source-group sensitivity（2 Taiwan events、9 scenes、single seed、既定 A/B budget policy），未回答海域、候選池／unique 資料量、augmentation、diversity 的因果效果；G3 單源最高但 macro 領先集中於 2026 event，B-G3 與 B-Mixed macro 僅差 0.0105（single seed 不宜排序）；A 只排除「實際 train patch 數不同」、不排除候選池多樣性混淆；A→B 為完整 budget policy 切換非純資料量實驗，無 seed-level 變異估計。同一 Claude session `d1d73839-7c91-48b3-8f43-aae92d9c9a73` 4 回合收斂，R1 REVISE→R2 ACCEPT(non-blocking 措辭)→R3 反方終審 REVISE(B1)→R4 雙方 ACCEPT/BLOCKING=[]｜詳見 decisions/20260913_P13海域與資料量最終結論.md
- 2026-09-11｜P13 patch-balanced 台灣 source sensitivity｜與 pixel-balanced 平行，8 arms=`A/B×G1/G2/G3/Mixed`，單 seed42；凍結 A/B patch 配額、train-proxy 選模、台灣137 patches/9 scenes/2 events 與 exposure 紀錄；限制為非獨立 val、B oil validation exposure 差異、非海域因果／非顯著性。第4回合 ACCEPT/BLOCKING=[]；全 arms `run:false`、未啟動｜詳見 decisions/20260911_P13_patch-balanced台灣source_sensitivity實驗設計.md
- 2026-09-02｜GeoAgent／A2C 尺度配置實驗設計與建碼｜六回合同一 Claude Opus 5/high session 收斂：R1–R5 REVISE→R6 ACCEPT/BLOCKING=[]；以 hard per-scene context budget 解決「無預算逐 patch 選尺度不是 genuine MDP」問題，定案 Track A=fixed local/3×/5×+supervised atom regressor/Dinkelbach、Track B=fixed/random/supervised/A2C/exact oracle；development=NOAA319、named-event36 sealed。隔離 worktree 已完成 fail-closed 程式；初審 REVISE 後聚焦複審 ACCEPT/NONE；CPU synthetic 16/16與 py_compile 通過，pytest 未安裝故未跑；未做真實 audit／Prithvi／GPU／training／commit／push｜詳見 decisions/20260902_GeoAgent_A2C尺度配置實驗設計與建碼.md
- 2026-08-31｜GeoAgent Global–Local 油污分割應用設計｜三回合同一 Claude Opus 5/high session 收斂：R1 REVISE（借 shared global-local、拒絕直接照搬 A2C）→R2 REVISE（fusion 18.9M→context-only、封印 outer-test 設計重用、30m 降為 physical-scale prior）→R3 ACCEPT/BLOCKING=[]；首臂=`256@10m local + 768→256 context + shared Prithvi + FixedROIResample(center 1/3) + 1024-channel zero-init gated add + 原 ASPP`，RL 預設 out of scope；僅概念設計，未改 code／prereg、未跑 CPU/GPU，所有後續 Stage 執行仍須使用者明確授權｜詳見 decisions/20260831_GeoAgent_GlobalLocal油污分割應用設計.md
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
- 2026-08-27｜P8 背景座標池擴充（expanded-pool）實驗設計，接續 P7 r=1.2 判定不予採用｜四回合收斂：R1 盲提案雙方獨立收斂同一原則→R2 REVISE(codex 撤回 pool 鎖死 10、改 expanded-pool 為首選)→R3 REVISE(Claude 認錯 F=6 無依據與配額式不守恆)→R4 ACCEPT/BLOCKING=[]；最終政策=既有 355 景 `min(C_s,10)` 不變、新景以 water-filling 分配並標記 `RATIO_INFEASIBLE_*`；P8 實驗=fixed-exposure expanded-pool k=22、non-inferiority 雙 co-primary gate、3 折、S2 未過不啟動 S6｜僅完成設計協商，未跑任何訓練、GPU 未啟動，S2/S6 均須另行授權｜詳見 decisions/20260827_P8背景座標池擴充實驗設計.md
- 2026-08-28｜油汙:背景**像素**曝光比例 ρ₀=23.0187 未經驗證,如何處置｜四回合收斂:雙方 ACCEPT/BLOCKING=[];決議做預註冊敏感度研究 ρ∈{15, 23.0187, 30}(以 ln ρ 建模的不等距 bracket,包住 1:20 與 1:25 但不直接測),巢狀以現行 3,510 張為不可變軸心、中臂重用正典三折,約 41.5 GPU-h、硬上限 45 h;ρ₀ 定位為採樣規則的衍生曝光統計,不主張最優;實驗未啟動,待使用者授權｜codex 原提窄設計 20/23.02/25 直接回答審稿人,被主持方以「同預算買到 3.1 倍 ln ρ 跨距、且窄設計在 MDE≈0.022 下先驗保證回傳 null」推翻;codex 的巢狀構造與 launch gate 被主持方採納;codex 保留少數意見(零 GPU 限制性陳述仍是合理替代,若撤回政策定位則實驗必要性消失)｜詳見 decisions/20260828_油背像素比例敏感度研究設計.md
