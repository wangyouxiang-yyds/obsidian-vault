# handoff.md — 工作階段交接檔

用途：任一 harness 接手時（例如 Claude token 用盡、使用者跳到 Codex 繼續），讀本檔即可掌握目前工作階段並繼續做工。
更新紀律：每完成一個工作段落就更新「目前狀態」，不要等 token 快用完才寫（真用完就來不及了）。
同機接手：Claude Code 與 Codex 都在這台 Linux server 上，讀寫同一份 vault，接手不需要 commit/push。只有要讓 Windows 端的 vault checkout 看到最新狀態時才需要 commit + push。

## 目前狀態（每次更新覆蓋本節）
- 更新時間：2026-07-11
- 更新者：Claude Code（Linux server）
- 當前任務：DeepLabV3+ 架構切換已實作完成待啟動訓練（使用者方法論決策：發現歷史模型實為 DeepLabV3 後，決定真改架構而非只改文字）
- 已完成追加：51 大圖 P/R 分解（tversky recall 0.474→0.584、precision 僅降至 0.740）；Claude×Codex 二回合協商定案 V3+ 實作方案（smp DeepLabV3Plus、model_family 切換鍵、歷史 checkpoint 逐位元回歸通過、EMA 移除）；兩份 v3plus yaml 備妥
- 下一步：待使用者明說啟動 v3plus 訓練（baseline_v3plus + tversky_v3plus 各 3-fold）；第一批工程修補（git init 等）仍待授權；ignorering 待決定順序
- 未提交變更：GT_expand repo 未進 git（即第一批 F1 項）

## 共識紀錄
- 2026-07-11｜Codex 前處理審查 F1-F10 處置方案｜一回合全條 AGREE：F2 降 Medium（空 tile=context-negative 保留）、F4 維持 High fail-fast、三批執行順序、ignorering 用凍結資料先跑與 v2 解耦｜雙方立場各經程式碼實查
- 2026-07-11｜DeepLabV3+ 架構切換實作方案｜二回合收斂：smp DeepLabV3Plus + model_family 切換鍵 + M1 預訓公平性 + rates(6,12,18) + dropout 維持出廠 0.5（Claude OBJECT Codex 的 0.1 成立）+ EMA 移除｜歷史 V3 checkpoint 逐位元回歸通過
- 2026-07-11｜協商制度升級 v2（收斂驅動：VERDICT 區塊、5 回合檢查點+10 回合硬上限、停滯偵測、反方終審限不可逆決策）｜兩回合收斂，雙方 ACCEPT｜詳見 decisions/20260711_收斂驅動協商制度升級.md
