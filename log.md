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

## [2026-05-22] experiment | Fold 1（第一輪）訓練完成：Best Val mIoU=0.5835（Oil IoU=0.201, BG IoU=0.966）at Epoch 33；Early stopping at Epoch 83（patience=50）
## [2026-05-22] bug | 發現 NIR-R-G PNG VRT 路徑錯誤：generate_nirRG_png.py 指向 /mnt/backup/ 的舊 60m VRT（1830×1830），導致 overlay 只有左上角有畫面（僅 2.8% 覆蓋）；已改指 /home/alanyh/ 的 10m VRT（10980×10980）
## [2026-05-22] fix | 刪除舊 66 個 NIR-R-G PNG，重新生成（3h9min）；新 PNG 為 10980×10980，93.7% 非黑像素，overlay 正確覆蓋全圖
## [2026-05-22] bug | 發現 reconstruct_module.py 缺少 /10000.0：VRT 讀出 uint16 raw 值（0–10000），未除以 10000 直接進 preprocess_patch，clip(0,10) 把所有值壓到 10，normalize 後得 ≈76（遠超分佈），模型全預測背景 → Oil IoU=0.0000
## [2026-05-22] fix | reconstruct_module.py line 315：加入 / 10000.0，與訓練時 _get_pos_vrt_item 一致
## [2026-05-22] bug | Fold 1 第一輪重組（66 個場景）仍全部 IoU=0：訓練 process 於 05:08 啟動時已載入舊版 reconstruct_module.py，17:04 的修正未被載入（Python 不重載已 import 的 module）
## [2026-05-23] verify | 快速重組測試（3 場景，使用 test_recon_quick.py）驗證 /10000 修正有效：Oil IoU 從 0 變非零（0.0007–0.0056）；Oil Recall=87.1%（模型確實找到油汙）；Oil Precision 極低因全圖油汙佔比僅 0.0015%，假陽性易累積
## [2026-05-23] perf | 實作 AMP（Automatic Mixed Precision）：修改 deeplab_adapter.py 5 處（GradScaler init、train forward+backward、accum step、epoch flush、val loop autocast）；YAML 新增 use_amp: true
## [2026-05-23] fix | 修正 experiments_CV.yaml 輸出路徑：results_base_dir 與 excel_log_path 改為絕對路徑，消除相對路徑導致資料夾名稱重複一層的問題
## [2026-05-23] experiment | 重啟 5-fold CV 訓練（AMP + num_folds=5）：log → train_log/train_5fold.log；Fold 1 Epoch 26 已達 Best mIoU=0.5926（Oil IoU=0.212），略優於上一輪

## [2026-05-24] perf | 診斷重組速度瓶頸：GDAL deflate 解壓縮為 CPU 單執行緒瓶頸，8-band 全場景 VRT 讀取耗時 166s；原設定 infer_batch_size=8 + stride=192 預估 9.7 小時/fold
## [2026-05-24] perf | reconstruct_module.py 新增 _read_vrt_parallel：解析 VRT XML → ThreadPoolExecutor(8) 並行讀 8 個 band TIF → cv2.resize 升採樣；band 讀取 166s → 39s（4.2×）
## [2026-05-24] perf | reconstruct_module.py 加入 torch.autocast('cuda') 包裹推論迴圈（fp16 inference）；preprocessing executor 跨 batch 重用（消除每 batch 重建 executor 開銷）
## [2026-05-24] perf | reconstruct_module.py 加入 _save_pool（async imwrite）與 bg PNG prefetch：NAS imread（40s）提交於推論前，儲存（26s）非同步執行；實際每場景 ~6 min
## [2026-05-24] config | experiments_CV.yaml reconstruction 區塊：infer_batch_size 8→64、stride 192→256（patch 數 3249→1849）、num_workers 2→8
## [2026-05-24] feat | main_runner.py 新增 _FoldTee 類別：stdout 同時 tee 至 fold 專屬 log 檔（fold_log_dir/YYYYMMDD_fold_N.log）；experiments_CV.yaml 新增 fold_log_dir 欄位
## [2026-05-24] feat | experiments_CV.yaml 新增 start_fold: 2；main_runner.py CV loop 改為 range(start_fold, num_folds+1)，允許從指定 fold 繼續
## [2026-05-24] bug | 發現 deeplab_adapter.py _get_coord_vrt_item（背景樣本路徑）line 267 缺少 /10000.0，與正樣本路徑不一致；已修正
## [2026-05-24] experiment | Fold 1/5 完成（Best mIoU=0.593，Oil IoU=0.212，Epoch 26）；Fold 2–5 以 start_fold=2 重啟（含 BG bug 修正 + 重組速度優化）

