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
| 學習率策略 | CosineAnnealingLR（lr0=**5e-5**, eta_min=lr0×0.01） |
| 梯度裁剪 | clip_grad_norm_（max_norm=1.0） |
| Epochs / Patience | 300 / 50 |
| Batch Size | **16**（之前是 24，已調整）|
| Workers | **8**（之前是 16）|
| AMP（混合精度）| **啟用**（use_amp=True，加速 + 省記憶體）|
| Gradient Accumulation | accumulation_steps=1 |
| Class Weights | [13.0, 1.0]（Oil : Background，Oil:BG 像素比 ≈ 1:13.8）|

---

## Mean/Std（VRT 模式）

- mean: [0.190978, 0.184417, 0.177819, 0.171368, 0.171265, 0.169873, 0.151570, 0.145186]
- std:  [0.136939, 0.130860, 0.129367, 0.131349, 0.136126, 0.131062, 0.072673, 0.062876]
- 計算方式：從訓練 fold 抽樣 500 個 patch 統計，寫入 args.yaml 後續直接讀取，避免重算

---

## ★ Checkpoint / Resume 機制（2026-06-15 新增）

為了應對突發狀況（電腦當機、訓練被踢掉），訓練流程加入了完整的 checkpoint 機制。

### 存了什麼

| 檔案 | 時機 | 內容 |
|------|------|------|
| `best.pt` | 每次 val_miou 創新高 | 只存 EMA model state_dict |
| `last.pt` | **每個 epoch 結束** | model / optimizer / scheduler / scaler / EMA / epoch / best_val_miou / patience / RNG 狀態 |

### Resume 流程

```python
# train() 開頭自動偵測 last.pt
if last_checkpoint_path.exists():
    ckpt = torch.load(last_checkpoint_path)
    self.model.load_state_dict(ckpt['model_state'])
    optimizer.load_state_dict(ckpt['optimizer_state'])
    scheduler.load_state_dict(ckpt['scheduler_state'])
    scaler.load_state_dict(ckpt['scaler_state'])
    model_ema.ema.load_state_dict(ckpt['ema_state'])
    best_val_miou    = ckpt['best_val_miou']
    patience_counter = ckpt['patience_counter']
    start_epoch      = ckpt['epoch']
    # RNG 恢復（資料順序可重現）
    torch.set_rng_state(ckpt['rng_state']['torch'])
    torch.cuda.set_rng_state_all(ckpt['rng_state']['cuda'])
    np.random.set_state(ckpt['rng_state']['numpy'])
    random.setstate(ckpt['rng_state']['random'])

for epoch in range(start_epoch, epochs):  # 從上次 epoch 繼續
    ...
```

### 安全機制

- **原子寫入**：先存 `last.tmp` 再 rename 成 `last.pt`，避免存到一半當機 corrupt
- **載入失敗自動回退**：除錯後從頭跑，不會卡死
- **Log append 模式**：resume 時不會覆蓋 `results.csv`
- **RNG 狀態完整保留**：torch / cuda / numpy / random 四種 RNG state 全部保存

### 使用方式

```bash
# 之前怎麼跑現在還是怎麼跑，會自動 resume：
conda run -n mamba_env python main_runner.py \
  --config experiments_3fold_random_noGulf.yaml

# 看到這行就表示成功 resume：
#   [Resume] ✅ 從 epoch N 繼續 | best_val_miou=... | patience=...

# 想從頭重跑：刪掉 last.pt 即可
rm result-seg/.../weights/last.pt
```

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
| `main/deeplab_adapter.py` | 模型定義與訓練邏輯（含 checkpoint）|
| `main/main_runner.py` | 實驗排程入口（--config 指定 yaml）|
| `main/experiments_CV.yaml` | 5-fold CV 實驗設定（舊版）|
| `main/experiments_3fold_all220.yaml` | 3-fold 全 220 場景 |
| `main/experiments_3fold_random.yaml` | 3-fold 純隨機（439 場景）|
| `main/experiments_3fold_random_noGulf.yaml` | **3-fold 排除 NOAA Gulf of Mexico（191 場景）** |
| `main/experiments_3fold_event.yaml` | 3-fold 依事件切分 |
| `main/experiments_3fold_temporal.yaml` | 3-fold 依時間切分 |
| `main/reconstruct_module.py` | Sliding Window 重組推論 |
| `result-seg/CV*/` | 各 fold 輸出結果 |
| `result_excel/result_CV_*.xlsx` | 指標彙整 |

---

## 已知問題與對策

| 問題 | 對策 |
|------|------|
| Oil 類別召回率偏低 | Class weight=13.0 + FocalLoss(gamma=2.0) |
| NAS 讀取慢 | 訓練資料搬至本地 SSD（詳見 `[[20260501_大圖重組效能優化]]`）|
| **訓練中當機 / 強制中斷** | **`last.pt` checkpoint（每 epoch 自動存，下次自動 resume）** |
| 資料不平衡（Gulf Mexico 佔比 56%）| 提供 `experiments_3fold_random_noGulf.yaml` 排除 NOAA Gulf Mexico（191 場景）|

---

## 相關頁面

- [OSDMamba.md](OSDMamba.md) — 對比模型
- [project_overview.md](../pipeline/project_overview.md) — 整體訓練流程
- [20260429_OSDMamba整合至0420.md](../experiments/20260429_OSDMamba整合至0420.md)
- [[20260614_DeepRX_EM_v3_全場景實驗總結]] — 無監督對照組
- [[20260613_iForest_LocalContrast_B4B8B11_全場景實驗總結]] — 無監督對照組
