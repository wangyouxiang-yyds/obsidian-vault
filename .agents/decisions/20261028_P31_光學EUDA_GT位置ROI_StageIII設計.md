# 光學 EUDA Stage III：GT 位置輔助 ROI 原型（事前凍結卡）

狀態：FROZEN → COMPLETED 2026-10-28 22:50；凍結時間 2026-10-28 Asia/Taipei（本 agent 無 Bash/exec 工具可取精確時刻，分秒待 parent 另行核對；日期確定為 2026-10-28）。沿 P30 後續另立新卡，不覆寫 P30。第7–9步已於 2026-10-28 22:50 完跑後補寫（非事後改寫1–6）。

## 1. Objective

檢驗使用baseline既有臺灣油污周邊窗口作target適應母體，是否能讓固定EMA StageIII原型在同一局部終點，相對StageI S42取得有實務幅度的提升。

## 2. Problem

P30以全景候選網格排序，easy18482窗只有6窗初始預測有油；正式teacher4999/5000批全背景，laststudent於固定19景270窗歷史recon OilIoU為0。背景CE+EMA回饋為較有支持的機制候選，BN/optimizer/adv等因果貢獻未分離。本次改變取樣位置先驗，不因原P30工程PASS而假定能改善。

## 3. Research question / 已否定路線檢查

使用相同S42與StageIII配方，target換成baseline既有270個GT-centric窗口後，fixed-last學生能否在同19景270窗的scene-macro歷史OilIoU上優於凍結S42？避免P30全背景為次要描述性問題，恢復非零油預測不等同優於S42。

P30已否定的是原全grid原型在該終點的效果。本次結構性改變是**target取樣新增GT位置資訊**、限制至既有油區附近；不是只調lr重試。P27/P28物理proxy不採用。

## 4. Hypothesis

ROI母體可提供較多持續的油類偽標籤訊號，使相同配方較不易陷入全背景，並可能改善相對S42的局部終點。仍可能退化或不可判定；原S42預測有油不等於正確偽標籤。

## 5. Method / 對照與estimand

- 使用者接受的範圍：沿baseline臺灣19張既有270個256x256 GT-centric座標，來源`analysis/euda_stage3_20261028/taiwan19_test.txt` SHA `3fbf624d160bcf040a1c0fc3a5e05dd4d2a643acaa9f6f8feaa007d31bfb8875`。270unique，與舊target排序精確座標0重疊。只從既有清單讀位置，不重新用pixel GT挑窗、擴張或補抽；VRT validity沿六band非finite/zero/extreme、<=10%無效；audit270/270有效、0排除、19景全保留。
- **GT座標＝使用；target pixel mask＝只在5000後固定終評；不用GT品質選lambda或checkpoint。** 這是GT位置輔助、ROI-conditioned target adaptation，同窗口適應/終評；不稱GT-free UDA、全景未知油搜尋、未見事件泛化或完整原文persistent PLR。
- 初始/主要對照：既有StageI `MS6_EUDA_S1_uncertainty_adv_S42/best.pt`，SHA `140f1d86e1783c084ae55427e42d31c02b9f92b789b27f93d25295fe20c1bdf3`。S42已用GoM source和臺灣full-grid target適應過，本輪不重訓StageI，不能稱zero-shot或全流程ROI。PrithviEO V2 300M+DeepLabV3、6active/8slot、DN/10000/meanstd不改。
- StageII：S42 eval/no_grad AMP single-forward重新計算ROI機率與valid像素平均entropy，原固定scale，global ROI-window升序、固定tie規則、lambda=.67，easy180/hard90；不做油分池、confidence threshold或物理篩選。初始偽預測分布只供描述，不是GT品質：easy161有油窗/180、油1218986/11796480=10.3335%；hard90/90、油1155287/5898240=19.5870%。manifest SHA `d2e1a39dac154f9543713fd6f4f75d14abfcb8a934a951ba04fede3c1ba1013c`。各來源SHA在新input_provenance，GT位置存取如實標示。
- StageIII：student/teacher從fresh S42同權重初始化；easy teacher online argmax CE、invalid255 ignore；hard U對抗lambda=.001；teacher參數EMA .99、buffers固定S42；D fresh同單通道5層FC、easy0/hard1 FP32；不加source CE/tau/oil balancing/BN freeze/固定teacher或未知PLR記憶。
- seed42、batch16easy+16hard、無accumulation，原順序/augmentation策略不改；5000成功更新、50個100-step記錄期，各80000 accepted draws，uniform replacement。AdamW encoder5e-6/head5e-5 WD.001，cosine eta_min5e-8；D Adam1e-4 betas(.9,.99)。AMP溢位/invalid skip沿原流程，不算成功步；數值失敗/incomplete不可當科學否定。
- 固定預算使easy每窗平均444.44 draws、hard888.89 draws，比P30曝光更多；窗口高度重疊、空間/事件相關，不當270獨立樣本。只比較**取樣位置先驗+母體與曝光分布改變的整包效果**，不能隔離entropy/EMA/adv或油先驗的單獨因果。
- 觀測不改梯度/RNG（小模型CPU on/off逐bit一致；CUDA warn-only非確定性仍不保證GPU逐bit重播）：逐successful batch記easyCE、hard adv、D、student_total=CE+.001adv、teacher/easy學生/hard學生油比例；科學記號stdout並tee到tmux和log。早期50/100/200保存fullstates，原每500 fullstate與固定last保留。這些為更新前當批輸出與診斷，非準確度或選模指標；不因油比例改checkpoint或預算，不對早期模型開targetGT評分。
- 正式只用main/main_runner.py，共用GPU lock、cleanGit/已追蹤config、W&B具ROI說明、共用三工作表。新結果目錄`.../_euda_stage3_runs/EUDA_STAGE3_ROI_20261028`，原P30結果/設定/卡不變。工程預檢fresh S42，預檢權重不可供正式初始化。

