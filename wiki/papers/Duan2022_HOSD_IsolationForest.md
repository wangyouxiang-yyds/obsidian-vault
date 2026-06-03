---
type: paper
title: "Hyperspectral Oil Spill Detection with Unsupervised Learning (HOSD Dataset)"
authors: Duan et al.
year: 2022
tags: [unsupervised, isolation-forest, hyperspectral, oil-spill, HOSD]
---

# Duan et al. 2022 — HOSD + Isolation Forest

## 核心貢獻

首次提出以純 Unsupervised 方式對高光譜影像做油汙偵測的完整 pipeline，並公開 HOSD 資料集。

## 關鍵方法

```
KPCA 降維（高光譜 224 波段 → 低維特徵空間）
    ↓
Isolation Forest（估計每個像素的異常分數）
    ↓
產生 pseudo label
    ↓
SVM 分類
    ↓
Extended Random Walker（優化邊緣）
```

## 資料集（HOSD）

- **來源**：2010 年墨西哥灣 Deepwater Horizon 漏油事件
- **感測器**：機載高光譜儀（AVIRIS 或類似）
- **格式**：18 個 .mat 檔（GM01–GM18）
- **影像大小**：1386 × 690 像素，**224 個波段**，int16
- **Mask**：0 = 海水，1 = 油汙（二值）
- **開源連結**：arXiv（搜尋 `HOSD dataset oil spill isolation forest Duan 2022`）

## 本專案實測結果（GM03，僅 iForest 步驟）

| 指標 | 數值 |
|------|------|
| F1（油汙） | 0.78 |
| AUC | **0.9899** |
| 油汙比例 | 4.12% |

> 只跑 iForest 單步（無 KPCA / SVM / RW）就達到 F1=0.78，AUC=0.99，說明 iForest 是整個 pipeline 的核心。

## 與本專案的關聯

| 論文設定 | 本專案設定 | 差異影響 |
|---------|-----------|---------|
| 高光譜，224 波段 | 多光譜，8 波段 | 光譜資訊量大幅減少，預期 AUC 下降 |
| 機載影像，高解析度 | Sentinel-2，10m | 單個油汙像素面積差異大 |
| binary mask（0/1） | mask 值 0/64/128/192 | 需轉換對照 |
| 僅 Sentinel-2 | Sentinel-2 + Landsat（sen2like harmonized）| 波段定義需確認 |

## 重要數據

- iForest 單步在 8 波段 Sentinel-2 上的實測 AUC：0.31 ~ 0.95（視場景而定）
- 高光譜 224 波段基準：AUC = 0.99

## 相關頁面
- [[unsupervised_oil_detection]]
- [[20260603_Unsupervised油汙偵測初探]]
