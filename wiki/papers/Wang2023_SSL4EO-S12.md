---
type: paper
title: "SSL4EO-S12: A Large-Scale Multimodal, Multitemporal Dataset for Self-Supervised Learning in Earth Observation"
authors: Yi Wang, Nassim Ait Ali Braham, Zhitong Xiong, Chenying Liu, Conrad M. Albrecht, Xiao Xiang Zhu
year: 2023
tags: [self-supervised-learning, foundation-model, pretraining, sentinel-1, sentinel-2, dataset, backbone, linear-probing, fine-tuning, deeplabv3plus]
---

# SSL4EO-S12（Wang et al., 2023）

> 來源：`raw/papers/SSL4EO-S12.pdf`
> 發表：IEEE Geoscience and Remote Sensing Magazine, 2023

---

## 核心貢獻

提出 SSL4EO-S12：一個全球尺度、多模態（SAR + 光學）、多時相（4 季）的 Sentinel-1/2 無標註資料集（25 萬個地點、約 300 萬張 patch），專為遙測領域的自監督預訓練（Self-Supervised Learning, SSL）設計。同時系統性地在 EuroSAT、BigEarthNet、So2Sat-LCZ42（場景分類）、DFC2020（語意分割）、OSCD（變化偵測）等下游任務上，比較 MoCo-v2/v3、DINO、MAE、data2vec 四種 SSL 演算法搭配 ResNet50/ViT-S 兩種 backbone 的 linear probing 與 fine-tuning 表現，並與既有資料集（ImageNet、SEN12MS、[[SeCo]]、BigEarthNet 自身）做對照。核心論點：**遙測影像有自己的統計特性（多波段、多時相、SAR/光學互補），直接沿用電腦視覺（ImageNet）預訓練會有 domain gap，而遙測領域專屬、大規模、低地理重疊的預訓練資料集能穩定帶來下游任務增益。**

---

## 關鍵方法

**資料收集**：沿用 [[SeCo]] 提供的 baseline（用 Google Earth Engine 下載處理），但做了兩項關鍵改良：
1. **Overlap filtering**：以全球人口前 10,000 大城市為中心，用高斯分布抽樣座標，再套用 grid search 演算法過濾掉地理重疊的 patch（SeCo/SEN12MS 都有嚴重的 patch 重疊問題）
2. **多感測器**：同時收集 Sentinel-1 GRD（VH/VV，10m）、Sentinel-2 L1C（13 波段，TOA 未校正）、Sentinel-2 L2A（12 波段，已大氣校正）三種資料，四季各一次，最終約 100 萬組 S1/S2-L1C/S2-L2A 三聯資料。

**四種 SSL 演算法 × 兩種 backbone**：
| SSL 方法 | 類型 | ResNet50 | ViT-S/16 |
|---|---|---|---|
| MoCo-v2/v3 | 對比學習（contrastive） | ✓ | ✓ |
| DINO | 師生蒸餾（teacher-student distillation） | ✓ | ✓ |
| MAE | 遮罩重建（masked autoencoding） | ✗（架構不相容） | ✓ |
| data2vec | 遮罩 + 特徵級聯合嵌入 | ✗ | ✓ |

MAE/data2vec 因遮罩機制需要 token 結構，只能用在 ViT，無法套用在連續卷積的 CNN 上。

**遙測專屬資料增強**：
- `RandomSeasonContrast`：兩個 view 各自從不同的隨機季節抽取（給對比/蒸餾類方法 MoCo/DINO 用）
- `RandomSeason`：單一 patch 隨機抽一個季節（給重建類方法 MAE/data2vec 用）
- `RandomSensorDrop`：訓練時隨機丟掉 SAR 或光學其中一種模態，用於多模態早期融合訓練

**評估協議**：
- **Linear probing**：凍結 backbone，只訓練一個線性分類頭 → 測試表徵本身的品質
- **Fine-tuning**：整個網路端到端訓練 → 測試可達到的效能上限
- 分割任務（DFC2020）以 [[DeepLabV3+.md|DeepLabV3+]] 作為下游分割頭；變化偵測（OSCD）用 U-Net，輸入為兩個時間點的特徵圖差值

---

## 重要數據/結果

**Linear probing（Table V，RGB only）**：SSL4EO-S12 全面領先——比 ImageNet 高約 10%，比 [[SeCo]] 高約 6%，比 SEN12MS 高 1.7%–3.5%（EuroSAT/BigEarthNet-10%/BigEarthNet-100%）。

**DFC2020 語意分割（Table VI，DeepLabV3+ 分割頭，ResNet50+MoCo-v2 預訓練，OA/AA/mIoU）**：
| 資料集 | OA | AA | mIoU |
|---|---|---|---|
| rand.init（無預訓練） | 81.97 | 56.46 | 42.11 |
| SeCo | 87.31 | 57.05 | 49.68 |
| SEN12MS | 88.64 | 67.69 | 54.83 |
| SSL4EO-S12 | 89.58 | 64.01 | 54.68 |

注意：SSL4EO-S12 在 AA/mIoU 上**略輸給 SEN12MS**（論文解釋為 DFC2020 的地理分布與 SEN12MS 更接近），並非「全面獲勝」。

**OSCD 變化偵測（Table VII，precision/recall/F1）**：
| 資料集 | precision | recall | F1 |
|---|---|---|---|
| rand.init. | 72.31 | 13.75 | 23.10 |
| SeCo | 74.85 | 17.47 | 28.33 |
| SEN12MS | 74.67 | 19.26 | 30.62 |
| SSL4EO-S12 | 70.23 | 23.38 | 35.08 |

