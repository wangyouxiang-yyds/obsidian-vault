---
date: 2026-06-13
type: experiment
stage: unsupervised
status: completed
tags: [unsupervised, isolation-forest, local-contrast, sentinel-2, B4B8B11, patch-level, full-scene, oil-detection, evaluation]
---

# iForest + Local Contrast（B4/B8/B11 三波段版）全 439 場景無監督油汙偵測實驗

## 實驗背景

無監督油汙偵測 pipeline 的設計目標：模擬研究者在 QGIS 上「調整 Min/Max 拉開對比 → 用 B4/B8/B11 三波段組合 RGB 顯示 → 視覺找跟周圍水體不一樣的區域」的標記行為。

對應到演算法上：

- 視覺對比 → Local Contrast（z-score 鄰域比較）
- 三波段組合 → 用 B4 (Red 665nm) / B8 (NIR 842nm) / B11 (SWIR 1610nm) 三波段均值當特徵
- 異常排序 → iForest 評分

---

## Pipeline 流程

```
VRT 場景（10980×10980×8 bands）載入
    ↓
Patch Pre-filter（patch_is_scoreable）
    擋掉海陸邊界、雲邊、紋理區、NoData
    ↓
Step 0: Local Contrast（B4/B8/B11 特徵）
    對每個 patch 算 B4/B8/B11 三波段均值
    與 ±5 patch 鄰域比 z-score（取絕對值，雙向偵測）
    score = max(|z_B4|, |z_B8|, |z_B11|)
    取 top 15% 進入下一階段
    ↓
Pass 1: iForest 訓練
    從乾淨深水 patch 抽 500 個訓練 StandardScaler + PCA(4) + iForest
    ↓
Pass 2: iForest 評分
    對 Local Contrast 候選做 PCA + iForest 評分
    每個 patch 抽 200 像素，取 p5 當 patch 分數
    取 top 20% 為最終候選
    ↓
驗證：cand_set ∩ oil_set → TP/FP/FN/Recall/Precision/F1
```

---

## 關鍵參數設定

### Patch 設定

| 參數 | 值 |
|------|----|
| PATCH_SIZE | 256 |
| STRIDE | 192（25% 重疊） |

### Patch Pre-filter（擋假油汙）

| 參數 | 值 | 說明 |
|------|----|------|
| NODATA_RATIO_MAX | 0.05 | >5% NoData 跳過 |
| COAST_LAND_RATIO_MAX | 0.02 | >2% 陸地像素跳過 |
| PATCH_HETERO_STD_MAX | 0.04 | B02 整片 std 過大跳過 |

### Local Contrast（Step 0）

| 參數 | 值 | 說明 |
|------|----|------|
| 特徵 | B4, B8, B11 | 三波段「水體像素」均值 |
| LOCAL_WINDOW_PATCHES | 5 | ±5 patch = 11×11 = 121 個鄰居 |
| 分數公式 | max(\|z_B4\|, \|z_B8\|, \|z_B11\|) | 雙向偵測，亮油暗油都抓 |
| LOCAL_CONTRAST_TOPK | 0.15 | top 15% 進 iForest |

### iForest 訓練（Pass 1）

| 參數 | 值 |
|------|----|
| N_FIT_PATCHES | 500 |
| PIX_PER_PATCH | 100 |
| PCA_D | 4 |
| IFOREST_T | 800 |
| IFOREST_PSI | 256 |

### iForest 評分（Pass 2）

| 參數 | 值 |
|------|----|
| SCORE_PERCENTILE | 80（保留 bottom 20% 最異常） |
| MIN_VALID_RATIO | 0.3 |
| SCORE_PIX_PER_PATCH | 200 |
| SCORE_AGG_PERCENTILE | 5（取 patch 內 p5 最異常） |

### 水體 / 雲遮罩門檻

| 參數 | 值 |
|------|----|
| NDWI_THRESH | 0.0 |
| NDWI_CLEAN_THRESH | 0.1 |
| BLUE_THRESH | 0.20 |
| SWIR1_THRESH | 0.05 |

### 加速優化

- 跳過 SVM 階段（產生不出有意義的 pseudo label）
- valid_mask + patch_is_scoreable 結果快取（避免重複計算）
- Multiprocessing：4 workers 平行處理場景（每場景獨立 RNG，依場景名 hash 種子）

---

## 量化指標（438 個有 GT mask 的場景）

### 整體 Micro Average

| 指標 | 數值 |
|------|------|
| 評估場景數 | 438 |
| Total TP | 104 |
| Total FP | 24,063 |
| Total FN | 4,516 |
| **Micro Recall** | **0.0225** |
| **Micro Precision** | **0.0043** |
| **Micro F1** | **0.0072** |

### 指標解讀

#### 三個指標的意義

| 指標 | 公式 | 直觀意義 |
|------|------|---------|
| **Recall（召回率）** | TP / (TP + FN) | 抓到的油汙 / 全部真實油汙 = 「有沒有漏掉？」 |
| **Precision（精準率）** | TP / (TP + FP) | 抓對的 / 全部候選 = 「猜得對嗎？」 |
| **F1** | 2·R·P / (R+P) | Recall 和 Precision 的調和平均，整體平衡 |

#### 該看哪個？

| 用途 | 主要看 | 為什麼 |
|------|------|--------|
| **本方法**（候選提示，人工複查為主） | **Recall** | 寧可多給候選，重點是別漏掉 |
| 自動化最終偵測 | Precision | 候選即結論，錯太多沒人要看 |
| 學術論文比較 | F1 | 一個數字代表整體表現 |

