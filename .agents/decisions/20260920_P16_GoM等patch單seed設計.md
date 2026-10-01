# P16 GoM／NonGoM 等 patch 單 seed — 九步驟卡（1–6 事前凍結）

**實驗編號**：P16
**狀態**：`FROZEN_prelaunch`；1–6 已凍結，尚未啟動；P16 新 report 的獨立 runtime parity 與 launch snapshot 校驗仍須通過才可送跑。
**凍結日期**：2026-09-20 20:15:11 Asia/Taipei。
**事後補寫**：否；本草稿建立時尚未啟動本實驗。
**取代哪張卡**：無；接續 P14 full-source 與 P15 重複曝光實驗，不取代其歷史結果。
**使用者授權**：2026-09-20 明確要求兩組先各跑 1 seed，安排完成並送跑後回報；不包含追加 seeds、commit 或 push。

## 1. Objective（目的）

釐清海外來源資料對台灣油污辨識的適用性，檢查 GoM 的候選優勢在訓練 patch 數量與訓練更新預算配平後是否仍存在。

## 2. Problem（問題）

P14 GoM-only 使用 1,188 GT patches＋1,550 BG patches，NonGoM-only 使用 584＋1,180；資料量與每 epoch 更新數不一致。P15 比較固定 GoM unique 資料下的重複曝光政策，不能排除 P14 的來源與資料量混淆，也不是 augmentation 開／關實驗。

既有 P15 報表的 event-macro 為各事件內 scene confusion matrices 相加後計算，與 P5 的同日重疊 tile 去重、日期等權公式不同；本案須明確採後者，不直接混用歷史報表值。

## 3. Research Question（研究問題）

在兩臂各有 584 GT＋1,180 BG 原始訓練 patches、相同 optimizer 更新預算及共同訓練設定的條件下，GoM-only 是否在目前兩個台灣測試事件皆比 NonGoM-only 有至少 0.02 的 Oil IoU 優勢？

**已否定路線檢查（R6-1）**：已查 institution 現況檔③。本案不是重開已否定的 loss／threshold／背景配額路線，也不重試 P15 曝光政策；此前尚無 GoM／NonGoM 同 GT/BG patch 數與同更新預算的對照。P13 僅為含多海域的來源群組比較，未隔離純 GoM。

## 4. Hypothesis（假說）

若 GoM 候選優勢不只是原始 patch 數與總更新機會較多造成，配平後仍可能在兩個事件觀察到正向差異。既有優勢主要集中於 Taiwan 大油污案例，因此不預設一定通過兩事件一致門檻；單一事件改善只支持條件性候選假說。

## 5. Method（方法）

### 資料與抽樣

- 兩臂各使用 584 GT＋1,180 BG＝1,764 個原始 patches；GT 類別指既有 GT-centric patch 清單，不能假設每個 patch 均含非零油污像素。
- NonGoM 全部保留；GoM 從 P14 既有 train pool 下採樣，不動原始資料。
- GoM 依 scene 分層、GT/BG 分開按容量比例作 Hamilton 配額，sampling seed＝42、無放回抽樣一次，清單凍結後不隨 epoch 重抽。抽樣 RNG 與 training RNG 獨立，固定落盤清單不因更換 training seed 連動重抽。audit 確認 GoM 156／NonGoM 120 train scenes 全部仍有 GT 清單覆蓋。
- 各臂 Val 與共同 Taiwan Test 清單 byte-identical 繼承 P14，train／Val scene-disjoint；兩臂 source-Val 本身不同，為殘餘混淆。
- audit 報告配額、唯一座標數、scene 覆蓋、清單 SHA256、train／Val／Test 交集，以及實際油污像素比例。相同 GT/BG patch 數不等於油污像素比例相同。
- mask 實算 GoM 油污像素＝4,624,577／115,605,504（4.00030867042455%），NonGoM＝4,212,049／115,605,504（3.6434675290200715%）；不將此差異隱藏成像素比例已配平。

### 訓練與選模

- 兩臂 training seed＝42；不新增其他 seeds 或抽樣清單。
- Prithvi-EO-2.0-300M＋既有 DeepLabV3 ASPP，Tversky α＝0.3／β＝0.7；既有 optimizer、scheduler、normalization、augmentation 維持相同。
- batch＝16，`drop_last=true`：每 epoch 使用 1,760／1,764 個 patches，共 110 updates；丟棄尾端 4 個，不代表永久刪除 4 個原始 patches。有效 GT/BG 曝光張數可隨 shuffle 浮動，只承諾落盤清單配額相同，不宣稱每 epoch 各類有效曝光數精確相同。
- 兩臂皆跑 300 epochs，patience＝301 使早停不在 300 epochs 內觸發，各 33,000 optimizer updates。每 epoch 驗證一次，更新間隔同為 110。
- checkpoint 依各臂獨立 source-Val 最佳 mIoU 選擇，僅在 `val_miou > best` 時更新，因此同分保留最早 checkpoint；報告 best epoch、updates-to-best、總更新數及終止原因。兩臂總預算相同，不代表選出的 best checkpoint 已接受相同更新數。
- Taiwan Test 僅作本次凍結後最終評估，不作 checkpoint／seed 選擇；因先前已被用於研究方向討論，不宣稱全新 untouched confirmatory test。

### 對照組與 estimand（R6-2）

