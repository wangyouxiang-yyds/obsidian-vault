---
status: FROZEN_PRE_LAUNCH
date: 2026-11-01
experiment: P37
supersedes: 20261101_P37_LFMDA_SegFormerB2_sourceonly設計.md
---

取代哪張卡: 20261101_P37_LFMDA_SegFormerB2_sourceonly設計.md

本卡（v2）於開跑前 readback 發現原卡（v1）第 5 步「Method」與「稽核」兩處有實質事實錯誤：(1) Method 誤寫「source 與 target 各自計算 mean/std」，實際凍結程式與 YAML 對 source（GoM R1）與 target（臺灣）都使用**同一組** GoM R1 source mean/std；(2) 稽核的 raw255 86.87%（train）／88.81%（val）是**GoM 油窗**的 raw255 占比，不是「GoM 全部窗」的占比，與其並列的背景 proxy raw255 占比為 train 3.92%、val 4.46%，原卡排版易讓人誤讀成兩者是同一範疇的對照。v1 因此**在開跑前被判定不得使用**；v1 檔案原樣保留、不修改、不刪除，僅作為錯誤版本的歷史紀錄。**v2 為本案唯一可用於開跑與結果判定的凍結卡**；凡引用 P37 方法或稽核數字者，一律指向本檔，不得指向 v1。

以下第 1–6 步內容逐項核對自 v1 並修正上述兩處錯誤（含用語澄清），其餘文字、所有數值門檻（支持／否定／不可判定）、source／target 角色劃分與 v1 完全一致，未做任何其他改動。

狀態：FROZEN_PRE_LAUNCH（凍結 2026-11-01 Asia/Taipei；建立後不修改）；本卡為 prelaunch 事前預註冊紀錄，非 post hoc。使用者已授權本機 git commit 與訓練開跑；第 7–9 步僅於完成後填寫。
共識：Claude Opus 5／high，同 session `2524a34e-f0a6-42dc-ae47-293be40622e2`，R1 REVISE → R2 REVISE → R3 ACCEPT → R4 REVISE → R5 ACCEPT／BLOCKING=[]；Codex ACCEPT。Repo 正本 `analysis/lfmda_sourceonly_baseline_20261101/consensus/DECISION.md`。

## 1. 目標

判斷作者原版 LFMDA（StuLiu/LFMDA）SegFormer-B2 source-only 能否作為本專案跨區域光學油污分割的基準（baseline）。本卡**不是**完整重現 DINOv2／PCD／SAM LFMDA 方法。

## 2. 問題

既有 P29 Prithvi／DeepLab source-only 與 EUDA 系列實驗，未單獨分離出「作者原版 source-only 架構＋RGB 輸入」這個變因；本基準只處理架構／前處理可行性與描述性表現，不是對 domain shift 或改善的受控歸因。GoM mask 有大量 raw255 被映射為 BG；臺灣 target 標籤是 GT-derived 的已知事件窗。

## 3. 研究問題（RQ）

固定 10,000 步 source-only 更新後：

- GoM source-val 最終 OilIoU ≥ 0.10；**且**
- 臺灣 19 景／270 窗 coverage-union pooled OilIoU ≥ max(0.10, 同覆蓋率 all-oil floor ＋0.05)？

查核過去否定路徑：institution research status 中 P29／P36 的 target 對抗塌縮或部分恢復，**不否定**本卡的 source-only 作者 SegFormer 架構；結構性差異在於作者 MiT-B2＋RGB 模型與 source-only 訓練；不宣稱本卡修復先前方法。

## 4. 假說

Source-only 作者 SegFormer 可能學到 GoM 油污訊號，並保留高於隨機水準的臺灣已知事件窗油污勾繪能力；但高 raw255 比例與標註差異可能導致失敗。單一 seed 無法建立穩健改善。

## 5. 方法

