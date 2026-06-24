---
type: pipeline
tags: [reconstruction, evaluation, sliding-window, tta, cloud-mask, prevalence, oil-detection, annot-only]
project: OIL_PROJECT_MutiBand_VRT
updated: 2026-06-24
---

# VRT Pipeline — 03 重組評估（Reconstruction & Evaluation）

> 系列文件：[[VRT_pipeline_01_前處理]] | [[VRT_pipeline_02_模型訓練]] | 本文

---

## 概覽

模型訓練完成後，對每個 test scene 做全場景 Sliding Window 推論、重組 prob_map、Cloud Mask 後處理，最後對照 GT 計算全局指標。這是本專案的**主交付評估指標**。

```
best.pt（EMA 權重）
    ↓ 對每個 test scene：
       讀整張 10980×10980 VRT → Sliding Window 切 patch → TTA × 3 forward
    ↓ prob_map 重組（10980×10980×2）
    ↓ argmax → final_mask
    ↓ Cloud Mask 後處理（SWIR1>0.05 AND Blue>0.20 → 強制 bg）
    ↓ 比對 GT → 累加 global confusion matrix
    ↓ Excel 輸出：recon_* + annot_recon_* 兩組指標
```

---

## A. Patch-level Evaluation（快速評估）

**模組**：`evaluation_module.py` → `DeepLabAdapter.val()`

**用途**：訓練迴圈中的 val 指標（用於 early stop）、快速確認模型學習狀況。

| 項目 | 設定 |
|------|------|
| 輸入 | 預先切好的 256×256 patch（test.txt 列表）|
| TTA | 原圖 + HFlip + VFlip（3 次 forward 平均 logits）|
| 指標計算 | 累加 confusion matrix → IoU / F1 / Precision / Recall |
| 特殊說明 | mIoU / F1-macro **排除 background class**（只報 Oil 指標）|
| 執行時間 | 分鐘級（快）|

---

## B. Reconstruction Evaluation（主交付指標）

**模組**：`reconstruct_module.py` → `run_reconstruction()`

**觸發方式**：yaml 內設定 `post_tests.reconstruction.enabled: true`，跟訓練同一個 `main_runner.py` 一起跑。

### 全場景重組流程（每個 test scene）

1. **讀整張影像**：從 VRT 讀 8-band 10980×10980（~60-90s，CPU-bound LZW 解壓）
2. **Sliding Window 切 patch**：~1849 個 256×256 patch（stride=192）
3. **TTA × 3 forward**：原圖 + HFlip + VFlip，logits 平均
4. **重組 prob_map**：probs 累加到 10980×10980×2 array，weight_map 處理重疊區域
5. **argmax → final_mask**
6. **Cloud Mask 後處理**：
   - 條件：SWIR1（B11）> 0.05 **AND** Blue（B02）> 0.20
   - 動作：符合條件的像素強制設為 background（排除雲遮罩誤偵）
7. **比對 GT**：
   - `recon_*`：用 JSON polygon rasterize 後比對（整張 10980×10980）
   - `annot_recon_*`：從 mask TIF 拿 GT，未標注區（255）ignore → **避免懲罰「模型抓到但人工未標」的區域**
8. **累加 global confusion matrix**（所有 test scene 加總再算指標）
9. **輸出**：
   - `overlays/`：衛星底圖（NIR-R-G）+ cyan 油汙著色 overlay
   - `mask_compare/`：GT vs Pred 並列上色

**執行時間**：fold2（146 scenes）約 4-5 小時（CPU-bound，B01 60m LZW 解壓是瓶頸）

### Infer 設定

| 項目 | 設定 |
|------|------|
| Patch Size | 256 |
| Stride | 192 |
| Infer Batch Size | 128（RTX 5090，VRAM 32GB；2026-06-23 從 64 升至 128）|
| TTA | Original + HFlip + VFlip |

---

## 兩組 GT 比對的差異

| 指標前綴 | GT 來源 | Ignore 區域 | 用途 |
|---------|--------|------------|------|
| `recon_*` | JSON polygon rasterize（整張）| 無 | 嚴格評估，包含所有未標注區 |
| `annot_recon_*` | mask TIF（GT 值 0/1/2/255）| 255（未標注）→ ignore | **主交付用**：只算人工確認的區域 |

> 選用 `annot_recon_*` 作為主交付指標的理由：油汙偵測實際標注困難，標注人員可能漏標，用 mask TIF 的 255-ignore 機制可以避免因標注不完整而低估模型性能。

---

## Fold2 最新交付指標（3fold_random）

### Annot-only 評估（主交付）

| 指標 | 數值 |
|------|------|
| Reconstruction Oil IoU | **39.80%** |
| Oil IoU（annot-only）| **55.74%** |
| Precision（annot-only）| **99.89%** |
| Recall（annot-only）| **55.78%** |
| F1（annot-only）| **71.58%** |

### 三組 Prevalence 對照（同一 fold2 model）

模型固有 Recall（TPR ≈ 55.77%）與 FPR（≈ 0.219%）跟 prevalence 無關，可由此反算不同 prevalence 下的交付指標：

