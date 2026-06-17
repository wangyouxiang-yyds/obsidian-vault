---
type: concept
tags: [deeprx, vae, anomaly-detection, unsupervised, hyperspectral, mahalanobis, background-only-training]
related: [Song_DeepRX_Hyperspectral_Anomaly, iforest_架構與運作機制, unsupervised_oil_detection]
date: 2026-06-17
---

# Deep-RX (VAE + RX) 架構與運作機制（深入理解版）

> 本頁為對話式學習紀錄，源自 2026-06-17 與 Claude 的 Q&A，逐步釐清 Deep-RX 的 VAE 架構本質、Song et al. 2023 在 San Diego I 的完整處理流程、訓測同源爭議的誠實討論，以及 Background-Only Training 的核心洞察。
>
> 論文摘要另見 [[Song_DeepRX_Hyperspectral_Anomaly]]，本頁不重複摘要，聚焦在使用者的提問脈絡與深層理解。

---

## 1. 提問脈絡（學習路徑）

1. 全部解釋 Deep-RX + VAE 一遍
2. 訓練/測試是不是同一批資料？
3. 是不是隨機抽幾個像素訓練後用回原圖打分？
4. 這樣不就是作弊嗎？
5. 論文裡實際是怎麼使用資料的？有講嗎？
6. 「沒講」那就是同一張圖訓+測啊，那就是作弊！
7. 如果只訓乾淨水體再對全圖打分，可不可以？ ← **本次最重要的洞察**

---

## 2. VAE 是什麼？

### 從 Autoencoder 說起

Autoencoder（AE）是一個「壓縮再還原」的神經網路：

```
輸入 x (224維)
    ↓ Encoder
潛在向量 z (10維)   ← 資訊被壓縮
    ↓ Decoder
重建 x̂ (224維)
```

訓練目標：讓 x̂ 盡量接近 x（最小化重建誤差 MSE）。

Encoder 被迫學「什麼特徵最重要」，學到的 z 就是一個緊湊的語意表示。

### VAE 的「V」加在哪

VAE = Variational Autoencoder。它在 AE 的基礎上加了一個約束：

| 面向 | 普通 AE | VAE |
|------|---------|-----|
| z 是什麼？ | 一個固定向量 | **一個分布** N(μ, σ²) |
| Encoder 輸出 | z | **μ 和 σ**（分布的兩個參數）|
| z 怎麼取得？ | 直接 | 從 N(μ, σ²) **抽樣** |
| Loss | 只有 MSE | MSE + **KL 散度懲罰** |

KL 散度的作用：強迫每個像素的潛在分布 N(μ, σ²) 趨近標準高斯 N(0, I)。這個約束讓整個 latent space 形成一個有結構的高斯分布。

### Reparameterization Trick

直接「從 N(μ, σ²) 抽樣」無法做反向傳播（抽樣操作沒有梯度）。VAE 的巧思：

```
z = μ + σ ⊙ ε，其中 ε ~ N(0, I)
```

- 隨機性由 ε 承擔
- 梯度只流過 μ 和 σ（可微）
- 數學等價，但可以訓練

### Loss = λ₁·MSE + λ₂·KL

```
L = λ₁ · ‖x - x̂‖²   (重建誤差)
  + λ₂ · KL[N(μ,σ²) ‖ N(0,I)]  (讓 latent 趨近高斯)
```

KL 散度的展開式（對角協方差 σ² 情形）：

```
KL = ½ Σ (σ² + μ² - 1 - log σ²)
```

這是解析解，不需要蒙特卡羅估計。

---

## 3. 為什麼要 VAE + RX 兩個串起來？

### 單獨用 RX 的問題

RX（Reed-Xiaoli）是高光譜偵測的經典方法，核心公式：

```
r(x) = (x - μ̂)ᵀ Σ̂⁻¹ (x - μ̂)   ← Mahalanobis 距離
```

**問題 A**：RX 假設背景服從高斯分布。高光譜資料（224 波段）實際上是多峰分布（水、植被、土壤、陰影各自成群），不是高斯。

**問題 B**：高維空間的協方差矩陣 Σ̂ 估計困難。224×224 的矩陣需要 >>224 個樣本才能可靠估計，而且很容易接近奇異（singular），求逆不穩定。

### 單獨用 VAE 的問題

只用重建誤差（‖x - x̂‖²）當異常分數也行，但：

- 浪費了 VAE 已經學好的 latent 高斯結構
- 重建誤差受 λ₁/λ₂ 權衡影響，不夠穩定

### 合起來的邏輯

```
VAE Encoder → latent z ~ N(μ, σ²)
                        ↑
          KL 約束強迫這裡是高斯
                        ↓
      在 latent space 做 RX → RX 的高斯假設在這裡成立
      維度只有 10 → Σ 估計穩定
```

Deep-RX 的核心思想：**用 VAE 把「對 RX 不友善的分布」轉換成「對 RX 友善的分布」**，再讓 RX 在轉換後的空間裡發揮。

---

## 4. Deep-RX 對 San Diego I 的完整流程

