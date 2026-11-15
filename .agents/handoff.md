# handoff.md — 工作階段交接檔（可覆寫快照）

用途：任一 harness 接手時（例如 Claude token 用盡、使用者跳到 Codex 繼續），讀本檔即可掌握目前在跑什麼、等誰決定、下一步是什麼。
更新紀律（2026-11-14 起）：本檔是**快照**。每次交接事件（process 啟動/結束、fold 完成、共識收斂、待授權事項變動）就**覆寫**「目前狀態」整節，不追加、不保留舊行；目標 < 10 KB（開場健康檢查門檻 20 KB）。永久紀錄寫 vault `log.md`，共識詳情寫 `.agents/decisions/`，研究結論寫 institution `research/oil_spill_project_status.md`。交接前檢查 STATUS_AS_OF 與訓練 log/最近事件的時間差，逾時標 STALE，不得宣稱最新。
同機接手：Claude Code 與 Codex 都在這台 Linux server 上，讀寫同一份 vault，接手不需要 commit/push。只有要讓 Windows 端的 vault checkout 看到最新狀態時才需要 commit + push。
歷史原文：2026-10-27 以前見 `.agents/handoff_archive_20261111.md`；2026-11-14 遷移前全文（含 54 條 STATUS_AS_OF 與共識紀錄）見 `.agents/handoff_history_20261114.md`。決議：`.agents/decisions/20261114_狀態檔拆分_現況快照與封存.md`。

## 目前狀態（每次覆寫本節）

- STATUS_AS_OF：2026-11-15（P54 launch 10:36 後；更新者：Claude 主 harness 經 obsidian-vault-manager）
- **執行中（GPU）**：**P54** 訓練自 2026-11-15 10:36 起（launch.sh 持共用 GPU 鎖 `/home/alanyh/oil_dataset/new/.gpu_main_runner.lock`；launcher pid 3211287）。預估訓練 ≤16 h（3.1 it/s、約 24 min/VE、最多 40 VE、patience 8）＋臺灣預測約 30 min＋evaluate（讀臺灣 GT，沿用 P53 既有授權）。Log：`/home/alanyh/oil_dataset/new/_p54_croma_global_20261113/{launch.log,train_P54_S42.log}`；wandb run `zleitpvz`。尚無結果，不寫結果卡。
- **P54**＝CROMA optical-large＋DLV3 接凍結的 P53 pipeline，seed 42；卡 **FROZEN r3**（GT_expand `analysis/p54_croma_global_20261113/P54_CARD.md`，commit 0318a65；Codex 實作審查 R1 REVISE → R2 ACCEPT）。使用者原話：「GPU回來了 可以幫我進行實驗了」。門檻於看 P54 GT 前已固定：τ 大林 .252／其他 22 張 .153（被 P53 S43 塌縮墊高；排除塌縮 seed 的敏感度各為 .09）。⚠ 現況檔「P54 須等 P53 塌縮處理完」的備註在 launch 前未被執行；比較對象 P53 S42 本身未塌縮；已回報使用者。
- **P53**（卡 FROZEN v2，GT_expand f5b321b）三 seed 全部 EXIT=0、評估完成：預登記判讀皆 no_decisive_difference；S43、S44 於 VE9 早停、best＝VE1（塌縮），S42 正常。結果卡 `wiki/experiments/20261115_P53_global_stratval_結果.md`；數字見該卡與 institution 現況檔。ctx k2k4 補seed 與推論加速 benchmark 先前已結束。
- **待使用者決定（最優先）**：**P53 塌縮處理**（不計／補 seed／查機制／改早停或選模規則）；未授權前不改卡、不補跑。
- **待使用者授權／決定**（研究細節與卡片見現況檔 ⓪）：
  1. （P54 已於 11-15 10:36 啟動，見上；不再待授權。）
  2. **ctx 推論加速**是否採用：預取不改數值；test 改 AMP 屬 operator 變更，須寫進新卡；採用前需整個 VAL 的 parity 檢查。
  3. **UDA（DACS＋MIC 接 P44 trainer）**提案、**Stage 1 自動找框 pilot**：只有方向共識，未凍結卡。
- **狀態檔拆分遷移**：2026-11-14 已完成（handoff 與現況檔改為快照，歷史原文封存；決議 `.agents/decisions/20261114_狀態檔拆分_現況快照與封存.md`）。其他 session 可恢復寫入，但一律照新格式：handoff 覆寫本節、現況檔照其 ⑦ 就地更新。
- **Git**：institution HEAD 2dd1650 之後另有「狀態檔拆分」遷移 commit（見 institution `git log`）；GT_expand HEAD e38990f（工作樹有 72 項未提交，非本次）；vault HEAD 464eeb3，工作樹有多個 session 的未提交修改（含 consensus.md、index.md、log.md、decisions/ 新檔），commit/push 由使用者決定。

## Harness continuity

- 程序正本：institution `40_harness_continuity.md`（接手順序、權限邊界、`--add-dir` 呼叫法、agy 停用狀態都在那裡，本檔不重抄）。
- 最近一次 parity audit：2026-11-14（vault-manager 規則同步後）`PARITY AUDIT: PASS (agents=3 skills=4 projects=2)`；NOTE＝0422 專案層 `.codex/config.toml` 為 inert orphan（Codex 不讀），處置仍 open。
- Codex 協商呼叫式（2026-11-13 起 `danger-full-access`＋唯讀指示）與模型：見 institution `10_model_dispatch.md` §2。
