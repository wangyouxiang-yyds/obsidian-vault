---
type: concept
reading_status: partial_abstract_introduction
tags: [PEFT, fine-tuning, transfer-learning, foundation-model, Prithvi, LoRA]
related: [MartiEscofet2025_PEFT_GeoFM, Szwarcman2026_PrithviEO2, self_supervised_pretraining_遙測, 跨海域_source-only_zero-shot油污偵測]
---

# 遙測基礎模型的參數高效率微調（PEFT）

> 狀態：部分建立。本頁目前只整理一般概念，以及 Marti Escofet et al. 2025 的 Abstract／Introduction 所提供的研究動機；方法細節與實驗結論待全文研讀。

## 一句話定義

Parameter-Efficient Fine-Tuning（PEFT）是在保留大部分預訓練模型權重不變的情況下，只更新少量既有參數，或加入少量新的可訓練參數，讓大型模型適配新的下游任務。

## Frozen、Full Fine-tuning 與 PEFT

| 適配方式 | 預訓練 encoder | 新增／更新的參數 | 任務 decoder | 核心取捨 |
|---|---|---|---|---|
| Frozen | 全部凍結 | 不更新 encoder | 通常正常訓練 | 成本最低，但 encoder 無法針對新任務調整 |
| Full fine-tuning | 全部或近乎全部更新 | 原模型大量參數 | 通常正常訓練 | 適應自由度最大，但訓練記憶體與過擬合／遺忘風險較高 |
| PEFT | 大部分凍結 | 少量既有參數，或新增 LoRA／Adapter／Prompt 等元件 | 通常正常訓練 | 在適應能力與資源需求之間取折衷 |

PEFT 描述的是**訓練哪些參數**，不是新的影像前處理、分割標籤或推論後處理。它也不是單一演算法，而是一組方法家族；LoRA、Adapter 與 Prompt Tuning 是常見形式。

## 概念上如何接到 Prithvi 油污模型

本專案的概念流程可以寫成：

```text
多光譜影像
    ↓
Prithvi encoder
    ↓
segmentation decoder
    ↓
油污機率圖／mask
```

導入 PEFT 後，輸入與輸出不變；差別只在 Prithvi 的訓練方式：

```text
凍結 Prithvi 原始權重
       +
訓練少量 PEFT 參數
       +
訓練 segmentation decoder
```

對本專案的潛在價值有兩層：

- **工程層**：減少需要更新的 encoder 參數，可能降低訓練記憶體需求；實際 wall-clock time 與推論成本不能只由參數量推定，須實測。
- **研究層**：在標註事件有限時，嘗試保留 Prithvi 的通用遙測表徵，只學習油污任務必要的修正；是否改善 Oil IoU 或跨海域泛化，仍需受控實驗證明。

若研究目標是 [[跨海域_source-only_zero-shot油污偵測]]，PEFT 本身不違反 source-only：只要所有可訓練參數、超參數與 checkpoint 都只由 source development data 決定。若使用 target 無標註影像來調整 PEFT 模組，問題就轉為 domain adaptation，不再是嚴格 target-blind zero-shot。

## 閱讀邊界與目前不能下的結論

- 不能因 trainable parameters 少，就宣稱訓練時間、顯存與推論成本會同比例下降。
- 不能從其他 EO 任務直接推論 PEFT 對低對比、極度不平衡的油污分割一定優於 full fine-tuning。
- 不能在尚未核對 decoder、split、搜尋預算與 seed 前，把跨資料集摘要主張當作本專案的實驗結論。

## 相關頁面

- [[MartiEscofet2025_PEFT_GeoFM]] — GeoFM PEFT 系統比較論文，目前僅完成 Abstract／Introduction
- [[Szwarcman2026_PrithviEO2]] — 本專案 Prithvi encoder 的來源與海面表徵假設
- [[self_supervised_pretraining_遙測]] — 預訓練、linear probing 與 fine-tuning 的上游概念
- [[deeplabv3plus_358clean_overfitting_改善方向]] — 本專案既有 pretrained backbone 與微調策略脈絡
- [[跨海域_source-only_zero-shot油污偵測]] — source-only、UDA 與 target-blind 的協定區分
