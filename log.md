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
