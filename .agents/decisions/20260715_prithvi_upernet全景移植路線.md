# Prithvi+UPerNet 全景（大圖/A 分支）移植路線

- 日期：2026-07-15
- 協商：Claude×Codex consensus v2，三回合收斂（R1 盲評 REVISE → R2 REVISE 4 blocking → R3 ACCEPT/BLOCKING=[]，CONFIDENCE 0.97）
- 背景：B 分支 Prithvi-300M+UPerNet+Tversky 3-fold 雙閘門全過（pooled mean 0.4498，large-slick +0.057 p=0.0078）；使用者問「計畫面小圖保持 DeepLabV3，Prithvi+UPerNet 能否用在大圖（A 分支全景 sliding-window）」。
- 結論：**有搞頭，但分三階段條件觸發，只有 Stage 0 立即可啟動（尚未啟動，待使用者授權）。**

## Stage 0：zero-shot 先導（純推論，無訓練）
1. 場景 12–20 景，模型輸出之前凍結選取；覆蓋 large/mid/tiny/無油/海岸/雲/sunglint hard negatives；刻意納入數景 external-84（A 有 B 沒有的場景，B 從未見過＝最乾淨 external validation）
2. **Leakage gate**：A/B 場景大量重疊（B=A 的清洗子集 355/439）→ B-overlap 景必須 fold-matched OOF（每景只用「該景屬 B test fold」的權重）；external-84 primary=fold1 權重（依既有 B 證據事前選定，非看結果後挑），fold2/3 全跑作 sensitivity，三折方向不一致→external 結論=inconclusive，不得挑折；B-overlap cohort 與 external cohort 分開報告再合併摘要
3. **評分三 support**：full valid-scene pooled Oil IoU／GT-aware bridge Oil IoU／ROI 外 FP per Mpix；統計單位=scene；argmax 為 primary（threshold calibration 降為選配診斷）；grid=A 分支現行 reconstruction 協定（兩模型同 grid；stride/blending 消融僅 2–3 景診斷用）
4. **三道 gate（看結果前鎖定）**：(a) bridge ≥ B recon 同景值 −0.03（non-inferiority margin）(b) full-scene Δ pooled Oil IoU ≥ 0 vs legacy A（fold-matched OOF）(c) FP piecewise：legacy ≥200 FP/Mpix 時 ≤1.25×，低於 floor 時 absolute ceiling ≤250 FP/Mpix；無油景另報 predicted-oil px/Mpix
5. **分流表（五列）**：bridge 保留＋full 不劣＋FP 過→進 Stage 2；bridge 保留＋full 落後＋FP 正常→查 stitching/邊界/calibration，不得直接歸因背景 FP；bridge 保留＋full 落後＋FP 超標→Stage 1；bridge 保留＋full 不劣＋FP 超標→Stage 1；bridge 失敗→技術一致性 audit（grid/前處理/nodata），重跑仍雙輸才停
6. 啟動前先做 500–1000 patch warmed throughput benchmark（含 VRT I/O）；「1–2 天」僅 calendar target

## Stage 1（條件觸發）：(c) B 權重 + A 協定 hard-negative fine-tune 一折
- 加純海面/海岸/雲負樣本；先凍 encoder 校 head 再低 LR 解凍；保留 B-style positives
- **雙重排除**：評估景 (i) 不在所用 B checkpoint 原訓練集 (ii) 不在 A fine-tune 訓練集 (iii) 未用於調任何超參/threshold；pilot 景只作 development，正式評估另留 untouched

## Stage 2（條件觸發）：(b) repaired 協定公平三折
- ResNet50 baseline 修 EMA(decay=0.999)/early-stop(Oil IoU) 同步重跑；Prithvi 從 foundation ckpt 依 A folds 重訓（無 B 權重洩漏）→ 才能做論文級架構主張
- practical margins：Δ pooled ≥ +0.02、precision non-inferiority 0.02、FP ≤1.25×、large/mid/tiny 無系統性反轉

## 措辭紅線
- pilot 結果只能稱 transfer 診斷，不得稱「已成功移植」
- B 的 gt_aware 數字不得直接推論全景優勢（分母不同，A/B IoU 不可互比）
- 雙層 comparator：legacy A baseline（現有系統）與 repaired baseline（公平協定）不得混稱

## Amendment 2026-07-16（Stage 0 啟動前修訂；Claude×Codex 三回合收斂 ACCEPT/BLOCKING=[]）

依 2026-07-16 provenance 實測（audit artifact：scratchpad leak_check3.py / external84_check.py；fold 互斥 0 重疊、355 景全對帳）修訂本計畫，於任何 Stage 0 預測產出之前落檔：

1. **external-84 重新定性**：實測 A-only 90 景 = 36 NOAA Atlantic + 53 NOAA Gulf of Mexico + 1 秘魯，**零個新事件群**。原文「B 從未見過＝最乾淨 external validation」措辭作廢，改稱 **A-only same-collection transfer cohort**（場景層面未見過、事件/集合層面全部見過）。
2. **Stage 0 改名**：zero-shot 先導 → **same-collection zero-shot transfer diagnostic**；三道 gate 數字不變，但判定語意降格為「protocol 內 transfer 診斷」，不得引為泛化證據。
3. **fold-matched OOF 的極限明載**：全語料僅 12 個名目事件群（NOAA 兩集合佔 315/355=89%），350/355 test 景在同 fold train 有同群姊妹景（332 同 tile 異日）。fold-matched 只防「同一場景」洩漏，不防姊妹景；Stage 0 任何結論一律受此限定。
4. **真 OOD 留待新事件資料**：2026 新資料若仍屬 NOAA Atlantic 集合區域，亦不構成事件級 OOD，需依 event/collection/sensor 軸另行定性。
