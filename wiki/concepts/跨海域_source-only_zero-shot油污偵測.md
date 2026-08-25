---
type: concept
tags: [domain-generalization, source-only, zero-shot, cross-region, target-blind, optical, oil-spill]
related: [wiki/datasets/dataset_split_strategy.md, wiki/concepts/acolite_vs_sen2like.md]
---

# 跨海域 source-only zero-shot 光學油污偵測

## 問題定義

本研究的主問題不是一般 transfer learning，而是更嚴格的 **source-only geographic zero-shot generalization**：只用海外有標註光學油污資料訓練及選模，最後一次性在封存的台灣資料上評估。

在最嚴格 protocol 下，台灣端不得提供：

- 訓練標註或 few-shot 樣本；
- 無標註影像統計、normalization 或 domain adaptation；
- threshold、early stopping、augmentation、模型或 checkpoint 選擇依據；
- 反覆查看台灣結果後再修改方法。

若使用台灣無標註影像調整模型，應另稱 UDA；若使用少量台灣標註，應另稱 few-shot adaptation，兩者不得和 source-only zero-shot 混稱。

## 為什麼跨海域會失敗

可能的 domain shift 至少包含：海色與懸浮物、近岸／外海背景、sunglint 幅度、雲與雲影、浪紋與船跡、油種與 weathering、季節與觀測幾何、感測器與處理鏈差異。Sen2Like 可減少 S2/L8/L9 的波段與空間不一致，但不能消除海域、事件、背景及油污物理狀態的差異。

## 第一輪文獻矩陣

| 文獻 | 光學／IR | 多地區 | 嚴格 target-blind？ | 對本研究的角色 |
|---|---|---:|---:|---|
| [[Du2022_CBF-CNN_HY1C]] | VIS+NIR | 是 | 否；各區域用當地標註訓練 | imbalance baseline |
| [[Sun2024_中解析度光學油污分割]] | VIS+NIR+SWIR | 是，全球十事件 | 未證明 | 最接近感測器／波段 baseline |
| [[Kang2024_PlanetScope_CNN_Transformer]] | VIS+NIR | 是，八事件 | 否；random patch CV | UPerNet／augmentation baseline |
| [[Wang2024_中國海多感測器油污製圖]] | 多感測器光學；涉及 VIS/NIR，實際模型輸入待確認 | 是 | 未證明 | operational domain breadth |
| [[Du2025_MS3OSD]] | UV+VIS+NIR | 是，四海域 | 未證明 | sunglint 與 multisensor fusion |
| [[Chou2021_U-Net光學海上油污]] | RGB | 未確認 | 未證明 | 台灣光學 U-Net prior art |

## 目前可防守的判讀

第一輪查核沒有找到上述六篇中任何一篇符合「海外 source-only 訓練、台灣完全封存測試」。因此目前不能說本題已被做過；但也不能只依六篇就宣稱全球從未有人做過。更穩妥的論述是：現有光學研究已涵蓋多地區、多感測器與不同 sunglint，但其評估多為 random pixel/patch split、當地重新訓練或未交代 region-held-out protocol；嚴格 geographic target-blind evidence 仍不足。

### SAR 的台灣相鄰先例

[[Chang2024_MorphologicalAttention_UNet_SAR]] 使用 SAR 模型並以影響台灣海域的真實油污事件作 effectiveness verification。Abstract／Introduction 沒有交代訓練資料的地理來源、台灣事件是否封存、84.55% mIoU 的評估資料或台灣驗證是否為定量 ground truth，因此目前只能列為「可能相關的 SAR→Taiwan application precedent」，不能先稱為嚴格 source-only zero-shot。它也不能直接代表本專案的 optical S2/L8/L9 domain transfer，因成像機制不同。

## 判定一篇論文是不是直接前例

至少要逐項回答：

1. target region 是否完全沒有參與訓練？
2. target 的無標註影像統計是否也未被使用？
3. split 是否先按 incident／site／region 分組，再裁 patch？
4. hyperparameter、checkpoint 與 threshold 是否只用 source validation 決定？
5. target 結果是否只開封一次，沒有反覆調整？
6. 評估是否區分 fully labeled scenes 與 positive-only annotations？

第 1–4 項不成立時，通常應依實際資料使用方式改稱 within-domain evaluation、domain adaptation 或 few-shot transfer。第 5 項不成立代表 target-test tuning／資料洩漏；第 6 項不成立則代表標註與評估範圍不足。這些情況都不能宣稱為合格的 strict source-only zero-shot 證據。

## 建議閱讀順序

1. [[Sun2024_中解析度光學油污分割]]：感測器與六個共同波段最接近。
2. [[Kang2024_PlanetScope_CNN_Transformer]]：直接回應 UPerNet vs DeepLabV3+，同時暴露 random-patch split 問題。
3. [[Du2022_CBF-CNN_HY1C]]：理解不平衡與「多區域但不是 zero-shot」的區別。
4. [[Wang2024_中國海多感測器油污製圖]]：補 operational mapping 與多感測器 domain shift。
5. [[Du2025_MS3OSD]]：補 sunglint、UV/VNIR 與 spatial-spectral fusion。
6. [[Chou2021_U-Net光學海上油污]]：界定台灣既有研究做到哪裡、沒做到哪裡。

## 相關頁面

- [[dataset_split_strategy]]
- [[acolite_vs_sen2like]]
- [[self_supervised_pretraining_遙測]]
