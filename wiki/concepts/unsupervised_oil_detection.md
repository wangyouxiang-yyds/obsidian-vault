---
type: concept
tags: [unsupervised, anomaly-detection, isolation-forest, oil-spill, sentinel-2, cloud-masking]
related: [Duan2022_HOSD_IsolationForest, hard_negative_mining, cloud_optimized_geotiff]
---

# Unsupervised 油汙偵測方法

## 一句話定義

不依賴標注資料，透過光譜異常分數（anomaly score）在衛星影像中初步定位疑似油汙區域，作為語義分割的前置過濾器。

---

## 研究動機

本專案的兩階段設計：
```
衛星影像（Sentinel-2 / Landsat via sen2like）
    ↓ 第一階段：Unsupervised 異常偵測（粗定位）
    ↓ 第二階段：DeepLabV3+ 語義分割（精確分割）
```

Unsupervised 初步定位的優勢：不需要 label 就能產生 pseudo mask，有助於縮小後續模型需要處理的範圍，也可用來擴充訓練資料。

---

## 主要方法分類

### 1. RX / Deep-RX（統計異常偵測）
- 基於 Mahalanobis 距離，假設背景像素符合高斯分佈
- 高光譜（HSI）領域的主流方法
- **問題**：雲和霧氣會嚴重污染整張影像的 covariance 估計，導致雲邊緣被誤判為異常
- **結論**：除非雲量 < 5%，否則對本專案資料集不穩定，暫不採用

### 2. Isolation Forest（最推薦）
- 隨機切割特徵空間，需要更少分割才能孤立的點 → 異常分數高
- 參考論文：[[Duan2022_HOSD_IsolationForest]]
- **優點**：
  - 不依賴整張影像的 covariance，雲遮罩後缺值不會讓模型崩潰
  - 純 Unsupervised，不需要 label
  - 多光譜（10-12 個波段）反而比高光譜更乾淨，雜訊更少
  - 不需要 GPU，sklearn 即可執行，幾分鐘出結果
  - 輸出連續 anomaly score（soft mask）或二值化（pseudo label）皆可

### 3. K-Means / GMM + 波段指數
- 設計油汙光譜指數（如 `(B1 + B2) / (B3 + B11)`），再用 K-Means（k=2）或 GMM 分成「油汙」vs「海水」
- 優點：計算量極小，論文好解釋
- 用途：當作 baseline 或消融實驗的對照組

### 4. Autoencoder / VAE
- 深度 AE 用於 SAR 油汙分割：Residual Encoder-Decoder，F1 達 93.01%（但為 SAR 影像）
- VAE 用於油汙時間演化預測
- **問題**：訓練時若混入雲像素，latent space 的高斯假設被污染，同樣會把雲邊緣當異常
- **結論**：與 Deep-RX 問題相同，暫列後期評估

---

## 建議實施路線（Isolation Forest）

```python
# 輸入：sen2like harmonized 多光譜 TIF（8 band）
# Step 1：雲與非水體遮罩
NDWI = (Green - NIR) / (Green + NIR)
water_mask = NDWI > 0          # 保留海面像素
cloud_mask = Blue > 閾值        # 排除雲（Blue 波段高反射）
valid_mask = water_mask & ~cloud_mask

# Step 2：選波段
# 推薦：B01, B02, B03, B8A, B11, B12（文獻建議對油水區分最有效）

# Step 3：Isolation Forest
from sklearn.ensemble import IsolationForest
clf = IsolationForest(contamination=0.01~0.05)  # 預期油汙佔比
clf.fit(pixels[valid_mask])
anomaly_score = clf.decision_function(all_pixels)

# Step 4：輸出
# anomaly score 圖 → 視覺化確認 → 二值化 threshold → pseudo label
```

---

## 雲遮罩問題與解法

sen2like 輸出**不含 SCL**（Scene Classification Layer），需自行用波段推算：

| 遮罩目標 | 方法 |
|---------|------|
| 水體範圍（保留） | `NDWI = (Green - NIR) / (Green + NIR) > 0` |
| 雲（排除） | `Blue > 0.2`（或 > 2000 DN，依資料縮放值調整） |
| 薄雲/霧（輔助） | SWIR 波段反射率：真實海面 SWIR 極低，薄雲中等 |

> 注意：SCL 對油汙的分類不準，厚油膜可能被分成 Unclassified(7) 或 Cloud Shadow(3)。即便有 SCL，也應保留 SCL=6（Water）+ SCL=7（Unclassified）兩類再跑 iForest。

---

## 執行順序建議

1. **本週**：用 NDWI 雲遮罩 + Isolation Forest 跑一張雲量較少的影像，看 anomaly score 圖
2. **確認有信號後**：調整 `contamination` 參數（預設 0.1 → 油汙場景建議 0.01~0.05）
3. **之後**：把 iForest output 當 pseudo label，接 DeepLabV3+ 訓練

---

## 文獻缺口（本研究定位）

| 現有研究 | 空白 |
|---------|------|
| 高光譜 HSI + iForest（HOSD dataset） | 多光譜 Sentinel-2/Landsat + RX/iForest 應用稀少 |
| SAR + Autoencoder | 光學影像 + VAE anomaly detection 幾乎沒有 |
| Sentinel-2 波段指數 + 監督式分類 | Unsupervised 初步定位 → 語義分割兩階段框架幾乎空白 |

---

## 相關頁面
- [[Duan2022_HOSD_IsolationForest]] — 最重要的對標論文
- [[hard_negative_mining]] — HNM 亦可與 iForest pseudo label 結合
- [[DeepLabV3+]] — 下游語義分割模型
