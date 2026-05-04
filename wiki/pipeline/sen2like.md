---
type: pipeline
stage: preprocess
tags: [sen2like, sentinel-2, landsat, fusion, harmonization, atmospheric-correction]
related:
  - wiki/concepts/
  - wiki/datasets/
---

# Sen2Like 處理流程

## 一句話定義
Sen2Like 是一套將 Sentinel-2 與 Landsat 8/9 進行大氣校正、光譜一致化（Harmonization）並融合（Fusion）至統一 10m 解析度的預處理工具。

---

## 1. 全自動 Pipeline（`run_pipeline.py`）

一鍵式流程：

```
原始 ZIP/TAR
  ↓ 解壓
  ↓ Sentinel-2 處理
  ↓ 跨區參考建立
  ↓ Landsat Fusion 處理
  ↓ 成果整理（Label_workplace / Dataset_workplace）
  ↓ 空間清理（刪除中間 .SAFE 資料夾，釋放數十 GB）
```

- `batch_process.py`：自動搜尋、解壓，依衛星類型（S2/L8/L9）分流處理。
- 強健性修正：`shutil.rmtree` 已修正以正確處理軟連結。

---

## 2. 產品類型選擇

### 使用 L2F（Fused）而非 L2H

| 產品 | 說明 | 建議 |
|------|------|------|
| L2H | 標準大氣校正，20m/30m 波段維持原始解析度 | 不推薦用於訓練 |
| **L2F** | Fusion 後 SWIR 波段提升至 10m | **優先使用** |

L2F 讓 B8A、B11、B12 等 SWIR 波段達到 10m 解析度，對油汙精細紋理識別至關重要。

---

## 3. 衛星波段對照（Harmonization Table）

Sen2Like 以下列「共同波段」進行跨衛星光譜一致化，**訓練模型時只選用這些波段**：

| 目標特徵 | Sentinel-2 | Landsat 8/9 | Sen2Like 統一輸出 |
|----------|-----------|------------|-----------------|
| 沿岸氣溶膠 (Coastal) | B01 (60m) | B01 (30m) | B01 (60m) |
| 藍 (Blue) | B02 (10m) | B02 | B02 (10m) |
| 綠 (Green) | B03 (10m) | B03 | B03 (10m) |
| 紅 (Red) | B04 (10m) | B04 | B04 (10m) |
| 近紅外 (NIR) | **B8A (20m)** | **B05 (30m)** | B8A (10m / Fused) |
| 短波紅外 1 | B11 (20m) | B06 | B11 (10m / Fused) |
| 短波紅外 2 | B12 (20m) | B07 | B12 (10m / Fused) |

> B08（S2 原生 10m 寬頻 NIR）因 Landsat 無直接對應，**不列入跨衛星訓練共同波段**。

---

## 4. 空間對齊策略（10m 為基準）

所有波段一律對齊至 **10m（10980 × 10980）**：

- **20m 波段**（B8A, B11, B12）：無 Fusion 時須 2× Upsampling。
- **60m 波段**（B01）：須 6× Upsampling。
- **NATIVE 資料夾**：存放未重採樣的原始解析度資料，**不可用於訓練**（解析度不統一且未 SBAF 校正，損害跨衛星泛化能力）。

---

## 5. Landsat Fusion 失敗排查

| 原因 | 修正方式 |
|------|---------|
| 搜尋路徑錯誤 | 修正 `my_config.ini` 的 `base_url` / `url_parameters_pattern`，改用 Tile ID 尋找參考圖 |
| 日期限制過嚴 | 修改 `S2L_Fusion.py`，移除「30 天內」限制，允許使用該 Tile 所有 S2 存量 |
| 缺乏當區參考 | 跨區借用策略：15RXP 缺 S2 時自動鏈結隔壁 15RXN 的參考圖強行觸發 Fusion |

---

## 6. 成果分流儲存

| 目錄 | 內容 | 用途 |
|------|------|------|
| `Label_workplace` | 標準命名 JPG 縮圖（`事件_日期_衛星_Tile.jpg`）| 標記軟體（LabelMe）快速讀取 |
| `Dataset_workplace` | 分層級 TIF 波段檔案 | 模型訓練原始資料 |

---

## 7. 影像比對工具箱

開發於 Acolite vs Sen2Like 差異分析階段：

| 腳本 | 功能 | 適用場景 |
|------|------|---------|
| `create_comparison_rgb.py` | 產出 1:1（10980px）NIR-R-G 影像 | LabelMe 比對，解決縮圖偏移問題 |
| `compare_spectral_diff.py` | 計算全波段（6通道）歐幾里得距離，產出全光譜總差異熱點圖 | 定量分析油汙邊緣歧見區域 |
| `create_side_by_side.py` | Acolite vs Sen2Like 左右並排 | 直觀對比油汙細節、噪點、解析度 |
| `create_full_montage.py` | 三排比對圖（Visible / NIR / SWIR）| 全方位分析各光譜段下油汙特徵 |

### 為何用 NIR-R-G（B08-B04-B03）比對？
海水在 NIR 幾乎全吸收（黑色），油汙反射特性不同，對比度遠優於真彩色 RGB，更利於檢查標記偏移。

### Acolite vs Sen2Like 亮度差異
- **Acolite**：針對水色優化，大氣扣除較多，畫面偏暗。
- **Sen2Like**：追求標準產品一致性，數值較高；SBAF 修正可能整體變亮，但關鍵是油汙與水面的對比是否提升。

---

## 8. 訓練前注意事項

1. **對齊確認**：使用 `Dataset_workplace` 下統一 10m 波段，防止像素位移。
2. **多波段輸入**：建議全 11 波段輸入 SegFormer，捕捉紅外微小特徵。
3. **標記位移檢查**：訓練前在 LabelMe 確認 Acolite 舊 JSON 與 Sen2Like 縮圖精確重合。

---

## 9. 系統監控

- CPU / 記憶體：`htop`
- 磁碟空間：`df -h`（`run_pipeline.py` 會自動清理，仍需確認）