### 資料集規格

San Diego I：100×100 像素，189 有效波段（AVIRIS 高光譜），異常目標為飛機（3 架，各自形狀不同）。

### 訓練階段：VAE 學習背景光譜結構

```
輸入影像 X shape = (100, 100, 189)
    ↓ 攤平
(10000, 189)   ← 每列是一個像素

VAE 架構（全連接 MLP）：
    Encoder: 189 → 128 → 64 → 32 → [μ(10), σ(10)]
    Decoder: 10 → 32 → 64 → 128 → 189

訓練：
    optimizer = SGD（mini-batch）
    epochs = 300
    所有 10,000 個像素都會在每個 epoch 被看到
```

每個 epoch，VAE 對「包含飛機」的全部像素都學一遍。但飛機只佔 10,000 個像素中的一小部分（<<1%），VAE 的 loss 主要由大量背景像素決定，它學到的是**背景的光譜結構**。飛機像素的存在對 loss 影響微小，VAE 基本上忽視它們。

### 推論階段：丟掉 Decoder，只用 Encoder

```
訓練好的 VAE → 只保留 Encoder 的 μ 分支（丟棄 σ 分支和整個 Decoder）

全部 10,000 個像素 → Encoder → μ 向量
    每個像素 xᵢ → μᵢ ∈ ℝ¹⁰

→ 得到 latent feature matrix Z shape = (10000, 10)
```

### 計算 RX 異常分數

```
在 latent space 估計背景統計量：
    μ̂_b = mean(Z, axis=0)    ← 10 維均值向量
    Σ̂_b = cov(Z)             ← 10×10 協方差矩陣

對每個像素計算 Mahalanobis 距離：
    r(xᵢ) = (μᵢ - μ̂_b)ᵀ Σ̂_b⁻¹ (μᵢ - μ̂_b)

→ anomaly score map shape = (100, 100)
→ AUC = 0.991（論文 Table 1）
```

---

## 5. 訓練 / 測試同一張的爭議（誠實版）

### 論文裡實際寫了什麼

- **§3.1**：用 X 表示輸入高光譜影像
- **§3.2**：用 T（to be detected）表示待偵測影像
- **Algorithm 1**：只有一個輸入 X，沒有分 train set / test set
- **§4.1**：三個資料集（San Diego I/II, HyperUrban）各自獨立介紹
- **§4.2**：「The network was trained for 300 epochs」——「The network」用單數，暗示每個資料集各自訓一個 VAE

### 論文沒明說的

- X 和 T 是否相同（= 是否同張影像訓+測）
- 是否每個 dataset 各自獨立訓練一個 VAE
- 訓練時是否包含全部像素（包含目標飛機）

### 合理推論

**高機率是：每張影像各自訓一個 VAE + 同張影像推論（per-scene transductive）**

推論依據：
1. Algorithm 1 只有一個輸入，暗示 X = T
2. 三個 dataset 波段數不同（189 / xxx / xxx），VAE 輸入維度不同，無法共用一個 VAE
3. 高光譜異常偵測（HAD）領域的慣例就是 per-scene：每張影像單獨訓練模型然後對同張影像偵測

---

## 6. 「這算不算作弊？」的誠實討論

### 嚴格定義：不算作弊

VAE 在訓練時從未看過 Ground Truth label（哪個像素是飛機）。它不知道「飛機」這個概念。它只是學「這 10,000 個光譜向量的整體分布長什麼樣」。

這類似於 [[iforest_架構與運作機制]] 第 5 節討論的 iForest transductive analysis——學的不是「x → y 對應」，而是「資料的幾何結構」。

### 但評估方式有實質問題

**問題 A：AUC 0.991 的泛化意義有限**

「AUC = 0.991」只證明：在 San Diego I 這張影像上，VAE 學到的背景結構能讓飛機的 Mahalanobis 距離排在水泥地 / 跑道 / 草地之前。這不等於：對一張新的機場影像，同樣的架構也能做到 0.991。

**問題 B：沒做 cross-scene 評估**

論文沒有做 Setup B 形式的評估：「在影像 A 訓練 VAE，在影像 B 偵測」。這種評估才能驗證「學到的東西是否可遷移」。

**問題 C：基準線比較不公平**

論文拿 Deep-RX（每張影像都訓了 300 epochs）去比 GRX/LRX/KRX（傳統方法，沒有機會「學」這張影像）。傳統方法在「不學習」的前提下輸了，這並不驚人——更有意義的比較是 Deep-RX vs 其他「也對同一張影像做 per-scene fitting」的方法。

### HAD 領域的慣例與侷限

Per-scene evaluation 是 HAD 領域的通行做法，不只 Deep-RX 這樣——大部分論文都這樣，所以論文被接受不代表這個評估方式本身沒有缺陷。

學術界確實有人批評：per-scene training 讓方法看起來比實際上更強，而且讓研究結果難以在真正的新場景部署時復現。

### 對本專案的直接影響

