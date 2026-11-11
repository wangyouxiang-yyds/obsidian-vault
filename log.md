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

## [2026-05-29] HNM | step3 完成：45,848 candidates，valid d_min=45,673；分布 p10=0.152 / p50=0.773 / max=21.3；決定採 p10 截點（移除最像真油的 10%）
## [2026-05-29] HNM | step4 完成：d_min < 0.1516 截除後剩 ~41,280 candidates；組裝 patch_level_GB1.0_cfhnm；train 加入 HN：fold1=466, fold2=467, fold3=543；最終比例 1:1.5
## [2026-05-29] training | 啟動 3-fold cfhnm 訓練：experiments_3fold_all220_cfhnm.yaml；結果→result-seg/CV_3fold_all220_cfhnm/；與 GB1.5 組比較 HNM 效益
## [2026-05-29] doc | 更新 wiki/experiments/20260527_CrossFoldHNM_執行紀錄.md：補入 step3 d_min 分布、step4 assembly 結果、訓練對比設計表
## [2026-05-30] doc | 新增 wiki/datasets/dataset_split_strategy.md（資料集切割策略規劃：定義場景層級 Scene-level 切割原則，規劃「完全隨機 K-Fold」、「特定事件留出」、「時序預測」三種實驗策略以驗證泛化能力）

## [2026-06-03] research | 文獻調查：Unsupervised 油汙偵測方法（RX/iForest/OSI/VAE）；確認 Duan 2022（HOSD+iForest）為最重要對標；規劃兩階段 pipeline（Unsupervised 粗定位 → DeepLabV3+ 精分割）
## [2026-06-03] experiment | HOSD 高光譜基準測試（GM03）：iForest 單步 F1=0.78 / AUC=0.9899（224 波段），確認 iForest 可行性
## [2026-06-03] experiment | S2 多光譜 iForest + OSI 批次測試（5 個 NOAA Gulf of Mexico 場景）：AUC 0.31~0.95，F1 偏低（Precision 問題）；實作雲遮罩（NDWI+B02）、F1 最佳閾值、空間後處理改善中
## [2026-06-03] doc | 新增 wiki/experiments/20260603_Unsupervised油汙偵測初探.md（完整實驗紀錄）
## [2026-06-03] doc | 新增 wiki/papers/Duan2022_HOSD_IsolationForest.md（論文摘要 + 本專案實測對比）
## [2026-06-03] doc | 新增 wiki/concepts/unsupervised_oil_detection.md（方法總覽、實施路線、文獻缺口）（資料集切割策略規劃：定義場景層級 Scene-level 切割原則，規劃「完全隨機 K-Fold」、「特定事件留出」、「時序預測」三種實驗策略以驗證泛化能力）

## [2026-06-05] fix | 補齊 183 個 NOAA 場景 JSON+Mask：check_dataset_completeness.py --fix-json + json_to_mask_tif.py；NOAA 可用 220→403，總場景 256→439
## [2026-06-05] cleanup | 刪除廢棄目錄：processed/（1.1TB）、JSON/、JSON_ms6/、175 個舊格式 mask TIF
## [2026-06-05] fix | 重建 JSON flat 目錄 MS6_sen2like_JSON/（439 個，以場景資料夾名命名）；更新三個 yaml json_dir 指向新目錄
## [2026-06-05] preprocess | 三策略 scene split 重跑（439 全場景）；patch coords GB1.5 三策略全部重生成
## [2026-06-05] doc | 新增 wiki/experiments/20260605_資料集修正與三策略重跑.md
## [2026-06-17] concept | 新增 wiki/concepts/iforest_架構與運作機制.md（Q&A 學習紀錄：樹結構本質、建樹過程含 8 像素例子、Duan 2022 GM01 完整 10 步驟、訓測同源 vs 資料外洩釐清、contamination 失敗點）
## [2026-06-17] concept | 新增 wiki/concepts/deeprx_vae_架構與運作機制.md（Q&A 學習紀錄：VAE 架構（reparam trick / KL 約束）、為何 VAE+RX、San Diego I 完整流程、訓測同源爭議的誠實討論含論文評估方式的批判、Background-Only Training 洞察）
## [2026-06-18] concept | 更新 wiki/concepts/deeprx_vae_架構與運作機制.md：追加第 8-12 章（論文 vs 真實場景複雜度差異 / v9 完整 12 步驟流程 / 跨影像不共用 VAE 的原因 / v3 濫抓四大根本原因與 v9 解法 / 投影片報告用三大 Takeaways）；更新 index.md 對應描述
## [2026-06-24] pipeline | 新增 wiki/pipeline/VRT_pipeline_01_前處理.md（完整前處理 pipeline：VRT 建置、GT Mask、GB1.5 patch 切分、3-fold 策略、HNM、NIR-R-G 背景圖；439 scenes / 8-band 最新狀態）
## [2026-06-24] pipeline | 新增 wiki/pipeline/VRT_pipeline_02_模型訓練.md（訓練 pipeline：DeepLabV3+ ResNet-50 from-scratch、FocalLoss(alpha=0.25,gamma=2)、class_weights=[13,1]、EMA decay=0.9997、checkpoint/resume、Docker v6 環境）
## [2026-06-24] pipeline | 新增 wiki/pipeline/VRT_pipeline_03_重組評估.md（重組評估 pipeline：全場景 sliding window TTA、Cloud Mask 後處理、annot-only/JSON 雙 GT 指標、prevalence 三組對照、reconstruct_v2 效能優化、fold2 Oil IoU=39.80%）
## [2026-06-24] index | 更新 index.md：新增三份 VRT pipeline 系列文件的索引條目

## [2026-07-02] pipeline | 新增 wiki/pipeline/OIL_PROJECT_MutiBand_0422_VRT_training.md（0422 主線入口索引，解決 TODO/README 中的斷鏈 [[OIL_PROJECT_MutiBand_0422_VRT_training]]）
## [2026-07-02] pipeline | 新增 wiki/pipeline/GT_expand_pipeline.md（GT_expand fork 版本完整 pipeline 說明：GT-centric patch 策略、與 0422 策略差異表、繼承 bug C1~C4、不繼承 A4/class_weights/TTA 分析）
## [2026-07-02] experiment | 新增 wiki/experiments/20260702_CV_358clean_gt_expand_進行中.md（當前 CV 實驗進度：fold1/2 完成、fold3 進行中 epoch 43、TODO 三份待動工、已知繼承 bug C2/C3/C4 對結果的影響說明）
## [2026-07-02] index | 更新 index.md：新增上述三份筆記的索引條目
## [2026-07-02] update | 修正 GT_expand_pipeline.md + 20260702_CV_358clean_gt_expand_進行中.md：依 code 驗證實況更新 TODO 狀態（C1~C7 幾乎全已修、class_weights→[1,1]、patience→25、A1 pretrained backbone 已啟用；C4 刻意 revert；02 公平比較協議部分待動工）；加註 repo TODO markdown 文件已過時
## [2026-07-02] ingest | 閱讀論文：Zakzouk et al. 2024《Novel oil spill indices for sentinel-2 imagery》，詳細總結 method details（859 波段組合窮舉、JM 距離、Index C/D 篩選流程）；判定核心相關（直接命中 Sentinel-2 油汙光譜指數，非邊緣相關）
## [2026-07-02] experiment | 新增 wiki/experiments/20260702_波段選擇消融實驗規劃.md：規劃 Baseline(8-band)／Band Subset(6-band)／Index C／Index D 四組消融實驗；確認現有 MS6_sen2like 資料集為 8 波段（非 CLAUDE.md 記載的 11 波段），解決波段對應疑點；列出 4 項待 server 確認事項

## [2026-07-10] update | 對照 B repo（OIL_PROJECT_MutiBand_GT_expand）逐行查證更新 5 篇筆記，反映分支 A/B 現況：GT_expand_pipeline.md 大改版（雙 gate 評估協議 pooled_oil_iou>0.362+Wilcoxon、背景 patch 機制、bbox>256 256-grid 鋪磚、3_fold_stratified_v2 split、cw31/tversky 實驗線、patch 數修正 2897→2893）；VRT_pipeline_02_模型訓練.md 修正三處硬錯（EMA decay=0.0 未修 bug、best.pt 依 val_miou 非 Oil IoU、FocalLoss 無 alpha 參數）並補分支 B 差異（weight_decay/augmentation/mean-std/執行環境）；VRT_pipeline_01_前處理.md 修正 Mask 路徑、patch 座標檔名編碼格式、VRT 雙路徑並存註記；ms6_sen2like.md 修正 JSON 路徑、uint16 DN vs /10000 轉換時機、COG 未建 overviews 澄清；dataset_split_strategy.md 補策略四（3_fold_stratified_v2）
## [2026-07-10] preprocess | Dataset manifest 制度上線：`build_manifest.py` 掃描產出單一真相帳本 `manifest.csv`（445 場景，issues=0，dirty 85）；修正先前誤報「菲律賓場景缺 JSON」——JSON 兩處都查（場景資料夾優先），實際 163 只在集中目錄、6 只在場景資料夾（含 5 張新場景）、276 兩處都有
## [2026-07-10] ingest | 5 張 2026 NOAA Atlantic（19TDE）場景入庫：20260309_S2/20260329_S2/20260414_L9/20260425_S2/20260508_L8，gpkg→mask→VRT 全部驗證通過（mask=VRT=10980×10980，值僅 0/1/255，oil_px=13663/345/3494/2286/1884），尚未加入任何 split
## [2026-07-10] doc | 更新 wiki/datasets/ms6_sen2like.md（新增第 6/7 節：manifest 制度、2026 新場景入庫記錄；修正 JSON 雙位置描述）、wiki/pipeline/VRT_pipeline_01_前處理.md（場景數 439→445、新增步驟 G manifest 章節、VRT/JSON 路徑表更新）、wiki/pipeline/GT_expand_pipeline.md（補 patch 座標腳本 SPLIT_DIR/OUT_DIR 環境變數硬化說明）
## [2026-07-10] infra | 建立 Claude Code × Codex 雙 harness 共用制度骨架：新增 .agents/README.md（正本索引）、.agents/dispatch_to_gemini.md（Gemini 降級為雜工角色定義）、.agents/agents/paper-extractor.md（subagent 規格正本）；將 AGENTS.md 由全文複製 CLAUDE.md 改寫為薄索引，修正其中過時的 Linux git sync 路徑；把家目錄 ~/.agents/skills/ingest-paper 併入專案 .agents/skills/ 作為正本位置（核對後三處內容原本即逐字相同，無實際漂移）。任務書 C/D/E（模型調度守則、判斷力外化、prompt 範本）本輪未展開，列在 .agents/README.md 待辦。

## [2026-07-11] doc | 新增 outputs/20260711_AI協作Harness現況快照.md（狀態快照：雙 harness+institution 架構、研究現況分支A/B與tversky/cw31數字、v3plus訓練進行中、Claude端subagent清單、Claude/agy/codex模型角色分工）；更新 index.md 分析輸出區塊

## [2026-07-14] ingest | 論文：SSL4EO-S12（Wang et al. 2023）——全球多模態多時相 Sentinel-1/2 SSL 預訓練資料集，四法（MoCo/DINO/MAE/data2vec）× 兩種 backbone benchmark；確認「SSL/foundation model 尚未應用於光學海面油汙偵測」研究缺口；新增 wiki/concepts/self_supervised_pretraining_遙測.md；順帶補齊 index.md 文獻摘要區塊遺漏的兩筆既有條目（Song_DeepRX、柯弈仲2008）

## [2026-07-14] ingest | 論文：Tversky loss for image segmentation（Salehi et al. 2017，arXiv:1706.05721）——以 α/β 取代 Dice 對稱權重直接控制 FP/FN 懲罰，3D U-Net MS 病灶分割 α=0.3/β=0.7 全面優於 Dice loss（α=β=0.5）；新增 wiki/concepts/分割損失函數與類別不平衡.md；**重要交叉確認**：此文獻最佳超參數與 GT_expand 分支現行 tversky 實驗線設定完全一致，該實驗線已實測勝出（pooled oil IoU=0.3915，唯一通過雙 gate，優於 cw31），已回頭補記於 wiki/pipeline/GT_expand_pipeline.md 第三-c 節

## [2026-07-24] experiment | 新增 wiki/experiments/20260724_Focal_Tversky小圖實驗可行性評估.md：記錄 Claude×Codex 對 Focal Tversky 的條件式結論、梯度交點、前置／fold1／三折 gate 與實作驗收；FTL 尚未啟動；同步更新 index.md

## [2026-07-24] doc | 補記 2026-07-16~07-24 缺口：新增 wiki/experiments/20260716_資料溯源洩漏與評估合約v1.md（12 事件群溯源稽核、350/355 姊妹景近重複、external-84 假設作廢、Evaluation Contract v1.0 取代舊雙 gate、grouped replay 結果、非對稱閘控雙軌 backbone 政策）
## [2026-07-24] doc | 新增 wiki/experiments/20260718_訓練不可重現根因與決定性修復.md（Albumentations 2.0.8 Compose 內部 RNG 未鎖致訓練不可重現，commit ba25391 修復並經 reproC/D 逐 bit 驗收；此前所有 grid/screen 掃描降 legacy、Dice 勝 Tversky 結論撤回；決定性正本重跑中）
## [2026-07-24] update | 更新 wiki/pipeline/GT_expand_pipeline.md：第三-c節補 tversky 完成數字＋修復前 caveat、第五節新增「五-a Evaluation Contract v1.0」小節取代舊雙 gate 描述、相關頁面補三個新連結
## [2026-07-24] update | 更新 wiki/datasets/dataset_split_strategy.md：核心原則小節加註 2026-07-16 校正、文末新增「資料溯源洩漏補充」章節
## [2026-07-24] index | 更新 index.md：新增兩篇新筆記索引條目，修訂 GT_expand_pipeline.md／dataset_split_strategy.md 描述反映 2026-07-24 更新

## [2026-07-24] review | Codex 獨立審查 vault，指出失效主張未就地標記＋freshness 抬頭過期＋GT_expand_pipeline.md 重複段落；新增 .agents/decisions/20260724_Codex審查與決定性重跑優先序.md 記錄 STANCE=CONDITIONAL GO、三個最優先動作、四個盲點、制度規則建議
## [2026-07-24] fix | 全 vault 搜尋 Wilcoxon／355／pooled／external-84／OOD／泛化／0.029／0.5-0.5 關鍵詞，逐處就地加失效標記（不改寫原文）：wiki/experiments/20260702_波段選擇消融實驗規劃.md、wiki/datasets/dataset_split_strategy.md、outputs/20260711_AI協作Harness現況快照.md、wiki/concepts/分割損失函數與類別不平衡.md、wiki/papers/Salehi2017_TverskyLoss.md
## [2026-07-24] fix | 合併 wiki/pipeline/GT_expand_pipeline.md 第三-c節雙 checkout 各自補的 tversky 重複段落為一段連貫敘述（保留兩邊實質內容：caveat＋Evaluation Contract 連結＋文獻交叉引用）
## [2026-07-24] fix | 修正 freshness 失真：.agents/handoff.md STATUS_AS_OF 由 2026-07-13 校正為 2026-07-24 並改寫狀態摘要；wiki/experiments/20260702_CV_358clean_gt_expand_進行中.md frontmatter status 由 running 校正為 completed 並加 freshness 說明

## [2026-08-05] fix | 補記 2026-07-28~07-31 缺口：更新 .agents/handoff.md「目前狀態」節，STATUS_AS_OF 由 07-24 校正為 08-05；記錄 P2 FTL 判定 null（保留 Tversky）、P3 ASPP-rate 判定 negative（保留 6,12,18）兩案結案、Prithvi+DeepLabV3-ASPP 決定性基線更新為 0.4399（取代舊 pre-fix 0.4167）、現況無訓練在跑（GPU 閒置）、result-seg 08-03 清理誤刪 P3 prereg 應保留的失敗殘骸（制度教訓）、/mnt/oil 新掛載、GT_expand HEAD 仍 f36e421 且 P3 產出未 commit
## [2026-08-05] experiment | 新增 wiki/experiments/20260731_ASPP_rate適配實驗.md：記錄 Prithvi 16×16 網格 ASPP atrous rate 適配實驗（候選 2,4,6 vs 決定性 6,12,18），grouped replay 判定「無證據」保留現行 rates，機制解讀為 global-pool+補零 dilation 卷積仍提供有效長距 context、非分支塌陷；副產品 dlv3_det 決定性基線 0.4399；誠實揭露候選 fold3 因系統重開機中斷、依 prereg 合法保留失敗殘骸、但殘骸事後於 08-03 清理中被誤刪
## [2026-08-05] update | 就地更新 wiki/experiments/20260724_Focal_Tversky小圖實驗可行性評估.md：frontmatter status 由 planned 改 completed，狀態列標示已結案；文末新增「8. 判定結果」章節記錄 2026-07-27~07-28 執行與判定（mean pooled 0.3859、grouped ΔS=+0.0011 CI 跨 0，保留 Tversky，loss 分支枯竭）；查核 yaml/code 發現實際執行的 07-27 prereg 凍結候選為 q=4/3（direct exponent, super-linear），與本篇第 2–3 節（07-24）原討論的 q=0.75（sub-linear）方向相反，已於新章節如實記錄此落差，不改動原第 1–7 節
## [2026-08-05] index | 更新 index.md：新增 P3 ASPP 筆記索引條目，修訂 FTL 筆記描述反映已結案判定結果
## [2026-08-05] fix | 追加更新 .agents/handoff.md「下一步」項：使用者裁決 /mnt/oil 只是 share folder（非新研究軸，SAR/UAV 空目錄為分類佔位）、Prithvi+UPerNet 決定性 3-fold 暫不排跑（降為已知 caveat，引用 0.4498 須註明 pre-fix legacy 單跑）；序列化共識案裁決仍待定；補記新增 Stop hook `/root/.claude/hooks/check_writeback_staleness.sh`（session 結束比對 result-seg 最新 run mtime vs institution 現況檔 mtime，自動化 30_maintenance.md §4 季檢第 4 條，僅 Claude 端有，Codex 接手需人工自查）
## [2026-08-05] experiment | 新增 wiki/experiments/20260805_巨型場景診斷與P4操作點實驗.md：既有 artifact 歸因分析（pooled 分數由最大 10 景=44.4% 油污像素主導、fold3 pooled 崩塌純屬巨型場景效應非整折變差）、JM 距離方法論教訓（高 JM 與 GT 外包絡吻合而非矛盾，term_cov 拆項糾正了「訊號在、是模型問題」的錯誤結論）、正典三折從未存過 pixel-exact 預測的資料保存發現；記錄由此催生的 P4 操作點（決策閾值）實驗設計、codex 審查 VERDICT REVISE 8 條 BLOCKING、以及卡住 prereg 凍結的使用者裁決點（pooled vs S 評成敗）；同步更新 index.md
## [2026-08-05] update | 第二次更新 .agents/handoff.md「目前狀態」：09:40 啟動的 mask/prediction-artifact JSON 序列化 schema 共識案已收斂，其輸出格式已被 P4 操作點實驗採用；記錄背景推論 job `analysis/p4_operating_point/dump_prob.py --all`（獨立 session leader，不受 /clear 影響，約 4 小時）；記錄 codex 對 P4 prereg draft 的 VERDICT REVISE 8 BLOCKING 重點三條；標記 🔴 卡住的使用者裁決（pooled vs S 評成敗）阻擋 prereg 凍結；更新 Git 狀態新增 analysis/p4_operating_point/ 與 recon_gt_aware_module.py 未 commit 項目
## [2026-08-06] experiment | 新增 wiki/experiments/20260806_P4操作點決策閾值_實質null.md：P4 結案，prereg v1.0 四輪收斂凍結（ACCEPT/BLOCKING=[]）；使用者裁決計畫任務以 pooled Oil IoU 評成敗（解除卡住的 BLOCKING #8）；判定 Δpooled=−0.0011 CI[−0.0067,+0.0034]＝實質 null，決策閾值槓桿正式關閉；H2a「模型過度保守」假說被推翻（GT-positive 像素可救回機率質量僅 2.1%，模型是有信心地判錯非保守）；estimand 標籤更正（招牌數字 0.4399＝逐折 pooled 再平均，非全語料 pooled 0.4335，兩者不同估計量）；§7 盲化標註稽核證實兩大巨景 GT 品質尚可、Pacific 20211005 一景輸入端無動態範圍；記錄兩則撤回聲明（「299/355 景逐位元相同」誤稱撤回、判準量綱教訓）；下一步槓桿轉向方向 C（重訓：面積加權/上限採樣、多層 tap 餵 ASPP，須另立 prereg）
## [2026-08-06] index | 更新 index.md：新增 P4 結案筆記索引條目
## [2026-08-06] update | 更新 .agents/handoff.md「目前狀態」：P4 操作點實驗由「進行中」改為「已結案」，記錄結案結論摘要與 estimand 標籤更正；「下一步」新增剩餘槓桿=方向 C（面積加權/上限採樣重訓 或 多層 tap 餵 ASPP，兩者皆須重訓+另立 prereg）為下一棒接手點

## [2026-08-21] ingest | 論文：CBF-CNN（Du et al. 2022）——HY-1C VIS+NIR 與 class-balanced loss；確認多區域實驗仍使用各區域當地標註訓練，不是 source-only target-blind transfer
## [2026-08-21] ingest | 論文：中解析度光學油污分割（Sun et al. 2024）——S2/L8/L9 共同六波段、全球十事件與 sun-glint feature；目前最接近 Sen2Like，但未證明台灣封存 zero-shot
## [2026-08-21] ingest | 論文：PlanetScope CNN/Transformer（Kang et al. 2024）——Swin-UPerNet 在 random patch CV 優於 DeepLabV3+；不能外推跨事件／跨海域能力
## [2026-08-21] ingest | 論文：中國海多感測器油污製圖（Wang et al. 2024）——七感測器 operational mapping；完整 bands、split 與模型指標標待全文補完
## [2026-08-21] ingest | 論文：MS3OSD（Du et al. 2025）——HY-1C/D UV–VIS–NIR 融合、四海域與 sunglint 分層；未證明 leave-one-region-out
## [2026-08-21] ingest | 論文：成大光學 U-Net 碩論（周芷蘭 2021）——確認正式題名沒有 Transfer Learning，摘要僅支持 RGB supervised U-Net 台灣先例
## [2026-08-21] concept | 新增 wiki/concepts/跨海域_source-only_zero-shot油污偵測.md；定義台灣 target-blind protocol、第一輪六篇文獻矩陣與建議閱讀順序；更新 acolite_vs_sen2like.md 波段對照

## [2026-08-22] ingest | 論文初讀：Chang et al. 2024《Marine Oil Pollution Monitoring Based on a Morphological Attention U-Net Using SAR Images》——完成 Abstract／Introduction 拆解；主 gap 是 SAR mask 碎裂與孔洞，台灣事件只可先標 application verification，是否為海外 source-only→台灣 target-blind 待 Section 2–3 判定

## [2026-08-22] update | Chang et al. 2024 Methods 統整：extended MKLab 1239張 Sentinel-1 VV／五類，FA-MobileUNet=MobileNetV3+CBAM+ASPP+full-scale aggregation，Stages 1–2 以 learnable morphological closing attention 取代 SAM，label smoothing α=0.1；判定最值得借的是嚴格 source/target protocol、early shape-aware feature 與 explicit lookalike，label smoothing 不宜當主要不平衡解法

## [2026-08-25] cleanup | 全場景 OOF baseline v1 產物經使用者裁決刪除：使用者以「主軸已於 2026-08-12 轉為海外 source-only→台灣 sealed 的小圖 patch 路線、全場景這條路未來不會再走」為由，裁決刪除 repo 外的 `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_Alan_outputs/`（203 MB），並明示只留一份紀錄、不保留二進位產物。刪除前已寫成自足決策紀錄 `OIL_PROJECT_MutiBand_Alan/docs/decisions/20260825_fullscene_oof_v1_退役與刪除.md`。被刪內容＝`alan_prithvi300m_dlv3_fullscene_oof_v1`：355 景（fold 119/118/118，三份 manifest 皆 complete: true、failed/skipped 皆 0）10980×10980 全場景 OOF 推論，每景 120,560,400 px、coverage mode=full，感測器 S2 283／L8 51／L9 21；執行 2026-08-08T09:53Z→08-11T06:32Z，合計 247,027 秒 ≈ 68.6 GPU-h；1,443 個檔案中 181 M 是兩份代表景 float16 機率 npz。關鍵事實：run_metrics.json／per_scene_metrics_v1_2.csv／threshold_curve_v1_2.csv 從未生成，這 68.6 GPU-h 從未產出任何可報告指標，刪除不推翻任何已成立結論。權重（三 fold best.pt）、split（3_fold_stratified_v2/scene_level/test_fold*.txt）、程式碼皆未刪，技術上可重建但須重新 finalize launch gate 並從第 1 景重算。2026-08-06 條目所引的 launch_gate 工件路徑即日起不存在，其關鍵數值（18 個閾值 tie 表、0.5 平手 44,149、gate checks 全過、決定性重跑 changed_px=0）已逐項抄錄於決策紀錄 §3。本次裁決作廢兩份相反的既有指示：archive/fullscene_v1_2/README.md 的「Do not delete the external 203 MB output」與 docs/OUTPUTS.md §6 遷移計畫（前者已加註 SUPERSEDED banner）。institution 正本 oil_spill_project_status.md 已同步追加同內容條目。

## [2026-08-26] ingest | 論文：N-FINDR + SAM 小型油污偵測（Lee et al. 2025）——核對 Sentinel-2 masks→N-FINDR（p=2）→SAM（τ=0.1）管線；判定最可借用為 nuisance masks 與 source-prototype SAM verifier，92.03%／F1=0.9585 僅屬同景 RGB proxy 評估，不能證明跨事件低誤警

## [2026-08-31] ingest | 論文：Prithvi-EO-2.0（Szwarcman et al. 2024，arXiv:2412.02732v3）——新增 wiki/papers/Szwarcman2026_PrithviEO2.md。IBM/NASA 第二代地理空間基礎模型：HLS（30m/6 波段）4.2M 時序樣本 MAE 自監督預訓練，新增 3D 時空 patch/positional embedding 與可選的 lat-lon + year/DOY metadata bias（訓練時 10% 隨機丟棄），300M（ViT-L）/600M（ViT-H）兩規模，400 epochs、21k/58k A100 GPU-hr。GEO-Bench 12 任務較前代平均 +8%；Sen1Floods11 water IoU 79.6→83.1、burn scar IoU 76.8→83.2；小樣本情境優勢最大（Landslide4Sense 僅 50 張訓練圖，600M mIoU 67.0% vs U-Net 59.7%）；已知弱點＝多類別 Burn Intensity mIoU 僅 31.2%、SAR 波段作為 encoder 額外輸入未提升（作者建議改用外部 SAR encoder 融合）。**與本專案的直接關聯**：此即現行固定對照組所綁的 Prithvi-EO-2.0-300M encoder 的來源論文，先前 wiki 只在 self_supervised_pretraining_遙測.md 以「研究缺口點名」的形式提及，未有本體摘要頁。本頁同時記錄一項假設殺手：預訓練取樣階段刻意降採樣 Sea 與同質化水體（Fig.1 中 Sea 僅佔訓練樣本 2%），故其表徵未必善於區辨「開闊水面上的細微膜層紋理」；Sen1Floods11 是「哪裡是水」而非「水上有無異常膜層」，不構成反證。若該假設成立，全模型 fine-tune 的效益可能被高估，建議先以低成本 linear probing 驗證。
## [2026-08-31] index | 更新 index.md 文獻摘要區塊：新增 Szwarcman2026_PrithviEO2.md 條目
## [2026-08-31] ingest | raw/papers/ 收錄 Liu et al.《Seeing Beyond the Patch: Scale-Adaptive Semantic Segmentation of High-resolution Remote Sensing Imagery》（ICCV 2023）PDF，**尚未閱讀、尚未建立 wiki/papers/ 摘要頁**，此處僅登錄檔案入庫，不得視為已 ingest
## [2026-08-31] doc | 更新 .agents/handoff.md：未償債務 #5（vault log.md 08-25 刪除紀錄未 commit）標記為已結（該紀錄已於 commit 1d765fa／e982f07 進入 master 並推送），並補記本次 08-31 vault 文件同步；本次僅動 vault 文件層，研究狀態（GPU 閒置、SAPP 探索實驗 claim 上限）未變

