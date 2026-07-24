---
type: concept
tags: [self-supervised-learning, pretraining, backbone, linear-probing, fine-tuning, foundation-model]
related: [Wang2023_SSL4EO-S12, DeepLabV3+, deeplabv3plus_358clean_overfitting_改善方向, unsupervised_oil_detection]
---

# 自監督預訓練（Self-Supervised Learning, SSL）在遙測影像的應用

## 一句話定義

不需要人工標註，讓模型先從大量未標記影像中學會「有用的通用表徵（representation）」，之後再用少量標註資料把這個表徵適配到具體任務（分類/分割/變化偵測）上。

## 詳細說明

### Backbone 是什麼

Backbone 指負責把原始影像轉換成「特徵表徵」的那段網路（例如 ResNet50 的卷積堆疊、或 ViT 的 patch embedding + self-attention 堆疊）。它本身不輸出任務答案，只輸出一組特徵向量/特徵圖；後面再接一個任務專屬的「頭」（分類頭、分割頭、變化偵測頭）才會產生最終輸出。SSL 預訓練訓練的正是這個 backbone。

### 兩種 backbone 架構

- **CNN（如 ResNet50）**：用卷積逐層降採樣，感受野隨深度增加。
- **Vision Transformer（ViT）**：先把影像切成固定大小的 patch（例如 16×16）轉成 token 序列，再用 self-attention 讓每個 token 跟所有其他 token 互動。

### 四種常見 SSL 演算法

| 方法 | 類型 | 核心機制 | 只能用在哪種 backbone |
|---|---|---|---|
| MoCo-v2/v3 | 對比學習（contrastive） | 同一張圖的兩個增強版本應該在特徵空間中靠近，不同圖應該遠離 | CNN 或 ViT 皆可 |
| DINO | 師生蒸餾（teacher-student distillation） | 教師網路（動量更新）產生「軟標籤」，學生網路模仿教師對同一張圖不同視角的輸出分布 | CNN 或 ViT 皆可 |
| MAE | 遮罩自編碼（masked autoencoding） | 隨機遮住大部分 patch，只用剩下的 patch 重建被遮住的原始像素 | 僅 ViT（需要 token 結構才能「移除」被遮住的 patch） |
| data2vec | 遮罩 + 特徵級聯合嵌入 | 類似 MAE 的遮罩機制，但重建目標是教師網路的特徵表示而非原始像素 | 僅 ViT |

MAE/data2vec 之所以無法套用在 CNN，是因為卷積運算需要連續、規則排列的輸入網格，無法像 ViT 的 token 序列一樣乾淨地「拿掉」被遮住的部分。

### 兩種評估/使用協議：Linear Probing vs Fine-tuning

|  | Linear Probing | Fine-tuning |
|---|---|---|
| Backbone 是否更新 | 凍結（frozen） | 全部或部分解凍，一起訓練 |
| 訓練的東西 | 只有最後一層線性分類/分割頭 | 整個網路（backbone + 頭） |
| 測試的是什麼 | Backbone 本身學到的表徵有多好（representation quality） | 整套系統實際能達到的效能上限 |
| 訓練成本 | 低（只更新一層參數） | 高（更新整個網路） |
| 常見用途 | 快速比較不同預訓練方法/資料集的表徵品質 | 實際部署前的最終效能追求 |

## 在本專案的應用

本專案的 [[DeepLabV3+.md|DeepLabV3+]] 目前是 **from-scratch 訓練**（無預訓練 backbone）。[[Wang2023_SSL4EO-S12]] 的實測數據顯示：同樣的 DeepLabV3+ 架構，只要 backbone 換成 ResNet50+MoCo-v2 這種相對輕量的 SSL 預訓練，DFC2020 分割任務的 mIoU 就能從 42.11（rand.init）提升到 54.68–54.83，是一個外部證據支持 [[deeplabv3plus_358clean_overfitting_改善方向.md]] 中 A1 方向（導入 pretrained backbone）的合理性。

**重要提醒（來自 SSL4EO-S12 論文的反例）**：預訓練+fine-tune 並非「全面碾壓」無預訓練——OSCD 變化偵測任務中，SSL4EO-S12 預訓練的 precision 反而是四組最低（因類別嚴重不平衡，rand.init 的高 precision 只是「什麼都不判」的假象）。評估預訓練效益時，必須看具體指標與資料集的類別分布特性，不能只看單一總分。

**目前已確認的研究缺口**：截至本會話的文獻調查（含閱讀 2025 回顧文獻 remotesensing-17-03681），尚無任何遙測專屬 SSL/foundation model（SSL4EO-S12、Prithvi、SatMAE、RingMo 等）被應用在**光學**海面油汙偵測上；既有相關工作僅限於高光譜、同域內自監督（Duan et al. 2021/2022、Kang et al. 2023 SSTNet），SAM 則僅作為 SAR 影像的標註加速工具。若本專案嘗試將 SSL 預訓練引入光學油汙分割，具有一定新穎性。

## 相關頁面

- [[Wang2023_SSL4EO-S12]] — 本概念的主要文獻來源，含完整資料集規格與 benchmark 數據
- [[DeepLabV3+.md]] — 本專案分割模型工程紀錄
- [[deeplabv3plus_358clean_overfitting_改善方向.md]] — pretrained backbone 導入的具體實作方向（A1）
- [[unsupervised_oil_detection.md]] — 既有 unsupervised 油汙偵測方法總覽
