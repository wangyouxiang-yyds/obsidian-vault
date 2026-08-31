---
date: 2026-08-26
type: paper
status: full-text-read
title: "Detection and Dispersion of Small-Scale Oil Spills in Pristine Coastal Waters Using Sentinel-2 Satellite Imagery: A Case Study From Jeju Island"
authors: [Jin-Ho Lee, Kyung-Ae Park, Kwang-Seok Moon, Jae-Jin Park, Tae-Sung Kim, Moonjin Lee, Min-Sun Lee]
year: 2025
doi: 10.1109/JSTARS.2025.3613018
tags: [optical, sentinel-2, oil-spill, n-findr, spectral-angle-mapper, spectral-unmixing, unsupervised]
---

# Detection and Dispersion of Small-Scale Oil Spills in Pristine Coastal Waters Using Sentinel-2 Satellite Imagery（Lee et al., 2025）

> 來源：附件 PDF（全文已核對）
> 發表：*IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing*, 18, 24848–24863
> DOI：10.1109/JSTARS.2025.3613018

---

## 基本資訊與研究問題

論文研究 2022-07-04 韓國濟州島城山港漁船火災後約 75,500 L 柴油外洩，使用 2022-07-06 與 2022-07-21 兩景無雲 Sentinel-2 影像，追蹤薄膜／銀白虹彩型小型油污的範圍、厚度、體積與擴散方向（PDF pp.2–3，Introduction、§II-A）。

偵測部分要回答的是：在缺乏現地油污圖的情況下，能否先排除主要非水體干擾，再從同一場景自動抽取 oil／seawater 光譜端元，以 spectral angle 形成油膜 extent（PDF pp.3–4，§II-B–D）。

## 核心貢獻

這篇的核心不是提出固定波段公式，而是建立一條分層的光譜偵測管線：**land／cloud／ship masks → N-FINDR 場景端元 → oil-reference SAM → thresholded oil extent**。N-FINDR 與 SAM 都使用整段多光譜向量；SAM 輸出的是逐像素 angle map，因此不能把它稱為一個固定的「油污指數」（PDF pp.3–4，§II-B–D）。

對本專案最重要的貢獻是流程分工：先用 nuisance masks 降低候選空間，再用 oil-specific spectral evidence 確認候選。這比把任何單一 anomaly score 直接當作 oil mask 更符合本專案已觀察到的「異常不等於油」。

## 關鍵方法

### 1. 輸入與前處理

- 影像：Sentinel-2A 2022-07-06、Sentinel-2B 2022-07-21；Level-1C 經 Sen2Cor 轉為 Level-2A BOA reflectance（PDF p.3，Table I、§II-A）。
- Sentinel-2 有 13 波段、442–2202 nm、10／20／60 m 原生解析度；但論文沒有逐一列出 N-FINDR／SAM 實際輸入波段，也沒有說明共同網格、重採樣方法或最終 SAM 解析度（PDF p.3，Table I、§II-A）。
- Land：Green／NIR NDWI 加 ESA SNAP refine；NDWI threshold 與 SNAP 細節未報（PDF p.3，§II-B、式 (1)）。
- Ship：SDI threshold=0.18，再用 CFAR；background／guard／target windows 為 7×7／5×5／3×3，detection threshold=1。正文稱 Green+NIR，但式 (2) 寫 Red+NIR，存在內部不一致（PDF pp.3–4，§II-B、式 (2)）。
- Cloud：HOT=`Blue − 0.5×Red − 0.08`，threshold=0.055；小於 20 pixels 的區域移除（PDF p.4，§II-B、式 (3)）。

### 2. N-FINDR 端元抽取

在 nonwater masking 後，作者指定 `p=2`，用 N-FINDR 最大化 spectral-space simplex volume，從同一場景抽取兩個光譜極端端元及其 abundance fractions（PDF p.4，§II-C；p.6，§III-A）。作者依可見光／NIR 高亮與約 1600 nm 較低的 C–H absorption 特徵，把 endmember 1 判為 oil、endmember 2 判為 seawater（PDF pp.6–7，§III-A、Fig. 5）。

這裡依賴三個強假設：

1. 像素近似 oil 與 water 的 linear mixture；
2. 場景中存在足夠 pure／extreme 的 oil 與 water pixels；
3. 遮罩後最主要的光譜極端確實只有 oil 與 seawater。

`p=2` 在單一、乾淨事件中可把問題壓成兩端，但本專案跨事件影像還有 turbidity、sunglint、cloud edge、ship wake 與 coastal background；它們都可能成為比薄油膜更極端的 nuisance endmember。因此 scene-specific N-FINDR 不宜直接當本專案主偵測器。

### 3. SAM 光譜確認

作者以 N-FINDR 判為 oil 的 endmember spectrum `r` 作 reference，對未遮罩像素 `x` 計算 spectral angle：

