# handoff.md — 工作階段交接檔

用途：任一 harness 接手時（例如 Claude token 用盡、使用者跳到 Codex 繼續），讀本檔即可掌握目前工作階段並繼續做工。
更新紀律：每完成一個工作段落就更新「目前狀態」，不要等 token 快用完才寫（真用完就來不及了）。
必寫事件:process 啟動/結束、fold 完成、共識案收斂。交接前檢查 STATUS_AS_OF 與訓練 log/最近事件的時間差,逾時標 STALE,不得宣稱最新。
同機接手：Claude Code 與 Codex 都在這台 Linux server 上，讀寫同一份 vault，接手不需要 commit/push。只有要讓 Windows 端的 vault checkout 看到最新狀態時才需要 commit + push。

## 目前狀態(每次更新覆蓋本節;本節只是摘要,研究細節唯一正本=institution research/oil_spill_project_status.md)
- STATUS_AS_OF:2026-07-12 01:50 UTC(更新者:Claude Code)
- 正本路徑:/home/alanyh/.agents/institution/research/oil_spill_project_status.md
- 最近完成事件:baseline_v3plus 3-fold 完跑+雙閘門正式判定=受控 negative result(pooled 0.3047 FAIL;per-scene vs V3 顯著更差 p=1e-10;純架構切換路線關閉,詳見狀態正本)
- 使用者資料計畫:5 個 2026 Atlantic 場景(現無 mask)待 v3plus 兩組跑完後加入資料集開後續實驗(=新資料版,屆時需重跑對照;ignorering 排序屆時交使用者)
- 下一步:等 v3plus 串行訓練完跑(~2天)→雙閘門判定;之後 threshold 校準+FN 分解(零重訓)
- 執行中程序:nohup tversky_v3plus fold1 訓練中(01:13 UTC 自動接跑;判準=須顯著優於 V3+tversky 0.3915 才留 V3+;log 同前)

## 共識紀錄
- 2026-07-11｜Codex 前處理審查 F1-F10 處置方案｜一回合全條 AGREE：F2 降 Medium（空 tile=context-negative 保留）、F4 維持 High fail-fast、三批執行順序、ignorering 用凍結資料先跑與 v2 解耦｜雙方立場各經程式碼實查
- 2026-07-11｜DeepLabV3+ 架構切換實作方案｜二回合收斂：smp DeepLabV3Plus + model_family 切換鍵 + M1 預訓公平性 + rates(6,12,18) + dropout 維持出廠 0.5（Claude OBJECT Codex 的 0.1 成立）+ EMA 移除｜歷史 V3 checkpoint 逐位元回歸通過
- 2026-07-11｜協商制度升級 v2（收斂驅動：VERDICT 區塊、5 回合檢查點+10 回合硬上限、停滯偵測、反方終審限不可逆決策）｜兩回合收斂，雙方 ACCEPT｜詳見 decisions/20260711_收斂驅動協商制度升級.md
- 2026-07-11｜harness 結構改革(入口路由三方化/subagent 定義版本化/狀態單一正本+handoff 瘦身/稽核 checklist/agy 型號 unverified 標註/v2 試行評估附錄)｜三回合收斂:codex 首輪 OBJECT BLOCKING[1,2,3]→候選稿 v1 REVISE BLOCKING[P1-2a push 授權]→主持方以 vault-manager 定義第19行(常設 auto commit+push)證據解消→ACCEPT BLOCKING[]｜詳見 decisions/20260711_harness結構改革.md
- 2026-07-11｜研究路線重排+named-incident OOD holdout(36 scenes/9 events 封存、TODO 00/01/11 關、02 升 P0、12 擱置、14 改寫、新 03/15、B 移植 A=投稿 gate)｜四回合收斂:OBJECT→REVISE→REVISE→ACCEPT[]｜詳見 decisions/20260711_研究路線重排與OOD_holdout.md
- 2026-07-24｜Focal Tversky 小圖實驗可行性｜兩回合同 session 收斂：R1 conditional worth、BLOCKING[plain Tversky 先完成/FTL 必須直接對 plain Tversky]→Codex 撤除未驗證的 soft-TI attenuation 推算→R2 雙方 ACCEPT/BLOCKING=[]；FTL 僅在 canonical Tversky 前置 gate 通過後取得一張 q=0.75 單變因 coupon，fold1 失敗即停、不掃 q｜詳見 decisions/20260724_Focal_Tversky小圖實驗可行性.md