## [2026-08-31] experiment | 週結：背景 patch 配額三部曲 P7→P8→P9。新增 wiki/experiments/20260831_P9油背像素比例bracket_inconclusive與凹性發現.md（訓練語料油汙:背景像素比例 ρ 首次敏感度檢驗，ρ∈{15, 23.0187, 30} 不等距 bracket、六道建構閘全過、29.6 GPU-h；主判定 inconclusive，S₃₀−S₂₃=−0.0201 與曲率 C=+0.0174 兩項達統計顯著未達 ±0.02 實質門檻，反應面呈凹形；現行 ρ=23.02 三水位中 S 最高但因 protocol continuity 暫留、不得稱最優）；新增 wiki/experiments/20260831_週結_背景patch配額三部曲P7P8P9.md（時間軸整合 + 三個方法論教訓）；更新 wiki/experiments/20260827_P7背景patch配額政策實驗_r12_null與機制發現.md（§8/§10 標記兩階段分配方案與 P8 已全線砍除、§9 已關閉槓桿總表新增 P9 一列、新增 §12 逐 size 四格拆解回顧）；更新 .agents/handoff.md（P9 已結案 inconclusive、現行 ρ 暫留、待辦論文段落須改寫與不得追加水位/seed）

## [2026-08-31] ingest | 論文：GeoAgent（Liu et al., ICCV 2023）——完成 PDF 全文研讀與九步分析，建立 `wiki/papers/Liu2023_GeoAgent_尺度自適應分割.md` 與 `wiki/concepts/尺度自適應_Global-Local分割.md`；釐清 RL 只選 context 尺度、分割與 reward 仍需 GT，並提出本專案 `10m local + 20/30m context + shared Prithvi` 尺度對齊假說。先前同日 raw-only 登錄至此升級為完整 ingest；未啟動實驗。

## [2026-09-01] decision | 新增 `.agents/decisions/20260901_P10油背比例細格研究設計.md`：P10 油背像素比例細格研究設計成立，6 點格 ρ∈{15,17,19,21,23.0187,30}（15/23.0187/30 沿用 P9 不重跑），本批次三臂 rho17/19/21 背景座標已建構並通過七道建構閘（連續性/單調性/純前綴/中臂身分閘/集合包含/比例複核/P9 舊臂磁碟一致性），三個新 config 與預註冊分析、詮釋閘腳本均已就緒；使用者在被完整告知功效不足（頂點增益上限僅 +0.0020，小一個量級於單一對比 sim CI 半寬 ±0.0139）後仍明示重申要求執行；**尚未啟動訓練，待使用者另行授權啟動**

## [2026-09-01] ingest | raw/papers/ 收錄兩篇 CVPR 論文 PDF，**尚未閱讀、尚未建立 wiki/papers/ 摘要頁**，此處僅登錄檔案入庫，不得視為已 ingest：(1) Astruc et al.《AnySat: One Earth Observation Model for Many Resolutions, Scales, and Modalities》(CVPR 2025)；(2) Cao et al.《CrossEarth-Gate: Fisher-Guided Adaptive Tuning Engine for Efficient Adaptation of Cross-Domain ...》(CVPR 2026，檔名於此截斷，完整標題待讀全文後補)。兩篇皆與現行主軸（跨域/跨海域 source-only 遷移、多解析度多模態 EO 基礎模型）表面相關，但相關程度未經 triage 判定。

## [2026-09-02] decision | 新增 `.agents/decisions/20260902_GeoAgent_A2C尺度配置實驗設計與建碼.md`：記錄 Codex×Claude 六輪方法協商 Round 6 ACCEPT、程式初審 REVISE 後聚焦複審 ACCEPT/NONE；P11 採 NOAA319-only development／named-event36 sealed，分為 Track A fixed local/3×/5× + supervised atom regressor/Dinkelbach 與 Track B hard-budget A2C + fixed/random/supervised/oracle；程式只寫入隔離 worktree，CPU synthetic selftest 16/16 與 py_compile 通過，pytest 未安裝故未跑；未做真資料 audit、Prithvi、GPU、training、commit 或 push。同步最小更新 `.agents/handoff.md`；institution 研究現況正本位於 vault 外，依本次僅可修改 vault 的範圍未動。

## [2026-09-03] partial-ingest | 論文：Fine-tune Smarter, Not Harder（Marti Escofet et al. 2025，arXiv:2504.17397v2）——本次僅研讀 Abstract＋Introduction，建立部分論文頁與 PEFT 概念頁；Methods／Experiments／Results／Supplementary Material 尚未研讀，效能數字、方法細節與全文結論皆待確認

## [2026-09-05] doc | 新增 `wiki/pipeline/GT_expand_workflow_onboarding.md` 新人導覽草稿；依使用者決定把原始 11-band／Sen2Like 列為外部上游，現行文件從整理後 8-band 場景開始，首輪記錄資料層級、VRT 邏輯對齊、Mask 原始語意、資料池與 355-scene 實驗語料邊界，以及 `nodata=0` 與 `Oil=0` 衝突的 metadata 風險；未修改資料、writer、訓練設定或既有 `GT_expand_pipeline.md`


## [2026-09-03] experiment | P10 油背像素比例六點格三臂訓練完成：rho17/19/21 各 3 折共 9 個 run 全數完成（run_metrics.json 齊全），起 2026-09-01 09:28、迄 2026-09-03 23:00；rho15/23.0187/30 沿用 P9 不重跑。（vault 回填於 2026-09-06，詳見 wiki/experiments/20260904_P10油背像素比例六點格_峰位辨識但增益不足.md）

## [2026-09-04] experiment | P10 主分析與詮釋閘完成：exposure_audit 判定 NOT_CENSORED（18 折皆乾淨早停，steps-to-best 跨臂差 2.19 倍，與 S 的 Spearman ρ=+0.829）；primary_results.json 六臂結果與六個對比皆通過 parity/regression 閘；E1 峰位 ρ*=19.751（bootstrap CI [18.873,21.091] 排除正典 23.0187）、E3 選擇機率 0.9255（peak_identified=True）、E2 峰增益 G=+0.0064（同時 CI [−0.00946,+0.02222] 跨 0）；預註冊換值規則六臂全不合格，判定 RETAIN_23.0187_protocol_continuity。（vault 回填於 2026-09-06）

## [2026-09-05] doc | P10 簡報圖表產出：fig_oil_iou_curve / fig_ratio_curve_S / fig_ratio_curve_pooled / P10_ratio_metrics.xlsx / fig_usable_band，論述「1:19–1:23 為可用區間」但明載兩條警語（不對未測比例作主張、S 與 pooled 口徑對區間內排序不一致）。（vault 回填於 2026-09-06）

## [2026-09-06] experiment | 補寫 P10 結案回填（訓練 09-01~03、分析 09-04、圖表 09-05，vault 當日未同步）：新增 wiki/experiments/20260904_P10油背像素比例六點格_峰位辨識但增益不足.md，更新 index.md；更新 .agents/decisions/20260901_P10油背比例細格研究設計.md 狀態列與新增 §11 結案回填；更新 .agents/handoff.md「目前狀態」P10 由「已授權待啟動」改為已結案，並記錄下一棒待裁決事項（ρ 軸是否正式結案為已耗盡的槓桿——決議 §10.2 自動觸發條件本次未成立，需另行裁決）。

## [2026-09-11] decision | 新增 `.agents/decisions/20260911_P13_patch-balanced台灣source_sensitivity實驗設計.md` 並更新 `.agents/handoff.md`：記錄與 pixel-balanced 平行的 8-arm patch-balanced 設計、配額、train-proxy 限制與固定台灣 holdout；Claude×Codex 第4回合 ACCEPT，全 arms `run:false`、未啟動。本次僅修改 vault，未碰 repo、未啟動／刪除／commit／push。

## [2026-09-20] decision-draft | 新增 `.agents/decisions/20260920_P16_GoM等patch單seed設計.md` 草稿（`DRAFT_NOT_FROZEN`），同步 index／handoff；使用者已授權 GoM／NonGoM 各 1 seed 配置並送跑，但本筆記建立時尚未啟動，待方法協商及資料／程式 audit 才凍結 1–6。擬每臂 584 GT＋1,180 BG、seed42、相同總更新預算，採 P5 同日去重→日期→事件等權主端點；來源內容與 source-Val 仍為殘餘混淆。未改原始資料、未 launch、未 commit／push。

## [2026-09-20] prereg-freeze | 20:15:11 Asia/Taipei 凍結 P16 卡片 1–6（`FROZEN_prelaunch`），同步 index／handoff；Claude×Codex R3 ACCEPT／BLOCKING=[]，獨立 input/config audit PASS；GoM156／NonGoM120 train scenes 全覆蓋，兩臂 GT584＋BG1180 unique、split 無 scene 交集，Val／Test 繼承 P14。P5 原公式 P12 CPU parity PASS；P16 新 report runtime parity 與 launch snapshot 校驗仍待通過。尚未 launch、未 commit／push；之後卡片不回改，process 狀態另寫 handoff／log。

## [2026-09-20] prelaunch-gates | 凍結後主線回報 P16 新 report P12 endpoint exact parity PASS，P14 checkpoint provenance GoM46／7866／Val mIoU0.7303328514 與 NonGoM64／7040／0.6536126733 皆 PASS，獨立 `--verify-frozen` PASS。卡 SHA256=`fe5410196e177946bb03cc2a203da16345cc904bbe5808ff567746964628efcb`，不回改已凍結卡；config run 開關已由主線依使用者授權啟用，仍待 snapshot／launch 證據。同步 index／handoff；未 commit／push。

## [2026-09-20] launch | 補記 P16 已啟動證據（實際 launch 20:17:47，晚前次 20:17 回寫 16 秒，原「尚無啟動證據」句已過期）：獨立 tmux session=`p16_matched_s42`、main PID=`23909`，兩臂串行 GoM→NonGoM、config 兩臂皆 `run:true`；config SHA256 實查=`5c8fdde49bd4140dc156699143b67d51a86f708f04455b52456d9f3a63ce290a`，與 institution 正本一致。300 epochs／patience301／110 updates/epoch=33,000 updates/臂、seed42。20:28～20:29 實查：GoM 臂 Epoch 3 完成、Epoch 4 進行中（Epoch 2 V-mIoU 0.5635／Oil IoU 0.2187），僅訓練中數字非可報告結果；粗估兩臂共約 28 小時。事前卡`.agents/decisions/20260920_P16_GoM等patch單seed設計.md`狀態仍`FROZEN_prelaunch`、SHA256=`fe5410196e177946bb03cc2a203da16345cc904bbe5808ff567746964628efcb`，未回改。更新 `.agents/handoff.md`「目前狀態」（新增 20:30 條目、標註 20:17 條目已被推翻但原樣保留）與「執行中程序」（P16 改列執行中、P14 完成敘述保留）；更新 `index.md` P16 條目移除過期的「尚未記錄啟動」字樣。未動 decisions/ 凍結卡；未 commit／push。

## [2026-09-20] incident | P16 首次送跑於 Epoch 15 遭 SIGINT 中斷、經使用者授權刪除並重跑：20:17:47 啟動的 GoM 臂（main PID `23909`、tmux `p16_matched_s42`）於 21:07 左右在 `torch.save()` 寫 checkpoint 當下收到 `KeyboardInterrupt` 而死，非 OOM、非程式錯誤、非磁碟問題；NonGoM 臂從未啟動。⚠ 成因未完全確證，僅觀察到 tmux `claudeXcodex_review` 建立於 `21:07:59` 與死亡時間吻合到秒，強烈暗示但無直接證據，不得寫成已確認因果、不點名任何人為責任。使用者於 21:10 明確授權刪除重跑（與 P14 09-14 中斷裁決一致）；刪除前重驗 `launch_files.sha256`（12 檔全 `OK`）與 `build_design.py --verify-frozen`（`PASS`），確認凍結未受污染。已刪除 `20260920-201818_MS6_P16_GoM_matched1764_S42`（7.9 GB，含 `last.tmp` 3.46 GB 寫一半殘骸），不可復原；刪除前將 `args.yaml`／`data.yaml`／`results.csv`（15 epochs）與改名 log 另存於 `_p16_runs/P16_gom_matched_patch/_interrupted_20260920-2107_record/`。中斷 run 的 15 epochs 最佳為 Epoch 13（Val mIoU `0.699347`／Oil IoU `0.436575`），僅供中斷存證，不得引用為任何 P16 結果。21:12:40 由 epoch 0 重跑：main PID `28418`、detached tmux `p16_restart_2111`，同一凍結 config、seed42 已鎖；新 GoM run=`20260920-211240_MS6_P16_GoM_matched1764_S42`，log=`P16_seed42_restart_20260920-2111.log`。事前卡 `.agents/decisions/20260920_P16_GoM等patch單seed設計.md` 狀態仍 `FROZEN_prelaunch`、SHA256 不變，未回改；設計／資料／端點全部沿用。institution 正本已於 21:12 寫入對應 CURRENT 條目。更新 `.agents/handoff.md`「目前狀態」（新增 21:15 STATUS_AS_OF、20:30 條目標註已被推翻但原樣保留）與「執行中程序」（PID／tmux／run 目錄／log 改列新重跑）；更新 `index.md` P16 條目反映中斷與重跑狀態。未動 decisions/ 凍結卡；未 commit／push。

## [2026-09-23] launch | P18 台灣 holdout × 背景配額 ρ∈{19, 21, 23} 已授權啟動：tmux session `P18_rho_grid`，串行跑 ρ19→21→23；開跑前閘 1–7 PASS，閘 8/9（build_report.py 與 P12 端點重現）尚未寫，須在讀任何台灣數字前補上並先重現 P12 0.46379088875726393 才能揭封。詳見 `.agents/decisions/20260923_P18_台灣holdout背景配額rho19_21_23設計.md`「啟動紀錄（2026-09-23）」。未 commit／push。

## [2026-09-24] decision | P19 GoM 形態分組獨立訓練設計：Claude×Codex 共識收斂（3 回合），共識稿存 `.agents/decisions/20260924_P19_GoM形態分組獨立訓練設計.md`，待使用者選方案 A/B 後凍結；未建檔、未訓練。

## [2026-09-24] decision | P19 凍結：使用者選方案 A（3 seeds×3 組），維持既有 loader；建檔 analysis/p19_gom_morph_split 完成（parity gate、val 39 景閘、loader 模擬、report selfcheck 重現 P17 0.455751／P16 NonGoM 0.281168、7 unit tests 全 PASS），共用 val k=45（251 oil＋310 BG）；未 launch（P18 仍在跑）。

## [2026-09-25] result | P18 台灣 holdout × 背景配額 ρ∈{19,21,23} 結案：主端點（best.pt event-macro Oil IoU 兩事件 pairwise Δ）三組皆判定不可判定，且對 checkpoint 規則敏感（last.pt 19−21 兩事件皆 ≥+0.02 支持 19，與 best.pt 19−21 不可判定不一致）；不收線也不採用新比例。build_report.py 因 2026-09-25 host 上 import torch 卡死而改以 torch-free 模式運行（P12 端點 exact parity 0.46379088875726393 先行重現 PASS），last.pt-internal best_epoch 交叉核對 PENDING（results.csv 復核 PASS）。詳見 wiki/experiments/20260925_P18_台灣holdout比例19_21_23_不可判定.md，設計正本 .agents/decisions/20260923_P18_台灣holdout背景配額rho19_21_23設計.md 已補「結案（2026-09-25）」節。未 commit／push。

## [2026-09-25] fix | 容器重啟後 import torch 恢復；P18 last.pt best_epoch 交叉核對全 PASS（ρ19/21/23 best_epoch=50/39/40，updates_to_best=13200/11349/12680，selection_reconciliation 全 PASS，輸出 `/home/alanyh/oil_dataset/new/_p18_runs/P18_taiwan_rho_grid/P18_checkpoint_crosscheck.json`）；P18 無待補項。已就地更新 wiki/experiments/20260925_P18_台灣holdout比例19_21_23_不可判定.md 與 .agents/decisions/20260923_P18_台灣holdout背景配額rho19_21_23設計.md 中原 PENDING 句（保留原句，附加補跑結果）。未 commit／push。

## [2026-09-25] experiment | P19 launch：09:29 首次 launch 因全機 import torch 卡在 kernel（SIGKILL 殺不掉）而停在 selfcheck，未訓練；使用者重啟容器後 10:16 重新 launch（tmux p19_morph_split），sha／verify-frozen／selfcheck PASS，10:21 開始 MS6_P19_MS_S42，loader 實載 train 538 oil＋630 BG／121 景、val 251＋310／39 景。9 runs 後自動跑 18 個台灣測試與 build_report.py。

## [2026-10-04] experiment | 使用者指定整理 Claude 舊圖 P16 0.3/0.7 vs P21 0.2/0.8：新增 wiki/experiments/20261004_P16_vs_P21_Tversky舊圖比較.md（第7–9步、事後描述性）；核對正式 JSON、既有 CSV／CM JSON，event-macro Oil IoU 0.455362→0.451826、recall↑／precision↓，未觀察實務改善、loss 因果不可判定。列入8張既有圖與CSV／JSON來源，區分9景未去重pooled與正式P5 event-macro，也區分P21同輪tv03對照。更新index／handoff；未改凍結卡、未訓練、未commit／push。

## [2026-10-06] decision | P24 全球排除台灣、GoM 額外增強曝光設計完成：新增 `.agents/decisions/20261006_P24_全球排除台灣_GoM增強曝光設計.md`，Claude×Codex 三輪同session收斂（R1 REVISE1→R2 REVISE5→R3雙方ACCEPT／BLOCKING=[]；claude-opus-5/high、CLI2.1.291、無降級、session累積USD0.889947）。所有來源保留基礎增強，僅GoM額外取樣；Original／P15倍率1.6420745069／2××seeds42/43/44九run，P5台灣兩事件主比較、固定epoch早停政策效果。CPU source-mask與AST-stub／parity／independent72項PASS；台灣清單byte-identical、本次未重掃台灣mask；actual Dataset/DataLoader與CUDA尚未執行。同步index／handoff，卡7–9待結果；未launch／排程／commit／push。

## [2026-10-07] experiment | P24 範圍 v2 並啟動訓練：使用者裁決（「就幫我跑這個就好，一樣幫我跑3 seed」）只跑 GomP15 組（全資料僅對 GoM 增強，每輪曝光 6260＝4496＋1764）× seed 42/43/44 共 3 run，不跑 Original、不跑 Gom2x；此縮減未再經 Claude×Codex 協商，故原卡 §6 的 GomP15−Original 同 seed 配對判準無法判定，本輪只產出三 seed 台灣 endpoint 數值，與既有全資料原量結果並列僅屬描述性對照。早上容器 /dev/nvidia* EPERM（cuda False），使用者 docker restart 後恢復（RTX 5090）；主機 CPU max_perf_pct 仍為 14。開跑前九 run 版 CPU 準備驗證重跑 PASS（含 P12 parity 0.46379088875726393），實際 CPU Dataset/DataLoader 前測 PASS（Train 1,772 oil＋2,730 BG、Val 444＋700、三 split 景交集 0；實讀 train 391 batch／6,256 張、val 72 batch／1,144 張）。GT_expand commit 7ace4d9，09:53 於 tmux `p24` 執行 `analysis/p24_global_gom_augmentation/launch_v2.sh`（`main/experiments_P24_gomp15_only.yaml`，三 seed 依序；取代卡 `analysis/p24_global_gom_augmentation/SCOPE_V2.md`），結果目錄 `/home/alanyh/oil_dataset/new/_p24_runs/P24_global_gom_augmentation`。S42 實測每輪約 7.3 分鐘；Epoch 1/2 source-Val mIoU 0.6092／0.6242、Oil IoU 0.270／0.300；上限 300 輪、patience 50；台灣結果尚無。詳見 `.agents/decisions/20261006_P24_全球排除台灣_GoM增強曝光設計.md`「2026-10-07 範圍 v2」。

## [2026-10-08] decision | 新增 `.agents/decisions/20261008_未來工項首波技術排序與文獻survey.md` 與附件目錄 `.agents/decisions/attachments/20261008_未來工項首波技術排序與文獻survey/`（8 檔：r1–r3 prompt／out、claude_r1_own、claude_survey），並更新 `.agents/handoff.md`（目前狀態＋共識紀錄）：未來工項(116–117)首波技術排序 Claude×Codex 三回合同 Codex session `01a11aa9-d215-7193-91c9-243f1ddc8153` 收斂（R1 盲提案→R2 REVISE[3]→R3 雙方 ACCEPT／BLOCKING=[]）；僅 survey＋優先序，未授權任何 launch／改檔／凍結。同步 index。

## [2026-10-09] result | P24 範圍 v2（GomP15 × 3 seed）完跑；台灣主要指標 Oil IoU 0.4606／0.4569／0.4142（平均 0.4439）；§6 無 Original 配對故不可判定；詳見 decisions/20261006_P24_全球排除台灣_GoM增強曝光設計

## [2026-10-09] launch | DADS-SAM B 六波段 100-epoch GoM pilot 啟動（22:03，使用者授權）：公開 DADS-SAM（SAM ViT-B＋LoRA）改吃六波段 B02/B03/B04/B8A/B11/B12，不經 Prithvi；config `main/experiments_DADS_SAM_gomonly_B_e100_pilot.yaml`（git_head `99c04c0`，launch gate PASS），tmux `dads_sam_b_pilot`，結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_pilot/20261009-220304_MS6_DADS_SAM_B_public6_GoM_e100_pilot_R1`。與正式 B 設定差異只有 epochs 300→100、Taiwan post-test 關閉、名稱與結果目錄。定位＝可行性 pilot，九步驟卡 1–6 未凍結故不跑臺灣測試，不產生任何臺灣數字，不是正式 B run；預估每 epoch 約 5.3 分鐘、最長約 9 小時（預計 10-10 早上結束）。細節正本見 institution 現況檔 `CURRENT 2026-10-09 22:03` 條目（institution commit `94a4880`），同步 `.agents/handoff.md`。

## [2026-10-09] incident | 主機 CPU 節流根因已定並解除（約 21:50，使用者於 host 執行）：元兇是 host 的 `thermald`，自 2026-10-04 07:27 開機後約 1 分鐘起把 CPU 壓到最低（`max_perf_pct=14`、關 turbo、powerclamp 拉滿、RAPL PL1 寫成 0 W），當時 CPU 僅 33°C；解除需四件事都做：`systemctl stop thermald`、`max_perf_pct=100`／`no_turbo=0`、cooling device 歸 0、RAPL `constraint_0` 寫回 125 W（mmio `constraint_1` 寫回 250 W）。⚠ `thermald` 尚未 disable，重開機後會再鎖。影響：2026-10-04 07:29 至 10-09 21:50 之間記錄的訓練時間都偏慢，不可與區間外比較，指標數值不受影響；B 節流時 52 分鐘／epoch，解除後 5.3 分鐘／epoch。細節正本見 institution 現況檔 `CURRENT 2026-10-09 22:03` 條目。

## [2026-10-10] result | DADS-SAM B 六波段 100-epoch GoM pilot 完跑（2026-10-09 22:03:06 → 2026-10-10 06:47:52，`EXIT=0`，tmux `dads_sam_b_pilot` 已結束，GPU 已釋出）：跑滿 100 epoch、未早停（最佳在第 87 epoch，patience 50 未觸發）；GoM 驗證集（單 seed 42）best val mIoU 0.7241、val Oil IoU 0.4809。對照 A（`MS6_P14_GoM_only_full_R1`）best val mIoU 0.7303、Oil IoU 0.4934（第 46 epoch），各一次 run、學習率排程長度不同（B 100／A 300 epoch），不作優劣判定；能主張的只有「六波段版本在 GoM-only 資料上學得起來，GoM 驗證表現與 A 同一量級」，本 run 沒有跑臺灣測試、沒有任何臺灣數字。總計 524.77 分鐘（8 小時 45 分），每 epoch 5.25 分鐘。產物 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_pilot/20261009-220304_MS6_DADS_SAM_B_public6_GoM_e100_pilot_R1/`（含 `weights/best.pt`、`results.csv`、`run_metrics.json`）。未決（屬使用者）：是否用 `best.pt` 補跑臺灣（需先凍結九步驟卡 1–6）、正式 B 用 100 或 300 epoch、C 是否啟動、host `thermald` 是否 disable；正式 B／C 設定檔仍是 `run: false`。細節正本見 institution 現況檔 `CURRENT 2026-10-10 06:48` 條目（institution commit `dba73d3`），同步 `.agents/handoff.md`。

## [2026-10-10] incident | 容器內新 process 無法使用 GPU（2026-10-10 06:48 發現）：`nvidia-smi` 回 `Failed to initialize NVML: Unknown Error`，新 process 的 `torch.cuda.is_available()` 為 False；10-09 22:18 時仍正常，已在跑的訓練不受影響。原因未定。在使用者於 host 重啟容器前，任何新的 GPU 工作都無法啟動。細節正本見 institution 現況檔 `CURRENT 2026-10-10 06:48` 條目（institution commit `dba73d3`），同步 `.agents/handoff.md`。

## [2026-10-10] result | DADS-SAM B 六波段 pilot 的 best.pt 臺灣測試完成（10:42 啟動、約 3 分鐘、`EXIT=0`，使用者授權「跑阿 就拿best去跑」）：固定第 87 epoch 的 best.pt，不重訓、不用臺灣挑 checkpoint；九步驟卡 1–6 沒有事前凍結，只能描述性陳述，不能當作 B 優於／劣於 A 的判定。B（本 pilot）vs A（`MS6_P14_GoM_only_full_R1`），各一次 run：9 景各算再平均（報告口徑）Oil IoU 0.3179／0.3657；兩事件平均 0.3431／0.4101（05-22：0.4460／0.4741；天使輪：0.2401／0.3461）；9 景像素加總 0.3871／0.4187。差距幾乎全來自單一景 `天使輪_20230912_S2_50QRL`（A 0.4437 → B 0.0331，recall 0.033，幾乎整景漏判）；其餘 8 景 B−A 介於 −0.034 與 +0.033，B 贏 5 景、輸 3 景。產物 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_pilot/20261010-104217_MS6_DADS_SAM_B_public6_GoM_e100_pilot_R1_taiwan_test/`；數值檔 `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/dads_sam_gomonly_20261005/taiwan_endpoint_B_pilot_vs_A.json`。細節正本見 institution 現況檔 `CURRENT 2026-10-10 10:45`（institution commit `21d3482`）。