論文 AUC = 0.991 是在 AVIRIS 機載高光譜（189 波段，100×100）、已知只有飛機異常的乾淨場景下拿到的。本專案的 v9 面對的是 Sentinel-2 多光譜（8-11 波段，真實海洋場景，雲/陸地/船都混在裡面）。論文的 0.991 不能直接套用到 v9 的期望值上。

---

## 7. 使用者的核心洞察：Background-Only Training

### 使用者的提問

> 如果只訓乾淨水體再對全圖打分，可不可以？

### 這個策略的學術名稱

**Background-Only Training** / Clean Background Bootstrap：

訓練 VAE 時，只餵入「確定是背景」的像素，讓 VAE 學到一個純淨的背景分布。推論時，對全部像素（包含潛在油汙）計算 Mahalanobis 距離。

### 雞生蛋問題的解

「你說只給乾淨水體，可是你不知道哪裡是油汙、哪裡才算乾淨水體」——這是合理的疑慮。

但關鍵是：**不需要知道「哪裡 IS 油汙」，只需要知道「哪裡絕對不是油汙」**。

可以用物理條件排除：

| 條件 | 說明 |
|------|------|
| NDWI 高（> 0.3） | 確定是水體，不是陸地 |
| B02（藍）低（< 0.1） | 深水，非淺灘反射 |
| B11（SWIR1）極低（< 0.05） | 乾淨水，油膜會讓 SWIR 升高 |
| 空間紋理均勻（local std 低） | 避開雲邊、浪花、船跡 |

這些條件的交集——即便影像上有油汙——仍然能找出「幾乎確定是乾淨水體」的像素，作為 VAE 的訓練樣本。

### v9 已經實作這個洞察

v9 pipeline 的 `patch_is_scoreable` 機制 + `loose_mask` 就是 Background-Only Training 的工程實作：

| 面向 | 論文 Deep-RX | v9 DeepRX pipeline |
|------|-------------|---------------------|
| VAE 訓練資料 | 全部像素（包含飛機目標） | 通過 `patch_is_scoreable` 的**乾淨水體 patch** |
| 對 contamination 的抵抗力 | 弱（若目標佔比高則失效） | 強（目標被排除在訓練之外）|
| 混雜場景適應性 | 低（純高光譜機載，場景乾淨）| 高（真實衛星影像含雲/陸地/船）|
| 訓練像素的品質控制 | 無 | NDWI + B02 + B11 + 紋理多條件篩選 |

**這是 v9 在概念上比論文 Deep-RX 更進一步的地方**——不是因為模型更複雜，而是訓練資料的篩選策略更嚴謹。

### Background-Only Training 的風險

| 風險 | 說明 | 對應措施 |
|------|------|---------|
| 篩太鬆 | 油汙污染訓練集 → VAE 學到的「正常」包含油汙 → 偵測失效 | 多條件交集篩選（NDWI + B02 + B11 + std）|
| 篩太嚴 | 訓練像素不足 → VAE 學不好 → latent space 結構差 | 降低門檻，但需 sensitivity 分析 |
| 乾淨水體 VAE 對非水體打高分 | VAE 學到的是「乾淨水體的分布」，對陸地/雲/油汙都會輸出高 Mahalanobis 距離 | 需要 NDOI z-score 等二次篩選 |

### 為什麼 v9 還需要 NDOI z-score 後過濾

Background-Only Training 的 VAE 會對「所有非乾淨水體」都給出高異常分數——包括陸地、雲邊、船隻、以及油汙。這就是「高召回、低精確」的問題。

v9 用 NDOI（Normalized Difference Oil Index）z-score 做二次驗證：即便 Mahalanobis 分數高，若該像素的光譜特徵不符合油汙的物理特性（SWIR 吸收、藍光抑制），就不列入最終偵測結果。

**邏輯**：VAE+RX 負責找「不像乾淨水體的」（高召回），NDOI z-score 負責確認「是不是油汙」（提高精確度）。

---

## 8. 一句話總結

> 使用者從「VAE 是什麼」問到「為什麼 VAE+RX 要串起來」，再問到「訓練和測試用同一張影像算不算作弊」——這個問題沒有簡單答案：VAE 沒看過標籤所以不算「作弊」，但論文的評估設計讓 AUC 0.991 難以泛化到新場景；而使用者自己推導出的「只用乾淨水體訓練」洞察，正好是 v9 pipeline 比論文更嚴謹的關鍵設計。

---

## 相關頁面

- [[Song_DeepRX_Hyperspectral_Anomaly]] — 論文摘要（方法、三個資料集 AUC、與 RX 系列比較）
- [[iforest_架構與運作機制]] — iForest 深入理解（同樣含「訓測同源」爭議的分析）
- [[unsupervised_oil_detection]] — Unsupervised 油汙偵測方法總覽
- [[20260614_DeepRX_EM_v3_全場景實驗總結]] — v9 DeepRX+EM 全場景實驗結果（Recall=0.371，16.4× vs iForest）
- [[柯弈仲2008_光學衛星影像海洋異常物偵測]] — 早期光學衛星海洋異常偵測文獻
