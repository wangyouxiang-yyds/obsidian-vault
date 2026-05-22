---
type: concept
tags: [training, technique, imbalance]
related: []
---

# Hard Negative Mining (HNM)

## 一句話定義
一種訓練技巧，透過模型主動尋找那些容易被誤判為正樣本（Oil）的困難負樣本（Background），並將其加入訓練集重新訓練，以降低模型的誤判率 (False Positive Rate)。

## 詳細說明
在油污偵測中，海面的波浪、船隻、雲影等背景紋理非常容易被誤判為油污。普通的隨機採樣背景可能不包含這些「困難」區域。

## 在本專案的應用
本專案實作了 HNM 五步驟流程：
1.  **Step 1**: 建立座標池 (Coordinate Pool)。
2.  **Step 2**: 準備 V1 訓練集（正樣本 + 少部分隨機背景）。
3.  **Step 3**: 訓練 Probe 模型 V1 (ConvNeXt-tiny)。
4.  **Step 4**: 使用 V1 模型掃描座標池，篩選信心值高的誤判區。
5.  **Step 5**: 組裝最終 Golden Dataset。

## 相關頁面
- [OIL_PROJECT_VRT_0422.md](../pipeline/OIL_PROJECT_VRT_0422.md)