## [2026-10-10] incident | 容器 GPU 存取中斷已解除：使用者 `docker restart` 後 GPU 恢復；觸發來源已查明＝host 的 snapd 自動更新在 06:33:01 觸發 systemd daemon-reload。`Docker/run.sh` 已加四行 `--device /dev/nvidia…`（未測試，重建容器才生效）；主機維護清單 `/mnt/backup/alanyh/HOST_REBOOT_CHECKLIST.md` 待使用者擇期執行（含 `thermald` disable）。目前沒有任何訓練 process 在跑。

## [2026-10-10] decision | Claude×Codex 就 DADS-SAM C（`MS6_DADS_SAM_C_prithvi_samdec_GoM_R1`）的做法四回合收斂（均 ACCEPT），九步驟卡 1–6 凍結：新增 `.agents/decisions/20261010_DADS_SAM_GoM_C_B正式run設計.md`。主指標＝臺灣 9 景平均 Oil IoU，判定用 X−R（R＝0.3862，δ＝0.05）。C 的 5-epoch GoM-only preflight 學習判準全過（epoch 5 val Oil IoU 0.396／mIoU 0.676）；原判準 2（GradScaler 零跳步）未過，經 Codex 第 4 回合同意事後修訂後通過。細節正本見 institution 現況檔 `CURRENT 2026-10-10 11:38` 條目（institution commit `d08ba58`），同步 `.agents/handoff.md`。

## [2026-10-10] launch | DADS-SAM GoM-only 正式 run 啟動（11:37:43，使用者授權：「我覺得你跟codex討論一下C的做法，你們兩個看過都OK 就你這邊負責這邊先測試跑看看，如果OK的話 就正式執行吧，然後先跑C 在B，然後都調成300 epoch比照A 辦理」）：config `main/experiments_DADS_SAM_gomonly_formal_C_then_B.yaml`（repo `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand`，git_head `f3b1001`，launch gate PASS），同一個 main_runner process（PID 12675）先跑 C（`MS6_DADS_SAM_C_prithvi_samdec_GoM_R1`）後跑 B（`MS6_DADS_SAM_B_public6_GoM_R1`），tmux `dads_sam_formal`；各最多 300 epoch、patience 50，設定比照 A，各自結束時以 best.pt 做唯一一次臺灣 post-test。log `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_formal_C_then_B_main_runner.log`（結束時末行為 `EXIT=`），結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly/`。預估 C 最多約 5 小時、B 最多約 26 小時（早停會更短）。正式 YAML 不得再改；run 若中斷須在新目錄從頭重跑；停止 process 須使用者明確授權。細節正本見 institution 現況檔 `CURRENT 2026-10-10 11:38` 條目（institution commit `d08ba58`），同步 `.agents/handoff.md`。

## [2026-10-10] result | DADS-SAM GoM-only 正式 C 完成（13:10）：早停於第 100 epoch（best 第 50，GoM val mIoU 0.7394）；臺灣 9 景平均 Oil IoU 0.3596，X−R −0.0266，判定「不可判定」（leave-one-scene-out 不改判定）；9/9 景、manifest complete、parity PASS；正式 B 於 13:07 在同一 process 接續執行。詳見 `.agents/handoff.md` 的 13:10 條。

## [2026-10-10] experiment | P12 seed 擴充（全球排除台灣，再補兩個 seed）已排隊，等 DADS-SAM 正式 B 結束後自動開始。使用者指示：「幫我安排一下在DADSAM的下一個實驗，我需要全資料排除台灣 在補兩個seed」。設定與原 P12 相同，只改 seed；runner 實際 seed＝設定 seed＋fold_idx，原 P12 實際 seed 為 43，新增兩個 run 實際 seed 為 42、44。tmux `p12_seeds`、launcher `analysis/p12_taiwan_deploy/launch_seed_extension.sh`、卡 `analysis/p12_taiwan_deploy/SEED_EXTENSION.md`、commit `1da93da`。未經 Codex 協商；結果尚無。

## [2026-10-11] result | DADS-SAM GoM-only 正式 B 完成（00:49）：早停於第 133 epoch（best 第 83，GoM val mIoU 0.7394／Oil IoU 0.5081，700.1 分鐘）；臺灣 9 景平均 Oil IoU 0.3732，X−R −0.0130，判定「不可判定」（leave-one-scene-out 的 X−R 介於 −0.023～+0.031，不改判定）；9/9 景、manifest complete、parity PASS；整個序列 EXIT=0。C（0.3596，X−R −0.0266）與 B 皆不可判定；B 非盲測（pilot 0.3179 在規則凍結前已看過）。報表 commit `b530e45`。排在其後的 P12 seed 擴充（tmux `p12_seeds`）於 00:50:15 自動啟動時被 launch 閘門擋下（追蹤檔 `endpoint_report.json` 被 build_report.py 改寫、約 20 秒後才 commit，EXIT=2），沒有在跑，GPU 自 00:50 閒置，重新啟動待使用者決定。結案卡見 `wiki/experiments/20261011_DADS_SAM_GoM_only_正式C與B_皆不可判定.md`；詳見 `.agents/handoff.md` 的 2026-10-11 08:15 條。

## [2026-10-11] experiment | P12 seed 擴充（全球排除台灣，實際 seed 42、44）：排隊在 00:50:15 觸發時被 launch 閘門擋下（DADS-SAM 收尾檔 endpoint_report.json 當下尚未 commit，時序相撞，EXIT=2，未產生結果）；08:13 工作區乾淨後以同一 launcher 重新啟動（tmux `p12_seeds`），閘門通過、訓練進行中。DADS-SAM 正式 B 已於 00:50 結束。

## [2026-10-11] result | P12 seed 擴充完跑（全球排除台灣，實際 seed 42／43／44，其中 43 為原 P12）。台灣 9 景平均 Oil IoU：0.3552／0.4052／0.3660，平均 0.3755、SD 0.0263；原 P12 單次 0.405 是三個 seed 中最高者。同口徑 3 seed 並列：GoM-only 0.393±0.026、全球＋GoM 增強（P24）0.401±0.018、GoM-only 增強 0.354±0.028；差距約 1 個 SD，不能主張 GoM 增強有效（P24 與本組切分、patience 不同，非配對；新舊 run 非同一 commit）。詳見 repo `analysis/p12_taiwan_deploy/SEED_EXTENSION.md`。

## [2026-10-11] decision | P25 HNM 背景替換（GT-centric 架構）單 seed pilot 設計卡 1–6 凍結：新增 `.agents/decisions/20261011_P25_HNM背景替換單seed_pilot設計.md`；使用者直接指示的單 seed pilot，未經 Codex 協商；訓練尚未啟動，預期結果為不可判定（單 seed）。

## [2026-10-11] launch | DADS-SAM GoM-only B seed 擴充（seed 43、44）於 21:07 Asia/Taipei 啟動：使用者指示原話「目前這邊DBDS-SAM在去幫我排 多跑兩個seed的實驗 讓他再多跑幾次吧 幫我安排 ，那剛剛我有跑HNM的實驗 就幫我接在他後面吧」；未經 Codex 協商，沿用 10-10 凍結卡（`.agents/decisions/20261010_DADS_SAM_GoM_C_B正式run設計.md`）的 B 設定，只多跑 seed 43、44。卡（結果出現前寫成）：repo `analysis/dads_sam_gomonly_20261005/B_SEED_EXTENSION.md`（GT_expand commit `d6f8ad5`）。設定 `main/experiments_DADS_SAM_gomonly_B_seeds.yaml`（`MS6_DADS_SAM_B_public6_GoM_S43` → `_S44`，與正式 B 逐鍵相同，只差 seed／名稱）；tmux `dads_sam_b_seeds`；launcher `analysis/dads_sam_gomonly_20261005/launch_B_seed_extension.sh`；launch gate PASS；啟動時 P25 HNM 的 main_runner 已於 21:06 結束（EXIT=0）。log：`/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_seed_extension_main_runner.log`（結束時末行 EXIT=）；結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_seeds/`。每 run 最多 300 epoch（約 26 小時）；seed 42 是 133 epoch 早停、700 分鐘；粗估兩個 run 合計 1–2 天，GPU 佔用中。報告方式（事前寫定）：B 三 seed（42 既有、43、44）與 GoM-only 基線 P15 Original 三 seed 並列；主數字依使用者 10-11 指示改為「每個 run 臺灣 9 景像素加總、跨 seed 取平均」（基線 Oil IoU 0.4808／0.4379／0.3704，平均 0.4297、SD 0.0557；B seed 42 為 0.4434），同時列 9 景平均；不訂新門檻、不改 10-10 卡對 B 單次 run 的判定；非盲測。使用者 10-11 同日指示：之後評估數字一律用像素加總（與混淆矩陣對得起來）。institution 現況檔已更新（commit `8e77698`）。結果尚無。

## [2026-10-11] result | P25 HNM 背景替換單 seed（43）pilot 結案：臺灣 9 景平均 Oil IoU，GoM-only＋HNM 0.3268（低於事前門檻 0.3632，否定；同 seed 差 −0.078）、全球排除臺灣＋HNM 0.4025（落在 0.3552–0.4052 內，不可判定；同 seed 差 −0.003）。兩組召回皆降（−0.085、−0.180）。路線收線、不補 seed。結案頁 `wiki/experiments/20261011_P25_HNM背景替換在臺灣終點無效_GoM否定_全球不可判定.md`；設計卡 `.agents/decisions/20261011_P25_HNM背景替換單seed_pilot設計.md`（未改動）。

## [2026-10-12] launch | P24 範圍 v3（全資料排除臺灣、不對墨西哥灣加量的 Original 組 × seed 42／43／44）於 10:26:08 Asia/Taipei 啟動，並同時停掉 DADS-SAM B seed 44（process 啟停當天回寫）。使用者指示（2026-10-12）原話：「我覺得我需要補跑這個實驗欸，我需要就是全資料(排除台灣) 然後訓練驗證是切80/20比例得下去做訓練」「就照那份然後去幫我安排實驗好了，也就是如你所說的seed43 跑完之後停掉 安插這一個 然後再回來跑seed 44 DBDSSAM」；未經 Claude × Codex 協商，使用者直接裁決。**實驗**：P24 九 run 設計裡當初沒跑的「Original」組 × seed 42／43／44；全資料排除臺灣，訓練 276 景 4,502 個 patch（1,772 GT-centric／2,730 純背景）、驗證 71 景 1,144 個 patch（約 80／20，以景為單位切）、不對墨西哥灣加量、每輪 281 步；與已跑完的 GomP15（期末報告表 3-7）同一份切分與設定，只差沒有墨西哥灣加量。背景：既有全部海域排除臺灣結果（P12）是 333 景訓練／14 景驗證，14 景是千里達、Golden Ray、斯里蘭卡三個事件整批留出，使用者想要一組 80／20 切分的全資料結果。**process 啟停時間線**：10:11 排隊腳本啟動（tmux `p24_v3_queue`，`analysis/p24_global_gom_augmentation/launch_v3_between_dads_seeds.sh`），只等待；DADS-SAM B seed 43 在第 151 epoch 早停（驗證最佳在第 101 epoch），臺灣測試於 10:24:55 寫出；10:26:03 排隊腳本停掉 DADS-SAM seed 擴充的 runner（當時剛開始 seed 44，在 epoch 1 第 14／171 步；原 log 末行 EXIT=143 為預期），被中斷的 seed 44 目錄移到 `/home/alanyh/oil_dataset/new/_dads_sam_runs/_interrupted_S44_20261012/`，沒有刪；10:26:08 P24 Original launch：launch gate PASS、git_head `bf04b95`、seed 42 已鎖，log 核對 train 1,772＋2,730、val 444＋700、每輪 281 步，epoch 1 約 40 秒；P24 三個 run 結束後，排隊腳本會自動把 DADS-SAM B seed 44 從 epoch 0 重跑（`main/experiments_DADS_SAM_gomonly_B_seed44.yaml`，區塊與原 seed 44 相同）。寫這筆紀錄時 P24 仍在訓練、seed 44 尚未重啟。**檔案位置**：實驗卡 `analysis/p24_global_gom_augmentation/SCOPE_V3.md`（結果出現前寫成）、設定 `main/experiments_P24_original_only.yaml`（以上皆在 repo `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand`，commit `bf04b95`）、結果 `/home/alanyh/oil_dataset/new/_p24_runs/P24_global_original/`；institution 現況檔已更新（commit `83a9fd8` 及其後一筆）。**已知限制**：與 GomP15 不是同一批跑的（GomP15 在 commit `7ace4d9`；其後 `6a59b58` 改了 main_runner.py 一行顯示訊息與 deeplab_adapter.py 的 DADS-SAM C 說明文字，不在 Prithvi＋DeepLabV3 路徑上，但無法保證逐位元相同）；不是盲測（GomP15 與 P12 的臺灣結果都已看過）；目前還沒有任何 Original 的臺灣結果。交接細節見 `.agents/handoff.md` 的 2026-10-12 10:30 條；P24 原決策卡 `.agents/decisions/20261006_P24_全球排除台灣_GoM增強曝光設計.md` 未改動。

## [2026-10-12] experiment | 使用者指示「先幫我停掉實驗一下」：11:41 停掉 P24 Original（全資料排除臺灣、80／20、不對 GoM 加量）的訓練與排隊腳本（tmux `p24_v3_queue`）。停止時 S42 已完成（含臺灣測試）、S43 中斷於第 23 輪（無臺灣結果）、S44 未開始；原訂之後自動重跑的 DADS-SAM B seed 44 也因排隊腳本停止而不會啟動。資料未刪。恢復需使用者明說。

## [2026-10-12] decision | P24 範圍 v4（Gom2xRaw）：使用者指示在全資料排除臺灣的訓練中，只對墨西哥灣資料多放一份、且多的那份不做影像增強。設計＝每輪 4,502 個 patch 各一次（照常增強）＋墨西哥灣 2,738 個 patch 再各一次不增強，共 7,240 張／452 步，seed 42／43／44。程式改動與卡已準備在分支 `p24-gom2x-raw`（commit `8faa058`，worktree `/mnt/backup/alanyh/oil_IR_Fullband/_wt_p24_gom2x_raw`），未合併、未 launch；需先處理 P24 Original 的 S43／S44 再合併。卡：`analysis/p24_global_gom_augmentation/SCOPE_V4.md`。未經 Codex 協商。

## [2026-10-12] experiment | P24 範圍 v4 Gom2xRaw（全資料排除臺灣＋墨西哥灣多放一份不增強的原圖）× seed 42／43／44 已於 11:59 啟動（使用者：「不行 先把這個跑了吧」，先於 P24 Original 的 S43／S44）。程式改動已併入主線（`8faa058`），設定 commit `ca82e94`；每輪 7,240 張／452 步。第一次啟動被另一 session 的推論佔用 GPU 鎖擋下，一分鐘後重試通過。tmux `p24_v4`，結果 `_p24_runs/P24_global_gom2x_raw/`。P24 Original S43／S44 與 DADS-SAM B seed 44 仍未跑。

## [2026-10-12] result | P24 範圍 v4 Gom2xRaw（全資料排除臺灣＋墨西哥灣資料多放一份原圖、不增強）× seed 42／43／44 完跑：11:59 啟動、約 16:17 三個 run 全部完成（EXIT=0），同一 commit `ca82e94`，GPU 已釋放。臺灣 9 景像素加總 Oil IoU：S42 0.4625、S43 0.4564、S44 0.2851；3 seed 平均 0.4013±0.1007（樣本 SD）；舊口徑（9 景各算再平均）0.3803±0.0394。S44 偏低來自 05-22 事件召回率 0.275（S42／S43 為 0.592／0.581），原因未查；0.2851 低於先前四組 12 個 run 的任何一個（最低 0.3704）。同口徑並列：GomP15 0.450±0.030、P15 Original GoM-only 0.430±0.056、P12 全球排除臺灣 0.420±0.052、本組 0.401±0.101；差距都小於本組 seed SD，僅描述性並列、不下有效／無效。完整表與混淆矩陣見 repo `analysis/p24_global_gom_augmentation/SCOPE_V4.md`「結果」段（commit `5f87a89`）；institution 現況檔已同步。未決、需使用者明說：P24 Original S43／S44 續跑、DADS-SAM B seed 44 重跑、成果表／報告表 3-9 是否換成本組數字。

## [2026-10-12] launch | DADS-SAM GoM-only B seed 44 重新啟動；seed 43 臺灣結果：使用者 2026-10-12 晚間指示（原話）：「DADSAM 的實驗幫我補跑吧 我記得是43 完成 跑44」。**(1) 背景**：10-12 10:26 seed 擴充 runner 跑到 seed 44 第 1 epoch 時，被 P24 的排隊腳本依使用者當時指示停掉（原 log 末行 EXIT=143）；之後使用者 11:41 停掉 P24 Original，seed 44 一直沒有重跑。中斷的目錄保留在 `/home/alanyh/oil_dataset/new/_dads_sam_runs/_interrupted_S44_20261012/`。**(2) 重新啟動**：20:10:27 重新啟動 seed 44（從 epoch 0）；設定檔 `main/experiments_DADS_SAM_gomonly_B_seed44.yaml`（與 seeds 設定檔的 seed 44 區塊相同，已用 build_B_seed44_relaunch.py 驗過）；tmux `dads_sam_b_seed44`；launcher `/home/alanyh/oil_dataset/new/_dads_sam_runs/launch_B_seed44_relaunch.sh`（不在 repo）；launch gate PASS、git_head `5f87a89`、seed 44 已鎖；log `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_seed44_relaunch_main_runner.log`（結束時末行 `EXIT=`）；run 目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_seeds/20261012-201029_MS6_DADS_SAM_B_public6_GoM_S44`。預估 11–14 小時。GPU 佔用中。**(3) commit 差異**：三個 B seed 的 commit 不同：S42 `f3b1001`、S43 `d6f8ad5`、S44 `5f87a89`；d6f8ad5→5f87a89 在 main/ 的程式差異只有 deeplab_adapter.py 的 epoch_sampling.repeat_without_augmentation（預設關，B 不使用 epoch_sampling）。**(4) B seed 43 結果**（10-12 10:24 完成；151 epoch 早停，best 第 101 epoch，驗證 mIoU 0.7468；臺灣 9/9 景）：臺灣 9 景像素加總 Oil IoU 0.4695（TP 224,200／FP 173,837／FN 79,476／TN 3,455,365；Precision 0.5633、Recall 0.7383、F1 0.6390、mIoU 0.7006）；9 景各算再平均 0.3395（0912_S2_50QRL 單景 0.034）。**(5) 並列（像素加總）**：B seed 42 為 0.4434；GoM-only 基線（P15 Original）seed 42／43／44 為 0.4808／0.4379／0.3704，平均 0.4297。B 兩個 seed 都落在基線三 seed 的範圍內；三 seed 彙整待 seed 44 完成，依 repo `analysis/dads_sam_gomonly_20261005/B_SEED_EXTENSION.md` 的事前寫法報告。已凍結卡（9 景平均、±0.05）的判定不回頭改。**(6) 未做**：seed 44 結束後的驗收、三 seed 比較表、結果 commit。

## [2026-10-13] result | DADS-SAM GoM-only B seed 44 完成；B 三個 seed 齊全
- seed 44 於 10-12 20:10 啟動、10-13 06:47 進臺灣 post-test，EXIT=0；早停於 epoch 121、best epoch 71（val mIoU 0.7247、val Oil IoU 0.4824）；臺灣 9/9 景；launch 時 git_head `5f87a89`。run 目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_seeds/20261012-201029_MS6_DADS_SAM_B_public6_GoM_S44`。
- 臺灣 9 景像素加總（每個 run 算一次）Oil IoU／Precision／Recall／F1／BG IoU／mIoU：S42 0.4434／0.5884／0.6428／0.6144／0.9344／0.6889；S43 0.4695／0.5633／0.7383／0.6390／0.9317／0.7006；S44 0.4355／0.5570／0.6664／0.6068／0.9297／0.6826。
- B 三 seed 平均 Oil IoU 0.4495 ± 0.0178；基線 P15 Original S42/43/44 為 0.4808／0.4379／0.3704，平均 0.4297 ± 0.0557。B 三個 seed 都落在基線三個 seed 的範圍內，依事前寫好的 `analysis/dads_sam_gomonly_20261005/B_SEED_EXTENSION.md` 寫「沒有可分辨的差異」；10-10 凍結卡的原判定（不可判定，用 9 景平均）不改。
- 9 景平均（僅列出）：S42 0.3732／S43 0.3395／S44 0.3880。
- 限制：三個 seed 的 commit 不同（`f3b1001`／`d6f8ad5`／`5f87a89`）；0912_S2_50QRL（整景 1 張 patch）三個 seed 為 0.141／0.034／0.590，seed 間差異很大。
- GPU 已釋放，沒有 main_runner 在跑。

## [2026-10-13] launch | P24 Gom2xRaw（全資料，墨西哥灣多放一份原圖）seed 44 重跑，15:34 啟動、進行中
- 使用者指示（原話）：「全資料(僅增加GoM一倍數量)…Seed44我覺得再幫我重跑一次」。
- 背景：第一次三個 seed 臺灣 9 景像素加總 Oil IoU 為 0.463／0.456／0.285（平均 0.401±0.101）。10-12 調查：seed 44 掉分集中在 2026-05-22 兩景重疊帶上的一大片油污（占全部測試油污像素 31.7%），該塊召回率 seed 42／43／44 為 0.72／0.55／0.03；只把這一塊換成 seed 42 的預測，seed 44 的 IoU 會由 0.285 變 0.435。其餘 7 景三個 seed 相近。為何是 seed 44 原因未定（訓練曲線正常、驗證 mIoU 0.7132 為三者最高）。
- 這次跑什麼：設定檔 `main/experiments_P24_gom2x_raw_S44_rerun.yaml`（seed 44 設定與第一次逐欄相同，只改結果目錄），launcher `analysis/p24_global_gom_augmentation/launch_v4_s44_rerun.sh`，repo commit `4090230`，launch gate PASS，tmux `p24_s44_rerun`。
- 結果另存 `/home/alanyh/oil_dataset/new/_p24_runs/P24_global_gom2x_raw_S44_rerun/`，不覆蓋第一次的結果。第一次耗時 79 分鐘，本次預估 1–2 小時。
- 同 seed 不會逐位元重現（第 1 輪驗證 mIoU 本次 0.6258、第一次 0.6353），所以這是「同設定再抽一次」。
- 未決：報告要兩次都列，還是以重跑取代（取代等同事後挑結果，須在表下註明），待使用者決定。
- 同日另產出：seed 42/43/44 臺灣預測圖（規範樣式：RGB 底圖＋綠 TP／紅 FP／黃 FN），在 `/home/alanyh/oil_dataset/new/_p24_runs/P24_global_gom2x_raw/P24_Gom2xRaw_Taiwan_RGB_TPgreen_20261012/`。

## [2026-10-13] result | **P24 Gom2xRaw（全資料，墨西哥灣多放一份原圖）seed 44 重跑 — 2026-10-13 完成，GPU 已釋放**
- `EXIT=0`。臺灣 9 景像素加總 Oil IoU 0.2949（第一次 0.2851）；Precision 0.5337／Recall 0.3973／F1 0.4555／BG IoU 0.9243／mIoU 0.6096；TP 120,649／FP 105,410／FN 183,027／TN 3,523,792。早停輪／最佳輪 115／65（第一次 99／49），驗證 mIoU 0.7032（第一次 0.7132）。
- 兩次訓練不是逐位元相同（第 1 輪驗證 mIoU 0.6353 對 0.6258），但臺灣結果幾乎一樣：都漏掉 2026-05-22 兩景重疊帶上那一大片油污，該塊召回率第一次 0.03、重跑 0.04（seed 42／43 為 0.72／0.55）。05-22 兩景 Oil IoU：51QTD 0.2633→0.2721、51QUD 0.2213→0.2297。
- 含意：seed 44 偏低不是單次 GPU 非決定性造成的偶發，同設定同 seed 可重複。seed 決定抽樣順序、增強與分類頭初始化，是哪一項、是否與「多放一份原圖」有關，未查。
- 未決：報告用哪個數字待使用者決定。四次並列 0.4625／0.4564／0.2851／0.2949；若以重跑取代第一次，三 seed 平均 0.4046±0.0951（原 0.4013±0.1007）。
- 紀錄：repo `analysis/p24_global_gom_augmentation/SCOPE_V4.md`「seed 44 重跑」段；結果目錄 `/home/alanyh/oil_dataset/new/_p24_runs/P24_global_gom2x_raw_S44_rerun/`。

## [2026-10-13] launch | DADS-SAM B × 公開碼訓練設定（seed 42）啟動
- 使用者指示（原話）：「我這邊想要請你幫我再跑一個實驗是 DADSSAM的 loss也幫我改成跟他一樣的，也就是使用Loss 為 0.7 DiceCE + 0.3 Focal，batch size = 2，固定訓練 100 epochs，取最後一個 epoch 的權重 所以簡單講就是改輸入與輸出而已」。
- 這次跑什麼：run `MS6_DADS_SAM_B_public6_GoM_PubRecipe_S42`，2026-10-13 21:21（Asia/Taipei）啟動；設定檔 `main/experiments_DADS_SAM_gomonly_B_public_recipe.yaml`，tmux `dads_sam_b_pubrecipe`，GT_expand repo commit `d2759a3`，launch gate PASS。
- 與正式 B（seed 42／43／44 已完成）的差別只在訓練設定，照公開碼 `train_port.py`：Loss 0.7×DiceCE＋0.3×Focal、Adam 學習率 1e-4 固定（weight decay 1e-4）、batch size 2、固定 100 epoch 不早停、取最後一個 epoch 的權重做唯一一次臺灣測試、無資料增強、無 gradient clipping、fp32。模型、六波段輸入、正規化、固定整張圖的 box prompt 都不變（公開碼由標註外接框算 prompt 的做法沒有採用）。
- 未經 Claude × Codex 協商；單一 seed；不訂判定門檻，結果只作描述。
- 啟動前短測試（GoM 訓練 300 步＋GoM 驗證，不讀臺灣資料）通過；新 loss 與公開碼（monai DiceCELoss＋公開碼 FocalLoss）數值比對一致。
- 結果目錄：`/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_public_recipe/20261013-212124_MS6_DADS_SAM_B_public6_GoM_PubRecipe_S42`；log：`/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_public_recipe_main_runner.log`（結束時末行為 `EXIT=`）。
- 時程：每個 epoch 約 7.1 分鐘，100 個 epoch 約 12 小時，預估 2026-10-14 上午 9–10 點結束。期間 GPU 使用中，其他 session 不要啟動 main_runner。
- 目前沒有任何結果。
- 事前紀錄（開跑前凍結）：`.agents/decisions/20261013_DADS_SAM_B_公開碼訓練設定_設計.md`；正本為 GT_expand repo `analysis/dads_sam_gomonly_20261005/B_PUBLIC_RECIPE.md`（commit `d2759a3`）。institution 現況檔已回寫（institution commit `cdaad40`）。

## [2026-10-14] result | DADS-SAM B × 公開碼訓練設定（seed 42）完成：臺灣像素加總 Oil IoU 0.1807
- 流程：2026-10-13 21:21 啟動、10-14 09:15 進臺灣測試、09:16 結束，整個序列 `EXIT=0`；跑滿 100 epoch（不早停），訓練 714.5 分鐘；results.csv 100 列皆為有限值、學習率全程 1e-4；run_record status trained、error null；launch 時 git_head `d2759a3`、launch gate PASS。臺灣 9/9 景，prediction manifest complete（ok 9／failed 0／skipped 0）。測試用權重是 `last.pt`（第 100 epoch），manifest 記錄的 model path 與 sha256（`bf5ba919…`）就是 last.pt；臺灣只評一次。
- 主數字（臺灣 9 景像素加總算一次）：Oil IoU **0.1807**／Precision 0.4274／Recall 0.2385／F1 0.3061／BG IoU 0.9150／mIoU 0.5478；混淆矩陣 TP 72,416／FP 97,019／FN 231,260／TN 3,532,183。9 景各算再平均 0.2761。
- 對照（同為像素加總 Oil IoU）：正式 B 三個 seed 0.4434／0.4695／0.4355（平均 0.4495）；基線 P15 Original 三個 seed 0.4808／0.4379／0.3704（平均 0.4297）。依事前紀錄的寫法：本 run 低於 B 三個 seed 的範圍（0.4355–0.4695），也低於基線三個 seed 的範圍（0.3704–0.4808）；單一 seed，不訂判定門檻，只作描述。
- GoM 驗證：最後一個 epoch（第 100）mIoU 0.6271／Oil IoU 0.2964；最佳 epoch 是第 43（mIoU 0.6985／Oil IoU 0.4333），最佳 epoch 的權重沒有拿去測臺灣。正式 B 三個 seed 在各自最佳 epoch 的驗證 Oil IoU 為 0.508／0.523／0.482。訓練 loss 前 10 個 epoch 平均 0.40、最後 10 個 epoch 平均 0.16；驗證 Oil IoU 第 41–50 epoch 平均 0.368、最後 10 個 epoch 平均 0.334（範圍 0.296–0.368）。
- 主因觀察（描述性）：05-22 兩景約占全部油污像素的 80%，這兩景的召回率只有 0.127／0.072，是像素加總偏低的主因；0801 兩景也低（正式 B 三個 seed 在這兩景為 0.18–0.41）；0912、0918、0922 五景與正式 B 相近或略高。逐景表見結案卡。
- 不可主張：(1) 差異來自 loss、batch size、optimizer、資料增強、取最後權重中的哪一項——九項訓練設定同時改，無法歸因；(2)「公開碼的方法本身比較差」——prompt 與公開碼不同（本 run 用固定整張圖的 box，公開碼用標註外接框）、資料是 GoM 衛星影像而非公開碼的無人機 RGB、且只有一個 seed。
- 限制：單一 seed；B 的臺灣結果事前已看過，不是盲測；臺灣只有 9 景、兩個事件；未經 Claude × Codex 協商。10-10 凍結卡對 B 的判定不因本 run 更改。
- 下一步：這條路線（續跑／收線／改設計）、是否補 seed 43／44——未填（待使用者）。
- 結案卡：`wiki/experiments/20261014_DADS_SAM_B_公開碼訓練設定_臺灣低於B三seed範圍.md`（1–6 正本 `.agents/decisions/20261013_DADS_SAM_B_公開碼訓練設定_設計.md`，未改）。run 目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_public_recipe/20261013-212124_MS6_DADS_SAM_B_public6_GoM_PubRecipe_S42`；log `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_public_recipe_main_runner.log`。institution 現況檔已回寫（institution commit `1c57574`）。
- GPU 已釋放，沒有 main_runner 在跑。

