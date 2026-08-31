---
type: paper
title: "Prithvi-EO-2.0: A Versatile Multi-Temporal Foundation Model for Earth Observation Applications"
authors: Szwarcman, Roy, Fraccaro et al. (IBM Research / NASA IMPACT / 多機構)
year: 2024（arXiv:2412.02732，本次閱讀版本 v3, 2026-03-06 修訂）
tags: [foundation-model, self-supervised-learning, MAE, ViT, multi-temporal, HLS, geospatial]
---

# Prithvi-EO-2.0: A Versatile Multi-Temporal Foundation Model for Earth Observation Applications（Szwarcman et al., 2024）

> 來源：`D:\LAB\Papers\Prithvi-EO-2.0A Versatile Multi-Temporal Foundation Model for Earth Observation Applications.pdf`（外部路徑，未收錄至 `raw/papers/`）
> 發表：arXiv:2412.02732v3 [cs.CV]；模型與程式碼開源於 Hugging Face（ibm-nasa-geospatial/Prithvi-EO-2.0）與 GitHub（NASA-IMPACT/Prithvi-EO-2.0），並整合進 TerraTorch 工具包

---

## 核心貢獻

IBM/NASA 團隊推出第二代地理空間基礎模型 Prithvi-EO-2.0：以 Masked Autoencoder（MAE）在 4.2M 筆全球 HLS（Harmonized Landsat Sentinel-2，30m，6 波段）時序樣本上做自監督預訓練，並首次為此系列模型加入 **3D（時空）patch/positional embedding** 與 **可選的時間+地理座標 metadata bias**。模型有 300M（ViT-L）與 600M（ViT-H）兩種規模，在 GEO-Bench 12 個任務上比前代 Prithvi-EO-1.0 平均進步 8%，並在 9 個 SME 主導的真實應用（防災、土地覆蓋/作物、生態系統動態）上多次達到 SOTA 或逼近多模態基準。

---

## 關鍵方法

- **資料取樣**：先以 Copernicus LULC 100m + RESOLVE Ecoregions 對 HLS tile 做分層抽樣，確保 LULC 類別與 846 個生態區都有代表（都市過取樣、"Sea"/沙漠類別刻意降採樣以避免同質化過度代表），最終得到 3,028 訓練 tile → 切成 256×256 patch，4 個時間戳記（間隔 1–6 個月，橫跨 2014–2023），得到 4.2M 訓練 / 46k 驗證樣本。
- **架構**：標準 MAE（ViT encoder-decoder + MSE 重建 loss）改造兩處：
  1. 2D patch/positional embedding → 3D 版本（Conv3D，t=1，配合 3D sin/cos 位置編碼），支援 T 張影像的時序輸入。
  2. 額外加入 lat/lon（地點）與 year/day-of-year（時間）metadata，各自用 1D sin/cos 編碼後串接，再以「學習到的權重」加權相加到 embedding 上（而非當作額外輸入），並在訓練時以 10% 機率隨機丟棄 metadata，讓模型能在缺 metadata 時也運作。
- **訓練規模**：400 epochs，300M 版用 80× A100（~21,000 GPU-hr），600M 版用 240× A100（~58,000 GPU-hr），global batch size 3,840。
- **評估協議**：GEO-Bench（Lacoste et al. 2023）標準化流程——10 trial 超參數搜尋 + 10 次不同 seed 重複訓練，回報 mean/std/max/min；下游任務另用各領域對應 baseline（U-Net、U-Net++、ViViT、XGBoost/RF、U-TAE 等）。

---

## 重要數據/結果

