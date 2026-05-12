---
type: pipeline
tags: [annotation, labelme, qgis, gpkg, json, mask]
related: [OIL_PROJECT_VRT_0422.md]
---

# 標注工作流程（LabelMe JSON / QGIS GPKG）

## 一句話摘要

新資料可以用 QGIS 標記（輸出 GPKG），再用 `gpkg_to_labelme.py` 轉換成 LabelMe JSON，後續 pipeline 完全不需要改動。

---

## JSON 在 Pipeline 中的角色

JSON 標注檔在兩個地方被使用，缺一不可：

| 位置 | 用途 |
|---|---|
| `preprocess/json_to_mask_tif.py` | JSON → 訓練用 mask TIF（pixel class index） |
| `main/reconstruct_module.py:376` | 大圖重組時讀取 GT，計算 mIoU / 混淆矩陣 |

mask TIF 的像素值定義：
- `0` = Oil
- `1` = Background
- `2` = Others
- `255` = 未標注（ignore_index）

---

## 為什麼舊 JSON 不含地理資訊

舊的 LabelMe JSON 是在 acolite 處理後匯出的 PNG/JPG 上標記的，影像本身沒有地理資訊，因此 JSON 內的座標全部是**像素座標**。pipeline 只需要像素座標，所以這不是問題。

---

## 新工作流程（QGIS 標記）

```
原始 Sentinel-2 TIF（含地理資訊）
    ↓ 載入 QGIS
QGIS 標記 → GPKG（WGS84 地理座標）
    ↓
gpkg_to_labelme.py（需要 REF_TIF 提供 affine transform）
    ↓ 地理座標 → 像素座標
LabelMe JSON（像素座標，格式與舊版相同）
    ↓
json_to_mask_tif.py / reconstruct_module（完全不動）
```

**關鍵前提**：每個 GPKG 場景都必須對應到一個 REF_TIF（原始 `.tif`），腳本透過它的 affine transform 把 WGS84 座標換算成像素位置。

---

## 橋接腳本

**位置**：`file_label_temp/gpkg_to_labelme.py`

**目前狀態**：單場景硬編碼（`GPKG_PATH`, `REF_TIF`, `OUTPUT_JSON`, table 名稱都寫死），每次處理新場景須手動修改。

**核心邏輯**：
1. 用 `sqlite3` 直接讀取 GPKG 的幾何二進位（WKB）
2. `_parse_wkb()` 解析出 exterior ring 的 (lon, lat) 列表
3. `geo_to_pixel()` 透過 rasterio 的 `warp_transform` 把 WGS84 → TIF 的 CRS，再用 affine transform 算出像素 (col, row)
4. 輸出標準 LabelMe JSON 格式

---

## acolite 舊標注 vs. sen2like 新資料：像素座標是否對齊？

**結論：對齊，舊 JSON 可直接用於 sen2like 資料。**

原因：JSON 存的是像素座標，不是地理座標。只要兩個工具處理同一個 Sentinel-2 tile 時像素網格一致，座標就不會偏移。

Sentinel-2 的 tile 邊界是固定的 UTM 座標，acolite 和 sen2like 處理同一個 granule（例如 34SGG）時，兩者的像素 (0,0) 都指向同一個地理位置（tile 左上角）。

**實際驗證（34SGG 場景）**：
- 舊 JSON：`imageHeight: 10980, imageWidth: 10980`
- sen2like TIF：`height: 10980, width: 10980`，`transform origin: (699960, 4200000)` = 34SGG tile 標準左上角

兩者完全一致。

### 唯一的風險

如果當初 acolite 是在**裁切過**的影像上標記的（不是完整的 10980×10980 tile），起點就不同，座標會偏移。

**判斷方式**：看 JSON 的 `imageWidth/imageHeight` 是否為 10980。若是，則像素網格與 sen2like 對齊，可直接使用。

---

## 待改進

- `gpkg_to_labelme.py` 目前為單場景硬編碼，若要批次處理多個場景需要改寫成接受資料夾或命令列參數的版本
- REF_TIF 路徑規則：`images/{scene_name}/10m/S2A_MSI_*.tif`，可用 glob 自動配對