## [2026-10-14] launch | DADS-SAM B × 公開碼訓練設定 seed 43、44 啟動（11:18 Asia/Taipei）
- 使用者授權（原話）：「對 你在幫我跑seed 43 / 44吧，阿我剛剛有導入wandb去幫我紀錄實驗，你幫我確認一下 有沒有問題 要是有先停止 告訴我錯在哪，要是正常就讓他跑下去吧」。未經 Claude × Codex 協商。
- 這次跑什麼：同一個 main_runner 依序跑 S43 → S44；兩個 block＝seed 42 block（`experiments_DADS_SAM_gomonly_B_public_recipe.yaml`）只改 seed／名稱，其餘訓練設定、prompt、資料都不變；各自以最後一個 epoch 的權重做唯一一次臺灣測試。設定檔 `main/experiments_DADS_SAM_gomonly_B_public_recipe_seeds.yaml`，GT_expand repo commit `836e33c`，launch gate PASS；tmux `dads_sam_b_pubrecipe_seeds`；結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_public_recipe_seeds/`；log `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_public_recipe_seeds_main_runner.log`（結束時末行 `EXIT=`）。
- 時程：每個 epoch 約 7.0–7.1 分鐘；預估 S43 約 10-14 23:20 結束、S44 約 10-15 11:30 結束。期間 GPU 使用中，其他 session 不要啟動 main_runner。
- 報告方式（事前寫定）：三個 seed（42／43／44）的臺灣 9 景像素加總指標取平均 ± 樣本標準差，與正式 B 三個 seed（平均 0.4495 ± 0.0178）及基線 P15 Original 三個 seed（平均 0.4297 ± 0.0557）並列，只作描述、不訂門檻。seed 42 的結果（0.1807）已看過，這兩個 run 不是盲測。
- wandb：這是第一次帶 wandb 的正式訓練（run `lkh4hy25`，專案 `oil-gt-expand`）。確認結果：數值正確（epoch 1 與 results.csv 完全相同）、速度不受影響、無警告。已知小問題：wandb 曲線比 results.csv 慢一個 epoch（`main/wandb_logging.py` 的 `run.log(row, step=epoch)` 沒給 `commit=True`），run 結束時會補上最後一列，不缺資料；未修，待使用者決定。
- 目前沒有任何結果。
- 事前紀錄（開跑前凍結）：`.agents/decisions/20261014_DADS_SAM_B_公開碼訓練設定_seed43_44_設計.md`；正本為 GT_expand repo `analysis/dads_sam_gomonly_20261005/B_PUBLIC_RECIPE_SEEDS.md`（commit `836e33c`）。institution 現況檔已回寫（institution commit `58421c5`）。

## [2026-10-14] queue | P24 Original seed 44 已排隊（等 DADS-SAM B 公開碼 seeds 跑完）
- 使用者指示（原話）：「那你再幫我安排進去吧， 我現在有一個實驗在跑， DADSSAM，等他跑完你就跑看看2」。目的：分辨 Gom2xRaw seed 44 漏掉 05-22 大片油污，是 seed 44 本身還是「多放一份不增強原圖」造成。
- 10-13～14 的分析：Gom2xRaw seed 44 兩次（0.2851／0.2949）在周圍 256×256 油污比例 >30% 的區域召回率只有 0.01–0.06（seed 42／43 為 0.41–0.88），稀疏油污則與其他 seed 相當（0.60–0.77）。49 個有機率圖的 run 裡，會整片漏掉那塊大油污（召回率 0.00–0.05）的只有沒用 GoM 訓練的模型和這兩次；模型對那塊的機率中位數不是約 0 就是約 1，調門檻救不回。那塊油污相對海水的亮度比和同景其他油污差不多。
- 使用者問 seed 44 是否抽到比較差的資料：程式查證，三個 seed 每輪看到的資料完全相同（抽樣只依輪數），seed 只決定順序、增強與分類頭初始值，所以不是抽到不同資料。
- 設定 `main/experiments_P24_original_S44.yaml`（凍結設計的 Original S44，只開 run、加 wandb notes），與 Gom2xRaw S44 只差有沒有多放那份原圖；repo commit `1107efc`。tmux `p24_original_s44`，launcher `analysis/p24_global_gom_augmentation/launch_original_s44_after_dads.sh`，等 DADS log 出現 `EXIT=` 與共用 GPU 鎖空出（Codex 馬祖全圖推論已先排隊）才啟動。結果會在 `/home/alanyh/oil_dataset/new/_p24_runs/P24_global_original_S44/`。預估 10-15 中午後開始、約 1.5 小時。

## [2026-10-15] result | DADS-SAM B × 公開碼訓練設定 seed 43、44 完成：三個 seed 平均臺灣像素加總 Oil IoU 0.3043 ± 0.1071，三個 seed 都低於 B 的範圍
- 流程：同一個 main_runner 依序跑 S43 → S44（2026-10-14 11:18 啟動）。S43 於 10-14 23:12 結束、S44 於 10-15 11:12 結束，整個序列 `EXIT=0`。設定檔 `main/experiments_DADS_SAM_gomonly_B_public_recipe_seeds.yaml`，GT_expand repo commit `836e33c`。訓練時間 S43 713.1 分鐘、S44 718.8 分鐘（seed 42 為 714.5 分鐘），都跑滿 100 epoch。每個 run 驗收項目全數通過：results.csv 100 列皆為有限值、學習率全程 1e-4、run_record trained、臺灣 9/9 景 manifest complete；臺灣測試用權重是 last.pt（第 100 epoch），sha256 S42 `bf5ba919…`、S43 `2a06c285…`、S44 `194a995c…`。
- 主數字（每個 run 臺灣 9 景像素加總算一次，再跨 seed 取平均 ± 樣本標準差 ddof=1）：Oil IoU 0.1807／0.3709／0.3614（seed 42／43／44），**平均 0.3043 ± 0.1071**；Precision 0.4274／0.7388／0.6175（0.5946 ± 0.1570）；Recall 0.2385／0.4268／0.4656（0.3770 ± 0.1215）；F1 0.3061／0.5411／0.5309（0.4594 ± 0.1328）；BG IoU 0.9150／0.9422／0.9341（0.9304 ± 0.0140）；mIoU 0.5478／0.6565／0.6477（0.6174 ± 0.0604）；9 景各算再平均 Oil IoU 0.2761／0.4052／0.3872（0.3562 ± 0.0699）。混淆矩陣 TP／FP／FN／TN：S42 72,416／97,019／231,260／3,532,183；S43 129,614／45,821／174,062／3,583,381；S44 141,399／87,592／162,277／3,541,610。
- 逐景 Oil IoU（0801_QRK、0801_QRL、0912_S2_QRL、0918_QRK、0918_QRL、0922_QRK、0922_QRL、0522_51QTD、0522_51QUD）：S43 0.260／0.173／0.385／0.622／0.653／0.455／0.438／0.331／0.330；S44 0.335／0.163／0.425／0.610／0.655／0.328／0.290／0.336／0.343；seed 42 逐景見 seed 42 結案卡。
- 對照（同為像素加總算法）：正式 B（我們的訓練設定，與基線相同：Tversky、AdamW、batch 16、早停、取驗證最佳）三個 seed 0.4434／0.4695／0.4355，平均 0.4495 ± 0.0178（Precision 0.5695、Recall 0.6825、F1 0.6201、BG IoU 0.9319、mIoU 0.6907）；基線 P15 Original（Prithvi＋DeepLabV3，GoM-only）0.4808／0.4379／0.3704，平均 0.4297 ± 0.0557（Precision 0.5473、Recall 0.6718、F1 0.5997、BG IoU 0.9278、mIoU 0.6788）。
- GoM 驗證（不用來選權重）：最後一個 epoch Oil IoU 0.2964／0.3391／0.4100（mIoU 0.6271／0.6510／0.6866）；最佳 epoch 43／27／12，Oil IoU 0.4333／0.4350／0.4237（最佳 epoch 的權重沒有拿去測臺灣）。正式 B 在各自最佳 epoch 的驗證 Oil IoU 為 0.508／0.523／0.482。
- 結論（依事前紀錄的寫法，只作描述、不訂門檻）：**三個 seed 都低於 B 的範圍**（0.4355–0.4695）；與基線範圍（0.3704–0.4808）相比，只有 S43（0.3709）落在基線最低值 0.3704 之上。三個 seed 的召回率都低於 B 的平均（0.24／0.43／0.47 對 0.68）；S43／S44 的 precision 高於 B 的平均。seed 間差異大（標準差 0.107，B 為 0.018、基線為 0.056），主要在 05-22 兩景（S42 召回率 0.13／0.07，S43／S44 為 0.35–0.42）。
- 不可主張：(1) 差異來自九項訓練設定中的哪一項——同時改，無法歸因；(2)「公開碼的方法本身比較差」——prompt 不同（公開碼用標註外接框 GT bounding-box prompt，測試時會洩漏答案，本實驗用固定整張圖的 box）、資料不同（GoM 衛星影像對公開碼的無人機 RGB）。限制：不是盲測（seed 42 的結果在 seed 43／44 開跑前已看過）；臺灣只有 9 景、兩個事件；未經 Claude × Codex 協商。
- wandb：S43 run `lkh4hy25`、S44 run `7lgnkgb0`（專案 `oil-gt-expand`），兩個都 finished，history 各 100 列與 results.csv 相同，summary 內有臺灣 recon 指標；seed 42 沒有 wandb 紀錄。已知小問題（未修，待使用者決定）：訓練期間 wandb 曲線慢一個 epoch（`main/wandb_logging.py` 的 `run.log(row, step=epoch)` 沒給 `commit=True`），最後一列在 run 結束時補寫，沒有資料遺失。
- 下一步：這條路線（DADS-SAM 線續跑或收線）、投影片寫法——待使用者決定。建議句（非已決定）：「照公開程式碼的訓練設定（只改輸入與輸出）時，三個 seed 平均 Oil IoU 0.30 ± 0.11，三個都低於改用 baseline 訓練設定的 DADS-SAM（0.45 ± 0.02）；因此本研究的 DADS-SAM 採用與 baseline 相同的訓練設定。prompt 與公開碼不同。」
- 結案卡：`wiki/experiments/20261015_DADS_SAM_B_公開碼訓練設定_三seed_臺灣皆低於B範圍.md`（1–6 正本 `.agents/decisions/20261013_DADS_SAM_B_公開碼訓練設定_設計.md`、`.agents/decisions/20261014_DADS_SAM_B_公開碼訓練設定_seed43_44_設計.md`，未改）。結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_public_recipe/`（S42）、`/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_public_recipe_seeds/`（S43、S44）；log `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_public_recipe_seeds_main_runner.log`。institution 現況檔已回寫（institution commit `18630f5`（S43）、`ed8eb66`（三個 seed））。
- GPU 交接：DADS session 已不佔 GPU，GPU 交給已排隊的馬祖推論（Codex）；P24 Original S44 排在馬祖推論之後（10-14 queue 條目）。

## [2026-10-15] result | P24 Original seed 44 完成（Gom2xRaw seed 44 漏判的對照）
- 10-15 11:14 起被 launch gate 擋（Codex 未 commit 的馬祖 README），使用者指示代 Codex 提交（repo commit `9203882`，內容未改）後啟動；`EXIT=0`，早停／最佳輪 111／61，驗證 mIoU 0.7099，59 分鐘。
- 臺灣 9 景像素加總 Oil IoU 0.4249（P 0.6025／R 0.5904／F1 0.5964；TP 179,276／FP 118,257／FN 124,400／TN 3,510,945）；05-22 兩景 0.4531／0.4497；天使輪 7 景加總 0.3740。
- 05-22 大片油污那一塊召回率：Original S44 0.57；Gom2xRaw S44 兩次 0.03／0.04。周圍油污比例 >50% 的區域召回率：Original S44 0.79；Gom2xRaw S44 0.01–0.03。
- 判讀：seed 44 不多放原圖時抓得到，所以漏判和 Gom2xRaw 設定有關，不是 seed 44 本身。但兩組每輪張數不同（7,240 對 4,502），同一個 seed 打亂出來的順序也完全不同，分不出是「不增強的原圖」還是「另一種順序」造成。單一 run，描述性。
- 紀錄：repo `analysis/p24_global_gom_augmentation/SCOPE_V4.md` 末段；結果 `/home/alanyh/oil_dataset/new/_p24_runs/P24_global_original_S44/`。

## [2026-10-15] launch | DADS-SAM B 只換 loss（0.7 DiceCE ＋ 0.3 Focal）seed 42、43、44 啟動（13:35 Asia/Taipei）
- 使用者授權（原話）：「那我想問問 這會不會是跟loss的有關? 或是batch size?」→「你是說我的baseline 設定 把tversky 換成 原架構的loss?」→「好吧，那就幫我安排下去 跑這個實驗」。未經 Claude × Codex 協商。
- 目的：公開碼訓練設定三個 seed（臺灣像素加總 Oil IoU 0.3043 ± 0.1071）都低於 B（0.4495 ± 0.0178），但兩組同時差九項訓練設定；本組只換 loss 一項，回答「在 B 的其他訓練設定下，把 Tversky 換成 0.7 DiceCE ＋ 0.3 Focal，結果會不會變低」。
- 設定：正式 B block 只改 loss、seed、名稱（另加 `wandb.notes`，不影響訓練）；同一個 main_runner 依序 S42 → S43 → S44，各自以驗證 mIoU 最高的權重做唯一一次臺灣測試。設定檔 `main/experiments_DADS_SAM_gomonly_B_dicece_focal.yaml`，GT_expand repo commit `7eb2c3b`，launch gate PASS；tmux `dads_sam_b_dicece_focal`；wandb S42 run `d5w2sjbn`；結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_dicece_focal/`；log `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_dicece_focal_main_runner.log`（結束時末行 `EXIT=`）。
- 開跑前 smoke PASS（一個 epoch、fp16、顯存 17.1 GiB、5.3 分鐘／epoch）。估每個 seed 11–14 小時，三個依序約 1.5–2 天。期間 GPU 使用中，其他 session 不要啟動 main_runner。
- 報告方式（事前寫定）：主要看 GoM 驗證最佳 epoch 的 Oil IoU／mIoU（對照 B Oil 0.5081／0.5231／0.4824、公開碼設定 0.4333／0.4350／0.4237）；臺灣照例報告但不拿來選 loss；只作描述、不訂門檻。B 與公開碼設定兩組的結果都已看過，不是盲測。
- 目前沒有任何結果。
- 事前紀錄（開跑前凍結）：`.agents/decisions/20261015_DADS_SAM_B_只換loss_DiceCE_Focal_設計.md`；正本為 GT_expand repo `analysis/dads_sam_gomonly_20261005/B_DICECE_FOCAL.md`（commit `7eb2c3b`）。institution 現況檔已回寫（institution commit `e271320`）。

## [2026-10-15] result | 台灣 8 景重測（媽祖 1＋天使輪 7）：P12 0.428 ± 0.002、GomP15 0.423 ± 0.017，兩組差距小於 seed 間波動
- 使用者要求把期末報告的台灣測試改成 8 景：新加入的媽祖 2022-06-22 一景＋天使輪 7 景（拿掉 2026-05-22 兩景）。模型：全資料排除台灣（P12）與全資料＋僅墨西哥灣增量（P24 GomP15），各 seed 42／43／44，用既有權重，只推論不訓練。
- 方法：與原 post-test 相同的 GT-aware 重建；每個 run 內 8 景像素加總算一次指標，跨 seed 取平均與樣本標準差。因 GPU 被 DADS-SAM 訓練占用，改在 CPU 推論；天使輪重測與原 GPU 結果每格差 ≤25 px。
- 結果 Oil IoU：P12 0.428 ± 0.002（0.431／0.426／0.428）；GomP15 0.423 ± 0.017（0.414／0.412／0.442）。兩組差距小於 seed 間波動，不能說增量有幫助。媽祖單景 IoU 0.53–0.64，天使輪 7 景合計 0.36–0.39。
- 程式：repo `OIL_PROJECT_MutiBand_GT_expand` 的 `analysis/taiwan8_matsu_angel_20261015/`（commit `ebd109f`）；輸出：`/home/alanyh/oil_dataset/new/_taiwan8_matsu_angel_20261015/summary.json`。

## [2026-10-16] result | DADS-SAM B 只換 loss（0.7 DiceCE ＋ 0.3 Focal）seed 42／43／44 完成：GoM 驗證與臺灣 Oil IoU 都與 B 的範圍重疊，沒有可分辨的差異（23:45 Asia/Taipei）
- 流程：10-15 13:35 啟動，10-16 23:34 全部結束，log 末行 `EXIT=0`（同一個 main_runner 依序 S42 → S43 → S44，tmux `dads_sam_b_dicece_focal`）。三個 run 都在 patience 50 早停：早停 epoch 158／118／112，最佳 epoch 108／68／62，訓練 829.2／619.0／587.4 分鐘（平均 678.5 ± 131.4）。驗收三個 run 全過：results.csv 全為有限值、cosine 學習率、run_record trained、臺灣 9/9 景 manifest complete、測試權重＝best.pt 且 sha 與 manifest 一致；wandb 三個 run 都 finished、history 列數與 results.csv 逐列一致、summary 有臺灣指標（S42 `d5w2sjbn`、S43 `uuq71rcq`、S44 `3spqtugu`）。
- 授權與設定：使用者 10-15 授權（原話）「好吧，那就幫我安排下去 跑這個實驗」；未經 Claude × Codex 協商。DADS-SAM B（Tversky 那組）只把 loss 換成公開碼的 0.7 DiceCE ＋ 0.3 Focal，其他照 baseline 訓練設定（AdamW 5e-5 cosine、batch 16、增強、clipping 1.0、fp16、最多 300 epoch／patience 50、取 GoM 驗證 mIoU 最佳權重）。設定檔 `main/experiments_DADS_SAM_gomonly_B_dicece_focal.yaml`，GT_expand commit `7eb2c3b`。
- GoM 驗證最佳 epoch（主要）：Oil IoU 0.4810／0.4986／0.4937，平均 0.4911 ± 0.0091；mIoU 0.7261／0.7348／0.7317，平均 0.7309 ± 0.0044。對照 B：Oil 0.5081／0.5231／0.4824（0.5045 ± 0.0206），mIoU 0.7394／0.7468／0.7247（0.7370 ± 0.0112）；B × 公開碼訓練設定（九項同時改）：Oil 0.4333／0.4350／0.4237（0.4307 ± 0.0061），mIoU 0.6985／0.6989／0.6924（0.6966 ± 0.0036）。
- 臺灣 9 景像素加總（每個 run 算一次、跨 seed 取平均 ± 樣本標準差）：Oil IoU 0.5394／0.4671／0.5354，平均 0.5140 ± 0.0406；Precision 0.6083 ± 0.0708、Recall 0.7746 ± 0.0574、F1 0.6783 ± 0.0361、BG IoU 0.9393 ± 0.0110、mIoU 0.7266 ± 0.0255；9 景各算再平均 0.4386 ± 0.0305。混淆矩陣 S42 TP 218,429／FP 101,266／FN 85,247／TN 3,527,936；S43 234,036／197,393／69,640／3,431,809；S44 253,250／169,340／50,426／3,459,862。對照：B 0.4434／0.4695／0.4355（0.4495 ± 0.0178）、公開碼訓練設定 0.1807／0.3709／0.3614（0.3043 ± 0.1071）、基線 P15 Original 0.4808／0.4379／0.3704（0.4297 ± 0.0557）。S44 的 0922 兩景 Precision 約 0.24–0.27、Recall 約 0.99（過度預測）。
- 判讀（照事前規則，只作描述、不訂門檻）：GoM 驗證 Oil IoU 的 S42（0.4810）比 B 最小值 0.4824 低 0.0014，S43、S44 在 B 範圍內 → 重疊 →「只換 loss 時與 B 沒有可分辨的差異」；mIoU 三個都在 B 範圍內。臺灣 Oil IoU 的 S42、S44 高於 B 最大值 0.4695、S43 在範圍內 → 重疊 → 同樣「沒有可分辨的差異」（不是三個全高）；臺灣依事前紀錄不拿來選 loss。
- loss 佔差距的比例＝（B 平均 − 本組平均）／（B 平均 − 公開碼設定平均）：GoM 驗證 Oil 0.18、mIoU 0.15，臺灣 −0.44（本組平均反而高於 B）。描述性結論：公開碼訓練設定與 B 的差距大部分不是只換 loss 造成，而是另外八項設定（optimizer 與學習率排程、batch size、訓練長度與選權重方式、增強、clipping、精度等；同時改、無法再細分）。不可寫成「Tversky 比 DiceCE＋Focal 好」或反之；不可推到其他模型或公開碼設定下；前一組與 10-10 凍結卡的結論不因本組更改。限制：三個 seed、B 與公開碼設定結果已先看過（非盲測）、DiceCE／Focal 權重照公開碼沒調、臺灣 9 景兩事件。
- 待使用者決定：投影片寫法、DADS-SAM 線續跑或收線、是否另外單獨測 batch size 等其他設定、wandb 曲線慢一個 epoch（`main/wandb_logging.py` 的 `run.log(row, step=epoch)` 沒給 `commit=True`，最後一列結束時補寫、無資料遺失，未修）。
- 結案卡：`wiki/experiments/20261016_DADS_SAM_B_只換loss_三seed_與B無可分辨差異.md`（1–6 正本 `.agents/decisions/20261015_DADS_SAM_B_只換loss_DiceCE_Focal_設計.md`，未改）。結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_dicece_focal/`（含 restructured Excel）；log `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_dicece_focal_main_runner.log`。institution 現況檔已回寫（institution commit `f1f0b29`）。

## [2026-10-17] launch | P24 Gom2xRaw seed 44 再重跑三次（rerun2／3／4）啟動（14:30 Asia/Taipei）

- 2026-10-17 14:30 P24 Gom2xRaw（全資料＋GoM 原圖多放一份）seed 44 再重跑三次（rerun2/3/4）已啟動，依序執行。使用者原話：「你可以再幫我多跑個三次seed 44嗎? 我想要觀察到底是怎樣」。目的：看 05-22 大片油污漏判是否每次重現（前兩次 Oil IoU 0.2851、0.2949）。設定與前兩次相同，只差結果目錄與 wandb notes；GT_expand commit 8b18edc，launcher analysis/p24_global_gom_augmentation/launch_v4_s44_rerun234.sh（tmux p24_s44_rerun234），launch gate PASS；結果 /home/alanyh/oil_dataset/new/_p24_runs/P24_global_gom2x_raw_S44_rerun{2,3,4}/。臺灣測試只用原 5 景 9 張（既有設定的重複，便於與前兩次比較）。

## [2026-10-17] result | P24 Gom2xRaw seed 44 三次重跑完成：後三次都正常（18:20 Asia/Taipei）
- seed 44 五次臺灣 9 景像素加總 Oil IoU 0.2851／0.2949／0.4807／0.4388／0.4314（平均 0.3862），S42／S43 為 0.4625／0.4564；05-22 大片油污召回率 0.03／0.04／0.67／0.54／0.66。判讀：漏判不是 seed 44 每次都會發生，是同一設定不同次訓練間的不穩定（Gom2xRaw 7 次中 2 次）；10-15「與 Gom2xRaw 設定有關」的判讀不成立（Original 只 1 次 run）；GoM 驗證 mIoU 看不出差別（0.703–0.713）。三次皆 EXIT=0，GPU 已釋放。紀錄在 GT_expand analysis/p24_global_gom_augmentation/SCOPE_V4.md 末節（commit 85409e1）。

