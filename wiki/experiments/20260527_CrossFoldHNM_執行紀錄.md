---
date: 2026-05-27
type: experiment
stage: HNM
status: running
tags: [HNM, Cross-Fold, FP-mining, spectral-filter, DeepLabV3+]
---

# Cross-Fold HNM — Step 2 & 3 執行紀錄

## 目標

利用 5-fold MS6 GB1.0 訓練完成的模型，對全部 220 個場景進行 False Positive 挖掘（Cross-Fold 設計），產出 Hard Negative candidates，補入 3-fold all-220 GB1.0 訓練集，最終得到 `patch_level_GB1.0_cfhnm`（1:1 + 0.5 HN = 1:1.5）。

## 設定

- **腳本位置**：`preprocess/Cross_Fold_HNM/`
- **挖掘模型**：5-fold fold 3 best.pt（val mIoU=50.3%，最高分 fold）
- **模型路徑**：`result-seg/CV/20260524-145713_MS6_CV_deeplabv3+_fold3/weights/best.pt`
- **挖掘對象**：全 220 個場景（VRT 動態讀取）
- **Patch 參數**：PATCH_SIZE=256, STRIDE=192
- **Patch-level 條件**：
  - mask==0（油）比例 = 0（避免污染）
  - mask==1/2（純 bg）比例 ≥ 50%
  - mask==255（unannotated）比例 < 30%
  - nodata 比例 < 30%
  - pred==oil ∩ mask==bg ≥ 1 pixel（有 FP 才算 HN candidate）
- **輸出目錄**：`HNM/step2_output/`

## Step 2 卡死問題與修正

### 根本原因

原始實作嘗試一次讀取整個場景的背景像素 bounding box → 最大高達 **3.86 GB** 的 NAS 單次讀取，對 NAS 而言實際會掛住 30–60 秒以上，無任何進度輸出，看起來像「卡死」。

### 修正方案

改為三階段流程：

1. **一次讀全場景 mask**（本機 NAS，~1.5 MB）→ Python loop 篩選 eligible patches
2. **只對 eligible patches 讀 VRT**（8 worker 並行 `ThreadPoolExecutor`）
3. **Prefetch pipeline**：reader thread（背景 NAS 讀取）與 GPU inference（主 thread）重疊執行

```
mask 讀取 → eligible 篩選 → [prefetch queue] → GPU inference
                              reader thread ↗        ↑ main thread
```

### 關鍵優化細節

- `as_strided` 向量化被測試但更慢（非連續記憶體存取，2× slower），仍保留 Python loop
- `queue.Queue(maxsize=3)`：預讀 3 個 batch，隱藏 NAS latency
- 每個 VRT patch = 8 次 NAS round trip（8 個 band TIF），latency dominated
- BATCH=64，每個 batch 讀完才推 queue（避免 race condition）

### 速度對比

| 方案 | 速度/scene |
|------|-----------|
| 原版（整 ROI 讀取） | ~290s（常卡死） |
| 修正版（mask-first + prefetch） | 平均 ~48s |
| **全 220 scenes 預計** | **~2.9 小時** |

## Step 2 結果（2026-05-28 完成）

| 統計項目 | 數值 |
|---------|------|
| 處理場景數 | 220 / 220 |
| FP patch candidates | **45,848** |
| n_eligible=0 的場景 | 4（純油或 bg 不足，正常） |
| 輸出檔案 | `cfhnm_mined_all220.txt`、`cfhnm_mined_summary.csv` |

4 個 eligible=0 場景（mask 條件無法通過）：
- `20201006_S2_19TDE`
- `20201016_S2_19TDE`
- `20201115_S2_19TDE`
- `20210926_S2_19TDE`

## Step 3 Bug 修正

`step3_spectral_safety_filter.py` 有 6 處引用了不存在的欄位 `cand_df["fold"]`，實際欄位名稱為 `source_model_fold`（step2 輸出格式）。全部修正完畢。

涉及位置：
- log 輸出列印（1 處）
- per-fold histogram 迴圈（2 處）
- d_min vs confidence scatter plot（2 處）
- top40 grid 標題與 summary 文字（各 1 處）

## Step 3 設定

- **輸入**：`HNM/step2_output/cfhnm_mined_all220.txt`
- **Oil reference**：`3fold_all220/patch_level_GB1.0/list_oil.txt`（3,494 個 oil patches）
- **Spectral feature（8-dim）**：MNDWI, OSI, B2_mean, B4_mean, B8_mean, B11_mean, B2_std, B11_std
- **d_min**：每個 candidate 到同場景 oil patches 的 z-scored L2 最短距離
- **輸出目錄**：`HNM/step3_output/`
- **預計耗時**：30–60 分鐘（CPU bound）

## 下一步

Step 3 跑完後：
1. 查看 `dmin_distribution.png` 與 `decision_summary.txt`
2. 決定 d_min cutoff（建議觀察 p10/p15 對應的感觀品質）
3. 執行 `step4_within_fold_assemble.py` 組裝 `patch_level_GB1.0_cfhnm`
4. 訓練 `experiments_3fold_all220_cfhnm.yaml`