| Prevalence | F1-Score | IoU | Accuracy | 說明 |
|-----------|---------|-----|---------|------|
| 0.025% | 10.97% | 5.80% | 99.77% | 原始全 scene 真實分布（油汙極稀少）|
| **0.5%** | **55.96%** | **38.85%** | **99.56%** | **建議主交付：模擬真實海域監測情境** |
| 8.09% | 70.48% | 54.42% | 96.22% | oil-only filter 焦點區評估 |

> **主交付建議用 0.5% prevalence**：代表「每 200 個背景像素有 1 個油汙像素」的監測情境，比原始分布更貼近實際部署場景，且統計上更穩定。

---

## reconstruct_v2.py — 效能優化版本（2026-06-22 ~ 06-24）

**原則**：所有改動都在 `reconstruct_v2.py` 和 `main_runner_v2.py`，原 `reconstruct_module.py` 不動。

### 改動清單

**Stage 1（清理 + 小優化）**：
- `rglob` 直接過濾 `.vrt` 副檔名（省 stat）
- `np.clip` 改為 in-place（每 scene 省一個 ~4 GB 暫存 array）

**Stage 2（windowed VRT read，opt-in）**：
- 設定：`use_windowed_read: true`
- 機制：先從 GT JSON 找含油 patch 位置，只 windowed read 那些 256×256 區域（B01 加 padding 處理 60m→10m resize 邊界）
- **適用場景**：sparse 評估（只算含油 patch）、快速驗證
- **速度**：~120s/scene → ~10-15s/scene（oil-only filter 模式效益顯著）

**skip_overlay 旗標**：windowed 模式自動跳過 NIR-R-G bg PNG 讀取 + overlay 渲染（每 scene 省 ~10-15s），只輸出 mask_compare

**Perf 優化（2026-06-24）**：
```python
torch.backends.cudnn.benchmark = True
torch.compile(adapter.model, mode='reduce-overhead')
tensors.pin_memory().to(device, non_blocking=True)
```

**入口**：
```bash
conda run -n mamba_env python main_runner_v2.py --config <yaml>
```

---

## Oil-only Filter 模式

**設定**：`eval_only_oil_patches: true` + `oil_patch_min_pixels: 100`

**作用**：只對「GT 含 ≥ 100 oil pixel 的 patch」評估，排除純背景區域。

**實作**：`reconstruct_module_oilonly.py` + `main_runner_oilonly.py`（v2 windowed 模式已整合此邏輯）

**注意事項**：
- 可作為「條件性評估」交付時補充說明
- **不可寫成 test set 整體評估**，必須清楚標明 "evaluated on oil-containing patches only"
- 對應 prevalence = 8.09% 的那組指標

---

## fuzzy_find_file Bug Fix（2026-06-23）

**問題**：原版 `fuzzy_find_file` 在目標檔不存在時，會 fallback 挑到錯誤的檔案，導致 overlay 用了錯誤的衛星底圖（歷史上 ~55% scene 配錯）。

**修正**：exact match 優先 + fuzzy fallback 加 keyword 閾值。

**影響**：此 bug 修正後，fold2 146 scene 的 overlay 視覺化全部正確。

**yaml 相關更新**（全部 yaml 一起改）：
- `vis_image_dir`：從 NAS 改為本機（`/home/alanyh/oil_dataset/new/full_band/NIR_R_G_Output_png/all_ms6/`）
- `infer_batch_size`：64 → 128

---

## 輸出格式

### Excel（result_excel/）

每個 fold 產出一個 xlsx，含以下欄位（每個 scene 一列）：

| 欄位 | 說明 |
|------|------|
| `recon_oil_iou` | JSON GT 比對的 Oil IoU |
| `recon_precision` | JSON GT 的 Precision |
| `recon_recall` | JSON GT 的 Recall |
| `annot_recon_oil_iou` | mask TIF GT（ignore 255）的 Oil IoU |
| `annot_recon_precision` | mask TIF GT 的 Precision |
| `annot_recon_recall` | mask TIF GT 的 Recall |
| `annot_recon_f1` | mask TIF GT 的 F1 |

最後一列為所有 scene 的 micro-average（global confusion matrix 計算）。

### 視覺化輸出

```
result-seg/
    {FOLD_NAME}/
        overlays/
            {SCENE_NAME}_overlay.jpg      # NIR-R-G 底圖 + cyan 油汙著色
        mask_compare/
            {SCENE_NAME}_compare.jpg      # 左：GT mask，右：Pred mask
```

---

## 主要腳本

| 腳本 | 說明 |
|------|------|
| `main/reconstruct_module.py` | 原版重組評估（穩定版）|
| `main/reconstruct_v2.py` | 效能優化版（windowed read + torch.compile）|
| `main/evaluation_module.py` | Patch-level 評估（val 用）|
| `main/main_runner.py` | 訓練 + 重組一體入口（原版）|
| `main/main_runner_v2.py` | v2 版本入口 |
| `main/main_runner_oilonly.py` | oil-only filter 模式入口 |

---

## 相關頁面

- [[VRT_pipeline_01_前處理]] — VRT 建立、GT Mask、Fold 切分
- [[VRT_pipeline_02_模型訓練]] — 訓練架構與超參數
- [[DeepLabV3+]] — 模型工程紀錄
- [[20260614_DeepRX_EM_v3_全場景實驗總結]] — 無監督對照（Recall=0.371）
- [[deeplabv3plus_358clean_overfitting_改善方向]] — A1/A4/A3 改善計畫