## [2026-10-17] result | P24 Gom2xRaw seed 44 第 5 次重跑（rerun5）完成（22:20 Asia/Taipei）
- 使用者原話「再給他跑一次 下去 這樣等於說重跑五次」；19:15 啟動（GT_expand commit 7b25efc，launch gate PASS），EXIT=0，GPU 已釋放。結果臺灣 9 景像素加總 Oil IoU 0.3785，05-22 大片油污召回率 0.18（部分漏判）；驗證 mIoU 0.7244 為六次最高。seed 44 六次：0.2851／0.2949／0.4807／0.4388／0.4314／0.3785，平均 0.3849 ± 0.0804。判讀不變：同一設定訓練間不穩定，GoM 驗證看不出。紀錄 SCOPE_V4.md 末節（commit 27896ff）。

## [2026-10-19] cleanup | 使用者指示停止並刪除 DADS-SAM 公開碼設定 LR／optimizer 單因子兩組（lr5e5 組、adamw 組）

- 使用者原話（2026-10-19）：「現在有在跑的實驗你先幫我停掉，並且幫我刪掉有關的紀錄好了，我覺得暫時沒有必較在繼續追下去，就先以目前結論來看的話 他原論文上的 Loss方法  在搭上我們的optimizer 與 lr好了 那我覺得是要去測試他的效果，請你去幫我跑跑看全資料訓練全資料測試，也就是3fold的訓練」。
- 被停止的是 10-18 由另一個 session（Codex）啟動的「DADS-SAM 公開碼設定 LR／optimizer 單因子兩組」（`lr5e5` 組與 `adamw` 組）。停止時 lr5e5 組 S42、S43 已跑完，S44 在第 19 個 epoch；adamw 組還沒開始。
- 已刪除：本機結果目錄與 log（`/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_public_lr5e5_20261018/`、`DADS_SAM_B_lr_optimizer_preflight_20261018/`、三個 log），以及 wandb run `spi0l3cz`、`2444ko10`、`n26gkxxe`。git 分支 `codex/dads_lr_optimizer_20261018`（commit `528b6bb`）與其 worktree 保留（刪分支需要另外授權）。
- 這兩組沒有結論，往後不要引用或重跑。不為被刪的實驗建卡。

## [2026-10-19] launch | DADS-SAM ＋ 公開碼 loss（0.7 DiceCE ＋ 0.3 Focal）全資料 3-fold 啟動（18:43 Asia/Taipei）

- 目的：GoM-only 實驗得出 DADS-SAM 用基線訓練設定時，loss 從 Tversky 換成 0.7 DiceCE＋0.3 Focal 在 GoM 驗證上沒有可分辨的差異；使用者決定 DADS-SAM 採「公開碼 loss＋我們的 optimizer 與 lr」，現在測它在全資料 3-fold 上的表現（授權原話見上一則）。未經 Claude × Codex 協商。
- 設定：模型 DADS-SAM 公開碼架構（`dads_sam_public6`，SAM ViT-B，六波段 B02／B03／B04／B8A／B11／B12，prompt 固定整張圖的框）；loss 0.7 DiceCE＋0.3 Focal（γ 2）；其餘照 Prithvi＋DeepLabV3 3-fold 基線 `_dlv3_det`：AdamW（weight decay 1e-3）、lr 5e-5 cosine、batch 16、最多 300 epoch、patience 25（GoM-only DADS-SAM 各組是 50）、取驗證 mIoU 最高的 epoch、資料增強、clipping 1.0、fp16 混合精度；資料 `3_fold_stratified_v2/patch_level_gt_expand`，每折 seed＝42＋折號，每折訓練一次。測試＝各折 test_fold 的 gt_aware 重建（argmax），另存逐像素預測檔。與基線不同的地方：模型、loss、模型專屬學習率（基線 Prithvi encoder 0.1×，DADS-SAM 全部 1×）。
- 啟動前檢查：smoke PASS——三折 train／val／test 的油汙與背景 patch 數、訓練集油汙／背景像素數與基線三折 run_metrics 完全相同（例如 fold 1 訓練 1,168＋2,110 patch）；fold 1 跑 200 步＋完整驗證，loss 有限、顯存峰值 17.1 GiB、6.3 分鐘。
- 啟動：2026-10-19 18:43，launch gate PASS，tmux `dads_sam_3fold`，wandb run `am48y66t`（https://wandb.ai/tree8454-national-central-university/oil-gt-expand/runs/am48y66t）。程式在 GT_expand 的 worktree `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019`（branch `claude/dads_sam_3fold_20261019`，commit `36ed46b`，未併入 main）；設定 `main/experiments_DADS_SAM_3fold_dicece_focal.yaml`，事前紀錄 `analysis/dads_sam_3fold_20261019/CARD.md`。結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_dicece_focal_20261019/`。每 epoch 約 6–7 分鐘，三折粗估 1–2 天。
- 事前寫定的報告方式：各折 gt_aware 重建 Oil IoU（每折內像素加總算一次）再三折平均，對照基線 0.4945／0.4774／0.3479、平均 0.4399；另報全語料 pooled（三折混淆矩陣加總），對照 0.4335；每折另報 P／R／F1／BG IoU／mIoU、混淆矩陣、最佳與停止 epoch、訓練時間。不訂門檻，只描述逐折差與方向。
- 不能主張：差距不能歸因於架構（loss 與模型專屬學習率也不同）；這是交叉驗證不是外部測試（語料含天使輪 7 張臺灣影像），不能主張外部泛化。沒有加臺灣 10 景 19 張測試，因為原臺灣測試集（5 景 9 張）中的天使輪 7 張就在 3-fold 語料裡，測了會洩漏；基線也沒有這項測試。
- 實驗卡：`wiki/experiments/20261019_DADS_SAM_公開碼loss_全資料3fold.md`（狀態執行中；結果欄位待結果，需使用者判斷的第 9 步未填）。

## [2026-10-20] result | DADS-SAM ＋ 公開碼 loss（0.7 DiceCE ＋ 0.3 Focal）全資料 3-fold 三折完成：三折測試 Oil IoU 皆低於基線（20:30 Asia/Taipei）

- 三折皆完成，`EXIT=0`；訓練 process 已結束、GPU 已釋放（2026-10-20 20:16 確認：沒有 main_runner process，GPU 顯存 15 MiB）。每折 loss 皆為有限值、`best.pt` 存在；重建測試的景數與基線各折相同（119／118／118 景），GT 油汙像素數與覆蓋像素數也與基線相同。wandb fold 1 `am48y66t`、fold 2 `58dwjjkl`、fold 3 `ku5a1wiq` 皆 finished，列數＝`results.csv`（41／127／54）。基線數字由 P4 逐像素 atoms（τ 0.5）重算，與正典 0.4945／0.4774／0.3479、0.4335 相同。
- 各折 gt_aware 重建 Oil IoU（每折內像素加總算一次）：本組 0.4431／0.3947／0.2984，基線 0.4945／0.4774／0.3479，差 −0.0514／−0.0826／−0.0495。逐折平均本組 0.3787、基線 0.4399（−0.0612）；全語料 pooled 本組 0.3669、基線 0.4335（−0.0666）。
- Precision／Recall：本組 fold 1 0.736／0.527、fold 2 0.785／0.443、fold 3 0.585／0.379、全語料 0.691／0.439；基線 fold 1 0.633／0.694、fold 2 0.699／0.601、fold 3 0.644／0.431、全語料 0.660／0.558。F1／BG IoU／mIoU：本組 0.614／0.953／0.698、0.566／0.929／0.662、0.460／0.920／0.609，全語料 0.537／0.933／0.650；基線 0.662／0.949／0.722、0.646／0.930／0.704、0.516／0.927／0.638，全語料 0.605／0.935／0.684。
- 停止／最佳 epoch：41／16、127／102、54／29。最佳驗證 mIoU 0.6206／0.7374／0.6956（基線 0.6620／0.7471／0.6713）。訓練時間 261／863／340 分鐘（基線的時間是 CPU 節流期間記錄的，不能比較）。
- 混淆矩陣（TP／FP／FN／TN）：本組 fold 1 1,465,145／526,350／1,314,826／37,354,673；fold 2 1,771,836／486,437／2,230,264／35,517,083；fold 3 1,641,584／1,165,768／2,694,530／44,583,835。基線 fold 1 1,928,513／1,120,035／851,458／36,760,988；fold 2 2,406,318／1,038,644／1,595,782／34,964,876；fold 3 1,867,284／1,031,684／2,468,830／44,717,919。
- 描述（事前紀錄不訂門檻）：三折都低於基線、方向一致，差 −0.05 到 −0.08；三折 Recall 都較低，差距主要來自漏判；fold 3 驗證 mIoU 高於基線但測試較低。每折只訓練一次，無法和訓練間雜訊比較。
- 不可主張：loss 與模型專屬學習率也不同，不能把差距全歸因於架構；這是交叉驗證，不是外部測試（語料含天使輪 7 張臺灣影像）。
- 來源：GT_expand worktree `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019` 的 `analysis/dads_sam_3fold_20261019/CARD.md` 末節與 `results.json`（commit `39e3e45`，branch `claude/dads_sam_3fold_20261019`，未併入 main）；結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_dicece_focal_20261019/`（含 restructured Excel）。institution 現況檔已回寫（institution commit `7f8be8d`）。
- 實驗卡：`wiki/experiments/20261019_DADS_SAM_公開碼loss_全資料3fold.md`（狀態已改為 completed；第 7–8 步已填，第 9 步未填，待使用者）。

## [2026-10-20] design | P26 SPOT6 四波段雙模型單景探索部署：實驗卡 1–6 事前凍結

- 使用者明確指示開始實驗，目標為明天看到 SPOT6 影像測試結果。凍結卡：`.agents/decisions/20261020_P26_SPOT6四波段雙模型設計.md`；P12 全球排除台灣與 P14 GoM-only R1 來源各訓練一個四波段 Prithvi-EO-2.0 300M＋DeepLabV3，單 seed 43；SPOT6 不參與擬合／選模，10 m 全景推論。GPKG 兩個 `Id=1` polygon 為油；標記外 unknown。事前門檻：任一模型標記油污 recall ≥0.50 且有效景預測油面積比例 ≤0.20 為單景值得繼續；兩模型 recall 都 <0.20 或預測油面積比例都 >0.50 為明顯不佳；其餘不可判定。不能計算標記外 FP、precision 或 IoU。兩來源驗證切分不同，結果只作描述。Claude Opus 5 已提供事前諮詢，但未形成正式雙 harness verdict；使用者已授權啟動。

## [2026-10-21] launch | P26 SPOT6 四波段雙模型實驗啟動（00:02 Asia/Taipei）

- 使用者明確授權開始；launch gate PASS，程式 worktree `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_spot6_20261020`，commit `9b2c755c9f9d688f25f62f4986cd606cf567321f`，設定 `main/experiments_P26_spot6_fourband.yaml`。tmux `p26_spot6` 於 00:02 啟動；00:09 檢查 A／全球組訓練進行中，B／GoM-only 組尚未開始。訓練 log `/home/alanyh/oil_dataset/new/_p26_runs/P26_spot6_fourband/P26_train.log`；後處理與驗證已分別排在 tmux `p26_spot6_post`、`p26_spot6_validate`。
- SPOT6 標記 JSON、10 m 四波段 TOA 輸入與 GT 位於 `/home/alanyh/oil_dataset/new/_spot6_4band_20261020/derived/`；兩個油 polygon 合計 575,698 個 10 m 油污像素，JSON／mask 對齊差異 0 像素。事前凍結設計卡 `.agents/decisions/20261020_P26_SPOT6四波段雙模型設計.md`。本條記錄啟動與資料準備，**尚無 SPOT6 推論或測試結果**。

## [2026-10-21] launch | DADS-SAM ＋ 公開碼 loss（0.7 DiceCE ＋ 0.3 Focal）全資料 3-fold 再跑兩次（base seed 43、44）啟動（10:53 Asia/Taipei）

- 授權：使用者指示（2026-10-21，原話）：「你在幫我跑一組好了  多跑個兩次」。解讀（Claude 的）：同一組設定再跑兩次，換 base seed 43、44（專案慣例 S42／43／44），連同 10-19 的 seed 42 共三次；每折實際 seed＝base seed＋折號（43 → 44／45／46，44 → 45／46／47）。未經 Claude × Codex 協商。
- 設定：與 seed 42 那次逐鍵相同，只改 seed、實驗名、重建測試名與 wandb 說明。設定 `main/experiments_DADS_SAM_3fold_dicece_focal_seeds.yaml`（由 `build_seeds.py` 產生），事前卡 `analysis/dads_sam_3fold_20261019/CARD_SEEDS.md`，都在 GT_expand worktree `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019`（branch `claude/dads_sam_3fold_20261019`，commit `dd4ebd9`，未併入 main）。
- 啟動：2026-10-21 10:53，launch gate PASS，tmux `dads_sam_3fold_seeds`，seed 43 fold 1 的 wandb run `kjsmqnjk`。結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_dicece_focal_seeds_20261021/`。兩次依序跑，預估約 2 天（1.5–3 天）；GPU 使用中。
- 事前寫定的報告方式：每次各折 Oil IoU、逐折平均、全語料 pooled 等，用與 seed 42 相同的算法；三次的逐折平均與全語料 pooled 報平均 ± 樣本標準差；對基線（只有一次訓練）只描述三次中幾次的逐折平均低於 0.4399、各折三次範圍是否都低於基線該折（0.4945／0.4774／0.3479）；不訂門檻。
- 限制：基線沒有重複，三次的變動只代表 DADS-SAM 這邊的訓練間變動，不能當兩邊差距的統計檢定。本條只記啟動，**尚無 seed 43、44 的任何結果**；seed 42 的結果與卡內描述不變。
- 實驗卡：`wiki/experiments/20261019_DADS_SAM_公開碼loss_全資料3fold.md`（末尾新增「後續：再跑兩次（base seed 43、44，執行中）」一節）。institution 現況檔 commit `7b603d6`。

## [2026-10-24] result | DADS-SAM ＋ 公開碼 loss（0.7 DiceCE ＋ 0.3 Focal）全資料 3-fold 三次完成：逐折平均 0.3683 ± 0.0118 對基線 0.4399（14:07 Asia/Taipei 跑完）

- seed 43、44 兩次於 2026-10-24 14:07 跑完（`EXIT=0`），GPU 已釋放，連同 10-19 的 seed 42 共三次。六折驗收通過：loss 有限、`best.pt` 存在、測試景數與 GT 像素和基線相同、wandb 六個 run finished 且列數＝`results.csv`。
- 各折 gt_aware 重建 Oil IoU（每折像素加總）：seed 42 為 0.4431／0.3947／0.2984（逐折平均 0.3787、全語料 pooled 0.3669）；seed 43 為 0.4077／0.3383／0.3205（0.3555、0.3479）；seed 44 為 0.4367／0.3722／0.3027（0.3706、0.3603）。三次平均 ± SD（ddof=1）：逐折平均 **0.3683 ± 0.0118**、全語料 pooled **0.3584 ± 0.0097**；基線 Prithvi＋DeepLabV3（一次）0.4945／0.4774／0.3479，逐折平均 0.4399、全語料 pooled 0.4335。
- 逐折平均的其他指標三次平均：P 0.6959／R 0.4382／F1 0.5361／BG IoU 0.9330／mIoU 0.6506（基線 0.6584／0.5752／0.6081／0.9355／0.6877）。
- 描述（不訂門檻）：三次逐折平均都低於 0.4399（3/3）；各折三次範圍（0.4077–0.4431／0.3383–0.3947／0.2984–0.3205）都低於該折基線；Recall 都較低，差距主要來自漏判。DADS-SAM 三次全距 0.023 小於與基線差距 0.072，但基線只訓練一次，不是統計檢定；loss 與模型專屬學習率也不同，不能全歸因於架構；這是交叉驗證，不是外部測試。
- seed 43 fold 3 的訓練時間 1,968 分鐘含 10-22 12:50～10-23 12:28 的 23.6 小時停擺（原因未定），扣除後約 555 分鐘；訓練數值不受影響。
- 來源：worktree `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019` 的 `analysis/dads_sam_3fold_20261019/CARD_SEEDS.md` 末節與 `results_seeds.json`（commit `2ac15c2`，branch `claude/dads_sam_3fold_20261019`，未併 main）；institution 現況檔 commit `085cff7`；結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_dicece_focal_seeds_20261021/`。
- 實驗卡：`wiki/experiments/20261019_DADS_SAM_公開碼loss_全資料3fold.md`（末尾新增「再跑兩次的結果（2026-10-24 補記）」一節；第 9 步仍為未填，待使用者）。

## [2026-10-24] design | P28 B08-dip 來源訓練集診斷第 1–6 步事前凍結

- 使用者要求在訓練集試用 FOCI 並呈現是否有用；本案以 P14 GoM-only R1 frozen 模型唯讀推論，在 14 日期固定 patch 比較 B08-dip proxy 與已標區域 Oil IoU 的關聯，對照平均 entropy。第 6 步門檻已事前寫定，尚未計算 P28 分數或結果，也不訓練模型。
- Sen2Like B08 未核實為原文 Rayleigh-corrected reflectance，GT=1 未確認為純海水，故名稱限於 B08-dip proxy；來源 train 的 in-sample 關聯不得直接稱 target strong／weak 效果。設計正本：`.agents/decisions/20261024_P28_B08_dip來源訓練集診斷設計.md`；`index.md` 已更新。

## [2026-10-24] result | P28 B08-dip 來源訓練集診斷依事前門檻不可判定

- 70 景／593 個 P14 GoM-only R1 source train S2 有油 patch 中 548 個可評分。固定 14 日期各一 patch 中 13 個可評分；`ρ(proxy, 已標區域 Oil IoU)=0.2473`，`ρ(−entropy, IoU)=−0.0055`，差 `+0.2527`。主 rho 未達事前支持線 0.50，也未觸及雙條件否定線，故不可判定；27 patch 敏感度景平均 rho `−0.0769`。
- 卡片 `.agents/decisions/20261024_P28_B08_dip來源訓練集診斷設計.md` 的 1–6 保持凍結原文，7–9 已事後補記；`index.md` 已更新。結果正本在 `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/foci_pilot_20261024/README.md` 與 `/home/alanyh/oil_dataset/new/_euda_optical_metadata_20261017/foci_pilot_20261024/`。本輪為 frozen 模型在自身來源訓練集的診斷，B08 不是已驗證 Rrc、GT1 不是已驗證海水，未做 Stage 2 訓練或 target 驗證。

## [2026-10-24] visualization | P28 全量 70 景／593 patch 圖庫補記

- 依使用者要求，補建 70 個場景頁、593 張逐 patch 圖與 593 列 manifest，另有 dip 高低雙向排序頁。入口：`/home/alanyh/oil_dataset/new/_euda_optical_metadata_20261017/foci_pilot_20261024/all_70_scene_patches_visualized/index.html`；`validation.json` 確認數量與圖連結。已將位置與閱讀限制補到 P28 卡第 7 步，事前第 1–6 步及「不可判定」結論不變。

## [2026-10-24] launch | DADS-SAM ＋ Tversky loss 全資料 3-fold 跑一次（base seed 42）啟動（22:03 Asia/Taipei）

- 授權：使用者指示（2026-10-24，原話）：「我覺得你這邊先幫我改成tversky loss然後跑跑看  先跑個一次好了」。未經 Claude × Codex 協商。
- 設定：DADS-SAM（`dads_sam_public6`）全資料 3-fold，與 DiceCE／Focal 3-fold seed 42 那次完全相同，只把 loss 從 0.7 DiceCE ＋ 0.3 Focal 換成基線用的 Tversky（α 0.3、β 0.7）；跑一次（base seed 42，每折 seed 43／44／45）。因此與 Prithvi＋DeepLabV3 3-fold 基線只差模型與模型專屬學習率（DADS-SAM 全部 1×，基線 encoder 0.1×）。目的：(1) 同一 DADS-SAM 設定 loss 換成 Tversky 後各折測試 Oil IoU 變多少；(2) 換後與基線還差多少。
- 啟動：2026-10-24 22:03，launch gate PASS，tmux `dads_sam_3fold_tversky`，wandb fold 1 `ecepktmm`；fold 1 log 顯示 `TverskyLoss(alpha=0.3, beta=0.7)`，每步約 1.8 秒；粗估 1–1.5 天。未另跑 smoke（資料、模型、流程與 DiceCE／Focal 3-fold 相同且都正常完成；DADS-SAM ＋ Tversky 在 GoM-only B 組三個 seed 也都正常跑完）。
- 事前寫定的報告方式：各折 gt_aware 重建 Oil IoU（每折像素加總）三折平均＋全語料 pooled；主要對照 DiceCE／Focal 同 seed 那次（0.4431／0.3947／0.2984，平均 0.3787；只差 loss），並描述是否落在 DiceCE／Focal 三次範圍內（逐折平均 0.3555–0.3787）；另對照基線（0.4945／0.4774／0.3479，平均 0.4399，pooled 0.4335）。只描述、不訂門檻；只跑一次，自身訓練間變動未知。
- 不可主張：與基線的差距只能歸因於「模型＋模型專屬學習率」整包；這是交叉驗證不是外部測試；沒加臺灣 19 張（天使輪 7 張在 3-fold 語料內會洩漏）。
- 來源：worktree `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019` 的 `analysis/dads_sam_3fold_tversky_20261024/CARD.md`、`main/experiments_DADS_SAM_3fold_tversky.yaml`（commit `58c15c0`，branch `claude/dads_sam_3fold_20261019`，未併 main）；institution 現況檔 commit `17cf011`；結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_tversky_20261024/`。本條只記啟動，**尚無任何結果**。
- 實驗卡：[[20261024_DADS_SAM_Tversky_全資料3fold]]（狀態 running；承接 [[20261019_DADS_SAM_公開碼loss_全資料3fold]]）。

## [2026-10-25] consensus | 光學耀光門檻與 EUDA-PLR Stage II：Codex × Claude Opus 5.5 討論三輪收斂（研究方向與文獻證據邊界共識，非凍結實驗）

- 同一 session 三輪（`ee08ab0c-2c83-4a6c-912b-3be4d6126cda`，requested／actual `claude-opus-5-5`／effort high、CLI 2.1.299、無降級；原生 Web 工具另有 `claude-haiku-5-5` helper，非主模型降級）；R1 Opus 獨立提案、R2 Codex 五項具體反對、R3 Opus `STANCE: ACCEPT`／`BLOCKING: []` 附三處改寫並已採納，Codex 接受修訂版本。首輪非完全雙盲（Opus 已讀共享舊碼稽核）。
- 結論：theta_m／Cox–Munk L_GN 可作候選觀測條件；S2/L8/9 目前無可直接移植的普適單一門檻，不預設全域單調或 U 型；可觀測性／可分性／偽標籤正確性三個研究終點須分開；lineage、角度跨度、確認背景尚待實查，資料不足為不可判定非否證。Stage II entropy＋geometry 精確排序演算法與保留油量不劣性接受條件未解；討論中出現的 coverage=30%、precision 差 +0.05、保留油量相對 80% 僅為示例性設計提案，非已核准判準，本輪未開跑任何新實驗。Pisano R 有原文定義，但僅 MODIS 4 景（3 景同一事件）小規模驗證，不等同已驗證「光學 SAR damping ratio 替代品」。
- 來源：`/home/alanyh/oil_dataset/new/_euda_optical_metadata_20261017/stage_feasibility_20261017/glint_opus55_discussion_20261025/REPORT.md` 與 `consultation_provenance.json`（本輪源文件已經 fresh-context agent 核對 PASS）；decision record `.agents/decisions/20261025_耀光門檻與光學EUDA_Opus55討論.md`；`index.md` 已更新。

## [2026-10-25] result | DADS-SAM ＋ Tversky 3-fold 完成，逐折平均 0.3688（DiceCE／Focal 三次範圍 0.3555–0.3787 內），低於基線 0.4399（18:39 Asia/Taipei 跑完）

- 三折於 2026-10-25 18:39 跑完（`EXIT=0`），GPU 已釋放。驗收通過：loss 有限、`best.pt` 存在、重建測試景數（119／118／118）與 GT 像素和基線相同、wandb `ecepktmm`／`pa56otf0`／`yn29axpk` finished 且列數＝`results.csv`（65／56／63）。
- 各折 gt_aware 重建 Oil IoU（每折像素加總）：本組（Tversky，seed 42）0.4256／0.3671／0.3139，逐折平均 **0.3688**、全語料 pooled **0.3632**；DiceCE／Focal seed 42 為 0.4431／0.3947／0.2984（0.3787、0.3669），差 −0.0176／−0.0276／+0.0155（逐折平均 −0.0099、pooled −0.0037）；DiceCE／Focal 三次範圍逐折平均 0.3555–0.3787、pooled 0.3479–0.3669；基線 Prithvi＋DeepLabV3 0.4945／0.4774／0.3479（0.4399、0.4335），差 −0.0689／−0.1103／−0.0340（逐折平均 −0.0711、pooled −0.0703）。
- 逐折平均 P 0.5969／R 0.5041／F1 0.5373／BG IoU 0.9250／mIoU 0.6469（DiceCE／Focal S42 0.7017／0.4494／0.5466／0.9341／0.6564；基線 0.6584／0.5752／0.6081／0.9355／0.6877）。停止／最佳 epoch 65／40、56／31、63／38；訓練 415／382／398 分鐘。
- 描述（不訂門檻）：與 DiceCE／Focal 同 seed 比，fold 1、2 較低、fold 3 較高，方向不一致；三折與逐折平均都落在 DiceCE／Focal 三次範圍內，單次結果下 loss 換成 Tversky 的效果與重跑一次的變動分不開（與 GoM-only「B 只換 loss」結論同向）。Tversky 這次 Precision 較低、Recall 較高，兩者抵銷，Oil IoU 相近。與基線比三折都較低，剩下的差別是「模型＋模型專屬學習率」整包，不能單獨歸因於架構；本組只跑一次，自身訓練間變動未知；這是交叉驗證，不是外部測試。
- 來源：worktree `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019` 的 `analysis/dads_sam_3fold_tversky_20261024/CARD.md` 末節與 `results.json`（commit `8de1b42`，branch `claude/dads_sam_3fold_20261019`，未併 main）；institution 現況檔 commit `e92b02e`；結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_3fold_tversky_20261024/`。
- 實驗卡：[[20261024_DADS_SAM_Tversky_全資料3fold]]（狀態 completed；第 7–8 步已填，第 9 步仍為未填，待使用者）。

## [2026-10-26] experiment | 建立 P29 光學 EUDA Stage 1 設計草案，尚未凍結或啟動

- 新卡：[[20261026_P29_光學EUDA_Stage1來源對抗設計]]；第 1–6 步為 `DRAFT_prelaunch`，包含 GoM source-only 對照、19 張臺灣 image-only target、三個配對 seed 與探索性數字門檻。正式 GPU smoke、磁碟 checkpoint 回載、target GT 隔離與設定讀回仍須驗收；本筆不宣稱已啟動訓練。

## [2026-10-26] experiment | P29 Stage 1 第 1–6 步預註冊凍結，GPU 預檢通過

- 卡片 [[20261026_P29_光學EUDA_Stage1來源對抗設計]] 狀態由 `DRAFT_prelaunch` 改為 `FROZEN_prelaunch`；repo commit `4cc96fb`、正式 YAML SHA-256 `49f8a6834d5633c24d6516c507dcee3d5a103e233900f41bf3d46cfd0dc84934`，六臂皆 `run:true`。GPU A/B seed 42 各單步 AMP batch16 預檢通過，初始 source loss 相同、loss／梯度有限、磁碟 checkpoint strict 回載；target 僅讀 image-only TSV。預檢證據：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/euda_stage1_20261025/GPU_SMOKE_20261026.md`。本筆凍結時**尚未啟動正式訓練**；正式 process 狀態須另行實查記錄。

