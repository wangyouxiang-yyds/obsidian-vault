---
date: 2026-07-02
type: pipeline
tags: [0422-vrt, deeplabv3+, sliding-window, oil-detection, paper-main]
project: OIL_PROJECT_MutiBand_0422_VRT_training
---

# OIL_PROJECT_MutiBand_0422_VRT_training（論文主軸）

> 本頁為 0422 VRT 訓練版本的入口索引。詳細說明分散在以下三份 pipeline 文件與一份架構概覽。

---

## 核心策略

- 整張大圖按 stride 切 patch（sliding-window，約 5 萬量級），油汙 patch 稀少、多為純背景
- 以 VRT 動態讀取，不預存實體 patch
- 訓練後用 `reconstruct_module` 拼回大圖，評估整圖 IoU（含大量 BG）
- 論文主軸：真實部署情境下的全場景偵測效能

---

## 文件索引

| 文件 | 說明 |
|------|------|
| [[OIL_PROJECT_VRT_0422]] | 架構概覽（VRT 動態讀取、全場景 Mask、評估機制進化、主要腳本）|
| [[VRT_pipeline_01_前處理]] | VRT 建立、GT Mask、Patch 切分（GB1.5）、3-fold 策略、HNM |
| [[VRT_pipeline_02_模型訓練]] | 訓練超參數、FocalLoss、EMA、checkpoint/resume |
| [[VRT_pipeline_03_重組評估]] | Reconstruction 評估、整圖 IoU、fold2 Oil IoU=39.80% |

---

## 與 GT_expand 版本的關係

本版本 fork 出 `OIL_PROJECT_MutiBand_GT_expand`，後者改採 GT-centric patch 策略（每個 patch 保證含油汙）。兩個版本並行運作，詳見 [[GT_expand_pipeline]]。

---

## 相關頁面

- [[GT_expand_pipeline]] — fork 版本（GT-centric patch，對照實驗）
- [[deeplabv3plus_358clean_overfitting_改善方向]] — 共用改善方向分析（A1/A2/C1~C4）
- [[DeepLabV3+]] — 模型工程紀錄