由於本方法目標是「候選提示器」協助人工標記，且實驗策略明確採用「寧可錯抓不要漏抓」，**主要應該看 Recall**。

#### 套用到本實驗結果

| 指標 | 數值 | 解讀 |
|------|------|------|
| **Micro Recall = 0.0225** | 抓到 **2.25%** 的真實油汙 patch | **漏掉 97.75%，這是核心問題** |
| Micro Precision = 0.0043 | 候選中只有 0.43% 是真油汙 | 大量 FP，但設計上可接受 |
| Micro F1 = 0.0072 | — | 平均後很低，對本用途意義不大 |

**結論：** 即便 Precision 的低分是設計允許的代價，Recall 僅 2.25% 表示這個方法在現階段**連「候選提示器」的門檻都很難達到**。要拿來協助人工標記，至少需要 Recall ≥ 0.3 才有實際幫助。

### 各事件表現分布

| 事件 | 場景數 | GT 油汙 patch | TP | 平均 Recall | 非零 Recall 場景 |
|------|--------|---------------|----|------------|------------------|
| 2024 千里達及多巴哥油駁船翻覆 | 8 | 164 | 8 | **0.155** | 5/8 |
| 2022 祕魯 La Pampilla 煉油廠洩油 | 6 | 75 | 12 | 0.087 | 3/6 |
| 2019 所羅門群島 MV Solomon Trader | 4 | 94 | 4 | 0.069 | 2/4 |
| NOAA Gulf of Mexico | 248 | 2,956 | 68 | 0.017 | 25/248 |
| NOAA Atlantic Ocean | 150 | 1,000 | 12 | 0.014 | 8/150 |
| 2017 希臘薩龍灣 Agia Zoni II | 4 | 91 | 0 | 0.000 | 0/4 |
| 2021 斯里蘭卡珍珠號化學品 | 4 | 19 | 0 | 0.000 | 0/4 |
| 2020 模里西斯洩漏 | 1 | 4 | 0 | 0.000 | 0/1 |
| 2021 美國 Golden Ray | 2 | 45 | 0 | 0.000 | 0/2 |
| 2023 帛琉籍天使輪 | 7 | 70 | 0 | 0.000 | 0/7 |
| NOAA Pacific Ocean | 4 | 102 | 0 | 0.000 | 0/4 |

### Patch Size 比較（10 場景子集驗證）

| 版本 | Micro Recall | 備註 |
|------|-------------|------|
| 256 (NIR p10 + NDWI p10) | 0.097 | 舊版單向偵測 |
| **256 (B4/B8/B11 max abs z)** | **0.106** | **新版雙向偵測（最終採用）** |
| 800 (NIR p10 + NDWI p10) | 0.060 | 大 patch 降精度 |
| 1024 (NIR p10 + NDWI p10) | 0.040 | 太大失效 |

結論：256 + B4/B8/B11 雙向偵測為最佳組合。

---

## 觀察與結論

### 表現好的事件特徵

- **千里達、La Pampilla、Solomon**：明顯且集中的油汙，光譜跟周圍水體差異大
- 這些都是化學品 / 重油類的污染，光譜上有明顯特殊性

### 全零事件特徵

- **希臘、斯里蘭卡、模里西斯、Golden Ray、帛琉、Pacific**：完全失效
- 可能原因：
  - 油汙太薄、光譜接近乾淨水體
  - 場景中有大量近岸 / 雲遮蔽
  - 油汙分布散亂，被 patch_is_scoreable 過濾掉
  - B4/B8/B11 特徵對這類油汙不敏感

### 整體限制

1. **單一方法難涵蓋所有油汙類型**：薄油膜、厚油膜、乳化油的光譜特徵差異很大
2. **無監督模式下假陽性極多**：FP=24,063 vs TP=104，比例約 230:1
3. **NOAA Gulf Mexico 主導訓練資料**：佔資料集 56.6%（248/438），其他事件被淹沒
4. **DeepRX + VAE 對照組無效**：實驗確認 VAE 訓練資料污染問題無法解決

### 後續改進方向

- [ ] 細看全零事件：實際看影像了解油汙視覺特徵，調整特徵設計
- [ ] 多特徵融合：兼顧亮油 / 暗油 / 光譜異常多種偵測邏輯
- [ ] 半監督方法：用少量人工標記做 fine-tune
- [ ] Patch Size 微調：考慮 128 或 192 看是否提升空間精度

---

## 實驗檔案位置

| 類型 | 路徑 |
|------|------|
| 程式碼 | `/mnt/backup/alanyh/oil_unsupervised/ms6_iforest/generate_iforest_patch_coords_B4B8B11_all.py` |
| 輸出目錄 | `/mnt/backup/alanyh/oil_unsupervised/ms6_iforest/output_B4B8B11_all/` |
| CSV 結果 | `/mnt/backup/alanyh/oil_unsupervised/ms6_iforest/output_B4B8B11_all/eval_results.csv` |

---

## 跑分資訊

| 項目 | 數值 |
|------|------|
| 平行 workers | 4（場景級 multiprocessing） |
| 記憶體消耗 | ~20 GB（4 workers × 5 GB/scene） |
| 總跑分時間 | ~26 小時（含 viz 輸出） |
| 中途問題 | 某場景 scored 列表為空導致 np.percentile 失敗；加入 resume 機制後完成 |

---

## 相關頁面

- [[20260603_Unsupervised油汙偵測初探]]
- [[20260604_iForest全場景改進]]
- [[20260607_iForest_Pipeline精簡化與全場景執行]]
- [[20260530_實驗結果彙整]]
