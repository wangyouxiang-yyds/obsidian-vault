# 油汙偵測研究 Wiki — 目錄

> 本頁列出所有 Wiki 頁面。每次新增頁面後請同步更新此目錄。

---

## Pipeline 說明（wiki/pipeline/）

- [sen2like.md](wiki/pipeline/sen2like.md) — Sen2Like 處理流程：大氣校正、光譜一致化、Fusion、空間對齊、比對工具箱
- [project_overview.md](wiki/pipeline/project_overview.md) — OIL_PROJECT_MutiBand_0420 專案概述：模型架構、前處理、訓練設定、I/O 優化、重要路徑
- [OIL_PROJECT_VRT_0422.md](wiki/pipeline/OIL_PROJECT_VRT_0422.md) — 0422 VRT 訓練專案：GDAL VRT 虛擬影像、動態 Patch 讀取、全場景 Mask 流程
- [annotation_workflow.md](wiki/pipeline/annotation_workflow.md) — 標注工作流程：LabelMe JSON 在 pipeline 的角色、QGIS GPKG → JSON 橋接腳本說明
- [add_new_data.md](wiki/pipeline/add_new_data.md) — 新增 Sen2Like 資料流程：目錄結構規範、Strip→COG 轉換、VRT 建置、fold split 重算、patch 座標重生成（含腳本指令與關鍵路徑索引）

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

---

## 模型（wiki/models/）

- [DeepLabV3+.md](wiki/models/DeepLabV3+.md) — DeepLabV3+ 工程紀錄：ResNet50 骨幹、8 通道輸入（B01/02/03/04/08/8A/11/12）、class_weights=[13.0,1.0]、FocalLoss、Sliding Window 推論設定
- [OSDMamba.md](wiki/models/OSDMamba.md) — OSDMamba 工程紀錄：SwinUMambaD 配置、Dice+Focal 聯合損失、EMA、RTX 5090 踩坑與對策

---

## 資料集（wiki/datasets/）

- [ms6_sen2like.md](wiki/datasets/ms6_sen2like.md) — MS6_sen2like 8-band 資料集：路徑、波段定義、Mask 映射與 COG 狀態
- [dataset_split_strategy.md](wiki/datasets/dataset_split_strategy.md) — 海面油汙偵測資料集切割策略規劃：完全隨機 (K-Fold)、特定事件留出 (Leave-One-Event-Out)、時序預測切割 (Temporal Split)

---

## 技術概念（wiki/concepts/）

- [unsupervised_oil_detection.md](wiki/concepts/unsupervised_oil_detection.md) — Unsupervised 油汙偵測方法總覽：iForest / OSI / RX / VAE 比較、雲遮罩替代方案（NDWI+Blue）、S2 實施路線、文獻缺口分析
- [cloud_optimized_geotiff.md](wiki/concepts/cloud_optimized_geotiff.md) — Strip TIF vs COG TIF：磁碟排列方式差異（index 粒度）、為何 COG 能直接跳到指定區塊、ASCII 圖解、在專案的 I/O 加速實測（9×）、與 VRT 的關係
- [acolite_vs_sen2like.md](wiki/concepts/acolite_vs_sen2like.md) — Acolite vs Sen2Like 格式差異：像素值單位（float32 vs uint16×10000）、波段數（11 vs 8）、SWIR 解析度（20m vs 10m Fusion）、大氣校正方向、標注對齊說明
- [hard_negative_mining.md](wiki/concepts/hard_negative_mining.md) — 困難樣本挖掘：降低誤判率的 HNM 五步驟流程說明
- [iforest_架構與運作機制.md](wiki/concepts/iforest_架構與運作機制.md) — iForest 深入理解：樹結構本質、建樹過程（含 8 像素具體例子）、Duan 2022 對 GM01 完整 10 步驟、訓測同源為何不是資料外洩、contamination 才是真正失敗點

---

## 文獻摘要（wiki/papers/）

- [IsolationForest_Liu2008.md](wiki/papers/IsolationForest_Liu2008.md) — Isolation Forest 原始論文（Liu et al. 2008/2012）：演算法原理、anomaly score 公式、sklearn 參數對照、在本專案 S2 油汙偵測的實測結果
- [Duan2022_HOSD_IsolationForest.md](wiki/papers/Duan2022_HOSD_IsolationForest.md) — Duan et al. 2022：HOSD 高光譜資料集 + Isolation Forest Unsupervised 油汙偵測；iForest 單步 AUC=0.99（224 波段）

---

## 環境設定（wiki/environment/）

- [Docker_環境設定.md](wiki/environment/Docker_環境設定.md) — Docker 環境與 MCP 設定紀錄：基礎映像檔、全域套件、CUDA 核心算子、MCP 自動化指令

---

## 分析輸出（outputs/）

_尚無頁面_
