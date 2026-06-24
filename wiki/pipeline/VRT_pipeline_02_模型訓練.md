---
type: pipeline
tags: [training, deeplabv3+, resnet50, focal-loss, ema, 8band, oil-detection, fold-cv]
project: OIL_PROJECT_MutiBand_VRT
updated: 2026-06-24
---

# VRT Pipeline — 02 模型訓練（Training）

> 系列文件：[[VRT_pipeline_01_前處理]] | 本文 | [[VRT_pipeline_03_重組評估]]

---

## 概覽

以 DeepLabV3+ ResNet-50 作為主力模型，從 VRT 動態讀取 8-band 256×256 patch，以 FocalLoss + class_weights 應對油汙稀有類別問題，每個 fold 訓練完後自動觸發重組評估。

```
patch 座標 TXT（train/val/test）
    ↓ SegmentationDataset（rasterio.Window 動態讀 VRT）
    ↓ DataLoader（batch_size=16, AMP=True）
    ↓ DeepLabV3+ ResNet-50（8-ch input conv, from-scratch）
    ↓ FocalLoss + class_weights [13.0, 1.0]
    ↓ AdamW + CosineAnnealingLR
    ↓ EMA（decay=0.9997）+ best.pt 存 val Oil IoU 最高
    ↓ 訓練完成 → 觸發 reconstruction evaluation
```

---

## 模型架構

| 項目 | 設定 |
|------|------|
| 骨幹網路 | ResNet-50（torchvision 內建）|
| 輸入通道 | **8**（原 3-channel RGB conv 改為 8-channel）|
| 輸出類別 | 2（Class 0 = Oil, Class 1 = Background）|
| 初始化方式 | from-scratch（`torchvision_weights: 'None'`）|
| 已知問題 | from-scratch 導致 val loss 約為 train loss 的 2 倍（欠擬合 + 起點弱）|

> **TODO/A1**：改用 ImageNet pretrained backbone，初始 3-channel conv 複製至 8-channel（mean copy 或 partial init），預期 Oil IoU 從 39% 拉到 55-65%。詳見 `TODO/01_A1_pretrained_backbone.md`。

---

## 訓練超參數

| 項目 | 設定 |
|------|------|
| Optimizer | AdamW（lr0=5e-5, weight_decay=1e-4）|
| Scheduler | CosineAnnealingLR（eta_min=lr0×0.01）|
| 梯度裁剪 | clip_grad_norm\_（max_norm=1.0）|
| Epochs | 300 |
| Patience（early stop）| 50（依 val Oil IoU 計算）|
| Batch Size | 16 |
| num_workers | 8 |
| pin_memory | True |
| persistent_workers | True |
| AMP（混合精度）| True（use_amp=True）|
| Gradient Accumulation | 1 step |

---

## 損失函數與類別平衡

### FocalLoss 設定

```python
loss = FocalLoss(alpha=0.25, gamma=2.0)
```

### Class Weights

```yaml
class_weights: [13.0, 1.0]   # [Oil, Background]
```

**理由**：油汙像素稀少（patch 內油汙佔比 ≈ 1:13.8），給 Oil 13× 懲罰使其 Loss 貢獻約等於 Background。

> **設計取捨**：`class_weights=[13.0,1.0]` 推高 Oil Recall，但 GB1.5（多給背景 patch）旨在降低誤報（Precision），兩者方向相反。需搭配評估指標一起判斷。

---

## EMA（指數移動平均）

| 設定 | 值 |
|------|----|
| 套件 | `timm.utils.ModelEma` |
| Decay | 0.9997 |
| 用途 | 驗證與 best.pt 均使用 EMA 權重（更穩定）|

---

## 資料 Pipeline

### SegmentationDataset（動態 VRT 讀取）

- `read_images_from_vrt: true`：啟用 rasterio.Window 從 VRT 讀取對應 patch 位置
- 輸入：patch 座標 TXT（`x, y, w, h` 格式）→ 動態切 8-band 256×256
- GT：同時從 mask TIF 對應座標讀取單 channel GT Mask
- 不預先存實體 patch 檔，省磁碟 + 彈性

### 資料增強（Albumentations）

