---
type: pipeline
stage: preprocess
tags: [COG, VRT, new-data, ms6, sen2like]
related: [OIL_PROJECT_VRT_0422.md, annotation_workflow.md]
---

# 新增 Sen2Like 資料流程

> 每次有新場景進來時，依序執行以下步驟。Step 1/2 有防呆（自動跳過已處理），可對整個目錄重跑。Step 3 每次都必須重跑。

---

## 前提：目錄結構

新資料必須符合以下三層結構才能被腳本掃描到：

```
/home/alanyh/oil_dataset/new/full_band/MS6_sen2like/
  {LOCATION}/
    {YEAR}/
      {SCENE_NAME}/
        *_B01.tif
        *_B02.tif
        *_B03.tif
        *_B04.tif
        *_B08.tif
        *_B8A.tif
        *_B11.tif
        *_B12.tif
```

例：
```
MS6_sen2like/NOAA_Gulf_of_Mexico_High_Confidence/2025/
  NOAA_Gulf_of_Mexico_High_Confidence_20250418_S2_19TDE/
    S2A_MSI_20250418_..._B01.tif
    S2A_MSI_20250418_..._B02.tif
    ...
```

---

## Step 1：Strip TIF → COG

**腳本**：`/home/alanyh/oil_dataset/new/full_band/convert_to_cog.py`

```bash
conda run -n mamba_env python3 /home/alanyh/oil_dataset/new/full_band/convert_to_cog.py
```

**行為**：
- 掃描 `MS6_sen2like/` 下所有 B02/03/04/08/B8A/B11/B12 的 TIF（共 7 個 target band，B01 60m 跳過）
- 已是 tiled 格式（COG）的檔案自動跳過，只處理新進來的 strip TIF
- 安全機制：先寫 `.cog_tmp.tif`，像素值驗證通過後才覆蓋原檔
- 8 workers 並行，log 輸出至 `cog_convert.log`

**為什麼需要這步**：Sen2Like 輸出預設為 Strip TIF（整行存放），讀 256×256 patch 時 GDAL 必須掃 256 整行（43× 浪費 I/O）。COG 改為 256×256 tile 分塊，讀 patch 只讀對應 tile，速度提升 9×。

**實測**（B03）：strip 397ms → COG 43ms

---

## Step 2：建立 VRT

**腳本**：`preprocess/build_vrt_ms6.py`

```bash
conda run -n mamba_env python3 /mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_0422_VRT_training/preprocess/build_vrt_ms6.py
```

**行為**：
- 掃描 `MS6_sen2like/*/*/*` 找出所有場景（三層：LOCATION/YEAR/SCENE）
- VRT 已存在則跳過（可整個目錄重跑）
- 8 個波段任一缺失則跳過並報告 fail
- 以 B02（10m，10980×10980）為 VRT 參考解析度，各波段設 SrcRect/DstRect，GDAL 自動 resample
- 輸出路徑：`MS6_sen2like_vrt/{LOCATION}/{SCENE_NAME}.vrt`

**波段順序**（依波長升序）：B01 → B02 → B03 → B04 → B08 → B8A → B11 → B12

**為什麼以 B02 為參考**：B02 是 10m 原生解析度，與 Mask TIF（10980×10980）座標系完全一致。若用 B01（60m，1830×1830）作參考，Patch TXT 的像素座標（最大 ~10979）會超出 VRT 邊界，讀出全零（這是舊版的致命 Bug）。

---

## Step 3：重建 Fold Split 與 Patch 座標

新場景加入後，fold 分配需要完整重跑：

```bash
# 3a. 重新分配場景到各 fold（stratified scene-level）
conda run -n mamba_env python3 /mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_0422_VRT_training/preprocess/build_scene_splits_stratified.py

# 3b. 重新生成 GB1.0 patch 座標 TXT
conda run -n mamba_env python3 /mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_0422_VRT_training/preprocess/generate_patch_coords.py
```

**輸出路徑**：
- Scene split：`/mnt/backup/oil_dataset/new/full_band/data_split/5_fold/scene_level/`
- Patch coords：`/mnt/backup/oil_dataset/new/full_band/data_split/5_fold/patch_level_GB1.0/`

**注意**：Step 3 每次新增資料都必須重跑，因為 stratified 分配需要感知所有現有場景才能均勻分配。

---

## Step 4（選用）：生成測試場景視覺化 PNG

若新場景屬於 test set（2025 年），補生成 NIR-R-G 假彩色 PNG：

```bash
conda run -n mamba_env python3 /mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_0422_VRT_training/preprocess/generate_nirRG_png.py
```

輸出：`/mnt/backup/oil_dataset/new/full_band/NIR_R_G_Output_png/test_ms6/`

---

## 完整流程圖

```
新 Sen2Like TIF 放入 MS6_sen2like/{LOCATION}/{YEAR}/{SCENE}/
        ↓
Step 1  convert_to_cog.py        Strip TIF → COG（跳過已轉換，可全目錄重跑）
        ↓
Step 2  build_vrt_ms6.py         建 VRT（跳過已存在，可全目錄重跑）
        ↓
Step 3a build_scene_splits_stratified.py   Fold split 重算（每次必跑）
        ↓
Step 3b generate_patch_coords.py           Patch 座標 TXT 重生成（每次必跑）
        ↓（若新場景為 test set）
Step 4  generate_nirRG_png.py    NIR-R-G 視覺化 PNG
```

---

## 關鍵路徑索引

| 項目 | 路徑 |
|---|---|
| Sen2Like TIF（原始） | `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like/` |
| VRT 虛擬影像 | `/home/alanyh/oil_dataset/new/full_band/MS6_sen2like_vrt/` |
| Mask TIF | `/home/alanyh/oil_dataset/new/full_band/mask/` |
| Scene split TXT | `/mnt/backup/oil_dataset/new/full_band/data_split/5_fold/scene_level/` |
| Patch coords TXT | `/mnt/backup/oil_dataset/new/full_band/data_split/5_fold/patch_level_GB1.0/` |
| NIR-R-G PNG | `/mnt/backup/oil_dataset/new/full_band/NIR_R_G_Output_png/test_ms6/` |
| COG 轉換腳本 | `/home/alanyh/oil_dataset/new/full_band/convert_to_cog.py` |
| VRT 建置腳本 | `preprocess/build_vrt_ms6.py` |
| COG 轉換 log | `/home/alanyh/oil_dataset/new/full_band/cog_convert.log` |
