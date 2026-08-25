---
type: paper
status: preliminary-needs-fulltext
title: "人工智慧之U-Net架構用於海上油污的光學影像偵測"
authors: 周芷蘭（Chih-Lan Chou）
year: 2021
tags: [Taiwan, thesis, optical, RGB, U-Net, oil-spill, reading-queue]
---

# 人工智慧之 U-Net 架構用於海上油污的光學影像偵測（周芷蘭，2021）

> 來源：https://doi.org/10.6844/NCKU202100163
> 學位：國立成功大學海洋科技與事務研究所碩士論文，109學年度，104頁
> 收錄狀態：第一輪紀錄；摘要可核實，感測器、地區、切分與全文方法仍待補。

---

## 核心貢獻

以監督式 U-Net 對海上油污光學 RGB 影像進行 segmentation，並比較 activation、optimizer 與 loss 的組合。它是台灣學位研究中的光學 U-Net 油污先例，但目前沒有證據顯示使用 transfer learning、海外資料遷移或台灣 target-blind 測試。

> **題名更正**：正式中英文題名都沒有「Transfer Learning」；可核實摘要也沒有 pretrained encoder 或遷移學習敘述。除非後續全文找到證據，不應把它稱為 transfer-learning 論文。

## 關鍵方法

- 模型：supervised U-Net segmentation。
- 輸入：摘要明確描述 RGB 三個光波段。
- 最佳組合：ELU activation、AdaMax optimizer、cross-entropy loss。
- 實際資料來源、感測器、地區、樣本量與 augmentation：待全文確認。

## 資料、波段與切分

| 項目 | 內容 |
|---|---|
| 影像 | 光學 RGB；摘要提到黃、黑、紅色油污案例 |
| IR 類型 | **沒有可核實的 NIR、SWIR 或 TIR 證據** |
| 平台／地區 | 摘要只泛稱衛星或（無人）飛行載具可應用；實際資料待確認 |
| 切分 | 有 training/testing，但比例與 scene/event grouping 待確認 |
| Transfer learning | 摘要與正式題名均未證實 |
| Zero-shot 判定 | **否／未證實** |

## 重要數據／結果

- 平均 accuracy 約 90%。
- Recall 92.2%、precision 93.9%、IoU 86.6%、F1 91.0%。
- IoU 較差案例主要涉及油水混雜、厚薄油混雜、波紋與反光。

## 與本專案的關聯

它可用於回答「台灣是否有人做過光學 U-Net 油污偵測」：有此先例。但它不能回答本專案的主問題，因為目前沒有海外-only 訓練、台灣 sealed target、NIR/SWIR 或跨海域 generalization 的證據。

## 研究缺口與假設

- 資料 provenance 與切分單位在摘要中不透明。
- RGB-only 與本專案八波段 Sen2Like 有明顯落差。
- 以 accuracy 為主可能受背景像素占比影響。
- 若 train/test 來自同一來源，結果不能代表跨域能力。

## 九步分析摘要

1. **領域地景**：台灣光學 RGB 油污 U-Net segmentation。
2. **矛盾偵測**：先前「with Transfer Learning」稱呼與正式題名／摘要不符。
3. **引用鏈**：上游為 U-Net 與一般 supervised segmentation。
4. **研究缺口**：缺多光譜、事件分組與跨海域測試。
5. **方法審核**：指標看似高，但資料 provenance 是可信度關鍵。
6. **假設殺手**：若背景占比極高，accuracy 90% 的資訊量有限。
7. **知識地圖**：連到台灣 prior art、U-Net baseline 與 source-only protocol。
8. **文獻綜合**：可作台灣光學油污先例，不是跨域 transfer 先例。
9. **So What**：後續優先取得全文，確認資料地區、感測器與 split。

## 相關頁面

- [[跨海域_source-only_zero-shot油污偵測]]
- [[DeepLabV3+]]

