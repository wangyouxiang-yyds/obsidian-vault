# 油汙偵測研究 Wiki — 目錄

> 本頁列出所有 Wiki 頁面。每次新增頁面後請同步更新此目錄。

---

## Pipeline 說明（wiki/pipeline/）

- [sen2like.md](wiki/pipeline/sen2like.md) — Sen2Like 處理流程：大氣校正、光譜一致化、Fusion、空間對齊、比對工具箱
- [project_overview.md](wiki/pipeline/project_overview.md) — OIL_PROJECT_MutiBand_0420 專案概述：模型架構、前處理、訓練設定、I/O 優化、重要路徑
- [OIL_PROJECT_VRT_0422.md](wiki/pipeline/OIL_PROJECT_VRT_0422.md) — 0422 VRT 訓練專案：GDAL VRT 虛擬影像、動態 Patch 讀取、全場景 Mask 流程
- [annotation_workflow.md](wiki/pipeline/annotation_workflow.md) — 標注工作流程：LabelMe JSON 在 pipeline 的角色、QGIS GPKG → JSON 橋接腳本說明
- [add_new_data.md](wiki/pipeline/add_new_data.md) — 新增 Sen2Like 資料流程：目錄結構規範、Strip→COG 轉換、VRT 建置、fold split 重算、patch 座標重生成（含腳本指令與關鍵路徑索引）
- [VRT_pipeline_01_前處理.md](wiki/pipeline/VRT_pipeline_01_前處理.md) — 完整前處理 Pipeline：VRT 生成、GT Mask、Patch 切分（GB1.5）、3-fold 策略、HNM、NIR-R-G 背景圖；445 場景 / 8-band；2026-07-10 更新場景數 439→445、新增 Dataset Manifest 制度章節、JSON 雙位置說明
- [VRT_pipeline_02_模型訓練.md](wiki/pipeline/VRT_pipeline_02_模型訓練.md) — 模型訓練 Pipeline：DeepLabV3+ ResNet-50（8-ch, from-scratch）、FocalLoss、class_weights=[13,1]、checkpoint/resume、fold2 基準；2026-07-10 修正三處硬錯（EMA decay=0.0 未修 bug、best.pt 依 val_miou 非 Oil IoU、FocalLoss 無 alpha 參數）並補分支 B（GT_expand）差異表
- [VRT_pipeline_03_重組評估.md](wiki/pipeline/VRT_pipeline_03_重組評估.md) — Reconstruction 評估 Pipeline：全場景 Sliding Window、Cloud Mask 後處理、annot-only / JSON GT 兩組指標、prevalence 對照表、v2 效能優化、fold2 Oil IoU=39.80%
- [OIL_PROJECT_MutiBand_0422_VRT_training.md](wiki/pipeline/OIL_PROJECT_MutiBand_0422_VRT_training.md) — 0422 論文主軸版本入口索引：sliding-window 策略說明、指向三份 VRT pipeline 系列文件、與 GT_expand 的關係
- [GT_expand_pipeline.md](wiki/pipeline/GT_expand_pipeline.md) — GT_expand Fork 版本 Pipeline：GT-centric patch 策略（bbox≤256 取中心 / bbox>256 用 256-grid 鋪磚，2,893 patches）、背景 patch 機制（bg_coords 3,510 筆）、3_fold_stratified_v2 split、雙 gate 評估協議（pooled_oil_iou>0.362 + Wilcoxon p<0.05）、cw31/tversky 實驗線；2026-07-10 大幅更新反映現況，並補 patch 座標腳本 SPLIT_DIR/OUT_DIR 環境變數硬化說明

---

## 實驗紀錄（wiki/experiments/）