- GEO-Bench 綜合表現：Prithvi-EO-2.0-600M-TL（含時空 metadata）與 600M 為最佳組合，全面優於前代 Prithvi-EO-1.0，並在 segmentation 任務上明顯超越 DeCUR/DINO-ResNet50（[[Wang2023_SSL4EO-S12]] 系列方法）。
- Sen1Floods11 洪水偵測：water class IoU 從 Prithvi-EO-1.0 的 79.6 提升到 600M-TL 的 83.1（+3.5）。
- 野火燒疤：burn scar IoU 從 76.8 → 83.2（+5.6，600M/600M-TL）。
- Burn Intensity（多類別燒傷強度）：mIoU 僅 31.2%（600M 為 31.1%），明顯低於前述二元任務，顯示模型對細碎、不連續的多類別標籤（如小面積高嚴重度區域嵌在大面積低嚴重度區域中）處理吃力，作者認為與 Prithvi 較大的 patch size 有關。
- Above Ground Biomass（BioMassters）：Prithvi-EO-2.0-300M 純光學（12 幀 S2 + LoRA）RMSE=33.40，仍輸給多模態（S2+S1）baseline 的 27.49；**加入 SAR 波段作為 encoder 額外輸入並未提升表現**（作者認為光學與 SAR 的本質差異使 Prithvi 難以直接吸收 SAR 輸入，建議改用外部 SAR encoder 做融合而非早期輸入層拼接）。
- GPP（總初級生產力）估計：600M-TL 版本 R² 全年份最佳（0.74–0.88），優於 RF/XGBoost/ResNet18 baseline 達 20%。
- 小樣本穩健性（Landslide4Sense，僅 50 張訓練圖）：Prithvi-EO-2.0-600M mIoU 67.0% vs U-Net 59.7%、U-Net++ 55.5%，顯示大型 GFM 在極小樣本情境下的優勢遠大於全量訓練時的差距。

---

## 與本專案的關聯

本專案的 [[self_supervised_pretraining_遙測.md]] 概念頁已明確點名 Prithvi 是「尚未被應用於光學海面油汙偵測」的 foundation model 之一——這篇論文正是該研究缺口所指的模型本尊（第二代）。與現有 wiki 中 [[Wang2023_SSL4EO-S12]]（MoCo/DINO 對比學習/蒸餾，10m Sentinel-2，1M patch，單時間戳）相比，Prithvi-EO-2.0 走的是完全不同的技術路線：MAE 重建式自監督、30m 中解析度、4 時間戳、且原生支援時空 metadata。這代表若要在本專案引入 pretrained backbone，SSL4EO-S12 與 Prithvi-EO-2.0 是兩種風格迥異的候選（對比學習 vs 遮罩重建；10m 高解析度單幀 vs 30m 中解析度多幀）。

此外，論文中 GPP 任務的 fine-tuning 架構（frozen Prithvi encoder + 輕量 decoder，將 HLS embedding 與其他模態特徵串接後回歸）提供了一個「凍結預訓練 encoder + 訓練小 decoder」的具體範本，可作為本專案若嘗試 linear probing 路線時的架構參考。

---

## 研究缺口與假設

- **論文承認的局限**：SAR 融合無效、燒傷強度多類別任務表現差、未與其他多模態 GFM 做系統性比較（列為 future work）。
- **假設殺手（本頁分析重點）**：預訓練資料在取樣階段刻意**降低「Sea」類別與同質化水體場景的比例**（見論文 Fig.1 與 III-A 節，"downsampled full sea and desert regions...to avoid over-representation of homogeneous areas"），且圖 1 顯示 Sea 佔訓練樣本僅 2%（原始 land tile 中佔 1%，但因訓練樣本另外還做了 sea-only 過濾，實際乾淨開闊水體樣本可能更稀少）。這意味著 Prithvi-EO-2.0 學到的表徵可能**對「開闊水面本身的細微紋理/光譜差異」（例如油膜 vs 乾淨海面）著墨甚少**，因為訓練目標從未特別強化這類同質化場景的重建難度。論文 12 個 GEO-Bench 任務 + 9 個 SME 任務中，最接近的是 Sen1Floods11（水陸邊界分類），但那是「哪裡是水」而非「水面上有沒有異常膜層」，兩者所需的視覺線索本質不同。
- **如果這個假設不成立**（即 Prithvi 的表徵其實仍能捕捉水面細微紋理），則 Prithvi-EO-2.0 對本專案會有直接助益；如果成立，則直接 fine-tune 全模型的效益可能被高估，需要先用低成本的 linear probing 驗證。

---

## 相關頁面

- [[self_supervised_pretraining_遙測.md]] — 已預先點名 Prithvi 為研究缺口的來源概念頁，本論文即為該缺口的具體模型
- [[Wang2023_SSL4EO-S12]] — 另一條 SSL 路線（對比學習/蒸餾 vs 本論文的 MAE 重建），可對照比較
- [[unsupervised_oil_detection.md]] — 本專案現行 unsupervised 方法總覽，與「pretrained encoder + fine-tune」路線形成對比
- [[deeprx_vae_架構與運作機制.md]] — 本專案目前的 frozen-encoder-like 設計（per-scene VAE），與 GPP 任務的 frozen-Prithvi-encoder 架構有方法論上的相似性可借鏡