```text
θ(x, r) = arccos((x · r) / (||x|| ||r||))
```

angle 越小代表光譜方向越接近 oil reference。兩日期都以固定 `τ=0.1` 產生最終油膜 extent；2022-07-06 的 oil-associated SAM <0.1，2022-07-21 源區仍可見 <0.05（PDF p.4，§II-D；p.7，§III-B）。論文沒有交代 `τ=0.1` 的校準集、搜尋範圍、選擇準則或 threshold 消融（PDF p.7，§III-B）。

### 4. 完整資料流

```text
Sentinel-2 L1C
  → Sen2Cor BOA
  → land / cloud / ship masks
  → N-FINDR（p=2；同景 oil / seawater endmembers）
  → abundance maps
  → oil-reference SAM angle map
  → τ=0.1 二值化 oil extent
  → 厚度／體積與 EFDC／Radon 擴散分析
```

偵測之後的 two-beam interference、bathymetry、EFDC 與 Radon 分析屬油污表徵與擴散研究，不是 92.03% 偵測指標的來源（PDF pp.5–6、8、10–13，§II-E–I、§III-C、§IV-B–E）。

## 主要結果與可信度審核

### 論文報告值

| 項目 | 報告值 | 證據範圍 |
|---|---:|---|
| Overall accuracy | 92.03% | 2022-07-06 同景 proxy reference（PDF p.9，§IV-A） |
| Precision | 92.03% | 同上 |
| Recall | 100% | 同上 |
| F1-score | 0.9585 | 同上；由 P=0.9203、R=1 可重算為 0.958495 |
| 偵測面積 | 307,500 → 5,400 m² | 2022-07-06 → 2022-07-21（PDF pp.7–8，§III-B/C） |

### 為什麼 92.03% 不能直接當跨場景能力

- 沒有現地 ground truth 或官方 spill map。作者以同一張 2022-07-06 Sentinel-2 RGB 正規化後計算 `SD`，再用 `SD>0.7` 建立 proxy oil reference，並與 SAM output 建 confusion matrix（PDF p.9，§IV-A、式 (13)）。
- 論文沒有提供 TP／FP／TN／FN counts、評估像素數或 evaluation mask，也沒有 baseline、threshold ablation、per-scene breakdown 或 confidence interval（PDF p.9，§IV-A；p.13，§IV-G）。
- 以標準定義核對，`Recall=1` 代表 `FN=0`，而 `Accuracy=Precision=0.9203`；若數字未經四捨五入，代數上會得到 `TN×FP=0`。因 precision<1 表示 FP>0，故只剩 TN=0。即使四捨五入容許極少 TN，沒有 counts 仍無法證明大背景下的低 false-positive burden。
- 研究只有一事件、兩景，沒有 train／validation／test split；同景 transductive endmember extraction 只能證明該場景內部可分，不能作 source-only 或跨事件泛化證據（PDF pp.3、7、9、13，§II-A、§III-B、§IV-A/G）。
- 論文稱面積由 307,500 降至 5,400 m² 是減少 82%；依兩個報告值實算為 `(307500−5400)/307500=98.24%`，兩者不一致（PDF pp.7–8，§III-B/C）。

## 與本專案的對照

| 面向 | Lee et al. 2025 | 本專案判斷 |
|---|---|---|
| 任務 | 同事件、同景端元抽取與小型油膜定位 | 跨事件多光譜 segmentation，背景干擾更複雜 |
| 核心訊號 | 場景內 spectral extremes + oil-reference angle | 已有 segmentation prediction；需要 oil-specific 後驗確認 |
| Nuisance control | land／cloud／ship masks | 高價值，可獨立加入現有 pipeline |
| Endmember | 每景 N-FINDR `p=2` | 優先改成 source-validation oil／known-water prototype library |
| 決策值 | SAM `τ=0.1`，未說明校準 | threshold 僅能由 source validation 決定 |
| 評估 | 一張 RGB proxy，同景、無 counts | event-macro IoU／F1／precision／recall + FP/km²；unknown 不得當 BG |

本專案已否定讓通用異常偵測器直接輸出油污：iForest 的 F1=0.0072、FP:TP≈230:1；DeepRX v3 的 F1=0.0081、FP:TP≈244:1；DeepRX v9 修正三項根因後 F1=0.0085，差距屬噪聲（來源：`/home/alanyh/.agents/institution/research/oil_spill_project_status.md:81-83`；對應實驗頁見相關頁面）。因此只把 iForest／DeepRX 換成 scene-specific N-FINDR+SAM，仍缺 oil-specific、跨事件穩定的監督錨點，優先度低。

## 最值得採用的部分

### A. 分層設計，而不是單一偵測器

最可借用的流程是：

```text
land / cloud / ship nuisance masks
  → 現有 segmentation 產生 candidate pixels
  → oil-specific spectral confirmation
  → verified oil mask
```

