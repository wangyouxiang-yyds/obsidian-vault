---
type: model
tags: [OSDMamba, SwinUMambaD, Mamba, SSM, segmentation, RTX5090]
related: [DeepLabV3+.md, ../papers/OSDMamba_摘要.md, ../pipeline/project_overview.md]
---

# OSDMamba（工程紀錄）

## 一句話定義

基於 Mamba SSM 的語義分割模型（SwinUMambaD），透過 SS2D 選擇性掃描擴大感受野，搭配非對稱解碼器處理油汙類別不平衡問題。

> 學術原理見 [OSDMamba 論文摘要](../papers/OSDMamba_摘要.md)；本頁專注工程配置與踩坑紀錄。

---

## 架構設定

| 項目 | 設定 |
|------|------|
| 架構類型 | SwinUMambaD |
| 模型定義路徑 | `/mnt/backup/alanyh/OSDMamba/OSDMamba.py` |
| 輸入通道 | 11（443~2202 nm 多光譜） |
| 輸出類別 | 2（Oil, Background） |
| depths | [2, 2, 9, 2] |
| dims | 96 |
| d_state | 16 |
| drop_path_rate | 0.2 |
| features_per_stage | [96, 192, 384, 768] |

---

## 訓練超參數

| 項目 | 設定 |
|------|------|
| 最佳化器 | AdamW（weight_decay=1e-4） |
| 學習率策略 | CosineAnnealingLR（lr0=0.0001, eta_min=lr0×0.01） |
| 梯度裁剪 | clip_grad_norm_（max_norm=1.0） |
| Epochs / Patience | 300 / 50 |
| Batch Size | 16（OOM 風險，視顯存調整） |
| Workers | 16 |
| EMA decay | 0.9997（每 batch 更新） |
| Class Weights | [30.0, 1.0]（Oil : Background） |
| AMP | 啟用（混合精度，降低顯存需求） |

> **EMA=0.9997 的原因**：OSDMamba 訓練震盪較大，EMA 可平滑模型權重提升驗證指標。DeepLabV3+ 訓練較穩定故不需要。

---

## 損失函數

**Dice Loss + FocalLoss 聯合損失**

- Dice Loss：優化邊界重疊，直接對 mIoU 友好
- FocalLoss（gamma=2.0）：壓低易分類背景的梯度貢獻，聚焦稀少的油汙像素
- 聯合：`loss = dice_loss + focal_loss`

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
| Infer Batch Size | 32（RTX 5090 專屬） |
| TTA | Original + HFlip + VFlip（3 次 forward 合併） |
| prob_map 累加 | GPU 側累加，整圖完成後才搬回 CPU |

---

## 相關腳本

| 腳本 | 說明 |
|------|------|
| `main/osdmamba_adapter.py` | 模型定義與訓練邏輯（含 AMP、EMA、向量化 Loss） |
| `main/experiments_osdmamba_CV.yaml` | 5-fold CV 實驗設定 |
| `main/reconstruct_module.py` | Sliding Window 重組推論 |
| `result-seg/CV_OSDMamba/` | 各 fold 輸出結果 |
| `result_excel/result_OSDMamba_CV_log.xlsx` | 指標彙整 |

---

## 環境需求（RTX 5090 / sm_120）

`mamba-ssm` 與 `causal-conv1d` 需從源碼編譯，不支援預編譯 wheel：

```bash
# 安裝步驟（需 CUDA 12.x + sm_120 支援）
pip install -e /path/to/mamba-ssm --no-build-isolation
pip install -e /path/to/causal-conv1d --no-build-isolation
```

詳細步驟見 `AI_help/環境建置.txt` 與 [Docker_環境設定.md](../environment/Docker_環境設定.md)。

---

## 已知問題與對策

| 問題 | 原因 | 對策 |
|------|------|------|
| OOM（顯存不足） | 模型比 DeepLabV3+ 大，RTX 5090 32GB 在 batch=24 時仍可能 OOM | 降低 batch_size + 啟用 AMP + gradient accumulation |
| 每 Epoch 過慢（~180s） | CPU-GPU 同步（loss.item()）+ NAS I/O + EMA 每 batch 更新 | 延遲同步 + 本地 SSD + 向量化 Dice Loss（詳見 [20260430_訓練效能優化.md](../experiments/20260430_訓練效能優化.md)）|
| selective_scan_fn 未啟動 | sm_120 CUDA 編譯失敗 | 從源碼重新編譯 mamba-ssm |

---

## 與 DeepLabV3+ 的差異對照

| 項目 | DeepLabV3+ | OSDMamba |
|------|-----------|----------|
| 骨幹 | ResNet50（CNN） | SwinUMambaD（SSM） |
| 損失函數 | FocalLoss only | Dice + FocalLoss |
| EMA | 關閉（decay=0） | decay=0.9997 |
| 顯存需求 | 較低 | 較高（需 AMP） |
| 訓練穩定性 | 穩定 | 需 EMA 輔助 |
| 感受野 | 有限（局部） | 全局（SS2D） |

---

## 相關頁面

- [DeepLabV3+.md](DeepLabV3+.md) — 對比 Baseline
- [OSDMamba 論文摘要](../papers/OSDMamba_摘要.md) — 學術架構說明
- [20260429_OSDMamba整合至0420.md](../experiments/20260429_OSDMamba整合至0420.md) — 移植工作紀錄
- [20260430_訓練效能優化.md](../experiments/20260430_訓練效能優化.md) — 訓練速度優化