- [20260429_OSDMamba整合至0420.md](wiki/experiments/20260429_OSDMamba整合至0420.md) — OSDMamba 整合至 0420 專案：移植工作、與 0422 版差異、下一步
- [20260430_訓練效能優化.md](wiki/experiments/20260430_訓練效能優化.md) — OSDMamba 訓練效能優化：CPU-GPU 同步瓶頸、向量化 Loss、EMA 調整
- [20260501_大圖重組效能優化.md](wiki/experiments/20260501_大圖重組效能優化.md) — 大圖重組效能優化：NAS I/O 瓶頸、GPU 側重組、Batch Size 提升
- [20260520_新批Sen2Like資料Pipeline重建計畫.md](wiki/experiments/20260520_新批Sen2Like資料Pipeline重建計畫.md) — 新資料引入完整 pipeline：8-band（B01/02/03/04/08/8A/11/12）、GPKG→JSON 轉換、GB1.0、2025 固定 test set、stratified scene-level fold split
- [20260521_MS6_Pipeline執行紀錄.md](wiki/experiments/20260521_MS6_Pipeline執行紀錄.md) — MS6 8-band DeepLabV3+ 完整執行紀錄：VRT 建置（B02 10m 參考）、COG 轉換（1546 TIF，9× I/O 提升）、class_weights、AMP、Bug 修正 6 項（VRT 60m 參考致訓練全零 / 三處 /10000 遺漏 / NIR-R-G PNG 路徑 / 輸出路徑）、重組速度優化（_read_vrt_parallel、async save、bg prefetch）、per-fold log / start_fold、5-fold CV 進行中（Fold 1 完成 mIoU=0.593）
- [20260525_架構演進與差異彙整.md](wiki/experiments/20260525_架構演進與差異彙整.md) — 2026-05-20 至 05-25 全面對比：資料格式（11→8 band、strip→COG、VRT Bug 修正）、前處理 pipeline、訓練設定（AMP/class_weights）、重組速度優化（_read_vrt_parallel / async save / prefetch）、Bug 修正彙總
- [20260527_CrossFoldHNM_執行紀錄.md](wiki/experiments/20260527_CrossFoldHNM_執行紀錄.md) — Cross-Fold HNM Step 2~4 完整紀錄：step2 卡死修正（45,848 FP candidates）、step3 d_min 分布（p10=0.152 截點）、step4 組裝（1:1+0.5HN=1:1.5）、cfHNM 訓練完成（avg Oil IoU=0.224，未超越 A 組 0.242）
- [20260530_實驗結果彙整.md](wiki/experiments/20260530_實驗結果彙整.md) — 截至 2026-05-30 全實驗彙整：Ref 5-fold GB1.0（avg Oil IoU=0.277）、A 組 3-fold GB1.5（0.242）、B 組 3-fold cfHNM（0.224）、跨組對比與後續方向
- [三策略比較實驗計畫.md](wiki/experiments/三策略比較實驗計畫.md) — 執行計畫（更新中）：策略一純隨機 / 策略二事件留出 / 策略三時序擴展，各跑 3-fold DeepLabV3+；scene split 完成，patch 座標生成進行中
- [20260602_三策略前置作業執行紀錄.md](wiki/experiments/20260602_三策略前置作業執行紀錄.md) — 三策略前置作業完整紀錄：資料現況修正（NOAA 可用 220 非 395）、事件 36 場景 GPKG→mask→VRT、COG 轉換 3073 TIF、scene split 三策略執行結果、patch 座標生成指令
- [20260605_資料集修正與三策略重跑.md](wiki/experiments/20260605_資料集修正與三策略重跑.md) — 183 NOAA 場景補齊 JSON+Mask（220→403 NOAA，總計 439）、廢棄目錄清理（1.1TB）、JSON flat 目錄重建、yaml json_dir 修正、三策略 scene split + patch coords 全部重跑、RGB viz 升級、early stop 觀察與 class_weights 矛盾分析
- [20260603_Unsupervised油汙偵測初探.md](wiki/experiments/20260603_Unsupervised油汙偵測初探.md) — Unsupervised 油汙偵測兩階段 pipeline 初探：HOSD 高光譜基準（AUC=0.99）、5 個 S2 場景 iForest + OSI 對比、雲遮罩設計、Precision 低問題診斷與改善方向
- [20260604_iForest全場景改進.md](wiki/experiments/20260604_iForest全場景改進.md) — iForest 全場景失效根因分析（污染比例/競爭異常/均值稀釋）、改進方案：乾淨水體訓練（NDWI>0.1+std<0.05）+ 5th Percentile 聚合取代均值、Fallback 機制
- [20260613_iForest_LocalContrast_B4B8B11_全場景實驗總結.md](wiki/experiments/20260613_iForest_LocalContrast_B4B8B11_全場景實驗總結.md) — iForest + Local Contrast（B4/B8/B11）全 439 場景評估：Micro Recall=0.0225，TP=104，FP=24,063；非零場景僅 43/438（9.8%）
- [20260614_DeepRX_EM_v3_全場景實驗總結.md](wiki/experiments/20260614_DeepRX_EM_v3_全場景實驗總結.md) — DeepRX + EM v3 全 439 場景評估：Micro Recall=0.3712（iForest 16.4×），TP=1,698，非零場景 275/433（63.5%）；VAE latent + Mahalanobis + EM 自動門檻
- [OSDMamba_摘要.md](wiki/papers/OSDMamba_摘要.md) — OSDMamba 論文摘要：首個基於 Mamba 的油汙偵測架構、SS2D 感受野擴張、ConvSSM 解碼器
- [20260702_CV_358clean_gt_expand_進行中.md](wiki/experiments/20260702_CV_358clean_gt_expand_進行中.md) — GT_expand 3-fold CV 訓練進度：fold1/2 完成（2026-07-01）、fold3 進行中（epoch 43 Oil IoU≈0.30）、TODO 三份待動工、已知繼承 bug（C2/C3/C4）
- [20260702_波段選擇消融實驗規劃.md](wiki/experiments/20260702_波段選擇消融實驗規劃.md) — 波段選擇消融實驗規劃：源自 Zakzouk et al. 2024 論文，比較 Baseline 全 8 波段／Band Subset(B1,2,3,8A,11,12)／Index C／Index D 四組；已釘死主指標 recon_pooled_oil_iou，組0 Baseline 3-fold 實測完成（0.332±0.026），組1 準備開跑
- [20260708_JM距離判別髒資料可行性診斷.md](wiki/experiments/20260708_JM距離判別髒資料可行性診斷.md) — 分支 B side study：三框架診斷（scene 級 oil-vs-bg JM AUC=0.50 / patch 級三指標 AUC≤0.65 / anomaly-JM AUC=0.64 但飽和）皆無法自動判髒資料，結論為「標籤性質 vs 工具性質」錯配非 JM 壞掉；附帶發現 SSL4EO band_map 波段錯位 bug
- [20260709_baseline錯誤分析_GT過寬與方向二降級.md](wiki/experiments/20260709_baseline錯誤分析_GT過寬與方向二降級.md) — SSL4EO 三配置打平 baseline 後的 error analysis：355 測試 scene dirty 命中=0（0.33 天花板與髒資料無關）、感測器無差、面積-IoU 倒 U 形（medium 0.456 最佳）、pooled 0.33 vs per-scene 0.42 落差幾乎全由 ~51 張大油汙 scene 造成；overlay 目視判定瓶頸為 FN + GT 過寬 extent-polygon（Gulf_20190428 IoU=0.753 vs Gulf_20210607 IoU=0.134 對照），判定方向二 detect-then-verify cascade 降級（結論已於同日修正，見文內〔2026-07-09 修正〕：51 張大油汙 scene precision/recall 硬化量化證實瓶頸是真實 under-segmentation，非 GT 過寬）