- 程式為 **thin runner**，匯入作者原始碼 `StuLiu/LFMDA` commit `88d9f2cda2e0c666611a0a8857240aaeb39a972b` 的 SegFormer／CE／`get_param_groups`／`adjust_learning_rate`／`loss_calc` 等元件組裝訓練迴圈；**非作者原始訓練腳本逐行複製**。
- MiT-B2 ImageNet checkpoint SHA-256 `ced22617efb7bae3c34ad0a80f20a9b8afb4d27368cb0835a23456baa9d0e092`（332/332 backbone keys matched）。
- GoM：R1 156 訓練景／2,738 窗，39 驗證景／702 窗，互斥。
- 原生 256，RGB（B04/B03/B02），DN/10000 → uint8；**source（GoM R1）與 target（臺灣）使用同一組 GoM R1 source mean [0.1433, 0.1342, 0.1087]、std [0.2178, 0.2179, 0.2248]，不為 target 另行計算或使用不同統計量**（已核對凍結程式與 YAML 一致）。
- Mask 重編碼：raw0＝oil，raw1／raw2／raw255＝BG；背景 proxy 窗全記為 BG。
- Seed 2333，batch 8，10,000 步；AdamW，LR 6e-5，weight decay .01，warmup 1500，poly power .9；CE，grad clip 32。
- 每 1,000 步在 GoM 驗證一次，依 source-val mIoU 選 best。**Final step10000 為臺灣頭條結果；best 為次要，且不可由臺灣 GT 選擇**。
- Target（僅影像）：既有 SHA-pinned TSV `analysis/euda_gtroi_pipeline_20261029/stage1_target_manifest.tsv`，SHA-256 `855a2307d229a3f436aabcc205fa50b3c278f325c35f7b40dfbaa1a440ea5162`，19 景／270 窗／256×256，row order 於本次 run 前已固定；不可調整 ROI 半徑或膨脹。
- Checkpoint、程式、protocol、TSV 與輸出須在讀取任何 target GT 前完成凍結並建立 SHA registry。final／best 兩份 prediction manifest 皆驗證完成後，才執行**一次**封存的 target GT 評估。
- 所有指標 `_kind: patch`：GoM source-val OilIoU 以全部 702 patch 計算；臺灣主要指標為 19 景 coverage-union pooled OilIoU（每景像素只算一次，重疊時以 TSV 後出現的列覆蓋前面；raw0＝oil，其餘＝BG）。
- Same-pass all-oil floor ＝ pooled GT 油像素 ÷ 全部覆蓋像素；all-BG floor＝0；另報告逐景 oil IoU 與 zero-IoU 景數。
- 既有 P29 僅作描述性參照：多個變因不同，不構成單因子因果估計，也不與論文 ISPRS 數字比較。
- 訓練／推論／選模全程不使用任何 target mask 像素。

## 6. 預期證據與判定

- **support baseline adequacy**：10k 步跑完、十筆 GoM 驗證記錄皆為有限值，且 final GoM source-val OilIoU ≥ 0.10 **且** final 臺灣 pooled OilIoU ≥ max(0.10, 同覆蓋率 all-oil floor ＋0.05)。
- **reject baseline adequacy**：final GoM source-val OilIoU ≤ 0.01 **或** final 臺灣 pooled OilIoU ≤ 同覆蓋率 all-oil floor。
- 其餘情形為 **indeterminate**：僅作描述性紀錄，不得用臺灣 GT 調參或宣稱可用對照；若要進一步宣稱效益，須另立預註冊多 seed／診斷卡。
- 三類別互斥；訓練失敗或未跑完屬方法失敗，不是數值否定。
- `best` 為次要指標，不可改變判定結論。單一 seed run 無法證明優於 P29，也無法證明泛化至全部臺灣景。
- 臺灣 target GT 的開封是後續、獨立 gate 的步驟，不屬於本次訓練開跑的一部分。

## 稽核（既有觀察，非判定依據）

- **GoM 油窗 raw255 占比**：train 86.87%、val 88.81%（此二數字特指 GoM 油窗，非「GoM 全部窗」）；**背景 proxy 窗 raw255 占比**：train 3.92%、val 4.46%；42.18% 的 train BG 監督像素來自 raw255 重編碼。不得稱背景 proxy 為真實水面，也不得稱衍生指標為 recall 下界。
- Image-only target audit：270/270 PASS。
- 2-step GPU smoke 與合成 protocol 測試：PASS。
- 目前尚無完整訓練或臺灣 GT 結果。

## 第 7–9 步

未填（待訓練完成後，依本卡第 6 步判定規則填寫；不得引用 v1）。
