---
type: paper
title: "Isolation Forest"
authors: Fei Tony Liu, Kai Ming Ting, Zhi-Hua Zhou
year: 2008
tags: [isolation-forest, anomaly-detection, unsupervised, iForest]
---

# Isolation Forest（Liu et al., 2008/2012）

> 原始論文：`raw/papers/IForest.pdf`
> 發表：ICDM 2008（會議版）；IEEE TKDE 2012（期刊擴充版）

---

## 核心貢獻

**逆向思考異常偵測**：傳統方法先建立「正常」的輪廓，再找例外。iForest 改為直接「孤立」異常點。

關鍵洞見：
- 異常點數量少（few）且特徵差異大（different）
- 因此異常點比正常點**更容易被隨機切割孤立**
- 孤立所需的分割次數越少 → 路徑越短 → 越像異常

---

## 演算法原理

### 建樹（Training）
1. 從資料中隨機取樣子集（subsample size ψ，預設 256）
2. 隨機選一個特徵維度
3. 在該維度的 min～max 之間隨機選一個切割值
4. 遞迴分裂，直到每個節點只剩一個點或達到樹高上限
5. 重複建 t 棵 iTree（預設 100 棵）

### 預測（Scoring）
對每個測試點 x，計算它在所有 iTree 中的平均路徑長度 E(h(x))。

**Anomaly Score 公式：**

```
s(x, n) = 2^{ -E(h(x)) / c(n) }
```

其中：
- `h(x)` = 點 x 在單棵 iTree 中從根節點到葉節點的路徑長度
- `c(n)` = n 個樣本的 BST 平均不成功搜尋路徑長（正規化常數）
  - `c(n) = 2H(n-1) - 2(n-1)/n`
  - `H(i) = ln(i) + 0.5772...`（Euler-Mascheroni 常數）

**分數解讀：**
| s 值 | 意義 |
|------|------|
| s → 1 | 很可能是異常 |
| s → 0.5 | 無法判斷 |
| s → 0 | 正常點 |

### sklearn 對應參數
```python
IsolationForest(
    n_estimators=100,      # iTree 數量
    max_samples=256,       # 每棵樹的子樣本數 ψ
    contamination=0.02,    # 預期異常比例（決定二值化閾值）
    random_state=42,
    n_jobs=-1
)
```

---

## 演算法特性

| 特性 | 說明 |
|------|------|
| 時間複雜度 | O(n)，線性 |
| 記憶體 | 極低，只需儲存 iTree 結構 |
| 不需距離/密度計算 | 比 LOF、KNN 快很多 |
| 高維資料 | 表現良好（隨機選特徵，不受維度詛咒影響）|
| 無監督 | 訓練時完全不需要 label |
| contamination | 控制多嚴格，可調整 |

---

## 實驗結果（原論文）

在多個 benchmark 資料集與 LOF、ORCA、Random Forest 比較：

| 資料集 | iForest AUC | LOF AUC |
|--------|-------------|---------|
| HTTP   | 1.000       | 0.997   |
| SMTP   | 0.904       | 0.716   |
| KDDCup | 0.993       | 0.965   |
| Forest | 0.862       | 0.568   |
| Shuttle| 0.997       | 0.529   |

→ 在大多數資料集上優於 LOF，且速度快上許多倍。

---

## 重要細節：subsample size ψ

論文發現 **ψ=256** 在大多數情況下已足夠，原因是：
- 異常孤立在少量樣本下就能完成
- 更大的 ψ 不顯著提升效果，反而增加計算量
- 這是 iForest 速度快的關鍵之一

---

## 在本專案的應用

### 油汙偵測場景
```python
# 每個像素的 8 個 S2 波段當作特徵向量
# 海水是「正常」背景，油汙是「異常」

clf = IsolationForest(n_estimators=100, contamination=0.02)
clf.fit(sea_pixels_scaled)           # 用隨機取樣的海面像素訓練
scores = clf.decision_function(all_pixels)  # 越負越異常
```

### 實測結果（5 個 NOAA S2 場景）

| Scene | AUC |
|-------|-----|
| 20220619_S2_15RXM | 0.9458 |
| 20220629_S2_15RXM | 0.7877 |
| 20221103_S2_16RCT | 0.7033 |
| 20220417_S2_16RBS | 0.6661 |
| 20221111_S2_15RXM | 0.5134 |

**觀察**：AUC 差異大，顯示某些場景的油汙光譜與海水差異不夠大（薄油膜、雲霧干擾），8 個波段在這種情況下資訊不足。

### 與高光譜比較
| 資料 | 波段數 | AUC（GM03） |
|------|--------|------------|
| HOSD 高光譜 | 224 | **0.9899** |
| S2 多光譜 | 8 | 0.31 ~ 0.95 |

---

## 相關頁面
- [[unsupervised_oil_detection]]
- [[Duan2022_HOSD_IsolationForest]]
- [[20260603_Unsupervised油汙偵測初探]]
