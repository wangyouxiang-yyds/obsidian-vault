---
type: pipeline
tags: [overview, OSDMamba, DeepLabV3+, multiband, segmentation]
---

# OIL_PROJECT_MutiBand_0420 專案概述

## 專案目標

針對 **11 波段 (Multi-spectral)** 衛星影像，利用深度學習模型進行油汙偵測與語意分割。目前對比實驗兩種架構：

- **DeepLabV3+** (ResNet50 Backbone)
- **OSDMamba** (SwinUMambaD，基於 Mamba SSM)

---

## 資料前處理

### 波段堆疊

- **腳本**: `preprocess/stack_multiband_tif.py`
- **波段順序 (Wavelengths)**: `443 → 492 → 560 → 665 → 704 → 740 → 783 → 833 → 865 → 1614 → 2202` (nm)
- **儲存格式**: Float32, BigTIFF (突破 4GB 限制), ZSTD 壓縮

### 切圖與取樣

- **腳本**: `preprocess/patch_from_stacked_tif_0421.py`
- **Patch Size**: 256×256 / **Overlap**: 64
- **取樣策略 (Balanced Sampling)**: Oil to BG Ratio = 1.5
- **標籤映射**:
  - Oil (128) → Class 0
  - Others/Ship (192) → Class 1
  - Background (64) → Class 1
  - 無標註 (0) → Class 1

---

## 模型架構

### DeepLabV3+ (`main/deeplab_adapter.py`)

- 骨幹網路：ResNet50，輸入層改為 11 通道卷積
- 損失函數：FocalLoss (gamma=2.0)
- EMA decay=0.0（驗證直接使用當下模型權重）

### OSDMamba (`main/osdmamba_adapter.py`)

- 架構：SwinUMambaD（`/mnt/backup/alanyh/OSDMamba/OSDMamba.py`）
- depths=[2, 2, 9, 2], dims=96, d_state=16, features_per_stage=[96, 192, 384, 768]
- 損失函數：Dice Loss + FocalLoss (gamma=2.0) 聯合
- EMA decay=0.9997

> **環境備注 (RTX 5090 / sm_120)**：mamba-ssm 與 causal-conv1d 需從源碼編譯，詳見 `AI_help/環境建置.txt`。

---

## 訓練機制與超參數

| 項目 | 設定 |
|------|------|
| 最佳化器 | AdamW (weight_decay=1e-4) |
| 學習率策略 | CosineAnnealingLR (lr0=0.0001, eta_min=lr0×0.01) |
| 梯度裁剪 | clip_grad_norm_ (max_norm=1.0) |
| Epochs / Patience | 300 / 50 |
| Batch Size | 24 (accumulation_steps=1) |
| Workers | 16 |

### 資料增強 (Albumentations)

- RandomResizedCrop (scale=0.75~1.0, p=0.8)
- HorizontalFlip / VerticalFlip / RandomRotate90
- GaussNoise / RandomBrightnessContrast
- Normalize（動態計算訓練集 Mean/Std）

---

## OSDMamba CV 關鍵設定 (`experiments_osdmamba_CV.yaml`)

### 類別平衡

- **Class Weights**: `[30.0, 1.0]`（Oil : Background）
  - 理由：油汙像素約佔全圖 1.76%，給予 30 倍權重使油汙對 Loss 貢獻達約 54%

### 重組推論設定

| 項目 | 設定 |
|------|------|
| 模式 | original_data_root（直接讀取本地 Stacked TIF）|
| Infer Batch Size | 32（RTX 5090）|
| TTA | Original + HFlip + VFlip |
| Stride / Patch Size | 192 / 256 |

---

## 評估方式

### 交叉驗證

- **模式**：5-fold CV
- **自動化**：`main_runner.py` 自動執行 5 次訓練與測試，計算各 fold 平均指標

### 重組測試

- **腳本**：`main/reconstruct_module.py`
- **方法**：Sliding Window（Stride: 192 / Patch: 256）
- **TTA**：三次 forward pass 結果合併

### 評估指標

- 核心：mIoU, Accuracy (OA), F1-macro
- 細節：IoU(Oil), Precision, Recall, F1

---

## I/O 效能優化（本地資料策略 v2.0）

- **本地路徑**：`/home/alanyh/oil_dataset/new/full_band/`（本地 SSD）
- 訓練 Patches 全數搬移至本地，消除 NAS 隨機讀取延遲
- 大圖改用本地 Stacked TIF 直接讀取（取代 VRT 模式），速度提升 **10 倍以上**

---

## 專案結構

```
OIL_PROJECT_MutiBand_0420/
├── main/
│   ├── main_runner.py
│   ├── training_module.py
│   ├── deeplab_adapter.py
│   ├── osdmamba_adapter.py
│   ├── reconstruct_module.py
│   ├── evaluation_module.py
│   ├── experiments_CV.yaml
│   └── experiments_osdmamba_CV.yaml
├── preprocess/
├── result-seg/
├── result_excel/
└── AI_help/
```

---

## 重要路徑

| 項目 | 路徑 |
|------|------|
| 專案根目錄 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_0420/` |
| OSDMamba 模型定義 | `/mnt/backup/alanyh/OSDMamba/OSDMamba.py` |
| Patch 資料集（本地）| `/home/alanyh/oil_dataset/new/full_band/processed/patched/` |
| 5-fold Split TXT | `/mnt/backup/oil_dataset/new/full_band/data_split/5_fold/patch_level_GB1.5/` |
| DeepLabV3+ 結果 | `result-seg/CV/` + `result_excel/result_CV_log.xlsx` |
| OSDMamba 結果 | `result-seg/CV_OSDMamba/` + `result_excel/result_OSDMamba_CV_log.xlsx` |

## 相關頁面

- [sen2like.md](sen2like.md)
- [wiki/experiments/20260429_OSDMamba整合至0420.md](../experiments/20260429_OSDMamba整合至0420.md)
- [wiki/experiments/20260430_訓練效能優化.md](../experiments/20260430_訓練效能優化.md)
- [wiki/experiments/20260501_大圖重組效能優化.md](../experiments/20260501_大圖重組效能優化.md)