這會把「哪裡值得檢查」與「光譜是否像油」分開。Masks 可先減少明確干擾，segmentation 保留空間／語意能力，SAM 則作 verifier，而不是與 backbone 競爭。

### B. Source-prototype SAM verifier（優先方案）

先不改 backbone，也不從 target／test scene 抽取端元。只用 source training／validation 中的已標註 oil 與 known-water pixels 建 prototype library，對現有 segmentation positive 計算：

```text
margin(x) = min θ(x, water prototypes) − min θ(x, oil prototypes)
```

`margin>0` 表示像素較接近 oil；實際門檻只能由 source validation 凍結。這是本專案對 Lee 方法的改造提案，不是論文原方法。它保留 SAM angle map 作 derived feature／verifier 的價值，同時避免把同景 N-FINDR 當作泛化證據。

### C. N-FINDR 的正確位置

Per-scene N-FINDR+SAM 只放在消融最後一階，用來回答「scene-specific extremes 是否比 source prototypes 多帶來資訊」。若它只在同景有效、跨事件不穩，結果本身也能支持不採用。

## 建議的小型消融

1. **Baseline**：現有 segmentation prediction。
2. **+ nuisance masks**：加入 land／cloud／ship 排除規則。
3. **+ source-prototype SAM verifier**：只用 source validation 選 margin threshold。
4. **最後才測 per-scene N-FINDR+SAM**：不得用 target labels 選 `p`、端元身分或 threshold。

固定報告 event-macro Oil IoU、F1、precision、recall 與 FP/km²，另列逐事件差異；`unknown/ignore` pixels 不得當作 background。這能分別量化 mask、oil-specific verifier 與 transductive endmember extraction 的增量，避免只報同一張影像的 proxy accuracy。

## 研究缺口與假設

- **Linear mixture**：混合像素是否真能由 oil+water 線性組合描述，沒有驗證。
- **Pure/extreme endmember**：薄油、濁水、耀光與雲邊可能改變誰是 spectral extreme。
- **只有兩端元**：`p=2` 把多種海況壓成 oil／water，不適合直接外推到沿岸複雜背景。
- **Identity assignment**：端元需由光譜形狀人工判定為 oil；自動部署時仍需穩定規則。
- **Threshold**：`τ=0.1` 無 calibration 或 ablation。
- **Protocol**：單事件兩景、無 split、同景抽端元；沒有跨事件或 source-only 證據。
- **Reference**：RGB `SD>0.7` 是 proxy，與 SAM 共享同景表觀，可能放大一致性。
- **Reproducibility**：N-FINDR 隨機初始 simplex 未報 seed／重複次數；程式碼與衍生 outputs 未提供（PDF p.4，§II-C；pp.9、13，§IV-A/G）。

## 九步分析摘要

1. **領域地景**：被動光學多光譜、小型油污的 unsupervised spectral unmixing 與物理擴散分析。
2. **矛盾偵測**：高同景 F1 與本專案異常偵測低 precision 並不矛盾；前者有乾淨場景、遮罩與同景 oil reference，且只對 RGB proxy 評估。
3. **引用鏈**：方法鏈由 Sen2Cor／NDWI-HOT-SDI-CFAR 前處理、N-FINDR、SAM、two-beam interference、EFDC 與 Radon transform 組成；本頁未逐篇展開其上游引用。
4. **研究缺口**：缺跨事件 split、threshold calibration、counts、baseline、現地 GT 與 nuisance-condition ablation。
5. **方法審核**：分層流程合理，但 detection 指標只能支持一張同景 proxy comparison。
6. **假設殺手**：若主要 spectral extreme 是 sunglint／turbidity／cloud edge 而非 oil，`p=2` 的端元身分就會失效。
7. **知識地圖**：直接連到本 Wiki 的 unsupervised anomaly 路線、iForest／DeepRX 失敗證據與下游 DeepLabV3+。
8. **文獻綜合**：論文展示乾淨沿岸單事件內，遮罩後以 N-FINDR+SAM 追蹤小型油膜的可行性；沒有證明跨事件低誤警泛化。
9. **So What**：優先移植 nuisance masks 與 source-prototype SAM verifier；不改 backbone，不把 scene-specific N-FINDR 升為主偵測器。

## 相關頁面

- [[unsupervised_oil_detection]] — iForest／RX／VAE 路線與 Lee 2025 的位置
- [[20260613_iForest_LocalContrast_B4B8B11_全場景實驗總結]] — iForest F1=0.0072、FP:TP≈230:1
- [[20260614_DeepRX_EM_v3_全場景實驗總結]] — DeepRX v3 F1=0.0081、FP:TP≈244:1
- [[DeepLabV3+]] — 現有 segmentation backbone／candidate producer
- [[Sun2024_中解析度光學油污分割]] — S2/L8/L9 中解析度光學分割對照
