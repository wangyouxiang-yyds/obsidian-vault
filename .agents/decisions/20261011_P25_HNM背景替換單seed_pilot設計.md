# P25 HNM 背景替換（GT-centric 架構）單 seed pilot 設計

**實驗編號**:P25  **凍結日期**:2026-10-11
**事後補寫?**:否（訓練尚未啟動；探勘推論與背景清單組裝已於凍結前完成，屬 Method 的前處理，未看任何臺灣結果）
**取代哪張卡**:無
**協商狀態**:使用者直接指示的單 seed pilot，未經 Codex 協商。

## 1. Objective
看「把大圖時期的 hard negative mining（HNM）搬回現行 GT-centric 架構」能不能提高臺灣 9 景的 Oil IoU。使用者指示：跑一版試試，沒有明顯變好就不採用。

## 2. Problem
現行訓練背景是每景在無油區隨機抽 10 張（stride 512 候選池）。臺灣終點的錯誤以誤判為主（P12 seed43 九景加總 precision 0.557、recall 0.773）。隨機背景不保證包含模型實際會誤判的海面型態。

## 3. Research Question
在訓練張數、背景比例、驗證清單、其餘設定都不變的前提下，把訓練景的部分隨機背景換成「基線模型誤判為油的無油窗」，臺灣 9 景平均 Oil IoU 是否高於隨機背景基線三個 seed 的範圍？

**已否定路線檢查(R6-1)**:是。現況檔③記載大圖時期 cfHNM Oil IoU 0.224 輸隨機背景 0.242（三 fold 全輸）。
這次的結構性改變：
- 新表徵：預訓練 Prithvi-EO-2.0 300M encoder＋DeepLabV3 ASPP head（當時為 from-scratch ResNet50）。
- 新取樣架構：GT-centric 油 patch＋每景固定 10 張背景（當時為滑窗 patch、HNM 為額外加量 0.25–0.5×油張數）；本次為等量替換，背景比例不變。
- 新終點：未參與訓練的臺灣 9 景（當時為同語料 CV）。

## 4. Hypothesis
預期**不可判定或無差異**。理由：步驟 0 量測（`analysis/hnm_step0_fp_distance/`，commit 9036e62）顯示臺灣 FP 中離標註 >32 px 的只佔 8–10%，即使全部消失且召回不掉，9 景平均 Oil IoU 上限僅 +0.011～+0.018，小於 seed SD（約 0.026）。約八成 FP 是貼邊圈胖，無油背景教不到。

## 5. Method
- 探勘（`analysis/hnm_bg_swap/mine.py`）：用各組 seed 43 基線的 best.pt，對該組**訓練景**的背景候選窗推論（fp32、無 TTA）。候選窗規則與 `preprocess/generate_bg_patches.py` 相同（256 px、stride 512、零油、mask 1|2 ≥30%、mask 255 <50%），另排除基線已用的座標、NaN>10%、全零像素 ≥30%。驗證景與臺灣景不參與。
- 過濾與組裝（`assemble.py`）：候選需預測油像素 ≥256 px；以 8 維光譜特徵（移植自 Cross_Fold_HNM step 3）算到同景訓練油 patch 的最小距離 d_min，剔除全體最低 10%（疑似漏標油）；每景依預測油像素數取前 K=5，覆寫該景 10 列背景的最後 k 列。
- 結果：GoM 組候選 30,455 窗、合格 871、替換 498／1,550 張訓練背景（135／155 景）；全球排除臺灣組候選 65,842 窗、合格 1,783、替換 972／3,290 張（280／329 景）。兩組的總列數、逐景張數、驗證與測試景的列完全不變（稽核 6 項全過）。
- 訓練（`main/experiments_P25_hnm_bg_swap.yaml`，由 `build_configs.py` 產生並斷言差異）：兩個 run，各為其基線設定區塊的複本，只有 `dataset.bg_coord_txt`、實驗名、測試名不同。
  - `MS6_P25_GoM_HNM_S43`：複本自 P15 `MS6_P15_GoM_Original_S43`（seed 43、patience 50）。
  - `MS6_P25_Global_HNM_S43`：複本自 P12 `MS6_P12_taiwan_deploy`（yaml seed 42＝實際 seed 43、patience 25）。
- 程式版本：GT_expand commit fd3ce65。

**對照組與 estimand(R6-2)**
- 對照組：GoM 組＝P15 Original S42/S43/S44（0.4111／0.4050／0.3632，平均 0.3931）；全球組＝P12 實際 seed 42/43/44（0.3552／0.4052／0.3660，平均 0.3755）。皆為既有 run，不重跑。
- 主指標：臺灣 9 景各算 Oil IoU 再平均。`_kind`：recon（gt_aware）。
- 聚合方式：scene-macro（9 景平均），單一 run。
- 兩組間唯一不同：訓練景背景座標的選法（隨機 vs 部分換成探勘結果）。**不乾淨之處**：兩個 seed 43 基線都跑在較早的 commit（P15 為 83ac2ba 且 dirty；P12 為 09-06 版本），P25 跑在 fd3ce65，程式版本差異無法排除；探勘模型即 seed 43 基線本身。

## 6. Expected Evidence
- 若假說（無差異）成立：兩個 run 的 9 景平均 Oil IoU 都落在各自基線三個 seed 的範圍內。
- **算支持 HNM 的門檻**：該組 run 的 9 景平均 Oil IoU **高於該組基線三個 seed 的最大值**（GoM >0.4111；全球 >0.4052）。達到才補 seed 42、44；單一 seed 超過最大值在零效果下仍約有 1/4 機率發生，不得據此宣稱有效。
- **算否定的門檻**：低於該組基線三個 seed 的最小值（GoM <0.3632；全球 <0.3552）。
- 落在範圍內 → 不可判定；依使用者指示**不補 seed、不採用**。
- 輔助（描述性，不作判定）：precision、recall、離標註 >32 px 的 FP 像素數（`analysis/hnm_step0_fp_distance/fp_distance.py`）、與同 seed 基線的差。
