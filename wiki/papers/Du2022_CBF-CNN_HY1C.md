---
type: paper
status: preliminary
title: "Detection of oil spill based on CBF-CNN using HY-1C CZI multispectral images"
authors: Kai Du, Yi Ma, Zongchen Jiang, Xiaoqing Lu, Junfang Yang
year: 2022
tags: [optical, multispectral, HY-1C, NIR, CBF-CNN, class-imbalance, oil-spill, reading-queue]
---

# Detection of oil spill based on CBF-CNN using HY-1C CZI multispectral images（Du et al., 2022）

> 來源：https://doi.org/10.1007/s13131-021-1977-x
> 發表：Acta Oceanologica Sinica, 41(7), 166–179
> 收錄狀態：第一輪紀錄；方法與主要表格已核對，後續可再做全文批判性深讀。

---

## 核心貢獻

以 Class-Balanced F loss 搭配 CNN，處理寬幅光學油污影像中油污像素遠少於海水的類別不平衡。論文在安達曼海與 Karimata Strait 的 HY-1C CZI 影像上評估油膜與乳化油分類，但各區域均使用當地標註像素訓練，因此不是 source-only、target-blind 的跨海域遷移研究。

## 關鍵方法

- 輸入：以中心像素為目標的 `11×11×4` patch。
- 模型：多尺度 CNN，包含 3×3、5×5 與 1×1 convolution。
- 損失：Class-Balanced F loss，直接針對類別不平衡設計。
- 增強：水平／垂直翻轉與 Gaussian noise；論文表示訓練樣本約擴充三倍。
- Baseline：cross-entropy CNN、DNN、RBF-SVM、Random Forest 等。

## 資料、波段與切分

| 項目 | 內容 |
|---|---|
| 感測器 | HY-1C Coastal Zone Imager（CZI），50 m，950 km swath |
| 波段 | 420–500、520–600、610–690、760–890 nm；最後一個為 NIR |
| IR 類型 | **NIR；沒有 SWIR 或 TIR** |
| 地區 | 安達曼海與 Karimata Strait；兩個日期／影像、三個 study areas |
| 觀測條件 | 弱 sunglint、油污主要呈負對比；部分區域有雲影與黑色背景干擾 |
| 切分 | study area 1 用於方法與參數選擇；areas 2/3 另各自抽取當地 train/test pixels |
| Zero-shot 判定 | **否**；測試區域並未封存，使用了當地標註訓練 |

## 重要數據／結果

- Study area 1：乳化油 F1 0.87、油膜 F1 0.94。
- Study area 2：乳化油 F1 0.88、油膜 F1 0.97。
- Study area 3：油膜 precision 0.94、recall 0.97、F1 0.96。
- 最佳鄰域大小為 11×11；class-balanced loss 對少數類別有明顯幫助。

## 與本專案的關聯

可借用的是類別不平衡處理、光學油污在 sunglint／雲影背景下的失敗型態，以及 NIR 與空間鄰域共同輸入的設計。不可借用的是「跨海域泛化已被證明」這個結論：Karimata 的測試仍使用 Karimata 當地標註訓練，而且 CZI 只有 VIS+NIR，與本專案 Sen2Like 的 VIS+NIR+SWIR 輸入並不等價。

## 研究缺口與假設

- 只涵蓋兩個日期、單一感測器與弱 sunglint 負對比條件。
- 以同景像素抽樣，可能高估跨場景或跨事件表現。
- 隱含假設是各 study area 都可取得足夠標註資料；這與台灣 target-blind 設定相反。
- 沒有 leave-one-sea、leave-one-event 或 sensor-held-out 實驗。

## 九步分析摘要

1. **領域地景**：屬於被動光學多光譜油污分類與類別不平衡研究。
2. **矛盾偵測**：多區域測試不等於跨區域 transfer，因各區域皆重新取樣訓練。
3. **引用鏈**：核心上游是 class-balanced loss、focal loss 與 pixel-centered CNN 分類。
4. **研究缺口**：缺嚴格事件／區域封存與強 sunglint 條件。
5. **方法審核**：局部分類設計清楚，但 pixel-level split 不能支持地理泛化主張。
6. **假設殺手**：若新海域不能取得標註，論文 protocol 無法直接部署。
7. **知識地圖**：連到類別不平衡、跨海域 split、NIR 光學油污特徵。
8. **文獻綜合**：它證明 CBF loss 對 CZI 少數類別有用，不證明 target-blind transfer。
9. **So What**：可作 loss baseline；不可作本研究 novelty 已被做過的反例。

## 相關頁面

- [[分割損失函數與類別不平衡]]
- [[跨海域_source-only_zero-shot油污偵測]]
- [[acolite_vs_sen2like]]