SSL4EO-S12 的 **precision 反而是四者最低**，論文解釋是類別嚴重不平衡（把所有像素都判成「未變化」也能拿到高 precision），rand.init. 的高 precision 是「什麼都不做」的假象，recall/F1 才是有意義的比較基準。**這是一個重要的反例：不能簡化為「有預訓練+fine-tune 全面贏無預訓練」，要看具體指標與資料集特性。**

**消融實驗（Section VI）**：多模態早期融合、季節增強、大氣校正（L1C/L2A/合併）作為增強手段、預訓練資料量（10 萬 → 100 萬 patch）規模效應、t-SNE 表徵視覺化，皆顯示正向但非線性的增益。

---

## 與本專案的關聯

**最直接的關聯是「研究缺口」而非「直接可用的方法」**：本季會話中特地做過文獻調查（WebSearch + 閱讀 remotesensing-17-03681.pdf 這篇 2025 回顧文獻）確認——**目前沒有任何遙測專屬 SSL/foundation model（SSL4EO-S12、Prithvi、SatMAE、RingMo 等）被應用在「光學」海面油汙偵測上**。既有的油汙 SSL 相關工作僅限於高光譜、同域內（in-domain，非 transfer）自監督（如 Duan et al. 2021/2022、Kang et al. 2023 SSTNet）；SAM 雖被用過，但主要當標註加速工具、且集中在 SAR 影像。

**方法論上可借用之處**：
- 本專案的 [[DeepLabV3+.md|DeepLabV3+]] 訓練目前是 **from-scratch**（見 `class_weights=[13.0,1.0]`、無預訓練 backbone 的設定，參考 [[deeplabv3plus_358clean_overfitting_改善方向.md]] 中 A1 方向已在嘗試導入 pretrained backbone）。SSL4EO-S12 證實：即使只用 ResNet50+MoCo-v2 這種相對輕量的預訓練，DFC2020 分割任務的 mIoU 就能從 42.11（rand.init）提升到 54.68/54.83——這對本專案「input conv 改造 + differential lr + 兩階段 fine-tune」的既定改善方向提供了外部文獻佐證。
- `RandomSeasonContrast`/`RandomSeason` 這類「跨季節當作資料增強」的思路，對本專案未來若要導入時序資訊（多期影像）可能有參考價值。

**方法論上的落差**：SSL4EO-S12 的 11 波段對照表跟本專案原先 CLAUDE.md 記載的 11 波段（443–2202nm）高度相似，但本專案實際資料集 [[ms6_sen2like.md]] 目前是 **8 波段**（B01/02/03/04/08/8A/11/12），並非 11 波段——若未來要直接借用 SSL4EO-S12 的預訓練權重，需要處理輸入通道數不匹配的問題（如 A1 方向已在做的 input conv 改造）。

---

## 研究缺口與假設

**論文自承的局限**：
- SSL4EO-S12 在部分下游任務（DFC2020 的 AA/mIoU）並未穩定贏過地理分布更貼近測試集的 SEN12MS，顯示「資料量大、地理覆蓋廣」不必然在所有下游任務上都占優，資料分布的匹配程度仍是關鍵變數。
- OSCD precision 偏低的例子說明：預訓練效益要看具體指標，類別不平衡場景下，天真的整體指標比較可能誤導。

**未明說的隱含假設**：
- 論文預設「下游任務的地理分布 / 感測器 / 波段組成」與 SSL4EO-S12 預訓練資料相近時效益才穩定顯現——但海面油汙偵測的資料分布（少量正樣本、強烈類別不平衡、特定地理事件如 Gulf of Mexico）與 SSL4EO-S12 的「全球城市中心取樣」在空間分布上差異很大，遷移效益需要實測驗證，不能直接假設成立。
- 論文的四種 SSL 演算法都是在「乾淨、無雲、良好標註」的基準資料集上驗證；本專案的實務資料存在雲遮蔽、標註品質不一（見 [[20260708_JM距離判別髒資料可行性診斷.md]] 提到的髒資料問題），SSL 預訓練在這類雜訊資料上的穩健性未經驗證。

**與本研究的關聯**：這個「SSL 尚未被應用於光學海面油汙偵測」的缺口，若本研究決定嘗試「用 SSL4EO-S12 預訓練權重 fine-tune 到油汙分割」，將會是具有一定新穎性的研究角度，而非單純複製既有做法。

---

## 相關頁面

- [[deeplabv3plus_358clean_overfitting_改善方向.md]] — 本專案現行 pretrained backbone 改善方向（A1），與本文獻的預訓練增益數據互為佐證
- [[DeepLabV3+.md]] — 本專案 DeepLabV3+ 工程紀錄，對照本文獻 DFC2020 分割任務用的同一套架構
- [[ms6_sen2like.md]] — 本專案實際 8 波段資料集，與本文獻 11 波段對照表的落差
- [[unsupervised_oil_detection.md]] — 既有 unsupervised 油汙偵測方法總覽，補充「SSL/foundation model 尚未應用於光學油汙偵測」這個文獻缺口
- [[20260708_JM距離判別髒資料可行性診斷.md]] — 本專案資料集髒資料問題，與本文獻預訓練資料乾淨假設的落差