- HorizontalFlip / VerticalFlip
- RandomRotate90
- RandomResizedCrop（scale=0.75~1.0, p=0.8）
- GaussNoise
- RandomBrightnessContrast
- Normalize（mean/std 從訓練 fold 抽樣 500 patch 統計）

### 正規化參數（fold2 / 3fold_random）

```yaml
mean: [0.190978, 0.184417, 0.177819, 0.171368, 0.171265, 0.169873, 0.151570, 0.145186]
std:  [0.136939, 0.130860, 0.129367, 0.131349, 0.136126, 0.131062, 0.072673, 0.062876]
```

---

## 存檔機制（Checkpoint / Resume）

| 檔案 | 存檔時機 | 內容 |
|------|---------|------|
| `best.pt` | val Oil IoU 創新高 | EMA model state_dict（推論用）|
| `last.pt` | 每個 epoch 結束（atomic write）| model / optimizer / scheduler / scaler / EMA / epoch / best_val / patience / RNG 狀態 |

Resume 邏輯：啟動時自動偵測 `last.pt`，若存在則從上次 epoch 繼續，RNG 狀態完整恢復（torch / cuda / numpy / random）。

---

## 訓練入口

```bash
# 在 Docker container v6（mamba_env）內執行
conda run -n mamba_env python main_runner.py \
    --config experiments_3fold_random.yaml

# Resume 時相同指令，程式自動偵測 last.pt
```

**YAML 清單**：

| YAML | 用途 |
|------|------|
| `experiments_3fold_random.yaml` | 3-fold 純隨機（**主線 fold2**）|
| `experiments_3fold_random_noGulf.yaml` | 3-fold 排除 Gulf of Mexico |
| `experiments_3fold_event.yaml` | 3-fold 事件留出 |
| `experiments_3fold_temporal.yaml` | 3-fold 時序擴展 |
| `experiments_3fold_all220.yaml` | 舊 220-scene（歷史備存）|

---

## 執行環境

| 項目 | 設定 |
|------|------|
| Docker image | `ghcr.io/wangyouxiang-yyds/osdmamba_env:v6` |
| conda env | `mamba_env` |
| PyTorch 版本 | 2.12.0.dev+cu128 |
| GPU | RTX 5090（VRAM 32 GB）|
| CUDA | 12.8 |

> **v7 嘗試（CUDA 12.9 + Python 3.10 + oil_models_env）已放棄**：pip 依賴衝突（libGL、libgthread、torch/torchvision CUDA mismatch），Dockerfile 改版來不及驗證。**目前仍用 v6**。

---

## 主要腳本

| 腳本 | 說明 |
|------|------|
| `main/main_runner.py` | 總控入口，依 yaml 排程 n-fold CV |
| `main/training_module.py` | 訓練迴圈、early stop、loss 計算 |
| `main/deeplab_adapter.py` | 模型定義、Dataset 定義、checkpoint 機制 |
| `main/evaluation_module.py` | Patch-level 驗證（TTA × 3 forward）|
| `main/reconstruct_module.py` | 全場景重組（詳見 [[VRT_pipeline_03_重組評估]]）|

---

## 目前已知問題

| 問題 | 狀態 | 對策 |
|------|------|------|
| from-scratch 初始化導致 overfitting（val loss ≈ 2× train loss）| 待修 | TODO/A1：ImageNet pretrained backbone |
| Oil Recall 偏低（~55%）| 部分改善 | class_weights=13 + FocalLoss(gamma=2) |
| 資料不平衡（Gulf Mexico 佔 56% 場景）| 已有對策 | `noGulf` yaml 切分 |

---

## 相關頁面

- [[VRT_pipeline_01_前處理]] — VRT 建立、Patch 切分、Fold 策略
- [[VRT_pipeline_03_重組評估]] — Reconstruction 與交付指標
- [[DeepLabV3+]] — 模型工程紀錄（詳細超參數與 checkpoint code）
- [[deeplabv3plus_358clean_overfitting_改善方向]] — A1/A2/A3 改善路線分析
- [[20260605_資料集修正與三策略重跑]] — 三策略訓練進度紀錄