## [2026-10-26] experiment | DADS-SAM 全資料（排除臺灣）訓練、測臺灣 19 張啟動（launch）

- 啟動 2026-10-26 21:46 Asia/Taipei；約 10.5 分／epoch，粗估 10–23 小時。P12 有效 seed 42 的 block 只把模型換成 DADS-SAM（`dads_sam_public6`，無 encoder 0.1× 學習率），其餘同 P12；跑一次。臺灣測試＝原 9 張＋新 9 張＋馬祖 1 張＝19 張（270 patch，`taiwan19_test.txt` SHA256 `3fbf624d160bcf040a1c0fc3a5e05dd4d2a643acaa9f6f8feaa007d31bfb8875`），訓練後只用 `best.pt` 測一次。非盲測；不訂門檻、只描述；未經 Claude × Codex 協商。**目前沒有結果。**
- worktree `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_dads3fold_20261019`（commit `7eabc92`）；tmux `dads_sam_global_taiwan19`；wandb `lva93t7a`；結果目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_global_taiwan19_20261026/`；log `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_global_taiwan19_20261026_main_runner.log`；institution 現況檔 commit `228aa9f`。
- 實驗卡：[[20261026_DADS_SAM_全資料排除臺灣_臺灣19張]]（狀態 running；第 7–9 步待結果）。

## [2026-10-27] result | DADS-SAM 全資料（排除臺灣）訓練、測臺灣 19 張完成，19 張像素加總 Oil IoU 0.5815（原 9 張 0.4796、新 10 張 0.6227）

- 2026-10-27 訓練完成 `EXIT=0`，GPU 已釋放：早停於第 69 epoch，最佳第 44 epoch（驗證 mIoU 0.6533、Oil IoU 0.3607），訓練 737.5 分鐘；臺灣以 `best.pt` 測一次（10:04），19／19 張都有預測檔，預測檔加總與重建 `run_metrics.json` 一致；wandb `lva93t7a` finished，70 列＝`results.csv` 69 列＋1 列推論報表。
- 臺灣像素加總：19 張 Oil IoU 0.5815／P 0.6599／R 0.8303／F1 0.7354／BG IoU 0.8911／mIoU 0.7363（TP 1,169,368／FP 602,669／FN 238,927／TN 6,889,862）；原 9 張 0.4796／P 0.5021／R 0.9142／F1 0.6482／BG IoU 0.9176／mIoU 0.6986（TP 277,631／FP 275,259／FN 26,045／TN 3,353,943；各算再平均 0.4081 僅供參考）；新 10 張 0.6227／P 0.7314／R 0.8073／F1 0.7675／BG IoU 0.8675／mIoU 0.7451（TP 891,737／FP 327,410／FN 212,882／TN 3,535,919）。
- 描述（不訂門檻）：原 9 張比 P12 同有效 seed 42 那次（0.3978）高 0.082，P12 三次範圍 0.3821–0.4789，本組略高於其最大值；單次、自身訓練間變動未知，差異只能歸因於「模型＋模型專屬學習率」整包；非盲測。原 9 張 Recall 高、Precision 低：天使輪 7 張 Precision 0.19–0.60（09-22 兩張約 0.2、Recall≈1），預測範圍大於 GT；05-22 兩張 Oil IoU 0.540／0.534。新 10 張各張 0.499–0.667，P12 無同一套數字、無直接基線對照。與 3-fold 結果（DADS-SAM 低於基線）方向相反，但測試集與比較基線都不同，不能直接對比。
- worktree commit `d23140f`（`analysis/dads_sam_global_taiwan19_20261026/` 的 `CARD.md`、`results.json`、`summarize_results.py`）；結果 post_test 目錄 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_global_taiwan19_20261026/20261026-214606_MS6_DADS_SAM_global_Taiwan19_eff42_fold1/post_test_20261027-100434_DADS_SAM_global_Taiwan19_reconstruction/`；institution 現況檔 commit `3560399`。
- 實驗卡：[[20261026_DADS_SAM_全資料排除臺灣_臺灣19張]]（狀態 completed；第 7–8 步已填、第 9 步未填待使用者）。

## [2026-10-28] decision | P30 光學 EUDA Stage III EMA 設計九步驟卡第 1–6 步事前凍結（20:12 Asia/Taipei）
- 方法共識：repo `analysis/euda_stage3_20261028/claude_round3_response.md`（Claude Opus 5／high × Codex 三輪收斂，R3 ACCEPT、BLOCKING=[]）；程式核對 `claude_code_check_round2_response.md`（ACCEPT）；正式 launch review `claude_launch_review_response.md`（ACCEPT，無 blocking）。使用者明示要求掛 tmux 開始本次實驗；本卡凍結時正式訓練尚未啟動。
- 設計：Stage I S42 `best.pt`（SHA `140f1d86…`）起點，沿用既有 Stage II 全域 256-window entropy 排序（λ.67，easy18,482／6 預測油，hard9,104／2,426）；Stage III target-only easy CE＋hard U 對抗（權重.001），EMA α.99 只更新 teacher 參數、buffers 固定於 S42；batch16 easy＋16 hard；正式預算 5,000 次成功 student 更新；最終 checkpoint＝預算打滿的 last student；不用臺灣 GT 或 source val 選模。固定終點：臺灣 19 張、270 個既有 GT-aware 視窗，frozen S42 vs last student 配對跑同一套既有重建；主指標 `recon(gt_aware)` per-image macro Oil IoU，pooled 副，另列 raw255 排除後 known-label-only 指標；歷史口徑非真全景 precision，屬非盲 transductive 評估，無 source-continuation control，不可歸因單一機制。
- 數值判準（探索性續做邊界，非統計顯著性）：primary Δ≥+.03 且已標油 recall Δ≥−.02 → 支持續做；primary Δ≤−.03 → 否定本原型於此終點優先性；其餘不可判定。
- GPU batch16 真實預檢（`GPU_PREFLIGHT_BATCH16.json`）：100 次成功更新／102 attempt、2 次初期 AMP overflow、最後 20 次無 overflow，peak reserved 12,320,768,000 bytes（12.32GB）／36.6%，teacher EMA／buffers frozen／checkpoint 回載四項 PASS；正式訓練一律用全新 S42，不沿用預檢權重。
- 紀錄：卡 `.agents/decisions/20261028_P30_光學EUDA_StageIII_EMA設計.md`（狀態 FROZEN，第 7–9 步未填）；程式與設定正本 `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/euda_stage3_20261028/`、正式 YAML `main/experiments_EUDA_stage3_20261028_run.yaml`；最終 YAML SHA 與訓練時 repo commit 由啟動後的 launch receipt 記錄，本卡未猜測。index.md「實驗紀錄」清單已插入最上方；`.agents/handoff.md` 已補 STATUS_AS_OF。vault 本次未 commit／push（需使用者另行授權）。

## [2026-10-28] result | P30 Stage III EMA 基準正式訓練跑滿，固定終點全背景塌縮，依事前門檻否定本原型
- 正式訓練：5,000 次成功 student 更新／5,002 attempts／2 次 skip（初期 AMP overflow）／`EXIT=0`。固定臺灣 19 景／270 既有 GT-aware 視窗終點：scene-macro historical Oil IoU S42 **.4708761605→0**，pooled **.4585928357→0**，last student 最終油預測像素 0。online teacher 5,000 批中僅 step50／attempt52 一批含油；原 S42 參考 easy 18,482 窗僅 6 窗預測有油，精確重建實際抽到這 6 窗 26 次／26 批。
- 依第 6 步事前門檻（primary Δ≤−0.03 → 否定本原型於此終點優先性）：Δ=−.4708761605，觸發**否定**；範圍限定於此配置於此終點，不概括為完整原文 EUDA-PLR 失效。
- 純 endpoint BN 假象檢查（CPU FP32 固定5窗 swap、實際 attempt52 的 easy16／hard16 BN-mode 對照）不支持「只恢復末期 BN 統計即可救回」，但未排除訓練歷程中的 BN 作用。
- 結案卡：`wiki/experiments/20261028_P30_StageIII結果與診斷.md`（第 7–9 步，1–6 正本未改動）。

## [2026-10-28] consensus | P30 Stage III 背景塌縮：Codex × Claude Opus 5／high 三輪診斷討論收斂（限縮描述性共識，非新凍結實驗）
- 同一 session `1f3947c9-3ebe-4c06-a217-f64dd7f0aa5c`，requested／actual `claude-opus-5`／effort high、CLI 2.1.312、無 fallback。R1 STANCE REVISE（2 項 blocking：BN 漂移 vs 方法退化未分離、缺崩潰時點證據）→R2 STANCE ACCEPT 但 blocking 仍非空（依新 CPU／實際 batch 探針撤回 R1「BN 優先」排序，另撤回 3 項不受支持說法）→R3 STANCE ACCEPT／BLOCKING=[]。
- 最終結論：較有支持的機制候選為 easy 全背景 CE 自蒸餾＋online EMA 追隨致稀少油 pseudo-label 消失，hard U（二分類對稱）結構上無法保留油類；純 endpoint BN 假象不受支持，訓練歷程中 BN 因果作用未排除，各項貢獻未分離，無塌縮起點。下一步僅排序為先查 Stage II 油候選監督與無 GT 監測、早期 checkpoint，未核准任何新訓練 recipe 或重跑；方法或 launch 另議。
- 紀錄：`.agents/decisions/20261028_P30_StageIII背景塌縮診斷.md`（不取新 P 編號、不是新實驗卡、不重新凍結方法）；P30 事前方法卡 `.agents/decisions/20261028_P30_光學EUDA_StageIII_EMA設計.md` 第 1–6 步內容未改動。index.md「實驗紀錄」清單已插入最上方；`.agents/handoff.md` 已補 STATUS_AS_OF 與「共識紀錄」。vault 本次未 commit／push（需使用者另行授權）。

## [2026-10-28] decision | P31 光學 EUDA GT位置ROI StageIII原型九步驟卡第1–6步事前凍結（另立新卡，沿用P30後續）
- 1–6原文保留自repo正本 `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/euda_stage3_roi_20261028/CARD.md`；本次只改header與狀態為FROZEN、補凍結時間，正式StageIII尚未啟動，第7–9步待末次終評。
- 設計摘要：target改用baseline既有臺灣19景270個GT-centric座標（與P30全grid排序精確座標0重疊），S42起點與StageIII配方（EMA.99、easyCE+hardUadv.001、batch16+16、5000成功更新）同P30不變，不重訓StageI；主estimand=19景scene-macro歷史`recon(gt_aware)` OilIoU，支持續作門檻Δ>=+.03且known-oil recallΔ>=−.02、否定門檻Δ<=−.03。
- 方法協商：ClaudeCLI2.1.312 Opus5／high requested=actual，session `62ee601c-91d6-49ee-8f47-8b1e2650d805`，R1 REVISE→R2 ACCEPT／BLOCKING=[]，Codex已接受；原始prompt/JSON/response/usage與`METHOD_CONSENSUS.json`在repo同目錄。工程GPU10步PASS、CPU15測試通過，但fresh驗證/程式readback仍進行中，不稱全部launch gate已過。
- 紀錄：卡 `.agents/decisions/20261028_P31_光學EUDA_GT位置ROI_StageIII設計.md`（狀態FROZEN）；index.md「實驗紀錄」已插入最上方；`.agents/handoff.md` 已補STATUS_AS_OF。vault本次未commit／push（需使用者另行授權）。

## [2026-10-28] result | P31 光學 EUDA GT位置ROI StageIII原型完跑並補寫第7–9步（rejects_this_prototype）
- 22:25開跑、22:50完跑，`EXIT=0`，5000成功步、skipped=2；GT_expand commit `32fcd96`；W&B run `daiyal5q`；270窗終評（同一批窗口、固定last student）per-image macro OilIoU：S42 0.4709→P31 last 0.0925，Δmacro=−0.378，依卡上事前判準 exploratory_outcome=rejects_this_prototype；pooled OilIoU 0.4586→0.1269（P 0.695→0.131，R 0.574→0.791），預測油像素1,164,126→8,480,159。
- teacher偽標籤油比例：step100 8.8%→1000 17.6%→2000 38.0%→3000 68.0%→3500 98.5%，之後>90%（step5000 91.6%）。解讀（單seed、探索性）：P30崩全背景、P31崩近乎全油，無source錨定的EMA＋online argmax自訓練往一邊漂、取樣位置只決定方向；因果未分離。下一步待使用者決定。
- 紀錄：卡 `.agents/decisions/20261028_P31_光學EUDA_GT位置ROI_StageIII設計.md` 狀態改FROZEN → COMPLETED 2026-10-28 22:50，第1–6步未改；`.agents/handoff.md` 置頂補STATUS_AS_OF。vault本次未commit／push。

## [2026-10-29] decision | P30/P31 塌縮後續 V32/V33 Claude×Codex 共識（未授權實作／開跑）
- Claude 主持、Codex `gpt-5.6-sol`（session `01a1836c-13f5-7e70-b4bd-7efcf1f02dc9`）三輪：R1 盲提案、R2 REVISE、R3 ACCEPT，BLOCKING 無。
- 診斷：P30（全背景）與 P31（全油）是同一個無錨點線上自訓練迴圈（逐像素 argmax 硬標籤、EMA teacher 跟隨 student、無 source 錨點、U-adversarial 類別對稱）的兩個吸引子；混淆因子=每個 easy 視窗曝光 P30 4.3 vs P31 444。
- 計畫：Step 0 無訓練診斷 → V32（S42 argmax cache 固定 5000 步、teacher 不改寫標籤）→ V33（V32＋.8/.2 對稱信心棄權）；q_t/q0 護欄（[0.25,4]，連續兩次越界即中止）；終點只取 last student；臺灣 GT 待 V32、V33 封存後才讀一次。單 seed 探索性，為 GT 位置輔助機制診斷、非 GT-free UDA；GT-free 主線（V35 等）另立卡。
- 紀錄：`.agents/decisions/20261029_P31塌縮後續_V32V33共識.md`；index.md「實驗紀錄」已插入最上方；`.agents/handoff.md` 置頂補STATUS_AS_OF。vault本次未commit／push。

## [2026-10-29] decision | P32 光學 EUDA GT位置ROI全流程設計凍結（未開跑）
- 事前卡第 1–6 步凍結：臺灣 270 窗為 Stage I/II/III target，Stage I 對抗重訓 S42/43/44 → entropy 分組 → 固定 Stage I argmax 偽標籤 Stage III；Claude×Codex 設計與程式審查皆 APPROVE，BLOCKING 無。
- 未授權開跑；第 7–9 步結案才填。紀錄：`.agents/decisions/20261029_P32_光學EUDA_GTROI全流程設計.md`；index.md「實驗紀錄」已插入最上方；`.agents/handoff.md` 置頂補 STATUS_AS_OF。vault 本次未 commit／push。

## [2026-10-29] decision | P33 光學 controlled IntraDA-style 對照組設計共識（未授權實作）
- Claude×Codex（gpt-5.6-sol，R1 盲稿→R2→APPROVE，BLOCKING 無，confidence .97）：補 EUDA-PLR Table 11 的 IntraDA baseline；對 P32 單因子（判別器輸入 U 圖 1ch→AdvEnt prob_2_entropy 2ch）；Δ_U 預註冊 ±.03；P32 phase C 延後、與 P33 合併單次評估；P33 程式走獨立 worktree，三 seed 封存後才合併。
- 未授權實作／開跑。P32 卡未改動（repo yaml 綁 vault_card_sha256）。紀錄：`.agents/decisions/20261029_P33_光學IntraDA對照組設計共識.md`；index.md「實驗紀錄」已插入最上方；`.agents/handoff.md` 置頂補 STATUS_AS_OF。vault 本次未 commit／push。

## [2026-10-29] decision | P32 Stage III 凍結 addendum（phase A 產物登錄；開跑已授權）
- 新增補充卡登錄 phase A 實際產物，設計不變：Stage I 三 seed 皆 trained、target_retries 0（GoM source-val best_epoch S42 91／S43 64／S44 79）；Stage II 三 seed strong 180／weak 90、tie_rate 0。Stage III 每 seed 綁定 Stage I best.pt sha 與 Stage II input_provenance sha（yaml 由 build_stage3_config.py 產生）；method fixed_stage1_argmax、5000 步 last、guard [.25,4] 連兩次、post_tests 空。開跑前隔離 10 步 GPU smoke（無 GT）。
- 使用者已授權 P32 A–C 一併開跑；P33 共識提議延後 phase C 與 P33 合併單次讀 GT，尚待使用者決定，phase B 不受影響。P32 事前卡未改動。
- 紀錄：`.agents/decisions/20261029_P32_StageIII凍結addendum.md`；index.md「實驗紀錄」已插入。vault 本次未 commit／push。

## [2026-10-29] experiment | P32 Stage III bug-fix 補充卡（fork 後 GDAL handle 共享；phase B 14:13 開跑）
- 隔離 GPU smoke 首次在 Stage III 第一批 target 讀取失敗（RasterioIOError TIFFReadEncodedStrip，`20230918_L9_50QRL` B01；同檔單獨讀正常）。根因：P32 新增 `load_probe` 在主程序經 `deeplab_adapter` 模組層 rasterio 快取讀 270 窗、handle 留快取，DataLoader 4 worker fork 後共用同一 GDAL dataset 與檔案 offset，並行讀取互相干擾（可能靜默讀錯）。P31／Stage I／Stage II 不受影響，phase A 結果有效。失敗產物留 `_smoke_failed_fdshare`，不得列入正式 run discovery／registry。
- 修正 commit `ee9610f`（僅 `main/euda_stage3.py`＋新測試；回歸 27＋9＋12＋6 通過；Codex R4 APPROVE .98；launch_files_stage3.sha256 已重產）；設定不變（config 凍結 `61034a1`）。重跑 smoke 7 項全過（q/q0 第 5 步 1.014、第 10 步 0.980；cache 180 窗、油比例 .029；峰值顯存 11.5 GiB）。
- Phase B 於 14:13 以 `ee9610f` 開跑（tmux `p32_phaseB`）；registry 訓練碼須記 `ee9610f`。P32 事前卡與凍結 addendum 未改動。紀錄：`.agents/decisions/20261029_P32_StageIII_bugfix_addendum.md`；index.md「實驗紀錄」已插入；`.agents/handoff.md` 置頂補 STATUS_AS_OF。vault 本次未 commit／push。

## [2026-10-29] experiment | P32 phase B（Stage III×3）完成
- 14:13–15:09（約 18 分/seed），commit `ee9610f`，三 seed 皆 trained、5000 步、guard 全程無越界。
- 臺灣 270 窗油像素比例 q/q0（student vs Stage I，非 GT）最後值：S42 .884、S43 .793、S44 .635（S44 在 step 2000→2500 由 .79 降至 .65 後持平）。teacher/cache 不一致率 ≤ .0011。
- main_runner 預設評估跑的是 GoM 驗證集（`GoM/R1/val_oil.txt`，39 景），未讀臺灣 GT。同一評估器 Oil IoU：Stage I best → Stage III last：S42 49.8→44.3、S43 48.2→36.4、S44 49.6→35.3（Stage III 無 source loss，屬預期的 source 遺忘；此 GoM 集也被 Stage I 選模用過；僅描述、不入任何判定）。
- P32 模型已封存；phase C（讀臺灣 GT）待使用者決定是否延後、與 P33 合併單次評估。vault 本次未 commit／push。

## [2026-10-29] decision | P32 phase C 延後與 P33 聯合評估；P33 事前卡凍結
- 使用者選方案 2：P32 phase C（凍結 registry、讀臺灣 GT）延後，待 P33 三 seed 跑完並封存後，P32＋P33 用單一聯合 registry、單一 evaluator、臺灣 GT 只讀一次。P32 主指標、門檻（±.03、≥2/3 seed）、method failure 定義、比較對象（matched P29 source-only S1）、checkpoint（Stage III last）不變，只改 GT 開封時間點與 registry 範圍。凍結 addendum：`.agents/decisions/20261029_P32_PhaseC延後聯合評估addendum.md`；P32 事前卡與兩張既有 addendum 未改動。臺灣 GT 至今未讀。
- P33 程式完成於分支 `claude/p33_intrada_entropy_20261029`（commits `82d1290`、`6b1cbb1`，自 P32 `ee9610f`）；Codex 程式審查 R1 APPROVE .96、無 BLOCKING。P33 事前卡已凍結：`.agents/decisions/20261029_P33_光學IntraDA對照組事前卡.md`。**P33 訓練尚未授權。**
- index.md「實驗紀錄」已插入兩條；`.agents/handoff.md` 置頂補 STATUS_AS_OF。vault 本次未 commit／push。

## [2026-10-29 23:46] experiment | P33 光學 IntraDA 對照組 phase A／B 訓練完成，待授權聯合讀 GT
- phase A（19:58–22:48，commit `efcf94b`）：Stage I×3 皆 prob_2_entropy／D 2 通道／17,100 步，launch gate PASS；best epoch S42 78／S43 85／S44 97；GoM 驗證 Oil IoU S42 .5179、S43 .5131、S44 .5050；Stage II 三 seed 皆 easy 180／hard 90。
- Stage III config 凍結（`main/experiments_EUDA_stage3_intrada_entropy_20261029.yaml` ＋ `launch_files_stage3.sha256`，commit `8bfe1d2`），隔離 GPU smoke 10 步 10/10 項通過。
- phase B（22:49–23:45，commit `8bfe1d2`）：三 seed 皆 trained、5000 步、無 guard_failure；臺灣 270 窗油像素比例 q/q0（非 GT）最後值 S42 .824／S43 .593／S44 .903，guard 讀數全程在 [.25, 4] 內。
- 臺灣 GT 仍未讀；P32＋P33 三 seed 模型皆已封存；聯合 registry／evaluator 經 Codex joint review R2 APPROVE .98（commit `1218aa0`）。待使用者授權：分支 fast-forward 併回 main → `build_joint_registry.py` 凍結 18 筆 registry 並人工核對 → 單次 `final_evaluate_joint.py`。
- 四張凍結卡（P33 事前卡、P32 主卡、P32 Phase C 延後 addendum、P32 Stage III 凍結 addendum）未改動；`.agents/handoff.md` 置頂補 STATUS_AS_OF，`index.md` 兩條 P33 條目狀態已更新。vault 本次未 commit／push。

## [2026-10-30] experiment | P32＋P33 聯合讀臺灣 GT：P32 rejects、P33 rejects、+U inconclusive
- 使用者授權讀 GT 後：P33 分支 fast-forward 併回 main（`8bfe1d2`，24 項測試過）→ `build_joint_registry.py` 凍結 18 筆 registry（全部 completed，sha256 `ddd39de1cb8823511897119657a1a8bfe3fa949a0f58c1ca7a5e5fef4eab6c26`，人工逐筆核對）→ `final_evaluate_joint.py` 單次讀臺灣 GT（EXIT=0，輸出 `/home/alanyh/oil_dataset/new/_intrada_ablation_20261029/joint/evaluation/JOINT_evaluation.json`）。臺灣 GT 已讀，不得再為選模／調參重讀。
- 主指標（270 窗重建覆蓋區 union pooled OilIoU，三 seed 平均）：P29 source-only 0.4115；P29 full-grid adv（歷史參照、不入判定）0.4432；P32 Stage I 0.2230、Stage III last 0.1894；P33 Stage I 0.2211、Stage III last 0.1743。
- 預註冊判定：P32 rejects（Δ −0.2221；−.221/−.212/−.233）；P33 rejects（Δ −0.2372；−.226/−.226/−.259）；+U（P32 S3−P33 S3）inconclusive（mean +0.0151；+.006/+.014/+.026，三 seed 皆正但未達 .03）。
- 機制（描述性）：GT-ROI Stage I 適應本身 P32 −0.189、P33 −0.190；Stage III 再降 P32 −0.034、P33 −0.047；Stage I 兩種表示差 +0.002。
- 錯誤型態：預測油／GT 油像素比 source-only 0.774、full-grid adv 0.798、P32 S1 0.366、P32 S3 0.281、P33 S1 0.360、P33 S3 0.268；precision 各組約 .67–.73，recall 由 ~.52 掉到 ~.19–.25 → 主要失敗是大量漏判（預測油量只剩 GT 的 3 成左右），Stage III 更嚴重。
- 結論：GT 位置輔助的 Stage I 對抗適應（U 圖或 entropy 圖）對臺灣是大幅負效果，兩種判別器輸入差異小；原因未定，不得用這批臺灣數字反推新規則。P34 以 P32 為基礎的前提需重新檢討，待使用者決定。
- 凍結卡（P33 事前卡、P32 主卡、P32 addendum 等）未改動；`.agents/handoff.md` 置頂補 STATUS_AS_OF，`index.md` P32／P33 條目狀態已更新。vault 本次未 commit／push。

## [2026-10-30] cleanup | 光學 EUDA 全景網格窗模型刪除（共 18G，不可復原）：路線只保留 GT-centric 270 窗
- 使用者 10-30 裁決：光學 EUDA 只保留 target 用 GT-centric 窗（270 窗）的路線；target 用臺灣全景網格窗（27,587 窗，含陸地和雲）的模型全部刪除。
- 已刪除（`rm -rf`，共 18G，不可復原）：
  - P29 全景對抗 Stage I 三個 run：`/home/alanyh/oil_dataset/new/_euda_stage1_runs/EUDA_STAGE1_20261025/` 底下的 `*uncertainty_adv_S42`、`*uncertainty_adv_S43`、`*uncertainty_adv_S44`。
  - P30 Stage III run：`/home/alanyh/oil_dataset/new/_euda_stage3_runs/EUDA_STAGE3_20261028/20261028-201839_MS6_EUDA_S3_entropy067_EMA_S42`。
  - P30 preflight：`/home/alanyh/oil_dataset/new/_euda_stage3_runs/preflight_20261028_batch16`。
- 保留：
  - P29 source-only S42/S43/S44（`_euda_stage1_runs/EUDA_STAGE1_20261025/` 內三個 `*source_only_S4x`），是 P32／P33 的對照組。
  - P30 的 `collapse_discussion_20261028/`、`main_runner.log`、`launch_receipt.json`（log 與 receipt）。
  - P31 GT-ROI Stage III。注意：它的 init checkpoint 是已刪掉的全景 S42，源頭已經拿不到。
  - `joint/evaluation/fullgrid_adv_s1_*` 的預測和疊圖，當歷史紀錄。10-30 registry 裡 3 筆 fullgrid 已經不能從 checkpoint 重算。
