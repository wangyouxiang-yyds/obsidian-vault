---
date: 2026-06-03
type: experiment
stage: preprocess
status: running
tags: [unsupervised, anomaly-detection, isolation-forest, OSI, sentinel-2, multispectral]
---

# Unsupervised 油汙偵測初探

## 目標

在沒有標注的情況下，利用多光譜影像的光譜異常偵測初步定位疑似油汙區域，作為 DeepLabV3+ 語義分割的前置過濾器（兩階段 pipeline 的 Stage 1）。

---

## 研究背景與文獻定位

參考 [[Duan2022_HOSD_IsolationForest]]，首次完整的 Unsupervised 油汙偵測流程為：
```
KPCA 降維 → Isolation Forest → SVM → Extended Random Walker
```
本實驗簡化為只取 iForest 核心，並與 Oil Spill Index（OSI）對比。

文獻缺口：現有 Unsupervised 油汙研究多針對高光譜（224+ 波段），對多光譜 Sentinel-2（8 波段）的應用幾乎空白。

---

## 實驗一：高光譜基準測試（HOSD Dataset）

### 設定
- 資料集：HOSD（2010 年墨西哥灣漏油，機載高光譜，18 張 .mat 檔）
- 影像大小：1386 × 690，224 個波段，int16
- Mask 類別：0 = 海水，1 = 油汙
- 腳本：`D:\LAB\oil_unsupervised\Hyperspectral_oil_spill_detection_datasets\run_iforest.py`

### 參數
- 油汙比例：4.12%（39,360 / 956,340 px）
- `contamination = 0.0412`（自動計算）
- `n_estimators = 100`
- 前處理：StandardScaler

### 結果（GM03）

| 指標 | 數值 |
|------|------|
| F1（油汙） | 0.78 |
| Precision | 0.78 |
| Recall | 0.78 |
| **AUC** | **0.9899** |

### 結論
- AUC 0.99 代表 iForest 的光譜排序能力極強
- 高光譜 224 個波段資訊豐富，是 S2 8 波段的最優上界參考

---

## 實驗二：Sentinel-2 多光譜（5 個 NOAA 場景）

### 設定
- 資料路徑：`F:\MS_IR\After_sen2like\NOAA_Gulf_of_Mexico_High_Confidence\2022\`
- 波段：B01/B02/B03/B04/B08/B8A/B11/B12（8 個波段，uint16，值域 ~1000–13000）
- 腳本：`D:\LAB\oil_unsupervised\run_iforest_batch.py`

### Pipeline
```
各波段 TIF（不同解析度）
    ↓ bilinear resample → 全部對齊 10980×10980
    ↓ 雲遮罩（NDWI > 0 保留水體，B02 < 3000 排除雲）
    ↓ oil.gpkg bbox + 5km buffer 裁切工作區域
    ↓ 隨機取樣最多 20 萬有效像素（True Unsupervised，不看 label）
    ↓ StandardScaler + IsolationForest(n_estimators=100, contamination=0.02)
    ↓ 對整個 crop 預測 anomaly score
    ↓ F1 最佳閾值（PR curve）+ 空間後處理（移除 < 50px 連通區塊）
    ↓ 評估（oil.gpkg rasterize mask 作為 GT）
```

### OSI（Oil Spill Index）
```python
OSI = (B01 + B02) / (B03 + B11)
```
同樣使用 PR curve F1 最佳閾值 + 空間後處理評估。

### 結果（v1：Youden 閾值，無空間後處理）

| Scene | Oil px | iF AUC | iF F1 | OSI AUC | OSI F1 |
|-------|--------|--------|-------|---------|--------|
| 20220619_S2_15RXM | 12,342 | 0.9458 | 0.079 | 0.8079 | 0.026 |
| 20221103_S2_16RCT | 3,685  | 0.7033 | 0.013 | 0.5732 | 0.008 |
| 20221111_S2_15RXM | 2,935  | 0.5134 | 0.008 | 0.8510 | 0.022 |
| 20220417_S2_16RBS | 4,858  | 0.6661 | 0.022 | 0.7525 | 0.019 |
| 20220629_S2_15RXM | 8,057  | 0.7877 | 0.007 | 0.7445 | 0.006 |

### 關鍵問題診斷

**AUC 還可以，但 F1/Precision 極低**

| 場景 | 問題 |
|------|------|
| 20220619（AUC=0.95）| AUC 良好，F1 低是 threshold 不適合不平衡資料 |
| 20221111（AUC=0.31）| 低於 0.5，油汙光譜與海水幾乎相同（薄油膜？），或雲霧干擾 |

根本原因：油汙像素佔 crop 總面積不到 1%，Youden 閾值對不平衡資料不適用，導致大量假陽性。

### 改善措施（v2，進行中）
1. **F1 最佳閾值**：改用 Precision-Recall curve 直接最大化 F1
2. **空間後處理**：移除面積 < 50px 的孤立連通區塊
3. 結果待更新

---

## 概念釐清：Pipeline 中的角色

目前評估腳本使用 `oil.gpkg` 來裁切工作區域，這**只用於評估**，真實部署時不存在。

**真實部署 Pipeline：**
```
整張衛星影像進來（無標注）
    ↓ 雲遮罩
    ↓ iForest / OSI 對全圖計算 anomaly score map
    ↓ 閾值 → 疑似油汙區域
    ↓ 送進 DeepLabV3+ 精確分割
```

下一步需要驗證：對**整張影像**（不依賴標注裁切）跑 iForest/OSI，anomaly score map 的品質如何。

---

## 技術備忘

### 波段解析度問題
sen2like 輸出各波段解析度不同，需在讀取時 resample：
- B01: 1830×1830（60m）→ resample to 10980×10980
- B8A/B11/B12: 5490×5490（20m）→ resample to 10980×10980
- B02/03/04/08: 10980×10980（10m，基準）

### GPKG 座標系問題
不同場景的 GPKG CRS 不一致（EPSG:3857 / EPSG:4326），需 `.to_crs(ref_crs)` 統一轉換到 TIF 的 UTM 座標系。

### 無 SCL 問題
sen2like 輸出不含 SCL，雲遮罩改用：
- `NDWI = (B03 - B08) / (B03 + B08) > 0`（保留水體）
- `B02 < 3000`（排除雲，閾值待校正）

---

## 下一步

- [ ] 跑完 v2（F1 閾值 + 空間後處理）看 Precision 是否提升
- [ ] 對**整張影像**（不用標注裁切）跑全圖 anomaly score map，驗證真實場景可行性
- [ ] 比較 iForest vs OSI 在各 scene 的強弱，確認哪個更穩定
- [ ] 若 AUC 持續低（如 20221111），研究是否為薄油膜造成，需要更敏感的特徵

---

## 相關頁面
- [[Duan2022_HOSD_IsolationForest]]
- [[unsupervised_oil_detection]]
- [[DeepLabV3+]]