---

## 模型（wiki/models/）

- [DeepLabV3+.md](wiki/models/DeepLabV3+.md) — DeepLabV3+ 工程紀錄：ResNet50 骨幹、8 通道輸入（B01/02/03/04/08/8A/11/12）、class_weights=[13.0,1.0]、FocalLoss、Sliding Window 推論設定
- [OSDMamba.md](wiki/models/OSDMamba.md) — OSDMamba 工程紀錄：SwinUMambaD 配置、Dice+Focal 聯合損失、EMA、RTX 5090 踩坑與對策

---

## 資料集（wiki/datasets/）

- [ms6_sen2like.md](wiki/datasets/ms6_sen2like.md) — MS6_sen2like 8-band 資料集：路徑、波段定義、Mask 映射與 COG 狀態；2026-07-10 補 Dataset Manifest 制度（445 場景，issues=0）、5 張 2026 NOAA Atlantic 新場景入庫記錄、JSON 雙位置盤點
- [dataset_split_strategy.md](wiki/datasets/dataset_split_strategy.md) — 海面油汙偵測資料集切割策略規劃：完全隨機 (K-Fold)、特定事件留出 (Leave-One-Event-Out)、時序預測切割 (Temporal Split)；2026-07-10 補策略四（油汙面積分層 3-fold，分支 B 現行主用）

---

## 技術概念（wiki/concepts/）

