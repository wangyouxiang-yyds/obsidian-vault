# 操作紀錄

> append-only，勿修改舊紀錄。格式：`## [YYYY-MM-DD] 操作類型 | 說明`

---

## [2026-05-04] init | 建立 Wiki 基礎結構（index.md、log.md、wiki/ 子目錄、raw/、outputs/）
## [2026-05-04] pipeline | 新增 wiki/pipeline/sen2like.md（整理自 GEMINI.md：全自動 pipeline、L2F 選擇、波段對照、Fusion 排查、比對工具箱）
## [2026-05-04] pipeline | 新增 wiki/pipeline/project_overview.md（整理自 project_descirbe.md：OIL_PROJECT_MutiBand_0420 模型、前處理、訓練、評估、I/O 優化全覽）
## [2026-05-04] experiment | 新增 wiki/experiments/20260429_OSDMamba整合至0420.md（整理自 session_0429 + task_0429）
## [2026-05-04] experiment | 新增 wiki/experiments/20260430_訓練效能優化.md（整理自 task_0430：RTX 5090 CPU-GPU 同步、向量化 Loss、EMA 優化）
## [2026-05-04] experiment | 新增 wiki/experiments/20260501_大圖重組效能優化.md（整理自 task_0501：NAS I/O、GPU 側重組、窗口化讀取）
## [2026-05-04] environment | 新增 wiki/environment/Docker_環境設定.md（彙整 Dockerfile：基礎映像、CUDA 編譯、Gemini/Claude MCP 持久化設定）
## [2026-05-05] pipeline | 新增 wiki/pipeline/OIL_PROJECT_VRT_0422.md（分析 0422 VRT 版與 0420 版差異：引入虛擬化影像讀取與全場景 Mask 機制）
## [2026-05-05] pipeline | 移植 0420 版 OSDMamba 優化至 0422 VRT 版：引入 AMP、EMA、class_weights、TTA 驗證，同時保留 VRT 動態讀取機制

## [2026-05-05] config | 更新 0422 OSDMamba YAML：切換至 GB1.0 split、啟用 VRT 動態讀取、優化本機重組路徑、設定 batch_size=16

## [2026-05-06] fix | 修正 experiments_osdmamba_CV.yaml：pixel_mapping 對齊 DeepLabV3+，Others/unannotated 改為 class 1（Background），避免推論時船隻被誤判為油汙
## [2026-05-05] papers | 新增 wiki/papers/OSDMamba_摘要.md（AI 閱讀總結：分析 OSDMamba 核心貢獻與非對稱解碼器架構）
## [2026-05-12] pipeline | 新增 wiki/pipeline/annotation_workflow.md（釐清 JSON 在 pipeline 的角色：json_to_mask_tif + reconstruct GT；確認 QGIS GPKG → gpkg_to_labelme.py → JSON 的橋接流程可行）
## [2026-05-12] pipeline | 更新 annotation_workflow.md：補充 acolite 舊標注與 sen2like 新資料的像素座標對齊分析，確認 10980×10980 tile 下兩者網格一致，舊 JSON 可直接沿用

## [2026-05-06] fix | 修正 0422 訓練速度過慢（每 epoch ~30 分鐘）：診斷為 NAS random seek I/O 瓶頸（每 sample 做 2 次 NAS rasterio.Window 讀取）；將 vrt_dir 改指本機 stack_tif、mask TIF 複製至本機，消除訓練階段所有 NAS I/O
## [2026-05-11] model | 新增 wiki/models/DeepLabV3+.md（工程紀錄：ResNet50 骨幹、訓練超參數、推論設定、已知問題）
## [2026-05-11] model | 新增 wiki/models/OSDMamba.md（工程紀錄：SwinUMambaD 配置、Dice+Focal 損失、EMA、RTX 5090 環境需求、與 DeepLabV3+ 對照表）
## [2026-05-20] experiment | 新增 wiki/experiments/20260520_新批Sen2Like資料Pipeline重建計畫.md（新資料引入完整 pipeline：GB1.0、2025作為固定test、stratified scene-level fold split 設計）
## [2026-05-20] update | 更新 20260520 計畫文件：新增 Step 0（GPKG→JSON 轉換）、確認 Sen2Like 輸出僅 8 波段（B01/02/03/04/08/8A/11/12）、更新 stack/YAML 修改說明

