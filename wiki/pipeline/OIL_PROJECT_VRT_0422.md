# OIL_PROJECT_MutiBand_0422_VRT_training 專案說明

**核心理念：全面虛擬化與動態讀取**
此版本是基於 0420 版本的重大架構升級，主要引入了 GDAL VRT 技術，實現了「零實體 Patch」的訓練流程。

---

## 🚀 核心升級：VRT 動態讀取架構

與 0420 版本需要預先裁切數萬張小圖（Patches）不同，0422 版本直接從原始影像中動態讀取資料。

### 1. 虛擬影像技術 (GDAL VRT)
- **機制**：使用 \`preprocess/build_vrt.py\` 將 11 個單波段的 TIF 檔案打包成一個數 KB 的 \`.vrt\` 虛擬多波段檔案。
- **優點**：
    - **節省空間**：不需要再儲存數十 GB 的 11-band 實體疊加圖 (Stacked TIF)。
    - **靈活性**：Dataset 在訓練時透過 \`rasterio.Window\` 直接讀取 VRT 指定座標，實現動態 Patching。

### 2. 全場景 Mask 化
- **機制**：\`preprocess/json_to_mask_tif.py\` 會將 LabelMe 的 JSON 標註轉換為與原始影像大小一致的「全場景 Mask TIF」。
- **訓練流程**：Dataset 同時對應「影像 VRT」與「Mask TIF」，根據座標 TXT 即時裁切資料送入模型。

---

## 🛠️ 主要腳本與職責

| 檔案 | 職責 | 關鍵點 |
|---|---|---|
| \`main/main_runner.py\` | 實驗總控、5-fold CV 排程 | 支援 \`{{experiment_name}}\` 預留位置進行細部調整 |
| \`main/deeplab_adapter.py\` | 核心模型與 Dataset 定義 | 實作了 \`read_images_from_vrt: true\` 的讀取邏輯 |
| \`preprocess/build_vrt.py\` | 建立多波段虛擬檔案 | 嚴格遵循 11 波長群組排列 |
| \`preprocess/json_to_mask_tif.py\` | 標註轉換 | 產生全場景 Label 來源 |

---

## 📈 評估機制進化

在 0422 版本中，**大圖重組 (Reconstruction)** 被視為標準評估手段：
- **5-fold CV**：每個 fold 結束後，會自動對該 fold 的測試場景進行 Sliding Window 重組。
- **指標計算**：以全圖的混淆矩陣為準，計算 \`recon_mIoU\` 與 \`recon_F1\`，這比 Patch-level 的平均分數更具參考價值。

---

## ⚠️ 開發防護機制 (防呆建議)

1. **GPU 記憶體清理**：程式碼中大量使用 \`gc.collect()\` 與 \`torch.cuda.empty_cache()\`，因為處理全場景大圖極易導致 OOM。
2. **Normalize 設定**：由於影像以 float32 讀入，Albumentations 的 \`max_pixel_value\` 必須設為 \`1.0\` 而非 \`255\`。
3. **輸入通道動態化**：模型初始化會自動讀取 YAML 中的 \`architecture_cfg.in_channels\` (通常為 11)，修改時需統一。

---

## 🔄 OSDMamba 優化移植 (來自 0420 版本)

為了提升 OSDMamba 的訓練效能與準確度，已將 0420 版本中的關鍵優化移植至 0422 專案中，同時確保不影響 VRT 動態讀取流程：

### 1. 訓練效能優化
- **AMP (混合精度)**：引入 `torch.amp` 自動混合精度訓練，顯著降低顯存佔用並提升計算速度。
- **EMA (權重平均)**：引入 `ModelEma` 並支援 `ema_update_interval`，驗證與儲存均使用平滑後的權重。
- **向量化指標計算**：在訓練迴圈中使用矩陣運算計算 IoU，減少 CPU 與 GPU 之間的同步等待。

### 2. 模型優化與損失函數
- **加權損失 (Class Weights)**：`_dice_loss` 與 `FocalLoss` 均支援類別權重，針對樣本不均的油汙類別進行加強學習。
- **標竿儲存 (Best Model Saving)**：模型儲存基準從整體的 `mIoU` 改為專注於 **Oil Class IoU**，確保最佳權重是對油汙偵測最有利的版本。

### 3. 進階驗證 (TTA Validation)
- **TTA (測試時增強)**：`val()` 方法支援「原圖 + 水平翻轉 + 垂直翻轉」三者融合預測，提升預測結果的穩定性。
- **詳細混淆矩陣**：自動生成百分比格式的混淆矩陣，並輸出 Recall, Precision, IoU, F1 等完整指標。

## 🔗 相關路徑
- **Dataset (NAS)**: \`/mnt/backup/oil_dataset/new/full_band/\`
- **VRT Files**: \`/mnt/backup/oil_dataset/new/full_band/vrt/\`
- **Results**: \`OIL_PROJECT_MutiBand_0422_VRT_training/result-seg/\`
