---
date: 2026-06-07
type: experiment
stage: unsupervised
status: ready-to-run
tags: [unsupervised, isolation-forest, sentinel-2, PCA, pipeline, patch-level, full-scene]
---

# iForest Pipeline 精簡化與全場景執行準備

## 背景

承接 [[20260604_iForest全場景改進]]，本次重新設計並大幅精簡 Duan et al. 2022 的六步驟 pipeline，目標是在合理品質下跑完 220 個 NOAA 場景。

---

## Pipeline 精簡：從六步驟縮減為三步驟

### 原始論文 Pipeline（Duan 2022）

```
① 拉普拉斯波段雜訊過濾 → ② KPCA 降維 → ③ Isolation Forest
→ ④ K-Means 初始化 → ⑤ SVM 精修 → ⑥ Extended Random Walker 邊界優化
```

### 本實作 Pipeline（Steps ①②③）

```
② PCA 8→4 → ③ Isolation Forest T=800, ψ=256 → anomaly score map
```

腳本：`generate_iforest_full_pipeline.py`
輸出目錄：`/mnt/backup/alanyh/oil_unsupervised/ms6_iforest/output_full_pipeline/`

---

## 各步驟捨棄原因

| 步驟 | 決定 | 原因 |
|------|------|------|
| ① 拉普拉斯波段雜訊過濾 | 跳過 | 在海洋影像上會把 B02/B03/B04/B08（光學波段）標為雜訊並移除，只留 B01/B8A/B11/B12，方向錯誤 |
| ② KPCA → PCA | 改用 PCA | KPCA 對 580K 像素需 313 秒；PCA < 1 秒，8 波段線性降維已足夠 |
| ④ K-Means 初始化 | 移除 | 作為 SVM 前處理，SVM 已移除故連帶刪除 |
| ⑤ SVM 精修 | 移除 | GridSearchCV cv=5 每場景約 211 秒，220 場景合計 ~13 小時，不可行 |
| ⑥ Extended Random Walker | 移除 | 在 1.2M 節點稀疏 CG 上執行，全場景尺度不可行 |

---

## 參數設定

```python
PATCH_SIZE       = 256      # 滑動窗口大小
STRIDE           = 192      # 步長（overlap = 64px）
GT_BUFFER_PX     = 500      # GT bbox 外擴距離（用於裁出工作區域）
KPCA_D           = 4        # PCA 輸出維度（論文 D=25，對應 224 波段）
IFOREST_T        = 800      # 樹的數量（論文預設值）
IFOREST_PSI      = 256      # 子採樣大小（Liu 2008 預設值）
MIN_VALID_RATIO  = 0.3      # patch 有效像素比例門檻
```

偵測門檻：所有 patch 分數的 bottom 5%（最異常）

---

## 評估邏輯修正

### 原始 Bug

```python
# 舊版：只對含油 GT 的 patch 評分 → Precision 恆為 1.0
for sx, sy in oil_patches:
    scored.append(...)
```

### 修正後

```python
# 新版：對 crop 內 ALL patches 評分，再與 GT oil_set 比對
all_scored = []
for pr in patch_rows:
    for pc in patch_cols:
        all_scored.append((sx, sy, float(finite.mean())))

det_thr = np.percentile(all_vals, 5)  # bottom 5% 為偵測
det_set = {(sx, sy) for sx, sy, sv in all_scored if sv <= det_thr}
```

修正後的 Recall/Precision/F1 才有實際意義，原先 Precision=1.0 是假象。

---

## 視覺化設計

每個場景的每個 region（獨立油汙連通區域）輸出一張 800×800 JPEG：
- Panel 1：RGB（B04/B03/B02）
- Panel 2：GT mask
- Panel 3：iForest anomaly score（heat map）
- 指標覆蓋在圖上：GT oil patch 數 / 偵測數 / Recall / Precision / F1

取代舊版的 N×256×256 patch JPEG（每場景可能輸出數百張）。

---

## 現有輸出（截至 2026-06-07）

已處理 89 個場景（包含事件場景與部分 NOAA 場景），共輸出：
- 91 個 `_full_iforest.txt`（偵測到的 patch 座標）
- 96 個 `_full_r{i}_overview.jpg`
- 96 個 `_full_r{i}_prob.tif`（anomaly score GeoTIFF）

---

## 全場景執行計畫

### 執行指令

```bash
python generate_iforest_full_pipeline.py --all --max-scenes 220
```

### 速度估算

| 步驟 | 舊版（T=800 + SVM + ERW） | 新版（T=800 只） |
|------|--------------------------|----------------|
| 單場景 | ~41 分鐘（16RCS 實測） | ~5 分鐘（估算） |
| 220 場景 | ~150 小時 | **~18 小時** |

前次跑了 1 個場景後中止（因速度不可接受），新 pipeline 尚未完整跑完 220 場景。

---

## 另一腳本：`generate_iforest_patch_coords.py`

為全場景（一次載入整張 10980×10980）設計的三段式 pipeline：

```
Pass 1：乾淨水體 patch 篩選 → 擬合 PCA + iForest
Pass 2：對全部 patch 評分 → score_map + patch_feats
Pass 3：patch 級 SVM（pseudo label + 空間過濾）
```

輸出：`_iforest.txt`（patch 座標）、`_score.tif`、`_svm.tif`、`_overview.jpg`

此腳本含 SVM，速度仍較慢，目前以 `generate_iforest_full_pipeline.py` 為主力。

---

## 下一步

- [ ] 執行 `--all --max-scenes 220`，完成全部 220 NOAA 場景
- [ ] 統計各場景的 Recall / Precision / F1，整理成匯總表
- [ ] 比較事件場景與 NOAA 場景的偵測性能差異
- [ ] 評估是否需要 post-processing（移除小連通區塊）
- [ ] 考慮是否值得對特定困難場景重新引入 SVM 精修

---

## 相關頁面

- [[20260603_Unsupervised油汙偵測初探]]
- [[20260604_iForest全場景改進]]
- [[Duan2022_HOSD_IsolationForest]]
