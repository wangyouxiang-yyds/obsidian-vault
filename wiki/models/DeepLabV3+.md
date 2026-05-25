---
type: model
tags: [DeepLabV3+, ResNet50, segmentation, baseline, FocalLoss]
related: [OSDMamba.md, ../pipeline/project_overview.md]
---

# DeepLabV3+（工程紀錄）

## 一句話定義

以 ResNet50 為骨幹的語義分割模型，作為本專案油汙偵測的 Baseline，輸入層改為 8 通道以支援 MS6 Sen2Like 多光譜影像。

---

## 架構設定

| 項目 | 設定 |
|------|------|
| 骨幹 | ResNet50（torchvision pretrained，第一層 conv 改為 8-ch） |
| 輸入通道 | **8**（B01/02/03/04/08/8A/11/12，443~2202 nm，不含紅邊 B05/06/07） |
| 輸出類別 | 2（Oil, Background） |
| 損失函數 | FocalLoss（gamma=2.0） |
| EMA | decay=0（直接用當下模型權重驗證） |

> **EMA=0 的原因**：DeepLabV3+ 訓練較穩定，實驗中發現開 EMA 對指標無明顯提升，故關閉以簡化流程。OSDMamba 則需要 EMA 穩定訓練（decay=0.9997）。

---

## 訓練超參數

| 項目 | 設定 |
|------|------|
| 最佳化器 | AdamW（weight_decay=1e-4） |
| 學習率策略 | CosineAnnealingLR（lr0=0.0001, eta_min=lr0×0.01） |
| 梯度裁剪 | clip_grad_norm_（max_norm=1.0） |
| Epochs / Patience | 300 / 50 |
| Batch Size | 24 |
| Workers | 16 |
| Class Weights | **[13.0, 1.0]**（Oil : Background，inverse frequency；Oil:BG 像素比 ≈ 1:13.8） |

---

## 資料增強（Albumentations）

- RandomResizedCrop（scale=0.75~1.0, p=0.8）
- HorizontalFlip / VerticalFlip / RandomRotate90
- GaussNoise / RandomBrightnessContrast
- Normalize（動態計算訓練集 Mean/Std）

---

## 推論設定（Sliding Window 重組）

| 項目 | 設定 |
|------|------|
| Patch Size | 256 |
| Stride | 192 |
| Infer Batch Size | 32（RTX 5090） |
| TTA | Original + HFlip + VFlip（3 次 forward 合併） |

---

## 評估指標

- 核心：mIoU, Overall Accuracy (OA), F1-macro
- 細節：IoU(Oil), Precision(Oil), Recall(Oil), F1(Oil)

---

## 相關腳本

| 腳本 | 說明 |
|------|------|
| `main/deeplab_adapter.py` | 模型定義與訓練邏輯 |
| `main/experiments_CV.yaml` | 5-fold CV 實驗設定 |
| `main/reconstruct_module.py` | Sliding Window 重組推論 |
| `result-seg/CV/` | 各 fold 輸出結果 |
| `result_excel/result_CV_log.xlsx` | 指標彙整 |

---

## 已知問題與對策

| 問題 | 對策 |
|------|------|
| Oil 類別召回率偏低 | Class weight=30.0 + FocalLoss(gamma=2.0) |
| NAS 讀取慢 | 訓練資料搬至本地 SSD（詳見 [20260501_大圖重組效能優化.md](../experiments/20260501_大圖重組效能優化.md)） |

---

## 相關頁面

- [OSDMamba.md](OSDMamba.md) — 對比模型
- [project_overview.md](../pipeline/project_overview.md) — 整體訓練流程
- [20260429_OSDMamba整合至0420.md](../experiments/20260429_OSDMamba整合至0420.md)
