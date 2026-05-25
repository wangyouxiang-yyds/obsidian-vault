---
type: concept
tags: [acolite, sen2like, data-format, atmospheric-correction, band]
related: [wiki/datasets/ms6_sen2like.md, wiki/pipeline/sen2like.md, wiki/experiments/20260525_架構演進與差異彙整.md]
---

# Acolite vs Sen2Like 資料格式差異

---

## 1. 像素值單位（最重要的差別）

| | Acolite | Sen2Like |
|---|---|---|
| 儲存類型 | `float32`，直接反射率 | `uint16`，**raw × 10000** |
| 數值範圍 | ~0.0 ~ 0.1（水色優化，偏暗） | 0 ~ 10000（除以 10000 才是反射率） |
| 讀進程式後 | 直接用 | **必須 `/ 10000.0`** |

Sen2Like 用 uint16 × 10000 是為了同時節省儲存空間（float32 → uint16，省一半）並保留 4 位小數的精度。Acolite 直接存 float，不需要這個轉換。

> ⚠️ **實際踩過的 Bug**：訓練讀取（`_get_pos_vrt_item`）和推論重組（`reconstruct_module.py`）都曾遺漏這個除法，導致 clip(0,10) 把所有有效值壓成 10，normalize 後嚴重偏移，模型全部輸出背景（Oil IoU=0）。

---

## 2. 波段數與選擇

| | Acolite | Sen2Like |
|---|---|---|
| 可輸出波段 | Sentinel-2 全 13 波段（含紅邊 B05/06/07） | **只有 8 個共同波段**（B01/02/03/04/08/8A/11/12） |
| 為何少三個 | — | 為了跨衛星一致性（Landsat 沒有紅邊波段），B05/06/07 不在 harmonization 範圍內 |
| 訓練 in_channels | 11 | **8** |

---

## 3. 空間解析度

| 波段 | Acolite | Sen2Like L2F |
|---|---|---|
| 可見光（B02/03/04） | 10m（原生） | 10m |
| NIR 窄頻（B8A）、SWIR（B11/12） | **20m（原生）** | **10m（Fusion 升採樣）** |
| 沿岸氣溶膠（B01） | 60m（原生） | 60m（VRT 內再升採樣至 10m） |
| 全場景尺寸 | 各波段不同 | **統一 10980×10980** |

Sen2Like 的 L2F 產品把 SWIR（B8A/B11/B12）從 20m 融合至 10m，所有波段對齊同一個像素網格，是它對模型輸入最直接的好處。

---

## 4. 大氣校正方向

| | Acolite | Sen2Like |
|---|---|---|
| 優化目標 | 水色遙測（water-color）；對水面做更積極的大氣扣除 | 標準地表反射率（surface reflectance）；追求跨衛星一致性 |
| 視覺效果 | 畫面偏暗 | 畫面較亮（SBAF 校正） |
| 油汙對比 | 油汙與海面對比較明顯 | 數值標準化，利於多衛星混訓 |

---

## 5. 地理資訊與標注對齊

| | Acolite | Sen2Like |
|---|---|---|
| GeoTIFF 輸出 | 有 CRS / affine transform | 有 CRS / affine transform |
| 舊版 PNG 匯出 | **無地理資訊**（舊 JSON 只存像素座標） | 不輸出 PNG |
| 與舊 JSON 標注對齊 | ✅（原生相同像素網格） | ✅（同一 Sentinel-2 tile，像素 (0,0) 指向同一地理位置） |

舊的 Acolite 時期標注是在非地理參考的 PNG 上做的，JSON 存像素座標。只要確認 `imageWidth/imageHeight` 為 10980，就能直接沿用於 Sen2Like 資料，不需要重標。

---

## 6. 訓練 pipeline 差異

| | Acolite 時期（0420） | Sen2Like 時期（0422） |
|---|---|---|
| 訓練讀法 | 預切實體 patch 存 NAS | **VRT 動態讀取**（零實體 patch） |
| 底層 TIF 格式 | Strip（預設）→ 後來搬本機 SSD 解決速度 | Strip → 轉 **COG**（9× I/O 提升） |
| Fold split | Random shuffle（patch-level） | **Stratified scene-level**（地點×月份分組） |

---

## 一句話總結

Acolite 給 float32 直接反射率、全波段、各波段原生解析度；Sen2Like 給 uint16×10000、8 個跨衛星共同波段、全部統一至 10m。**讀進模型一定要記得除以 10000**，這是兩者最關鍵的格式差別。

---

## 相關頁面

- [sen2like.md](../pipeline/sen2like.md)
- [ms6_sen2like.md](../datasets/ms6_sen2like.md)
- [20260525_架構演進與差異彙整.md](../experiments/20260525_架構演進與差異彙整.md)
