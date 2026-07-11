# handoff.md — 工作階段交接檔

用途：任一 harness 接手時（例如 Claude token 用盡、使用者跳到 Codex 繼續），讀本檔即可掌握目前工作階段並繼續做工。
更新紀律：每完成一個工作段落就更新「目前狀態」，不要等 token 快用完才寫（真用完就來不及了）。
同機接手：Claude Code 與 Codex 都在這台 Linux server 上，讀寫同一份 vault，接手不需要 commit/push。只有要讓 Windows 端的 vault checkout 看到最新狀態時才需要 commit + push。

## 目前狀態（每次更新覆蓋本節）
- 更新時間：2026-07-11
- 更新者：Claude Code（Linux server）
- 當前任務：GT_expand 前處理審查後續——Claude×Codex 協商完成，待使用者授權第一批工程修補
- 已完成：tversky 3-fold 雙 gate 全過（詳見 institution 現況檔⑤-9）；Codex 審查 F1-F10 協商一回合收斂全條 AGREE
- 進行中：51 大圖 P/R 分解（tversky，背景推論中）
- 下一步：使用者授權後執行第一批（git init 獨立 repo、fail-fast、stable hash、N_FOLDS、drop_last、preflight）；ignorering 待使用者明說啟動
- 未提交變更：GT_expand repo 未進 git（即第一批 F1 項）

## 共識紀錄
- 2026-07-11｜Codex 前處理審查 F1-F10 處置方案｜一回合全條 AGREE：F2 降 Medium（空 tile=context-negative 保留）、F4 維持 High fail-fast、三批執行順序、ignorering 用凍結資料先跑與 v2 解耦｜雙方立場各經程式碼實查