## [2026-05-25] doc | 新增 wiki/experiments/20260525_架構演進與差異彙整.md：彙整 5/20–5/25 所有改動，對照資料格式（11→8 band、strip→COG、VRT Bug）、前處理 pipeline、訓練設定、重組速度優化、Bug 修正共 6 項
## [2026-05-25] doc | 更新 wiki/models/DeepLabV3+.md：in_channels 11→8（B01/02/03/04/08/8A/11/12）、class_weights [30.0,1.0]→[13.0,1.0]（inverse frequency，Oil:BG≈1:13.8）；更新 index.md 對應描述
## [2026-05-25] doc | 新增 wiki/pipeline/add_new_data.md：新增 Sen2Like 資料的完整流程（目錄結構規範、Step 1 convert_to_cog.py、Step 2 build_vrt_ms6.py、Step 3 fold split + patch coords 重算、Step 4 NIR-R-G PNG）；更新 index.md
## [2026-05-25] doc | 擴充 wiki/concepts/cloud_optimized_geotiff.md：加入 Strip vs COG 磁碟排列 ASCII 圖解、index 粒度差異說明、與 VRT 的關係、實測數字表
## [2026-05-25] doc | 新增 wiki/concepts/acolite_vs_sen2like.md：Acolite vs Sen2Like 完整格式比較（像素單位、波段數、解析度、大氣校正、標注對齊、pipeline 差異）；更新 index.md

## [2026-05-26] preprocess | 新建 3-fold all-220 實驗前置準備
## [2026-05-26] preprocess | build_scene_splits_3fold_all220.py：全 220 場景打散 3-fold；Fold1:train=106,val=39,test=75 / Fold2:train=113,val=30,test=77 / Fold3:train=121,val=31,test=68；三個 test set 不重疊合集=220
## [2026-05-26] preprocess | generate_patch_coords_3fold.py：GB1.5 patch 座標生成；Fold1 train=2333 / Fold2=2337 / Fold3=2715；output→3fold_all220/patch_level_GB1.5/
## [2026-05-26] preprocess | generate_nirRG_png_all220.py：8-worker 並行 JPEG 生成全 220 場景；output→NIR_R_G_Output_png/all_ms6/（220 jpg + 66 png）
## [2026-05-26] fix | reconstruct_module.py line 356：fuzzy_find_file 改為先找 .jpg 再 fallback .png，支援新 all_ms6/ JPEG 底圖
## [2026-05-26] config | 新建 main/experiments_3fold_all220.yaml：num_folds=3, GB1.5, split_dir→3fold_all220, vis_image_dir→all_ms6

## [2026-05-27] training | 5-fold MS6 GB1.0 CV 全部完成（fold1-5 best.pt 就緒）；3-fold all-220 GB1.5 訓練完成；兩組結果均可作為 HNM baseline 比較
## [2026-05-27] HNM | Cross-Fold HNM step2_per_fold_mine_fp.py 卡死 bug 診斷：原實作一次讀取整場景背景 bounding box（最大 3.86 GB NAS），對 NAS 等同掛住；改為 mask-first + 8-worker prefetch pipeline（reader thread 與 GPU inference 重疊）
## [2026-05-27] fix | Cross-Fold HNM step2 卡死修正完成：平均 ~48s/scene（原 ~290s），全 220 scenes ≈ 2.9 小時
## [2026-05-28] HNM | step2 執行完成：45,848 FP patch candidates，220/220 場景全數處理，4 個場景 n_eligible=0（mask 條件正常過濾），輸出→HNM/step2_output/cfhnm_mined_all220.txt
## [2026-05-28] fix | step3_spectral_safety_filter.py：修正 6 處 cand_df["fold"] → cand_df["source_model_fold"]（欄位名稱不符，原本會 KeyError crash）
## [2026-05-28] HNM | step3 spectral safety filter 開始執行（CPU bound，預計 30–60 min）；輸出 dmin_distribution.png + decision_summary.txt 後由使用者決定 cutoff
## [2026-05-28] doc | 新增 wiki/experiments/20260527_CrossFoldHNM_執行紀錄.md（step2 卡死修正、prefetch pipeline 設計、step2 結果、step3 bug 修正）