- 實驗組：`MS6_P16_GoM_matched1764_S42`；對照組：`MS6_P16_NonGoM_matched1764_S42`。設定及卡片 SHA256 另由 launch snapshot 留存；不將可變 run 開關的 hash 嵌入本卡。
- estimand：固定一次來源抽樣、seed＝42、相同 GT/BG patch 數與總訓練更新預算下，兩套來源＋source-Val 選模政策在既有台灣資料的表現差；不是純海域地理因果效果。
- 主指標：P5 聚合合約的 event-macro Oil IoU；`_kind=recon`，GT-aware 支援域。2 events、9 scenes、5 dates；Angel2023 為 4 dates，Taiwan20260522 為 1 date。
- 先對同事件同日的重疊 tiles 作地理鑲嵌與去重，計算日期級 Oil IoU；事件內各日期等權平均，最後兩事件等權平均。不得以 scene confusion matrices 直接相加取代去重與日期等權。
- 次要描述：每事件／每日期 Oil IoU，以及清楚標明聚合方式的 Precision、Recall、F1、BG IoU、mIoU；scene-macro／未去重 pooled 僅作補充，不替代主判定。
- 尚未控制：來源 scene 數、感測器、油污型態／像素比例、來源事件與多樣性、source-Val 組成、抽樣隨機性及訓練隨機性。不是只有地理位置不同的完全受控實驗。

## 6. Expected Evidence（預期證據）

Δ 定義為 `GoM − NonGoM`；門檻 δ＝0.02，沿用 P14 描述性量級，**不是顯著性檢定或等價界值**。

- **支持（pilot）**：兩個事件各自的日期等權 Oil IoU 差皆 `Δ >= +0.02`。
- **反向證據（pilot 否定方向性假說）**：兩個事件皆 `Δ <= −0.02`；只否定本次凍結條件下的正向預測，不宣稱整條 GoM 路線普遍無效。
- **不可判定**：其餘所有情況，包含方向不一致或幅度不足；不將小差異當成等價或無效證明。
- event-macro 與逐事件差值一併報告；不以較有利的次要指標改判定。
- 單 seed、單次抽樣及兩事件不提供跨 seed／抽樣／全台灣穩健性；本案定位為探索性 pilot。

## 凍結與稽核紀錄

- 正式方法協商：主持 harness 回報 R1=`REVISE`（事前凍結、metric parity、scene audit 為 blockers），R2 待收斂；不得冒稱雙 harness 已同意。Claude CLI `2.1.278`，requested／actual model=`claude-opus-5`、effort=`high`、session=`4621f47a-e40e-4880-92f5-d757f1b6c1f8`。R1 usage：input 2／cache-create 12,403／cache-read 8,257／output 6,943 tokens，cost USD 0.3017435；未回報降級。
- R2：`STANCE=ACCEPT` 但 `BLOCKING` 非空，**尚未收斂／不可凍結送跑**；剩餘 launch gates 為 source-Val mIoU tie 規則、實際 best.pt SHA／epoch 與 prediction manifest／Val 記錄一致性、P12 端點 CPU parity、launch 前卡片 hash 留存。累計 usage：input 4／output 12,791／cache-read 28,917／cache-create 20,086 tokens，cost USD 0.5351135；同一 session，待 R3。
- R3：主持 harness 回報 Claude `ACCEPT / BLOCKING=[]`，Codex 主線亦 `ACCEPT`，正式方法協商收斂；同一 session／model／effort／CLI、無降級。累計 usage：input 6／output 14,906／cache-read 57,260／cache-create 26,494 tokens，cost USD 0.66625。**此時仍待最終實作 audit，不代表本卡已凍結或實驗已啟動。**
- P12 獨立 CPU endpoint parity：主持 harness 回報 `PASS`，event-macro＝`0.46379088875726393`；Taiwan＝`0.5286993936592828`、Angel＝`0.39888238385524516`。
- report gate：`best.pt` 僅含 state_dict，不虛構其 epoch metadata；assert prediction manifest 的 actual model SHA 等於實際 `best.pt` SHA，`last.pt` 的 best_epoch／best_optimizer_step 與 run_metrics 一致，並核對 CSV 六位小數四捨五入最高 Val mIoU。最終程式 audit 待主線回報。
- INPUT／CONFIG audit：獨立 agent `earlier_evidence` 回報 `PASS`，兩臂 GT584／BG1180 皆 unique；train／Val／Test scene 交集為 0，清單 hash 一致；NonGoM 7 檔 byte-identical 繼承 P14，兩臂 Val／Test 4 檔及 Val BG rows／order byte-identical；兩臂 config 除 paths／names 外一致。audit 檔已確認存在：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p16_gom_matched_patch/generated/audit.json`。
- config 已確認存在：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/main/experiments_P16_gom_matched_patch.yaml`；結果根目錄已建立：`/home/alanyh/oil_dataset/new/_p16_runs/P16_gom_matched_patch`。
- 原 P5 公式獨立 CPU parity 已通過；**P16 新 report runtime parity 在本卡凍結時仍在獨立測試中，屬待完成 launch gate，不得混稱已通過。**
- launch 必須先驗證 `launch_files.sha256` snapshot 與 builder `--verify-frozen`，並設定 `CUBLAS_WORKSPACE_CONFIG=:4096:8`、`PYTHONHASHSEED=42`。卡片 hash 由主持 harness 在 launch 前留存於對話／launcher log；未授權 commit／push。
- 1–6 凍結前不得啟動；凍結後不得回改，變更須另立新卡。
- 7–9 待結果產出後另建 `wiki/experiments/` 結案卡；即時 process 狀態寫入 handoff、log 與 institution 現況正本。