- [unsupervised_oil_detection.md](wiki/concepts/unsupervised_oil_detection.md) — Unsupervised 油汙偵測方法總覽：iForest / OSI / RX / VAE 比較、雲遮罩替代方案（NDWI+Blue）、S2 實施路線、文獻缺口分析
- [cloud_optimized_geotiff.md](wiki/concepts/cloud_optimized_geotiff.md) — Strip TIF vs COG TIF：磁碟排列方式差異（index 粒度）、為何 COG 能直接跳到指定區塊、ASCII 圖解、在專案的 I/O 加速實測（9×）、與 VRT 的關係
- [acolite_vs_sen2like.md](wiki/concepts/acolite_vs_sen2like.md) — Acolite vs Sen2Like 格式差異：像素值單位（float32 vs uint16×10000）、波段數（11 vs 8）、SWIR 解析度（20m vs 10m Fusion）、大氣校正方向、標注對齊說明
- [hard_negative_mining.md](wiki/concepts/hard_negative_mining.md) — 困難樣本挖掘：降低誤判率的 HNM 五步驟流程說明
- [iforest_架構與運作機制.md](wiki/concepts/iforest_架構與運作機制.md) — iForest 深入理解：樹結構本質、建樹過程（含 8 像素具體例子）、Duan 2022 對 GM01 完整 10 步驟、訓測同源為何不是資料外洩、contamination 才是真正失敗點
- [deeprx_vae_架構與運作機制.md](wiki/concepts/deeprx_vae_架構與運作機制.md) — Deep-RX 深入理解：VAE 架構（reparam trick / KL 約束）、San Diego I 完整流程、訓測同源爭議的誠實討論、論文評估方式的學術批評、Background-Only Training 洞察（v9 已實作）、v9 完整 12 步驟流程（per-scene VAE + Mahalanobis + EM + NDOI）、論文 vs 真實場景複雜度差異（one anomaly vs multi-anomaly）、v3 濫抓四大根本原因與 v9 對應解法
- [deeplabv3plus_358clean_overfitting_改善方向.md](wiki/concepts/deeplabv3plus_358clean_overfitting_改善方向.md) — DeepLabV3+ 358-clean 訓練改善方向統整：overfitting 根本診斷（初始化+資料量錯位）、A~C 三組改善方向表（依 ROI 排序）、A1 pre-trained backbone 完整實作細節（input conv 改造 code、differential lr、兩階段 fine-tune）
- [self_supervised_pretraining_遙測.md](wiki/concepts/self_supervised_pretraining_遙測.md) — 遙測 SSL 預訓練總覽：backbone 概念、MoCo/DINO/MAE/data2vec 四法比較、Linear Probing vs Fine-tuning 對照表、SSL4EO-S12 實測佐證 pretrained backbone 效益、「SSL 尚未用於光學油汙偵測」研究缺口

---

## 文獻摘要（wiki/papers/）

- [IsolationForest_Liu2008.md](wiki/papers/IsolationForest_Liu2008.md) — Isolation Forest 原始論文（Liu et al. 2008/2012）：演算法原理、anomaly score 公式、sklearn 參數對照、在本專案 S2 油汙偵測的實測結果
- [Duan2022_HOSD_IsolationForest.md](wiki/papers/Duan2022_HOSD_IsolationForest.md) — Duan et al. 2022：HOSD 高光譜資料集 + Isolation Forest Unsupervised 油汙偵測；iForest 單步 AUC=0.99（224 波段）
- [Song_DeepRX_Hyperspectral_Anomaly.md](wiki/papers/Song_DeepRX_Hyperspectral_Anomaly.md) — Song et al. 2023：Deep-RX 高光譜異常偵測，VAE + RX 結合，深度學習版馬氏距離異常評分
- [柯弈仲2008_光學衛星影像海洋異常物偵測.md](wiki/papers/柯弈仲2008_光學衛星影像海洋異常物偵測.md) — 柯弈仲 2008（NCU 碩論，指導：陳繼藩）：光學衛星影像海洋異常物偵測，RX + EM + Region Growing
- [Wang2023_SSL4EO-S12.md](wiki/papers/Wang2023_SSL4EO-S12.md) — Wang et al. 2023：全球尺度多模態多時相 Sentinel-1/2 自監督預訓練資料集（25萬地點/300萬patch），MoCo/DINO/MAE/data2vec × ResNet50/ViT 全面 benchmark；確認「SSL 尚未應用於光學海面油汙偵測」的研究缺口

---

## 環境設定（wiki/environment/）

- [Docker_環境設定.md](wiki/environment/Docker_環境設定.md) — Docker 環境與 MCP 設定紀錄：基礎映像檔、全域套件、CUDA 核心算子、MCP 自動化指令

---

## 分析輸出（outputs/）

- [20260711_AI協作Harness現況快照.md](outputs/20260711_AI協作Harness現況快照.md) — 2026-07-11 狀態快照：雙 harness（Claude Code × Antigravity）+ institution 正本架構、研究現況（分支A/B、tversky/cw31 數字、v3plus 訓練進行中）、Claude 端 subagent 清單、Claude/agy/codex 模型角色分工與調度原則
