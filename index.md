# 油汙偵測研究 Wiki — 目錄

> 本頁列出所有 Wiki 頁面。每次新增頁面後請同步更新此目錄。

---

## Pipeline 說明（wiki/pipeline/）

- [sen2like.md](wiki/pipeline/sen2like.md) — Sen2Like 處理流程：大氣校正、光譜一致化、Fusion、空間對齊、比對工具箱
- [project_overview.md](wiki/pipeline/project_overview.md) — OIL_PROJECT_MutiBand_0420 專案概述：模型架構、前處理、訓練設定、I/O 優化、重要路徑
- [OIL_PROJECT_VRT_0422.md](wiki/pipeline/OIL_PROJECT_VRT_0422.md) — 0422 VRT 訓練專案：GDAL VRT 虛擬影像、動態 Patch 讀取、全場景 Mask 流程
- [annotation_workflow.md](wiki/pipeline/annotation_workflow.md) — 標注工作流程：LabelMe JSON 在 pipeline 的角色、QGIS GPKG → JSON 橋接腳本說明

---

## 實驗紀錄（wiki/experiments/）

- [20260429_OSDMamba整合至0420.md](wiki/experiments/20260429_OSDMamba整合至0420.md) — OSDMamba 整合至 0420 專案：移植工作、與 0422 版差異、下一步
- [20260430_訓練效能優化.md](wiki/experiments/20260430_訓練效能優化.md) — OSDMamba 訓練效能優化：CPU-GPU 同步瓶頸、向量化 Loss、EMA 調整
- [20260501_大圖重組效能優化.md](wiki/experiments/20260501_大圖重組效能優化.md) — 大圖重組效能優化：NAS I/O 瓶頸、GPU 側重組、Batch Size 提升
- [20260520_新批Sen2Like資料Pipeline重建計畫.md](wiki/experiments/20260520_新批Sen2Like資料Pipeline重建計畫.md) — 新資料引入完整 pipeline：8-band（B01/02/03/04/08/8A/11/12）、GPKG→JSON 轉換、GB1.0、2025 固定 test set、stratified scene-level fold split
- [20260521_MS6_Pipeline執行紀錄.md](wiki/experiments/20260521_MS6_Pipeline執行紀錄.md) — Steps 2–5 執行結果、VRT 解析度 Bug（60m→10m 修復）、/10000 資料單位 Bug、I/O 瓶頸診斷（strip TIF 397ms→COG 44ms）、COG 轉換進行中
- [OSDMamba_摘要.md](wiki/papers/OSDMamba_摘要.md) — OSDMamba 論文摘要：首個基於 Mamba 的油汙偵測架構、SS2D 感受野擴張、ConvSSM 解碼器

---

## 模型（wiki/models/）

- [DeepLabV3+.md](wiki/models/DeepLabV3+.md) — DeepLabV3+ 工程紀錄：ResNet50 骨幹、11 通道輸入、FocalLoss、Sliding Window 推論設定
- [OSDMamba.md](wiki/models/OSDMamba.md) — OSDMamba 工程紀錄：SwinUMambaD 配置、Dice+Focal 聯合損失、EMA、RTX 5090 踩坑與對策

---

## 資料集（wiki/datasets/）

- [ms6_sen2like.md](wiki/datasets/ms6_sen2like.md) — MS6_sen2like 8-band 資料集：路徑、波段定義、Mask 映射與 COG 狀態

---

## 技術概念（wiki/concepts/）

- [cloud_optimized_geotiff.md](wiki/concepts/cloud_optimized_geotiff.md) — COG 效能優化：原理與在專案中帶來的 I/O 加速實測
- [hard_negative_mining.md](wiki/concepts/hard_negative_mining.md) — 困難樣本挖掘：降低誤判率的 HNM 五步驟流程說明


---

## 文獻摘要（wiki/papers/）

_尚無頁面_

---

## 環境設定（wiki/environment/）

- [Docker_環境設定.md](wiki/environment/Docker_環境設定.md) — Docker 環境與 MCP 設定紀錄：基礎映像檔、全域套件、CUDA 核心算子、MCP 自動化指令

---

## 分析輸出（outputs/）

_尚無頁面_
