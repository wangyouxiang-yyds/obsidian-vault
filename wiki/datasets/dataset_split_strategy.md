---
updated: 2026-07-10
---

# 海面油汙偵測資料集切割策略規劃 (Dataset Splitting Strategy)

> ⚠️ 以下策略一～三為**分支 A（0422_VRT_training）視角**的規劃文件。分支 B（GT_expand，目前主力）現行主用切法是文末新增的「策略四」，見該節。

## 📌 核心原則：防止資料洩漏 (Data Leakage Prevention)
本專案所有資料切割（Train / Val / Test）皆**嚴格遵守「大圖層級 (Scene-level)」為最小分割單位**。
絕不將同一張大圖產生的不同 Patches 分散至訓練集與測試集中，以確保模型不會透過記憶特定海象、雲層形狀或海岸線特徵來「作弊」，確保測試結果真實反映模型泛化能力。

---

## 策略一：大圖層級完全隨機 (Scene-Level Complete Random K-Fold)
**目標**：評估模型在整體資料分佈下的基礎特徵提取能力與穩定性。

*   **實作邏輯**：
    *   將資料庫中所有獨立大圖（Scene）混合成一個總池 (Pool)。
    *   採用 **K-Fold Cross Validation (例如 5-Fold)** 進行隨機抽樣。
    *   每次 Fold 迭代中，保證 Train/Val/Test 集合內單個大圖互不重疊。
*   **優點**：能夠全面且無偏差地評估模型在現有資料域 (Data Domain) 內的平均效能，分數最具統計代表性。
*   **預期挑戰**：可能會因為時間或空間特徵的隨機混合，導致測試集難度較低（相較於後兩種策略）。

---

## 策略二：特定事件留出法 (Leave-One-Event-Out / Event Generalization)
**目標**：測試模型面對「從未見過的全新漏油災難事件」的極限泛化能力。

*   **資料特性定義**：
    *   **NOAA 日常監測資料**：檔名帶有 `NOAA_` 開頭，代表跨時間、多地點的長期監測，背景多樣性高。
    *   **特定大事件資料**：非 NOAA 開頭（例如特定船隻洩漏事件），代表單一事件產生的大量密集影像。
*   **實作邏輯**：
    *   **NOAA 日常資料**：按照大圖隨機打散，作為穩定模型背景認知的主力。
    *   **大事件資料處理**：採用 **「事件隔離 (Event Isolation)」**。
        *   將大事件歸類為 A, B, C... 等不同事件群。
        *   在 K-Fold 實驗中，指定 **「某個完整事件 (例如事件 C) 全部 100% 歸入 Test Set」**，而 Train Set 中完全不包含該事件的任何影像。
*   **優點**：驗證模型是否具備「真實災難應變能力」的最嚴格標準。如果模型在此策略下分數依然很高，代表它真正學會了「油汙物理特徵」，而不是死背特定的海域背景。

---

## 策略三：時序預測切割 (Temporal Split / Future Prediction)
**目標**：模擬真實系統上線運作 (Production) 的情境，以舊資料訓練，預測未來的影像。

*   **實作邏輯**：
    *   依據影像拍攝年份將大圖排序。
    *   **Training Set (訓練)**：使用歷史資料（例如 2018 ~ 2023 年）。
    *   **Validation Set (驗證)**：使用近期資料（例如 2024 年），用於調參。
    *   **Test Set (測試)**：保留最新年份資料（例如 2025 年）作為最終盲測基準。
*   **優點**：能真實檢驗模型是否能抵抗「時間飄移 (Temporal Drift)」，例如衛星感測器老化、氣候變遷或不同年份的季節光照差異。

---

## 總結與建議行動 (Action Items)

1.  **實驗執行順序**：建議先以「策略一」建立 Baseline 分數，隨後以「策略二」挑戰泛化極限，最後以「策略三」評估部署可行性。
2.  **資料前處理腳本更新**：需要確保 `split_dataset.py` 支援基於「年份標籤」以及「事件名稱關鍵字」的大圖分派邏輯。

---

## 策略四：油汙面積分層 3-fold（3_fold_stratified_v2）（2026-07-10 補充，分支 B / GT_expand 現行主用）

> 以上策略一～三皆為分支 A（0422_VRT_training）視角的規劃。分支 B（GT_expand，目前主力）現行主用的是本節描述的策略，腳本為 `preprocess/resplit_stratified_v2.py`，詳見 [[GT_expand_pipeline]] 第三節。

**目標**：確保每個 fold 的 test 難度（油汙大小分佈）均衡，避免某折剛好抽到特別多大油汙或特別多小油汙場景，導致 fold 間指標波動失真。

**實作邏輯**：
1. 排除髒場景（`dirty.txt` / `blur.txt` 標記）
2. 讀 mask 計算每個場景的 oil pixel count
3. 依油汙面積分 3 個 tertile 作為分層依據
4. 用 `StratifiedKFold(3)` 依 tertile 分層切 3-fold（scene-level）
5. 每個 fold 內的 train+val 再依 80/20 同樣做分層抽樣
6. `seed=42`

**輸出目錄**：`/mnt/backup/oil_dataset/new/full_band/data_split/3_fold_stratified_v2/`