- 之後的比較表不再列全景對抗（0.443）。現行比較：臺灣 270 窗，每個 run 內像素加總算 IoU，三個 seed 取平均：

  | 組 | IoU | 預測油／GT 油 |
  |---|---|---|
  | source-only | 0.411±0.018 | 0.77 |
  | P32 Stage I | 0.223 | 0.37 |
  | P32 Stage III | 0.189 | 0.28 |
  | P33 Stage I | 0.221 | 0.36 |
  | P33 Stage III | 0.174 | 0.27 |

  各組 precision 都約 0.7。
- 漏判機制假說（**未證實**）：source batch 有 57% 是純背景 patch（1550/2738，油像素佔 9%），target 的 270 窗則全部含油；判別器因此可以拿「有沒有油」來分 domain，對抗梯度只經過 target，結果是壓低臺灣的油預測。佐證：GoM 驗證集的預測／GT 從 source-only 的 0.85–0.92 降到 GT-ROI 的 0.78–0.82，IoU 沒有掉；臺灣則從 0.77 降到 0.36。此為假說，尚無對照實驗驗證。
- institution 紀錄已寫在 `research/oil_spill_project_status.md`，commit `bb709d3`。
- 凍結卡（P32 主卡、P32 Stage III addendum、P33 事前卡、P32 Phase C 延後 addendum、P29 設計卡、P30 設計卡）未改動；`wiki/experiments/20261028_P30_StageIII結果與診斷.md` 頁尾追加「2026-10-30 模型檔已刪除」附註（只追加）；`.agents/handoff.md` 置頂補 STATUS_AS_OF。vault 本次未 commit／push。

## [2026-10-30] experiment | P35 LFMDA 凍結原型教師可行性事前卡（1–6，pre-registered，尚無結果）
- 新增 `wiki/experiments/20261030_P35_LFMDA凍結原型教師可行性_事前卡.md`（status in-progress）；第 1–6 步原文取自 repo `analysis/p35_pcd_teacher_feasibility_20261030/CARD.md`（SHA-256 `283522f2…6584`），未改判準與數字；第 7–9 步留白。
- 範圍：只管 A（兩個凍結 Prithvi teacher 原型訊號，不更新權重）；PCD 訓練 B 須另立卡。Go／No-go／不可判定判準見卡第 6 步。
- `index.md` 新增條目；`.agents/handoff.md` 置頂補 STATUS_AS_OF。未執行任何實驗、未讀臺灣 GT。vault 本次未 commit／push。

## [2026-10-30] correction | P35 事前卡同步 repo CARD 現行版與程式 SHA
- 更正上方同日 P35 條目所記舊 SHA（該條原文依 append-only 保留）：repo `CARD.md` 現為 FROZEN，SHA-256 `da141cf0d779a7495215251c243a022104c290b170d924d4ab13ba78ddceb868`（舊 `283522f2…6584`）；`run_feasibility.py` 現為 SHA-256 `0bc5ba3c7d4314038bb98bdd0d87a99c63cd022a0798cdeed044f62af6c02324`（新增 `valid_windows` 記錄；舊 `26702798…41f6`）。
- `wiki/experiments/20261030_P35_LFMDA凍結原型教師可行性_事前卡.md` 第 5 步已同步 GoM R1 Train／Val「整窗影像無效像素 >10% 即排除並記數」規則與「約 20/39」用語，並更新兩個 SHA；第 7–9 步仍未填，正式實驗尚未執行。
- `.agents/handoff.md` 的 P35 STATUS_AS_OF 條目同步改為新 SHA。vault 本次未 commit／push。

## [2026-10-30] result | P35 A 凍結原型教師可行性：事前 gate GO、teacher=S42，但不優先作改善方案
- 正式輸出 `/home/alanyh/oil_dataset/new/_p35_pcd_feasibility_20261030/20261030-1145_A_frozen_prototype/result.json`；報告 repo `analysis/p35_pcd_teacher_feasibility_20261030/RESULTS.md`（SHA-256 `8b3073b4…6635`）。程式 SHA-256 `0bc5ba3c…2324`、manifest `855a2307…a5162`；Git HEAD `48cdeca`，程式／卡／報告未 commit。
- 主量（GoM val 合格景 20/39 等權 AUROC vs 景內打亂 null95）：S42 0.9878 vs 0.5476；預訓練 Prithvi 0.6511 vs 0.5480。臺灣 270 窗原型油 token：S42 9.35%、預訓練 69.51%。→ 事前 gate **GO**，依規則 teacher=**S42**。
- 描述：S42 decoder 臺灣油 token 12.16%；原型與 decoder 同判 96.75%（κ 0.8313、油 Jaccard 0.7378），decoder-only 油 2,094 vs prototype-only 150。原型近似 decoder 的保守子集，未顯示互補知識。
- 結論：不證明臺灣改善；研究判斷不優先把 S42-PCD 當改善方案。PCD 訓練 B 未啟動，若做須另立卡、定位為可能無效／傷害對照。
- Claude peer postrun（session `7636a263-…`）R1 錯誤經核對撤回，R2 ACCEPT／BLOCKING=[]；原始檔在 repo `consensus/`。
- 卡 `wiki/experiments/20261030_P35_LFMDA凍結原型教師可行性_事前卡.md` 改 completed、填第 7–9 步（1–6 未改）；`index.md` P35 條目、`.agents/handoff.md` 置頂 STATUS_AS_OF 同步。vault 本次未 commit／push。

## [2026-10-30] prereg | P36 Stage I 判別器 GoM 含油範本事前卡已凍結，等待使用者授權 GPU
- 新卡 `.agents/decisions/20261030_P36_StageI判別器GoM含油範本設計.md`（status FROZEN_PRE_LAUNCH）。
- 內容：單一因素（P32 Stage I 只改 `stage1_uda.discriminator_source: oil_only`）、三 seed 配對 source-only、判定規則由 `health_check.py` 的 `verdict()` 在 CPU 執行；不讀臺灣 GT、不做 Stage II／III。**未授權 GPU、未開跑**。既有凍結卡（P29／P30／P31／P32／P33／P35）未改動；vault 本次未 commit／push。

## [2026-10-30] result | P36 Stage I 判別器 GoM 含油範本：依事前卡凍結規則判定 not_improved
- 新頁 `wiki/experiments/20261030_P36_結果.md`（連回事前卡 `.agents/decisions/20261030_P36_StageI判別器GoM含油範本設計.md`，該卡 sha 3e02a1e8… 未改動）。
- 執行：2026-10-30 15:18 tmux 開跑，S42／S43／S44 依序各約 55 分鐘，最後 EXIT=0；GPU smoke 先 SMOKE PASS；GT_expand commit 3d29e67、c23d2ac、c2211a0、1e5e7f7（未 push）；產物 `/home/alanyh/oil_dataset/new/_euda_doil_20261030/`，判定依據 `health/health_check.json`；全程未讀臺灣 GT。
- 判定 **not_improved**：R＝臺灣 270 窗預測油比例 ÷ 同 seed source-only，S42 0.625／S43 0.642／S44 0.463，平均 0.577 < 0.60（P32 參照 0.477）；全空窗數比 source-only 多 +53／+47／+79（平均 +59.7），比 P32 少 17／40／13；GoM val 油 IoU 差 +0.017／+0.033／−0.006，未觸發 source 受損。
- 描述：6 景持續崩壞（天使輪 0801 L9 50QRK 0.04／50QRL 0.42、0522 L8 51QUD 0.07／51QTD 0.14、中油大林 50QRK 0.35／51QTE 0.43）；中段景回升（0924 RTK 0.38→0.74、1001 RQN 0.69→0.86、馬祖 0.79→0.94）；S42／S43 單看落「部分」區間、S44 幾乎同 P32，三 seed 不一致。判別器範本含背景至多只解釋一小部分崩壞；R 只表示有沒有往背景崩，不代表預測正確。
- 下一步（事前卡第 5 步）：先比較 GoM 純背景與含油範本在 U 圖、判別器 logits、embedding 的差異，再做不更新權重的梯度稽核（依 target margin 分箱），不先調 λ；target 錨點、降曝光待使用者決定。**未授權任何新實驗**。前置診斷：BN 交換（GT_expand 48cdeca）已排除 BN 是主因。institution 同步紀錄在 `research/oil_spill_project_status.md`（commit f7a57a1）。vault 本次未 commit／push。

- 2026-11-01｜P37 Lane B 結果回寫：新增 `wiki/experiments/20261101_P37_臺灣24景補充_LaneB結果.md`；Lane A 未開封。
- 2026-11-01｜P37 Lane A 結果回寫：新增 `wiki/experiments/20261101_P37_LFMDA_SegFormerB2_sourceonly_結果.md`；臺灣 19 景 final pooled OilIoU 0.3563、GoM-val-best 0.3224，判定 support baseline adequacy；Lane B 結果頁加指向行。事前卡 sha 9e5de5b1…／619e1dad… 未變；vault 未 commit／push。

- 2026-11-01｜P38 事前卡協商紀錄：新增 `.agents/decisions/20261101_P38_LFMDA_warmupPCD事前卡協商.md`（Claude×Codex 四輪 R4 ACCEPT/[]；卡 v5 待程式 SHA 後凍結，未 launch）；handoff 共識紀錄加一行；vault 未 commit／push。

- 2026-11-02｜P38 卡 v7 回寫：`.agents/decisions/20261101_P38_LFMDA_warmupPCD事前卡協商.md` 檔尾新增「v6→v7：作者實際 warmup 設定（R5）」（Codex R5 ACCEPT/[]，repo 卡 8e1fa87、程式 176c409，smoke 通過）；`.agents/handoff.md` 共識紀錄加一行；正式凍結卡未建立、未 launch；vault 未 commit／push。

- 2026-11-02｜P38 正式凍結卡建立：新增 `.agents/decisions/20261101_P38_LFMDA_warmupPCD設計.md`（cp 自 repo P38_CARD_DRAFT.md，SHA256 700d4883…e8824，程式 commit 176c409）；`.agents/handoff.md` 共識紀錄加一行；`index.md` 實驗紀錄區加條目；未 launch、未讀臺灣 GT；vault 未 commit／push。

- 2026-11-02｜P38 正式 run 啟動回寫：`.agents/handoff.md` 於「P38 凍結卡 FROZEN v7」行下加一行（tmux `p38`、launch.sh、repo commit c9c0075、run 順序、log 路徑；讀臺灣 GT 計分需另行授權）；未改凍結卡；vault 未 commit／push。

- 2026-11-03｜P38 正式 run 完成回寫：`.agents/handoff.md` 於「P38 正式 run 已啟動」行（第 222 行）正下方（新第 223 行）加一行完成紀錄（五 run 全完成封存、GoM val OilIoU C 0.3742／T 0.3245、Δ̄_GoM −0.0497、H2 未保留；臺灣 GT 未讀）；未 commit。

- 2026-11-03｜P38 評估程式封存回寫：`.agents/handoff.md` 於「P38 五個正式 run 全部完成並封存」行（第 223 行）正下方（新第 224 行）加一行（evaluate_p38.py 封存、repo commit 710eb4c、eval_seal.json 13 組預測驗證、讀臺灣 GT 待授權、P39 方針）；臺灣 GT 未讀；未 commit。

- 2026-11-03｜P38 臺灣評估結果回寫：新增 `wiki/experiments/20261103_P38_LFMDA_warmupPCD_結果.md`（19 景 pooled OilIoU C 平均 0.3147 vs T 0.2103，Δ̄ −0.1044 → H1 rejected；H2 未保留；H4 positive_signal 但不推翻 H1；整體 not improved；eval_result.json SHA 7b18bd53…c967f3）；`.agents/handoff.md` 於評估程式封存行下加一行；`index.md` 實驗紀錄區最上方加條目；凍結卡 `.agents/decisions/20261101_P38_LFMDA_warmupPCD設計.md` 未修改；vault 未 commit／push。

- 2026-11-03｜P39 凍結卡建立回寫：新增 `.agents/decisions/20261103_P39_LFMDA_stage2_SAM_LRH設計.md`（自 repo `analysis/p39_lfmda_stage2_sam_lrh_20261103/P39_CARD_DRAFT.md` 以 cp 位元組複製，無 frontmatter，SHA f89c9470…fee5，FROZEN v4）；`.agents/handoff.md` 於「P38 臺灣評估完成」行下加一行；`index.md` 實驗紀錄區最上方加條目；臺灣 GT 未讀；vault 未 commit／push。

- 2026-11-03｜P39 正式 run 啟動回寫（process 啟動，非新協商輪次）：使用者指示「先啟用並且跑一遍」，tmux `p39` 執行 repo `analysis/p39_lfmda_stage2_sam_lrh_20261103/launch.sh`（commit 81c381b），順序 S2333→S2334→S2335，log `/home/alanyh/oil_dataset/new/_p39_lfmda_stage2_20261103/launch.log`；`.agents/handoff.md` 於 P39 凍結卡行（第 226 行）正下方加一行（新第 227 行）；凍結卡 `.agents/decisions/20261103_P39_LFMDA_stage2_SAM_LRH設計.md` 未修改；臺灣 GT 未讀；vault 未 commit／push。

- 2026-11-03｜P39 訓練完成回寫（process 結束，非新協商輪次）：tmux `p39` 三 run（S2333／S2334／S2335）全部完成，EXIT=0，各約 33.5 分，9 組預測各 306 窗已封存；GoM val final OilIoU 0.3287／0.3044／0.2893（平均 0.3075 vs C 0.3742，Δ̄ −0.067，三個皆低於 C）→ H2 依卡規則未保留（GoM-only、正式值待評估器）；機制紀錄：偽標籤油僅 1–2.5% 像素、LRH 幾乎單向油→背景、SAM 270 窗中 168 窗僅 1 區域／221 窗區域覆蓋整窗（LRH 等於整窗多數決）；`.agents/handoff.md` 於 P39 正式 run 啟動行（第 227 行）正下方加一行（新第 228 行）；凍結卡 `.agents/decisions/20261103_P39_LFMDA_stage2_SAM_LRH設計.md` 未修改；臺灣 19 景未評、GT 未讀；vault 未 commit／push。

- 2026-11-03｜P39 臺灣評估結案回寫（使用者授權讀 GT，非新協商輪次）：19 景 pooled OilIoU C 平均 0.3147 vs P39 0.1759／0.2097／0.0908，Δ̄ −0.156，0/19 景改善 → H1 否定；H1b 無正向 change（平均 −0.052）；H2 GoM 未保留（0.3287／0.3044／0.2893 vs C 0.3742）；H4 新五景 recall 無正向訊號（ΔR −0.317）；整體 not improved。seal SHA `990c3744…`、result SHA `c03963fa…`，repo commit 86d2f4c。機制：SAM 在 8-bit proxy 上多半整窗一塊、LRH 單向擦油。LFMDA 作者兩階段都已走完一次、皆 not improved；回頭改善需另立卡、未授權。新增 `wiki/experiments/20261103_P39_LFMDA_stage2_SAM_LRH_結果.md`；`index.md` 在 P39 凍結卡條目上方加結果頁條目；`.agents/handoff.md` 於 P39 訓練完成行（第 228 行）正下方加一行（新第 229 行）；凍結卡 `.agents/decisions/20261103_P39_LFMDA_stage2_SAM_LRH設計.md` 未修改；vault 未 commit／push。

- 2026-11-03｜臺灣測試規則變更回寫（使用者裁決，非新協商輪次）：臺灣新增五張（11-02 補的 Landsat 8：天使輪 2023-07-24 50QRK／50QRL／51QTF、2023-08-25 51QTF；taiwan_file 2026-05-29 50QRK）油標註以外視為背景，取代 11-01「標註外為 unknown、不可合併」規則；往後新實驗主報臺灣 24 張 pooled OilIoU，19 張與新增五張並列；五張用 coverage-union、排除六波段無效像素，19 張照原 Lane A 口徑；已凍結卡判定不改。事後補算（純算術、未重讀 GT 或預測檔；`GT_expand/analysis/taiwan24_merged_iou_20261103/merge24.py`，結果 SHA `96daa0b0fff4bcbbd3288f0e69c9a0c403fc912f7b6f291604772d12f3dc5e58`）學生 final 三 run 平均（19 張／新增五張／24 張）：C 0.3147／0.3344／0.3156；P38 T 0.2103／0.3933／0.2183；P39 0.1588／0.0813／0.1561；24 張排序與 19 張相同。預測圖：五張上未標註的預測油往後改當 FP（紅）。更新 `wiki/experiments/20261103_P38_LFMDA_warmupPCD_結果.md`、`20261103_P39_LFMDA_stage2_SAM_LRH_結果.md` 末節「事後補充：臺灣 24 張」；`20261101_P37_臺灣24景補充_LaneB結果.md`、`20261101_P37_LFMDA_SegFormerB2_sourceonly_結果.md` 附加規則變更註記（原文保留）；`.agents/handoff.md` 於 P39 臺灣評估行（第 229 行）正下方加一行（新第 230 行）；`.agents/decisions/` 凍結卡未修改；vault 未 commit／push。

- 2026-11-04｜P40 凍結回寫（LFMDA student／teacher 改 6 波段，單 seed 探索；Codex R1 REVISE→R2 REVISE→R3 ACCEPT，session 01a1a441…；未 launch，非新實驗結果）：使用者方針先只改 LFMDA 輸入波段 3→6（B04/B03/B02/B8A/B11/B12），單 seed S2333；C6＝P37 配方＋6 波段、T6＝P38 warmup＋6 波段（teacher 只放開 patch_embed.proj；新通道用 RGB 平均初始化×3/6）；REF＝P15 GoM-only Original Prithvi S42/43/44 既有 best.pt 在 306 窗推論；主報臺灣 24 張 pooled OilIoU。repo commit 41418f2；launch 與讀 GT 待使用者授權。新增 `.agents/decisions/20261104_P40_LFMDA_6波段_C6T6設計.md`（repo `P40_CARD_DRAFT.md` 逐位元複製，SHA `a2474e1996a3097eb63e4c78673f21631f8b03097a6294b4d9ff311fd4fc7c8c`）；`index.md` 於實驗紀錄區最上（第 25 行）加一行；`.agents/handoff.md` 於 2026-11-03 臺灣測試規則變更行（第 230 行）正下方加一行（新第 231 行）；其他 `.agents/decisions/` 凍結卡未修改；vault 未 commit／push。

- 2026-11-04｜P40 正式 run 啟動回寫（process 啟動，非新協商輪次、非新實驗結果）：使用者授權啟動，10:08 tmux `p40` 執行 repo `analysis/p40_sixband_lfmda_20261104/launch.sh`（commit 41418f2，凍結程式 SHA 檢查通過），順序 C6→T6→8 組預測（C6 final/best、T6 三種、REF S42/43/44），C6 約 6.6 it/s，估約 1.5 小時跑完；log `/home/alanyh/oil_dataset/new/_p40_sixband_20261104/launch.log`；讀臺灣 GT 需使用者另授權。`.agents/handoff.md` 於「P40 凍結」行（第 231 行）正下方加一行（新第 232 行）；`.agents/decisions/` 凍結卡未修改；vault 未 commit／push。

- 2026-11-04｜P40 跑完與封存回寫（執行紀錄，非結果；process 結束，非新協商輪次）：10:08 使用者授權 launch（commit 41418f2），11:47 全部跑完 LAUNCH_EXIT=0（C6 10:08–10:35、T6 10:35–11:45、8 組預測 11:45–11:47）；12:16 seal，`eval_seal.json` SHA-256 `aeeed8dee4b8…`，8 組預測。GoM-val（來源域，非臺灣）OilIoU：C6 final 0.3763（峰 0.450@step5000；P37 S2333 final 0.3763）、T6 student final 0.1887（峰 0.218@step4000；P38 T_S2333 final 0.296、峰 0.369@step8000）、T6 teacher final 0.135（P38 0.253）。尚未讀臺灣 GT、未看預測或 mechanism log，讀 GT 待使用者另授權；LFMDA 線（P37–P40）訓練器都沒接 wandb。新增 `wiki/experiments/20261104_P40_LFMDA_6波段_執行紀錄.md`；`index.md` 更新 P40 條目（原第 25 行）並於其上加執行紀錄條目；`.agents/handoff.md` 於「P40 已啟動」行正下方加一行；`.agents/decisions/` 凍結卡未修改；vault 未 commit／push。

- 2026-11-04｜P40 臺灣 24 張結果回寫（使用者授權讀 GT 後 `evaluate_p40.py evaluate`；單 seed、描述性、不判定；非新協商輪次）：24 張 pooled OilIoU——C6 final 0.4628（P37 S2333 0.3514，+0.111；P 0.760／R 0.542）、C6 best 0.4799、T6 student final 0.3369（P38 T_S2333 0.2646，+0.072）、T6 student best 0.2709、T6 teacher final 0.2388；REF P15 S42/43/44 0.4888/0.4074/0.3931，平均 0.4298。C6 高於 REF 平均、落在 REF 範圍內，相對 3 波段主要是 recall 上升；T6 蒸餾仍低於 C6；+0.11 與 C 組 seed 間範圍 0.26–0.35 同量級；補 seed 由使用者決定。來源 repo `analysis/p40_sixband_lfmda_20261104/P40_RESULTS.md`（commit 076697e）、`eval_result.json` SHA-256 `be9ea4e2…`。更新 `wiki/experiments/20261104_P40_LFMDA_6波段_執行紀錄.md`（status→completed）；`index.md` P40 條目（第 25 行）；`.agents/handoff.md` 於「P40 已跑完」行下方加一行；`.agents/decisions/` 凍結卡未修改；vault 未 commit／push。

- 2026-11-04｜P41 啟動回寫（process 啟動，非新結果、非新協商輪次）：使用者 14:11 授權 launch，tmux `p41`、repo commit `8399df2`，預計 15:00 前後完成；LFMDA 線首次掛 wandb（run `P41_T3_S2333`）。teacher＝固定 P15 S42（GoM val Oil 0.514、mIoU 0.7407，原生輸入；8-bit 量化輸入 0.437），τ=0.9（門檻閘門 0.5–0.99 全不過，使用者裁決用作者值）；對照 C6 final 0.463／best 0.480、REF 平均 0.430。尚未讀臺灣 GT。新增 `.agents/decisions/20261104_P41_LFMDA_PCD_固定P15teacher設計.md`（逐字複製 repo 草稿，SHA-256 `2194e3bc…` 兩檔一致）與 `wiki/experiments/20261104_P41_LFMDA_PCD_固定P15teacher_執行紀錄.md`；`index.md` 加兩條；`.agents/handoff.md` 於「P40 臺灣 24 張結果」行下方加一行；P40 凍結卡未修改；vault 未 commit／push。
- 2026-11-04｜P41 臺灣 24 張結果回寫（使用者授權讀 GT 後 `evaluate_p41.py evaluate`；單 seed、描述性、不判定；非新協商輪次）：14:11–14:55 跑完（訓練 43 分），seal `2d923489…`、eval_result `ffda8d4e…`。24 張 pooled OilIoU——T3 final 0.4906（P .695 R .625）／best 0.4747；C6 final／best 0.4628／0.4799；T6 0.3369；REF 平均 0.4298；T3−C6 final +0.028、best −0.005（seed 雜訊內，未見 PCD 增益）；GoM val T3 final 0.353、峰 0.400@6000；target 97.6% 過 τ、約 14.7% 為油。補 seed 與 source-only PCD ablation 待使用者決定。更新 `wiki/experiments/20261104_P41_LFMDA_PCD_固定P15teacher_執行紀錄.md`（status: completed）、`index.md` P41 條目、`.agents/handoff.md`；P41 凍結卡未修改；vault 未 commit／push。

- 2026-11-04｜P42 啟動回寫（process 啟動，非新結果、非新協商輪次）：使用者授權 launch，2026-11-04 21:14 起（訓練→2 組預測→image-only seal，預計約 40 分），repo commit `c962d03`，wandb run `P42_S2333`。LFMDA stage 2（自訓練＋SAM＋LRH）接在 P41 T3 之後：使用者方針先照原架構 SAM（P39 設定）跑一次，之後再討論 SAM 是否適合（候選 DADS-SAM）；起點 P41 T3_S2333 student_final＋warmup_prototypes（硬 pin SHA），teacher＝固定 P15 S42，SAM 區域＝P39 封存 ViT-H（manifest `05fa132b…`），單 seed S2333。Codex session `01a1a6ab-daa4-7f93-80ef-0afc004b39c9` R1 REVISE→R2 ACCEPT。對照 T3 final 0.491、C6 0.463、P39 舊線 stage 2 失敗。尚未讀臺灣 GT。新增 `.agents/decisions/20261104_P42_LFMDA_stage2_from_P41_T3設計.md`（逐位元複製 repo 草稿，SHA-256 `2fde70f6…` 兩檔一致）與 `wiki/experiments/20261104_P42_LFMDA_stage2_from_P41_T3_執行紀錄.md`；`index.md` 加兩條；`.agents/handoff.md` 於「P41 臺灣 24 張結果」行下方加一行；P41 凍結卡未修改；vault 未 commit／push。

- 2026-11-05｜P42 訓練完成回寫（process 結束＋image-only seal，非新結果判定、非新協商輪次）：2026-11-04 21:14:57–21:44:12 訓練（約 29 分）、預測 final／best 各約 15 秒、21:44:43 seal，launch.log 全部 EXIT 0；狀態＝訓練完成、已封存、臺灣 GT 待授權。6000 步完成、teacher_unchanged=true、target 抽樣 24000 次、翻轉計數 [11953, 12009, 11897]。GoM val 油 IoU：step1 0.345、1000 0.261、2000 0.386、3000 0.378、4000 0.358、5000 0.363、6000 0.385（mIoU 0.674；P 0.701／R 0.461），best＝final＝6000（P41 T3 step 10000 為 0.353）。機制（封存後揭露、未用 GT，每 1000 步）：LRH 油→背景約 1.53–1.58M（約 4.4% of 選中油約 35.4M px），忽略→背景約 1.1–1.25M，忽略→油約 7–8k，背景→油約 2k，幾乎只往背景推；teacher 偽標籤油機率 >0.5 比例 0.159。seal `eval_seal.json` SHA-256 `6cc54123…c7722`、target_mask_values_read=false。尚未讀臺灣 GT，讀 GT 待使用者另授權（`evaluate_p42.py evaluate --seal-sha256 <sha> --allow-target-gt`）。更新 `wiki/experiments/20261104_P42_LFMDA_stage2_from_P41_T3_執行紀錄.md`（status 改、加「2026-11-04 21:44 完成」章節）、`index.md` P42 兩條、`.agents/handoff.md` 加一行；P42 凍結卡未修改；vault 未 commit／push。