### 固定終點與對照

5000成功步後同時重新跑S42與fixed laststudent，使用同19景/270座標、FP32三視角mean-logit TTA、argmax、last-write-wins既有重建與prediction artifacts。stage3 ROI與endpoint座標集合相同要先驗SHA/roster核對；缺景/不同覆蓋/未完成預算failclosed。

主要estimand：`_kind=recon(gt_aware)` **19景等權scene-macro historical OilIoU**，GT255沿既有歷史口徑作背景。副指標pooled OilIoU/P/R/F1、19景paired差、known-label0/1/2-only recall等、unknown255預測油量與覆蓋。GT255未知使歷史IoU不是完整全景精度。P30 0僅作同終點歷史失敗對照；S42需在本輪相同終評pipeline重新算。單seed、non-blind、GT-position-guided、完全transductive（適應窗=終評窗），不作獨立test泛化主張。GoM val僅末次workbook bookkeeping，非新checkpoint選擇或獨立test。

## 6. Expected evidence / 事前數字門檻

延用P30的探索性實務幅度規則（不是顯著性）：
- 支持續作：Δ(scene-macro historical OilIoU，新last−S42)>=+.03，且Δ(known-oil recall同覆蓋)>=−.02；仍報pooled/所有19景差與油量，不能因通過而宣稱泛化或純selector因果。
- 否定此ROI原型於該局部終點的增益假說：Δmacro<=−.03。
- 其餘不可判定；macro提升但recall guard不通過亦不可判定。未完成/缺景/數值失敗為invalid/incomplete，非科學否定。
- 是否再次完全全背景，描述性記final predicted_oil_pixels==0及teacher/oil軌跡；即使避免collapse，也必須過上面對S42的增益門檻才支持續作。ROI分位easy不代表與P30同一絕對entropy條件；若歷史255-as-background主指標與known-label-only指標方向不同，仍按上述事前主指標判定，並明記口徑差異，不事後改判。

## 7–9. 結果 / 結論 / 下一步

（以下為 2026-10-28 22:50 完跑後補寫；第 1–6 步未改。）

### 7. 結果

- **執行**：22:25 開跑、22:50 完跑，`EXIT=0`；5000 成功步，skipped=2；GT_expand commit `32fcd96`；W&B run `daiyal5q`；log `/home/alanyh/oil_dataset/new/_euda_stage3_runs/EUDA_STAGE3_ROI_20261028/main_runner.log`。
- **270 窗終評**（同一批窗口、固定 last student）：

| 指標 | Stage I S42 | P31 last student |
|---|---|---|
| per-image macro OilIoU（主 estimand） | 0.4709 | 0.0925 |
| pooled OilIoU | 0.4586 | 0.1269 |
| pooled Precision | 0.695 | 0.131 |
| pooled Recall | 0.574 | 0.791 |
| 預測油像素 | 1,164,126 | 8,480,159 |

- **Δmacro = −0.378**（P31 last − S42）。
- **訓練中 teacher 偽標籤油比例**：

| step | 100 | 1000 | 2000 | 3000 | 3500 | 5000 |
|---|---|---|---|---|---|---|
| 油比例 | 8.8% | 17.6% | 38.0% | 68.0% | 98.5% | 91.6% |

  3500 步之後維持 >90%。

### 8. 判定與解讀

- **判定**：依第 6 步事前判準（Δmacro<=−.03），exploratory_outcome = `rejects_this_prototype`。
- **解讀**（單 seed、探索性，非結論性因果）：P30 崩成全背景、P31 崩成近乎全油；同一套無 source 錨定的 EMA＋online argmax 自訓練會往一邊漂，取樣位置只決定漂的方向。因果尚未分離（EMA、無 source CE、無信心門檻皆可能）。

### 9. 下一步

待使用者決定，不預設。

---

## 方法協商 Provenance（附加段，非1–6原文）

ClaudeCLI 2.1.312、Opus 5／high，requested=actual；session `62ee601c-91d6-49ee-8f47-8b1e2650d805`。R1 STANCE REVISE → R2 STANCE ACCEPT／BLOCKING=[]；Codex 已接受。原始 prompt／JSON／response／usage 記錄於 repo 同目錄（`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/euda_stage3_roi_20261028/`）；`METHOD_CONSENSUS.json` 已核。resume 成本可能累計不可直接相加，無 fallback。工程 GPU 10 步 PASS 與 CPU 15 測試已通過，但 fresh 驗證／程式 readback 仍進行中，不能假稱全部 launch gate 已通過。
