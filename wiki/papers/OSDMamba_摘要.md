# OSDMamba 論文摘要整理

**論文標題**：OSDMamba: Enhancing Oil Spill Detection From Remote Sensing Images Using Selective State-Space Model (IEEE LGRS, 2025)

---

## 🎯 核心貢獻 (Core Contributions)

1. **首個基於 Mamba 的油汙偵測架構**：提出 OSDMamba，這是第一個專門為油汙偵測 (OSD) 設計的狀態空間模型 (SSM) 架構，突破了傳統 CNN 與 Transformer 的限制。
2. **選擇性掃描機制 (SS2D)**：利用 Mamba 的選擇性掃描機制有效擴大感受野 (Receptive Field)，能捕捉全局上下文資訊，解決了 CNN 因感受野受限而難以偵測微小油汙面積的問題。
3. **非對稱解碼器 (Asymmetric Decoder)**：設計了整合 **ConvSSM** 與 **深層監督 (Deep Supervision)** 的解碼器，增強了多尺度特徵融合，提升模型對少數類別（油汙）的敏感度。
4. **SOTA 效能表現**：在兩個公開資料集 (M4D, MADOS) 上分別取得了 8.9% 與 11.8% 的 mIoU 提升，展現了卓越的偵測準確度與對類別不平衡問題的魯棒性。

---

## 🏗️ 網路架構 (Architecture)

### 1. 編碼器 (Encoder)
- **基底**：參考 VMamba-Tiny 與 Swin-UMamba 的設計原則。
- **構成**：由四個階段的 **視覺狀態空間區塊 (VSS Blocks)** 組成（配置為 {2, 2, 9, 2}）。
- **關鍵技術**：每個 VSS 區塊採用 **2-D 選擇性掃描 (SS2D)**，將影像 Patch 展開為四個方向的序列進行處理，確保能同時提取局部與全局特徵。

### 2. 解碼器 (Decoder)
- **非對稱設計**：針對不同階段設計了兩種上採樣區塊：
    - **前期階段**：結合 **ConvSSM** 與 VSS 區塊，專注於保留空間邊界與細節，強化多尺度特徵融合。
    - **後期階段**：採用輕量化的 Patch Expansion 與雙 VSS 區塊設計。
- **深層監督 (Deep Supervision)**：在 1/4, 1/8, 1/16 尺度輸出處應用 1x1 卷積，引導特徵學習。

### 3. 損失函數 (Loss Function)
- **混合損失**：結合 **Focal Loss**（處理類別極度不平衡，Oil 佔比極低）與 **Dice Loss**（優化邊割邊界）。

---

## 📊 實驗結果
- **M4D Dataset**：mIoU 達到 70.25%，在油汙類別上比 U-Net 提升了 12.18%。
- **MADOS Dataset**：在 F1-score 與 mIoU 上均超越了目前主流的模型（如 MarineXt, SegNext）。
- **效率**：與 Transformer 模型相比，OSDMamba 在參數路與計算成本 (FLOPs) 上更具競爭力。

---

## 🔗 相關資源
- **原始 PDF**: [raw/papers/OSDMamba.pdf](../../raw/papers/OSDMamba.pdf)
- **官方程式碼**: [https://github.com/Chenshuaiyu1120/Oil-Spill-detection](https://github.com/Chenshuaiyu1120/Oil-Spill-detection)
