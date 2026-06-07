---
type: paper
title: "Deep-RX for Hyperspectral Anomaly Detection"
authors: Yingjie Song, Shuaikai Shi, Jie Chen
year: 2023
tags: [anomaly-detection, hyperspectral, VAE, RX, deep-learning, unsupervised]
---

# Song et al. — Deep-RX for Hyperspectral Anomaly Detection

> 原始論文：`raw/papers/Deep-RX_for_Hyperspectral_Anomaly_Detection.pdf`
> 單位：Northwestern Polytechnical University（西北工業大學），Shenzhen

---

## 核心貢獻

將傳統 **RX 偵測器**（Reed-Xiaoli）與 **VAE（Variational Autoencoder）** 結合，解決 RX 在高光譜資料上的兩個根本問題：
1. RX 假設背景服從高斯分布 → 高光譜實際分布非高斯
2. RX 在高維空間（224 波段）估計共變異矩陣不穩定

**解法**：先用 VAE Encoder 將高光譜壓縮到低維高斯潛在空間，再在潛在空間做 RX 偵測。

---

## 方法架構

```
高光譜影像 X (B×N)
    ↓ VAE Encoder
潛在空間 z ~ N(μ, σ²)    ← 強制高斯分布（VAE 的 KL 約束）
    ↓ RX Detector
異常分數 r = (z-μ̂)ᵀ Ĉ⁻¹ (z-μ̂)
    ↓
偵測結果
```

**VAE 的角色**：
- Encoder 學習光譜背景的壓縮表示
- KL divergence 約束讓潛在空間服從高斯 → RX 假設成立
- Decoder 重建損失保留光譜資訊

**損失函數**：
```
L = λ₁ · 重建損失 + λ₂ · KL 散度
```

---

## 實驗結果（AUC）

| 資料集 | Deep-RX | KRX | GRX | LRX | CRD |
|--------|---------|-----|-----|-----|-----|
| San Diego I | **0.991** | 0.986 | 0.952 | 0.953 | 0.956 |
| San Diego II | **0.985** | 0.972 | 0.905 | 0.825 | 0.923 |
| HyperUrban | **0.979** | 0.940 | 0.840 | 0.832 | 0.949 |

> 三個資料集均為 AVIRIS 機載高光譜影像，異常目標為飛機（airport scenes）。

---

## 與 RX 系列方法比較

| 方法 | 核心思想 | 限制 |
|------|---------|------|
| GRX（Global RX） | 全圖估計背景統計 | 高維不穩定 |
| LRX（Local RX） | 局部視窗估計 | 視窗大小敏感 |
| KRX（Kernel RX） | 核函數非線性映射 | 核參數難選 |
| **Deep-RX** | VAE 壓縮 + RX | 需訓練 VAE |

---

## 與本專案的關聯

| 面向 | Deep-RX | 本專案（iForest） |
|------|---------|-----------------|
| 輸入 | 高光譜 224 波段 | 多光譜 8 波段 |
| 背景建模 | VAE 潛在空間 | iForest 隔離樹 |
| 非線性能力 | ✓（深度學習） | △（ensemble trees） |
| 訓練成本 | 高（需 GPU） | 低 |
| 海洋油汙應用 | 未測試 | 本專案主軸 |

**啟發**：若 iForest 在 8 波段的 AUC 太低，可考慮改用 Deep-RX 架構（VAE Encoder + RX）作為升級方向。

---

## 相關頁面
- [[unsupervised_oil_detection]]
- [[IsolationForest_Liu2008]]
- [[Duan2022_HOSD_IsolationForest]]