## [2026-05-21] preprocess | Step 2：build_vrt_ms6.py 建立 220 個 8-band VRT，輸出至 MS6_sen2like_vrt/；1 個場景（20250529_S2_15RXM）缺 B02 失敗，已排除
## [2026-05-21] preprocess | Step 2 附帶：generate_nirRG_png.py 生成 66 個 test 場景的 NIR-R-G 假彩色 PNG，輸出至 NIR_R_G_Output_png/test_ms6/
## [2026-05-21] preprocess | Step 3：build_scene_splits_stratified.py 生成 stratified 5-fold scene split（66 test + 154 非2025）
## [2026-05-21] preprocess | Step 4：generate_patch_coords.py 生成 GB1.0 patch 座標 TXT（fold1: train=2044, val=362, test=1334）
## [2026-05-21] config | Step 5：更新 experiments_CV.yaml（in_channels=8、新 VRT/mask/JSON 路徑、pixel_mapping 對齊）；建立 JSON_ms6 扁平化 symlink 目錄（221 個 symlink）
## [2026-05-21] experiment | 新增 wiki/experiments/20260521_MS6_Pipeline執行紀錄.md（Steps 2–5 執行結果、試跑 Fold 1 卡住問題、資料單位 Bug 診斷）
## [2026-05-21] bug | 發現 deeplab_adapter.py 資料單位 Bug：MS6_sen2like TIF 為 raw×10000 格式，但 _get_pos_vrt_item 未除以 10000，導致 patch>100 clip 歸零所有有效像素；待修正
## [2026-05-21] bug | 發現 VRT 解析度不對齊 Bug（致命）：build_vrt_ms6.py 以 B01（60m/1830×1830）為 VRT 參考，導致 VRT 座標系為 60m；但 Patch TXT 座標在 10m Mask 空間（最大~10979），座標超出 VRT 邊界全讀零 → 62 個 epoch Oil IoU=0 → 訓練全部無效
## [2026-05-21] fix | 修正 build_vrt_ms6.py：改以 B02（10m/10980×10980）為 VRT 參考，各波段用實際 SrcRect→VRT DstRect，GDAL 自動 resample；VRT_OUT_DIR 改至本機 SSD；刪除舊 220 個 VRT，重建完成
## [2026-05-21] fix | 重算 mean/std（排除 nodata 零像素）：mean=[0.191,0.184,0.178,0.171,0.171,0.170,0.152,0.145]，std=[0.137,0.131,0.129,0.131,0.136,0.131,0.073,0.063]；更新 experiments_CV.yaml
## [2026-05-21] experiment | 清除無效 fold1 結果，重啟 Fold 1 訓練（修正後首次有效訓練）
## [2026-05-21] perf | 診斷訓練慢（7–9 min/epoch）：GPU=0%，worker CPU=49% → rasterio.open 每 sample 重開 VRT+8 TIF 為瓶頸
## [2026-05-21] fix | deeplab_adapter.py 加入 _cached_rasterio_open（per-process file handle cache）+ DataLoader persistent_workers=True，消除跨 epoch cache 失效

## [2026-05-22] perf | 診斷 epoch 仍慢（8-9 min）真正根本原因：MS6 TIF 為 strip 格式（block_shapes=[(1,10980)]），每次讀 256×256 patch 掃 256 整行 strip，I/O 量是需求的 43×。Benchmark：B03 strip=397ms vs COG tiled=43.6ms（9× 差距）
## [2026-05-22] perf | 舊「1:20/epoch」為虛假速度：60m VRT 版本 patch 座標超出 VRT 邊界，讀出全零，根本無真實 I/O
## [2026-05-22] preprocess | 啟動 COG in-place 轉換（convert_to_cog.py）：B02/03/04/08（10m）+ B8A/11/12（20m）共 1546 個 TIF，8 workers 並行，PID 49030，log → cog_convert.log，預計 2.7 小時完成
## [2026-05-22] preprocess | COG 轉換完成：1546/1546 成功，0 失敗，耗時 234 min；實測 B03 strip→COG 讀速 397ms→43ms（9.2×），VRT 8-band 1600ms→450ms（3.6×）
## [2026-05-22] config | 更新 experiments_CV.yaml：新增 class_weights=[13.0, 1.0]（Oil:BG=1:13.8，inverse frequency）
## [2026-05-22] experiment | 清除舊 checkpoint，重啟 Fold 1 訓練（PID 63443）；log → /home/alanyh/oil_dataset/new/full_band/train_fold1.log
## [2026-05-22] experiment | Fold 1 Epoch 1-9 結果：Oil IoU 0.028→0.164（epoch 8 best），epoch ~3 min；GPU 使用率 0%（仍 I/O bound），VRAM 6.4/32GB

## [2026-05-22] init | 建立知識庫骨架與 MS6 資料集頁面 (wiki/datasets/ms6_sen2like.md)
## [2026-05-22] update | 新增技術概念頁面：COG (wiki/concepts/cloud_optimized_geotiff.md), Hard Negative Mining (wiki/concepts/hard_negative_mining.md)
## [2026-05-22] lint | 更新 index.md 目錄結構
