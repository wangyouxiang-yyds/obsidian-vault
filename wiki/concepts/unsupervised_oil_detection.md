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

### 2. Isolation Forest（已否定作為單獨主偵測器）
- 隨機切割特徵空間，需要更少分割才能孤立的點 → 異常分數高
- 參考論文：[[Duan2022_HOSD_IsolationForest]]
- **優點**：
  - 不依賴整張影像的 covariance，雲遮罩後缺值不會讓模型崩潰
  - 純 Unsupervised，不需要 label
  - 多光譜（10-12 個波段）反而比高光譜更乾淨，雜訊更少
  - 不需要 GPU，sklearn 即可執行，幾分鐘出結果
  - 輸出連續 anomaly score（soft mask）或二值化（pseudo label）皆可

> **2026-08-26 現況校正**：上述是早期候選理由，不是現行推薦。全場景實驗的 iForest F1=0.0072、FP:TP≈230:1；DeepRX v3 F1=0.0081、FP:TP≈244:1，v9 F1=0.0085，已否定「通用異常分數直接當油污結果」這條路。來源：`/home/alanyh/.agents/institution/research/oil_spill_project_status.md:81-83`、[[20260613_iForest_LocalContrast_B4B8B11_全場景實驗總結]]、[[20260614_DeepRX_EM_v3_全場景實驗總結]]。

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

## Lee et al. 2025：N-FINDR + SAM 的可借用位置

Lee et al. 先遮罩 land／cloud／ship，再以 N-FINDR `p=2` 從同一 Sentinel-2 場景抽取 oil／seawater endmembers，最後以 oil endmember 計算 SAM angle map、用 `τ=0.1` 形成油膜 extent。N-FINDR／SAM 使用多光譜向量；**不是固定波段指數**，SAM angle map 比較適合視為 derived feature 或 oil-specific verifier（PDF pp.3–4、6–7，§II-B–D、§III-A–B；DOI `10.1109/JSTARS.2025.3613018`）。

本專案最值得移植的是 `nuisance masks → candidate/prototype → oil-specific spectral confirmation` 的分層邏輯。優先方案是在 source validation 以已標註 oil／known-water 建 prototype library，計算 `min θ(water) − min θ(oil)` margin，放在現有 segmentation positives 後確認；先不改 backbone，也不把 scene-specific N-FINDR 當主偵測器。

原因是 N-FINDR `p=2` 依賴 linear mixture、pure/extreme endmember 與「主要只剩 oil+water」；跨事件的 turbidity、sunglint、cloud edge、ship wake、coastal background 可能反而成為極端端元。論文也只有單一事件兩景、無 split，`τ=0.1` 無 calibration／ablation，同景 transductive extraction 不能當 source-only 泛化證據（PDF pp.3–4、7、9、13，§II-A/C–D、§III-B、§IV-A/G）。

若做小型消融，順序為 baseline → +nuisance masks → +source-prototype SAM verifier → 最後才測 per-scene N-FINDR+SAM；threshold 僅由 source validation 決定，報 event-macro IoU／F1／precision／recall 與 FP/km²，unknown mask 不得當 BG。完整方法與指標可信度審核見 [[Lee2025_NFINDR_SAM_小型油污]]。

---

## 相關頁面
- [[Duan2022_HOSD_IsolationForest]] — 最重要的對標論文
- [[hard_negative_mining]] — HNM 亦可與 iForest pseudo label 結合
- [[DeepLabV3+]] — 下游語義分割模型
- [[Lee2025_NFINDR_SAM_小型油污]] — N-FINDR + SAM 方法、限制與 verifier 消融建議