- 2026-11-05｜P42 臺灣 24 張結果回寫（使用者授權讀 GT「讀GT吧」後 `evaluate_p42.py evaluate`；單 seed、描述性、不判定因果；非新協商輪次）：24 張 pooled OilIoU——P42 final＝best（step 6000）0.4786（19 張 0.4785／新增5 0.4812／P 0.671／R 0.626）；P41 T3 final（起點）0.4906；P40 C6 final 0.4628、best 0.4799；REF P15 S42/43/44 0.489／0.407／0.393。P42−T3 −0.012、P42−C6 +0.016；TP 913,417 vs 912,235（幾乎不變）、FP 448,464 vs 399,630（+49k）、FN 546,512 vs 547,694。24 張中 20 張下降、3 張上升（>0.01）：降最多天使輪 20230918 L9 QRL −0.127／QRK −0.114、20230801 L9 QRK −0.126（細線斷開、FN 增加）；升 20260522 L8 51QTD +0.033／51QUD +0.029、馬祖 20220622 +0.068（大張像素權重大，pooled 只降 0.012）。判定 not improved，與 SAM/LRH failure mode 相容（LRH 幾乎只往背景推、約 4.4% 被擦，內部補油幾乎沒有）。eval_result SHA-256 `728f9e011695d1e4cab033f1c885426979b1fa3dc1fc384993baae20c99e1a97`；疊圖 `/home/alanyh/oil_dataset/new/_p42_prithvi_stage2_20261104/figures_final/`。LFMDA 新線（P41→P42）兩階段皆走完一次；下一步討論 SAM 替代（DADS-SAM encoder＋原 SAM decoder 點網格，A 方案，先不讀 GT 做區域統計），需另立卡＋使用者授權。更新 `wiki/experiments/20261104_P42_LFMDA_stage2_from_P41_T3_執行紀錄.md`（status 改為完成：not improved，加「2026-11-05 臺灣 24 張 GT 評估」章節）、`index.md` P42 兩條、`.agents/handoff.md` 加一行；P42 凍結卡未修改；vault 未 commit／push。

- 2026-11-05｜P43 凍結卡落地與啟動回寫（process 啟動，非新結果判定；Codex session `01a1a957-b2b1-7310-9fc6-5e3c8514e896` R1 REVISE→R2 ACCEPT）：使用者 11-05 要求把 P41 T3 stage 1 → P42 stage 2 整條流程多跑兩次看穩定性（student seed 2334、2335；S2333 沿用封存結果不重跑；teacher 固定 P15 S42、SAM 用 P39 封存區域；`pipeline43.py` 原樣 import 凍結程式只改 run 身分）；之後再討論 SAM 換 DADS-SAM 或其他 SAM。repo commit `a24d9b6`，2026-11-05 09:41 launch，wandb stage 1 S2334 run `n9gcp2je`，輸出 `/home/alanyh/oil_dataset/new/_p43_lfmda_seeds_20261105/`，預計約 2.5 小時；評估臺灣24 pooled 三 seed mean／SD／range 與各 seed stage2 − 自己的 stage1 final，讀 GT 另需授權。新增 `.agents/decisions/20261105_P43_LFMDA流程補seed穩定性設計.md`（逐位元複製自 repo `P43_CARD_DRAFT.md`，SHA-256 `800f834122b2e512442aecebfeced56835966f2094cb8f367f4e0810f549e35f` 已核對）、`wiki/experiments/20261105_P43_LFMDA流程補seed穩定性_執行紀錄.md`（status 執行中）、`index.md` 兩條、`.agents/handoff.md` 加一行；vault 未 commit／push。

- 2026-11-06｜P44 凍結卡落地（launch 前，非新結果判定；Codex session `01a1aeb7-9605-7f91-8a86-ec99294c1458` R1 REVISE→R2 ACCEPT→R3 REVISE→R4 REVISE→R5 ACCEPT/BLOCKING=[]）：GT 衍生錨點 ROI＋重疊滑窗訓練／推論，全球排除臺灣，評估臺灣 24 景 ROI pooled OilIoU，對照 P12 三 artifact。GT_expand commit `0bcc8c4`，repo 卡 SHA-256 `51c828bc26a61c0977da9a477b0560c45733ca1e73e2dcbe4c21a91a3de5b098`。新增 `.agents/decisions/20261106_P44_ROI重疊滑窗全球排除臺灣設計.md`、`.agents/handoff.md` 加一行；未 launch、未讀臺灣 GT，兩者各需使用者另行授權；vault 未 commit／push。

- 2026-11-06｜11:35 P44（ROI 3,840＋步距128重疊滑窗、全球排除臺灣）經使用者授權 launch：launch.sh（commit 0bcc8c4，launcher pid 938760，wandb run srjpbqsh），log 在 /home/alanyh/oil_dataset/new/_p44_roi_sliding_20261106/launch.log；流程 train→5 組臺灣預測→VAL 診斷→seal，使用者已授權 seal 後直接讀臺灣 GT 計分（evaluate_p44.py evaluate）。決策檔見 [[20261106_P44_ROI重疊滑窗全球排除臺灣設計]]。

- 2026-11-06｜16:20 P44 完成，判定 upgrade：臺灣 24 張 ROI 口徑 pooled OilIoU P44_best 0.300 vs P12 三 seed 0.209／0.217／0.223（mean 0.216，+0.084）；增益來自誤判減少（precision 0.27→0.43、recall 持平約 0.50）；單 seed、P44_last 只有 0.163（checkpoint 敏感）。seal 435024cf…，結果 /home/alanyh/oil_dataset/new/_p44_roi_sliding_20261106/eval_result.json。見 [[20261106_P44_ROI重疊滑窗全球排除臺灣設計]]。

- 2026-11-06｜16:54 P45 launch：P44 流程上只改 batch 組成（每 batch 固定 k 個含油窗，k＝1／2／4，VAL 選 k、臺灣不參與選 k；對照沿用 P44 封存預測），起因是 P44 漏判（3 景幾乎全漏）。未經 Codex（用量用盡，使用者授權 Claude 自審後直接跑）。commit 0bb16ca，launcher pid 985864，root /home/alanyh/oil_dataset/new/_p45_oil_per_batch_20261106/，預估 11–17 小時；讀臺灣 GT 待使用者授權。見 [[20261106_P45_每batch固定含油窗數]]。
- 2026-11-07｜00:48 P45 launcher 全部完成（未讀臺灣 GT）：VAL best 油 IoU k1 0.212（ve2）／k2 0.212（ve9）／k4 0.224（ve15）→ 依規則選 k=4（P44 VAL best 0.176）；k4 VAL 診斷（測試算子）IoU 0.225、precision 0.36、recall 0.37、pred/gt 1.03；三組臺灣預測＋seal 完成（seal SHA bb55822c9a939efca93542631d6e6e8cb6c8b6f4dc418924b5355cdbd1b77f54）。讀臺灣 GT 待使用者授權。
- 2026-11-07｜P45 臺灣 GT 計分（使用者授權）：VAL 選中 k=4，臺灣 24 張 ROI pooled OilIoU 0.238 → worse_than_P44（P44 0.300）；相對 P12（0.209／0.217／0.223）indeterminate；硬失敗皆無。描述性（不改判定、不回頭選 k）：k1 0.284、k2 0.238；VAL 排序沒有轉移到臺灣。P44 全漏 3 景（50RQN／51RTK／50RPN）k4 recall 0.82／0.87／0.79 救回；但 Landsat 11 景 k4 IoU 0.111（P44 0.286），S2 13 景 0.304 與 P44 0.308 相當，整體掉分來自 Landsat（感測器分組為事後描述）。FP 像素 1.34M（P44 0.99M）。
- 2026-11-07｜HNM_SI 改寫到 P45 k=4 管線（困難背景＋光譜指數安全閘）經 Claude×Codex 共識 R1 REVISE→R2 REVISE→R3 ACCEPT→R4 ACCEPT，程式 commit b4ae2a3／a2022d7、42 項測試與 GPU smoke 通過，未正式探勘、未訓練，下一步待使用者決定，見 [[20261107_HNM_SI改寫到P45k4]]。
- 2026-11-07｜HNM_SI 正式探勘（使用者授權，未訓練）gate 未過：評分 106,156 個已標背景≥70% 純背景窗，FP≥256 px 候選僅 790 窗（0.74%，156 景）；SI 安全閘剔除 589（S2 467/613、Landsat 122/177）→ 核可 201 窗（57 景）；排程實際替換比例 overall 0.070（S2 0.059、Landsat 0.116，可替換上限 0.389）＜事前門檻 0.25／0.20 → 依規則 H1 不得訓練，回設計。主因推測：礦工為 in-sample（P45 k4 在 TRAIN 已標背景幾乎不誤判），且 SI 把約 3/4 候選判為像油。對照圖：/home/alanyh/oil_dataset/new/_hnm_si_p45_20261107/figures/si_gate/。
- 2026-11-07｜P46 大圖版配比（油：隨機背景：困難背景＝1：1：0.5）與 OOF 困難背景探勘：卡 P46_CARD.md FROZEN v1（commit 8ae6e95），Claude×Codex R1 REVISE→R2 ACCEPT，47 項測試與 GPU smoke 通過；launch 未授權，下一步＝使用者授權 launch。詳見 decisions/20261107_P46大圖版配比與OOF探勘.md。
- 2026-11-07｜P46 v2（launch 前）：使用者改額外背景為全隨機（H0′＝1.5 份隨機、H1′＝1.0 隨機＋0.5 OOF HNM，gate S≥1,781），Claude×Codex R3 REVISE→R4 ACCEPT，凍結 v2（commit 39e0a2d）、55 項測試與 GPU smoke 通過；launch 未授權。詳見 [[20261107_P46大圖版配比與OOF探勘]]。
- 2026-11-07｜P46 v2 launch（14:10）：使用者授權一次跑完並授權跑完後直接讀臺灣 GT 計分；外層 run_all.sh（commit 39e0a2d；H0′ wandb xs8khiwm，約 10 it/s，預計約 17 h）；gate 不過則 H1′ 不跑、不 evaluate。見 [[20261107_P46大圖版配比與OOF探勘]]
2026-11-08｜P46 gate 未過（SI 後 772＜1,781）→ H1′ 未訓練；H0′ 完成、臺灣未讀；待使用者決定結案或修訂補 H1′。見 [[20261107_P46大圖版配比與OOF探勘]]
2026-11-08｜P46 H0′ 臺灣24 0.185（<P44 0.300）；v3 SI 改大圖剔 5% 已預先登記；H1′ 待授權。見 [[20261107_P46大圖版配比與OOF探勘]]
2026-11-08｜P46 v3 H1′ 09:43 launch（SI 改大圖剔 5%，S 2,184；Codex R8 ACCEPT）；約 14:00 自動計分。見 [[20261107_P46大圖版配比與OOF探勘]]
2026-11-08｜P46 H1′ 臺灣24 Oil IoU 0.318 vs H0′ 0.185，Δ+0.133＞margin 0.03 → H1_better_than_H0（seal 969fb3ee）；recall 0.28→0.69、pred/gt 1.85；best=ve1 早停、單 seed 非盲；後續訓練未授權。見 [[20261107_P46大圖版配比與OOF探勘]]
2026-11-08｜P48 補 seed（H1′ 與 P44 各 seed 43、44，H0′ 不補）14:47 launch；預先登記 Δ≥0.03 且 >2·se 才判 H1 較好；純 seed 複製未經 Codex。見 [[20261108_P48補seed_H1與P44]]
2026-11-09｜P48 完成 → indistinguishable（H1′ 0.311±0.014 vs P44 0.232±0.164；P44 S44 塌縮成全背景）；後續待使用者決定。見 [[20261108_P48補seed_H1與P44]]
2026-11-09｜P47（SpectralGPT+＋DeepLabV3 接 P44 ROI 流程只換模型，單 seed）完成 → indistinguishable：臺灣24 ROI pooled OilIoU P47_best 0.315（P 0.474／R 0.485）vs P44_best S42 0.300（|Δ|<0.09）；P47_last 0.145；19 張 0.326 vs 0.304、新增5張 0.121 vs 0.252；4/24 景勝過 P44；09:22 launch、20:35 預測完、21:15 讀臺灣 GT；未經 Codex。見 [[20261108_P47_SpectralGPT_ROI3840]]
2026-11-09｜P43 完成（使用者授權讀臺灣 GT；描述性、無判定）：臺灣24 pooled OilIoU stage 1 final 0.488±0.003（S2333／2334／2335＝0.491／0.487／0.485）、stage 2 final 0.481±0.002（0.479／0.481／0.483）；配對差 −0.012／−0.005／−0.002，平均 −0.006±0.005，三 seed 全小幅下降；流程 seed 間極穩定、SAM＋LRH 無增益；seal da8ea1ef…、eval_result SHA ff173b18…；只換 student seed、舊 306 窗口徑不可與 ROI 3840 比較；DADS-SAM／其他 SAM 未授權。見 [[20261105_P43_LFMDA流程補seed穩定性_執行紀錄]]
2026-11-09｜ROI probe（既有 LFMDA 模型直接套 P44 ROI 3840 推論口徑，只推論不訓練、非預先登記；使用者先同意「先做個推論試試」、後以「跑評分阿」授權讀臺灣 GT）完成：22:03–22:32 推論 8 個模型（P15_S42、C6_S2333、T3×3、P42×3），seal 8f5e5c3d…；臺灣24 ROI pooled OilIoU T3 0.340／0.351／0.333（平均 0.341±0.009）、P42 0.328／0.338／0.321（平均 0.329±0.009）、C6 0.251、P15 teacher 0.178（P 0.199／R 0.628、pred/gt 3.16）、P44_best 重算 0.300；對照 P12 0.216、H1′ 0.311±0.014；T3 逐張勝 P44 僅 11/24；stage 2 仍比 stage 1 低約 0.012。LFMDA student 搬到 ROI 口徑沒崩，但 T3／P42 的 PCD 看過臺灣 19 張影像（transductive）、臺灣為重用基準，不能寫成「LFMDA 勝過 ROI 取樣」。同日裁決：之後 LFMDA-ROI 的 target 改用臺灣 24 張；LFMDA-ROI 設計（teacher 換 H1′、source 用 H1′ 清單、target 24 張 ROI 窗）未授權。見 [[20261109_ROI_probe_既有LFMDA模型套ROI3840_執行紀錄]]
2026-11-10｜P49（LFMDA 整條流程搬到 P44 ROI 資料，三 seed；source＝H1′ 串流、teacher＝H1′ S43、target＝臺灣 24 張 ROI 窗；使用者 00:00 前授權無人值守「接下來都自行執行不用我授權我要去睡了」，未經 Codex、描述性）完成：00:16 第一輪 launch（run_all.sh，commit 7815100），S2333／S2335 stage 1 在 step 1000 VAL batch 64 GPU OOM（3 訓練＋2 SAM 同卡 32GB），04:03 第一輪結束只有 S2334 完成；07:51 第二輪 run_rest.sh（VAL batch 16＋empty_cache，commit baf527f），08:45 stage 1 完成無塌縮，09:4x stage 2 完成；10:10 重新封存（probe_seal sha d04460831c4b…），10:12 評分完成（eval_p49.json sha 1bc478442120…）。臺灣24 ROI pooled OilIoU：S2 final 0.307／0.422／0.336（0.355±0.060，19 張 0.359、新增5張 0.295）、S1 final 0.297／0.001／0.235（S2334 塌成全背景）、C6-ROI 0.097；對照 P44 0.300（重算 0.3002）、H1′ 0.311±0.014、ROI probe T3 0.341±0.009。分解：大林兩張（約占 GT 59%）IoU 0.52–0.62 vs P44 0.37／0.31；扣大林 0.109–0.152 vs P44 0.267；Landsat 0.120–0.174 vs 0.286；S2 衛星 13 張 0.378–0.504 vs 0.308；逐張勝 P44 僅 7／7／8 張。判讀：描述性、大林驅動，不能說 LFMDA-ROI 整體改善；transductive、stage 2 為子集。見 [[20261110_P49_LFMDA搬到ROI_三seed_執行紀錄]]
2026-11-10｜P50（H1′ 1:1+0.5 HNM+SI × Prithvi-EO-2.0-300M + UperNet，一般監督式分割三 seed；只換分割頭 DLV3→UperNet，動機=P49 student 照抄 teacher H1′ S43，想試 UperNet 當 teacher；使用者授權、未經 Codex、描述性）smoke 通過（5.5 it/s、峰值 8.6 GiB），11:40 launch（S42、S43 並行，接 S44 → 臺灣 24 張 ROI 預測 → 評分）；產物 /home/alanyh/oil_dataset/new/_p50_upernet_h1_20261110/；結果待完成。見 [[20261110_P50_H1prime_UperNet_三seed_執行紀錄]]
2026-11-10｜基線裁決：自然比例（P44 流程、不做任何背景挑選）。使用者原話「我們的baseline設定 自然比例，也就是我們不做任何的背景挑選去做為我們的baseline好了」。基線＝P44 流程（Prithvi-EO-2.0 300M＋DeepLabV3、Tversky 0.3/0.7、TRAIN 298／VAL 37，同 P44 卡 FROZEN v1）：GT 衍生錨點 ROI 3,840×3,840、ROI 內 256 窗步距 128、背景不挑（不做配比、HNM／SI，也不固定每 batch 油窗數）。ROI 規則：以 298 張訓練影像最大單一連通油區（GoM_20190428_S2_15RYM，外框 2,782×3,430）長邊 3,430 為分組上限；油污依面積由大到小 first-fit，合併後外框寬高皆 ≤3,430 同組，否則另開；每組以外框中心為錨點放 3,840（15×256）ROI，起點對齊 256、靠邊推回影像內。現有數字（臺灣 24 張 ROI pooled Oil IoU）：P44 S42 0.300／S43 0.351／S44 0.045（S44 塌成全背景），三 seed 0.232±0.164，不計 S44 為 0.325（描述性）；參考 H1′（1:1＋0.5 HNM）0.311±0.014，P48 判定 indistinguishable。注意：自然比例 1/3 seed 塌縮，之後拿基線當對照要報三個 seed 並揭露塌縮；報告採含或不含 S44 的數字待使用者決定；進行中 P50（H1′×UperNet）、P49（source＝H1′ 串流）以 H1′ 為底、不是此基線，兩者不改動、不追溯。未改任何凍結檔、未啟動訓練；institution 對應 commit 611b524。見 [[20261106_P44_ROI重疊滑窗全球排除臺灣設計]]、[[20261108_P48補seed_H1與P44]]
2026-11-11｜P50 完成（H1′ × Prithvi-EO-2.0-300M + UperNet 一般監督式分割三 seed；描述性、未經 Codex）。11-10 12:44 使用者要求「停掉43啦 一個 一個跑」：停掉並行 S43（epoch 3，移到 train_UPN_S43_failed_attempt1/）與原 launch.sh，改 launch_seq.sh（commit 8f9d78a）依序跑；S42 11:40–16:23（best ve 12）、S43 16:23–20:59（best ve 14）、S44 20:59–11-11 01:24（best ve 13）；預測 01:24–01:57；01:57 評分（eval_p50.json sha b31a49ceea50…）。結果（臺灣24 ROI pooled OilIoU，UperNet vs DLV3）：全部 0.299±0.016 vs 0.311±0.014（配對 −0.000／−0.002／−0.032）；扣大林 0.172 vs 0.184、Landsat 0.147 vs 0.175（三 seed 並未皆升）；Sentinel-2 0.378 vs 0.378；大林 0.451 vs 0.493；P／R／pred-gt 0.44／0.48／1.09 vs 0.40／0.60／1.52；逐張勝 14／14／13 張。結論：not better，不建議換 P49 teacher；UperNet seed 間差小、誤判少但漏判多；兩種頭扣大林／Landsat 都遠低於 P44（0.267／0.286）。卡片 commit ab23095，產物 /home/alanyh/oil_dataset/new/_p50_upernet_h1_20261110/。見 [[20261110_P50_H1prime_UperNet_三seed_執行紀錄]]
2026-11-11｜臺灣結果呈現裁決：改為「大林 vs 其他 22 張」。使用者裁決，臺灣結果呈現改為「大林 vs 其他 22 張」。使用者原話：「以後不用呈現甚麼台灣24 /19 / 5 我覺得這很沒意義 以後就幫我呈現大油污(大林) 跟其他剩下的台灣油污 這樣比較知道差在哪」。之後臺灣成績分兩欄：大林（中油大林 2 張，GT 849,893 px，約佔 58%）與其他 22 張（GT 610,036 px）；兩欄各自在 run 內做像素加總後算 IoU，再跨 seed 平均；不再列 24／19／5。凍結卡的原判定不改，只改呈現方式。自然比例基線三 seed（P44 S42、P48 S43、P48 S44）：大林 0.342／0.427／0.007、其他 0.267／0.280／0.089；平均大林 0.258±0.222、其他 0.212±0.107；不計塌縮的 S44 則為 0.385、0.274。H1′ 三 seed（參考）：大林 0.453／0.537／0.491，平均 0.493±0.042；其他 0.228／0.139／0.185，平均 0.184±0.044。描述性觀察：H1′ 強在大林；自然比例在未塌縮的 seed 上，其他 22 張較高。institution 現況檔已記錄。
2026-11-11｜P51 啟動（GoM-only 資料制度三 seed）。P51 GoM-only 資料制度三 seed（S42／S43／S44），09:54 launch；使用者原話：「接下來我想要試試看 GoM的資料 測試在台灣」。使用者決定：VAL 只用 GoM 25 景；跑 3 個 seed（42／43／44）；送 Codex 審；workbook 比照 P44～P50 豁免；「執行 實驗」。設計：從 P44 凍結清單取 GoM 子集，TRAIN 158 景／181 ROI／148,684 窗（含油窗 4,511，含油窗:背景窗＝1:32），VAL 25 景／26 ROI／5,850 窗；ROI 與 P44 相同，設定只改 seed 與清單路徑。這是整體制度的比較：訓練資料、VAL、曝光量（約 1.8 倍）、正負比同時改變。實驗卡：GT_expand `analysis/p51_gom_only_roi_20261111/P51_CARD.md`，Codex 審三輪（R1 REVISE、R2 REVISE、R3 ACCEPT），現為 FROZEN v3（commit 9e85ed5）。判讀：大林、其他 22 張分開判，對照自然比例 baseline（P44 S42、P48 S43、P48 S44）；Δ ≥ 0.03 且 > 2·se 判 GoM_higher／GoM_lower，否則 no_decisive_difference（啟發式判讀，非顯著性檢定）。執行狀態：launcher pid 1925228，全程持有 GPU 鎖；順序 S42 → S43 → S44，每 seed 各含臺灣預測，最後自動讀臺灣 GT 評分；wandb jfhl1902；產物 /home/alanyh/oil_dataset/new/_p51_gom_only_roi_20261111/；預估 6–17 h；結果待完成。institution 已記錄。

## [2026-11-11] decision | ROI 固定倍率 context（GeoAgent 雙分支 k=2/k=4，無 RL）實驗設計共識
- 新增 `.agents/decisions/20261111_ROI固定倍率context_k2k4實驗設計.md`（附件 `.agents/decisions/attachments//` r1–r3 prompt/out）。
- Claude×Codex 三輪收斂 ACCEPT／BLOCKING=[]；K1/K2/K4、Stage A 9 runs＋futility 停止、Stage B 6 runs；前置 VAL FN 拆解。**僅設計共識，未寫碼、未跑、未凍結 prereg、未授權執行**。

## [2026-11-11] harness | vault CLAUDE.md 改版 v2（使用者授權；改為每半月更新）
- 規則校正：index 只當一行一頁的目錄、log 回到 `## [日期] 類型 | 說明` 格式、status 用英文、Git commit/push 需授權（取代舊「git add . && push」）
- 新增 §6 研究基線快照（8 槽 VRT／6 波段輸入、mask 0＝油、P44 baseline、run 內像素加總口徑、臺灣分大林 vs 其他 22 張）；舊 11 波段與 128/64/192 mask 定義移出（屬 0420/0422 舊 pipeline）
- 下次檢查：2026-11-25

## [2026-11-11] correction | 修復 vault 導覽與近期研究問題入口
- 重建 `AGENTS.md`、`.agents/README.md` 的損壞 `NaN` 段落；重整 `index.md` 為一頁一行，補全部內容頁與決策頁，移除 P51 壞連結和過期執行敘述。
- 新增 P51 凍結卡唯讀鏡像、`wiki/datasets/20261111_來源分折與封存測試稽核.md`、`wiki/concepts/20261111_研究問題與未解事項.md`；更新 `.agents/handoff.md` 的現場觀察。
- 例外路由原因：本工作階段 vault-manager 因 vault 不在其宣告可寫 workspace roots 而停止寫入；依 `CLAUDE.md` §4 由主代理直接寫入，並作唯讀稽核與連結驗證。未改凍結卡原件、資料、模型或 process，未 commit／push。

## [2026-11-11] writeback | P44–P48 結果補入實驗頁
- 新增 `wiki/experiments/20261106_P44_ROI自然比例基線_結果.md` 至 `20261109_P48_H1prime與P44補seed_結果.md` 共五頁，連回原設計／決策卡；更新 `index.md` 與研究問題入口。
- `wiki/papers/Wang2023_SSL4EO-S12.md` 三處無目標的 `[[SeCo]]` 改為純文字，修復 Obsidian 斷鏈；不改論文主張。

## [2026-11-11] writeback | 回填 GT-centric、資料、環境、模型與新資料路由
- 新增兩份 `outputs/` 快照，以及 `wiki/datasets/` 逐景風海流／標記清冊和 457 景 CSV 快照、`wiki/environment/` Docker/Codex 遠端紀錄、`wiki/models/` 模型路線、`wiki/pipeline/` 新資料路由；更新舊頁歷史提示及 `index.md`。
- 更新 `CLAUDE.md` 與 vault-manager 角色的內容頁驗收要求，並在 `.agents/handoff.md` 標記 P51 舊 process 觀察過時；`wiki/papers/`、主線 Dockerfile、資料與 process 未動。
- 例外路由：此工作階段 vault 不在子代理宣告的可寫 workspace roots，沿用先前已核實的阻斷條件，由主代理依 `CLAUDE.md` §4 直接寫入並做連結／內容核對；未 commit／push。

## [2026-11-11] harness | Codex vault-manager 掛載別名驗證
- `/home/alanyh/obsidian-vault` 與 workspace 內 `/mnt/backup/alanyh/.container_state/oil_deeplab/obsidian-vault` 的 device:inode 同為 `66307:26214429`；已在 Codex adapter 寫入同 inode 別名規則。
- 本 session 新派出的 vault-manager 仍載入舊 developer instruction，因 canonical 路徑不在 writable roots 而停止；**未能驗證代理實際可寫**。本筆由主代理追加，後續重載角色後需再驗證；未 commit／push。

## [2026-11-11] decision | ROI context k2/k4 融合改為論文 branch aggregation（v5）
- 使用者裁決改用 GeoAgent 論文 branch aggregation，γ 版（F_local+γ⊙C）作廢；Claude×Codex R4 REVISE→R5 ACCEPT／BLOCKING=[]，未授權執行。詳見 `.agents/decisions/20261111_ROI固定倍率context_k2k4實驗設計.md` 文末「設計變更」節，附件補入 r4／r5。

## [2026-11-11] decision | vault 知識庫修復 WP0 目標設計定案（D0–D5，使用者裁決）
- 新增 `.agents/decisions/20261111_vault知識庫修復_WP0目標設計.md`；WP1 起分工待使用者討論，未開始

## [2026-11-11] harness | Codex vault-manager 寫入規則修正（WP5，使用者要求）
- `.codex/agents/obsidian-vault-manager.toml`：改為「先嘗試寫入、實際權限錯誤才停」，加 `[sandbox_workspace_write] writable_roots` 指向 vault，移除同 inode 別名寫法；根因＝舊指示依 parent 宣告的 workspace roots 自行停止、從未嘗試寫入
