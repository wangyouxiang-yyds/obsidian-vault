---
type: paper
status: methods-read
reading_scope: abstract-introduction-and-methods
title: "Marine Oil Pollution Monitoring Based on a Morphological Attention U-Net Using SAR Images"
authors: Lena Chang, Yi-Ting Chen, Ching-Min Cheng, Yang-Lang Chang, Shang-Chih Ma
year: 2024
tags: [SAR, oil-spill, U-Net, FA-MobileUNet, morphological-attention, CBAM, label-smoothing, Taiwan, reading-queue]
---

# Marine Oil Pollution Monitoring Based on a Morphological Attention U-Net Using SAR Images（Chang et al., 2024）

> 來源：https://doi.org/10.3390/s24206768
> 發表：Sensors 24(20), 6768；2024-10-21
> 閱讀進度：已讀 Abstract、Introduction 與 Section 2 Methods；Results 僅保留 Abstract 已報數字，尚未正式評讀。

---

## 這一階段先讀到的核心

這篇的主問題不是跨海域遷移，而是 **SAR 油污 segmentation mask 容易破碎、出現孔洞，如何得到較完整的污染範圍**。作者以 morphology 改造 FA-MobileUNet 的 attention module，並使用 label smoothing；台灣油污事件在 Abstract 中被定位為應用驗證，但這兩節沒有交代台灣是否是嚴格封存的 target domain。

## Abstract 拆解

| 要素 | 作者在摘要中的內容 | 初讀時要保留的疑問 |
|---|---|---|
| 問題 | SAR 油污分割結果有碎片與孔洞，不利於描述事故位置與範圍 | 完整外形是否真的比 boundary accuracy 更好，需看定量指標 |
| 方法 | 改造 FA-MobileUNet 的 CBAM，加入 Morphological Attention Module（MAM） | MAM 是真正學到形態，還是可學習 closing／平滑化？待 Methods |
| 不平衡 | 使用 label smoothing，降低模型對多數類別的過度自信 | label smoothing 並不直接重平衡類別數量，此機制宣稱需再審 |
| Dataset | 摘要只說使用 SAR | 感測器、訓練地區、事件與切分均未交代 |
| 主要數字 | mIoU 84.55%，作者稱比原始 U-Net 高 17.15% | 17.15% 的計算口徑及比較是否公平，均待 Results 核對 |
| 台灣驗證 | 偵測範圍與台灣油污事件報告記載區域一致 | 摘要沒有說 84.55% 來自台灣；「一致」可能是質性驗證 |

### Abstract 的一句話白話版

作者想把 SAR 油污分割圖中的小洞與碎片補得更完整，讓模型輸出的油污範圍更接近事故報告可使用的污染區域。

## Introduction 的論證鏈

### 第 1 段：為什麼油污監測重要

海運與工業活動帶來平台洩漏、船舶事故、非法排放、管線及港口作業等油污來源；油污造成生態損害，也提高清理成本。

### 第 2 段：為什麼使用 SAR

船舶及飛機巡查覆蓋有限；visible／IR 也受夜間與惡劣天候限制。SAR 可大範圍、全天候觀測；油膜抑制海面毛細波，使後向散射降低，所以在 SAR 中常呈暗區。

### 第 3 段：傳統 SAR 油污方法

作者依序回顧 CFAR＋target decomposition、co-polarized phase difference、region segmentation＋GLRT、degree of polarization，以及 SVM、tree ensemble、GAM、PLDA 等以人工特徵為主的方法。

### 第 4 段：深度學習與 U-Net 路線

論證從 CNN／DNN 取代 handcrafted features，進入 FCN／U-Net segmentation；列舉 adversarial U-Net、結合極化資訊與風速的 attention U-Net、自動產生訓練資料、multi-level feature fusion，以及 MLP＋U-Net 等方向。

### 第 5 段：真正的 research gap

前身 FA-MobileUNet 已整合 CBAM、ASPP 與 full-scale aggregation，也能區分 oil spill 與 lookalike；作者認為非平穩、非均勻的 sea clutter **可能**造成結果碎裂或形狀不完整。作者因此導入 morphology，目標是強化 spatial feature extraction，輸出更完整的油污區域。

### 第 6 段：文章結構

只交代後續 Sections 2–5，沒有新增貢獻主張。

## 從 Abstract／Introduction 可整理出的 contributions

作者沒有列正式的 numbered contribution list，但這兩節可重述為：

1. 以 morphology 改造 FA-MobileUNet 的 spatial attention，降低孔洞與碎片。
2. 以 label smoothing regularize 訓練，作者把它定位為類別不平衡處理。
3. 將模型應用於影響台灣海洋環境的真實油污事件，檢查輸出範圍與事故紀錄是否相符。

## Methods 統整

### 1. Dataset 與任務

| 項目 | 論文設定 |
|---|---|
| Dataset | extended MKLab |
| 原始來源 | EMSA CleanSeaNet 記錄的事件，依時間與位置下載 Sentinel-1 |
| 擴充來源 | 另搜尋 2015–2022 油污事件並取得 Sentinel-1 影像 |
| 地理資訊 | Methods 沒有列國家、海域或逐事件清單 |
| 感測器 | Sentinel-1、C-band、VV polarization、10 m pixel spacing |
| 數量 | 1,239 張；1,129 train、110 test |
| 原始尺寸 | 1250×650 pixels |
| 類別 | oil spill、lookalike、ship、sea surface、land，共五類 |
| 切分疑點 | 未說明按事件、場景或隨機影像切分 |

### 2. FA-MobileUNet 架構資料流

```text
Sentinel-1 SAR
      ↓
MobileNetV3 encoder
  ├─ Stage 1–2：CAM + Morphological Attention（保留空間形狀）
  └─ Stage 3–4：原始 CBAM（保留高階語意）
      ↓
ASPP bottleneck（多尺度 context）
      ↓
Full-scale aggregation decoder
  └─ 對齊並串接不同 encoder／decoder 尺度
      ↓
5-class segmentation
```

- **MobileNetV3**：以 depthwise separable convolution、inverted residual 與 SE module 降低計算量。
- **CBAM**：原本由 channel attention（CAM）與 spatial attention（SAM）組成。
- **ASPP**：置於 encoder／decoder bottleneck，擷取不同 receptive fields 的 context。
- **Full-scale aggregation（FA）**：把不同尺度的 encoder features 對齊後送入各 decoder stage，概念接近強化版 multi-scale skip connection。

### 3. Morphological Attention Module（MAM）

MAM 只取代 CBAM 的 **spatial attention module**，channel attention 仍保留。資料流為：

```text
feature F
  → 1×1 convolution
  → Dilation2D
  → Erosion2D          （dilation + erosion = closing）
  → 可重複 k 次 closing
  → 1×1 convolution + sigmoid
  → spatial attention map × 原始 feature F
```

- morphological kernel 初始大小為 3×3；作者只說它可在訓練中調整，沒有釐清調整的是 structuring-element 數值、權重或尺寸。
- 只放在 encoder Stages 1–2；Stages 3–4 仍用原始 CBAM。
- 作者的理由是 closing 有助於連接空間結構、填補小孔洞，但也可能損失細節，因此只處理較低階的 spatial features。
- Methods 未交代 kernel 初始化值、參數限制、erosion／dilation kernel 是否共享等實作細節。

### 4. Loss 與 label smoothing

Loss 是五類 categorical cross-entropy，平滑標籤為：

```text
y_ls = (1 − α)y + α/K
α = 0.1，K = 5
```

因此 true class target 為 0.92，其餘四類各為 0.02。這個操作對所有類別一視同仁，沒有改變 class frequency，也沒有針對 oil class 加權；所以它更接近 overconfidence regularization，而不是完整的 class-imbalance solution。

## 哪些可以參考？

### 高度值得參考

#### A. 把跨海域應用變成正式 protocol

論文清楚報告模型使用 extended MKLab 訓練，之後再展示台灣油污事件；但 Methods 沒有：

- 列出 source incidents／regions；
- 證明 Taiwan incidents 不在 2015–2022 擴充資料中；
- 說明 train/test 是否 event-held-out；
- 定義 Taiwan target 是否完全封存；
- 提供 Taiwan pixel-level GT 與跨域定量指標。

因此最值得本專案借用的不是它的「答案」，而是把它未形式化的 application demonstration 升級成：source incident-held-out validation → source-only model selection → sealed Taiwan target evaluation。

#### B. 低階空間特徵與高階語意特徵分開處理

Stages 1–2 使用 MAM、Stages 3–4 保留 CBAM 的設計原則可以移植到光學模型：低階層關注邊界與連續性，高階層保留油污／背景的語意判斷。若未來使用 UPerNet，可把它轉成「只在 early/FPN high-resolution features 加 shape-aware module」的單一消融，而不必照搬整套 FA-MobileUNet。

#### C. 明確把 lookalike 當類別

五類設定把 lookalike、ship、land 從一般 background 中拆開，對降低 false positive 很有意義。若本專案未來能可靠標註這些類別，可做 auxiliary classes；但現有標註若沒有穩定區分，就不能硬套。

### 可以做小型消融，但不宜當主軸

#### D. Morphological attention

MAM 在 feature space 做作者描述為「可在訓練中調整」的 morphological closing，但其 parameterization 與約束不明。這個概念理論上不綁定 SAR，可測試在光學油膜上是否減少孔洞；然而光學油膜可能本來就是細碎、斷裂或混合像素，closing 也可能錯誤合併分離區塊、擴張 mask。若要測，除 IoU 外還應檢查 boundary、component count、hole area、precision／recall 與小油膜召回。

#### E. Full-scale aggregation

多尺度 encoder–decoder aggregation 是合理 baseline，但 UPerNet 本身已具備 FPN／多尺度融合，因此對本專案不是強 novelty；最多用來解釋為何選擇 UPerNet 或做等容量 baseline。

### 不建議優先採用

#### F. Label smoothing 作為不平衡解法

`α=0.1` 不會增加 oil samples、降低 dominant event exposure 或直接處理 oil/background prevalence。對本專案而言，event-balanced sampling、明確的 ignore mask，以及已驗證的 Tversky loss 更直接。Label smoothing 最多只能作 calibration／overconfidence regularizer，不能當跨海域 generalization 的核心方法。

#### G. MobileNetV3 輕量化

只有在論文目標包含 edge deployment、推論速度或船載即時系統時才重要；對目前的研究問題定義沒有直接貢獻。

## 可放進 related work 的精確定位

> Chang et al. 將以 extended MKLab 訓練的 Sentinel-1 SAR segmentation model 應用於台灣油污事件，顯示跨地理應用的實務需求；然而該研究未定義 geographic source/target split、target-blind model selection 或台灣 pixel-level evaluation，因此不能視為嚴格的 source-only zero-shot 證據。本研究則把這個未形式化的 transfer 問題改造成主要評估協定，並轉向 S2/L8/L9 光學多光譜影像。

## 與本專案主問題的關係

這篇值得讀，因為它確實出現「模型拿台灣事件驗證」的表面結構；但目前不能稱為本研究的直接前例。Abstract 與 Introduction 尚未回答：

- 訓練資料是否完全不含台灣影像；
- 台灣影像、標註或事件報告是否參與模型設計、調參或 checkpoint 選擇；
- 台灣事件是否只開封一次；
- 84.55% mIoU 是 dataset test set 還是台灣事件結果；
- 台灣驗證是定量 ground truth，還是只和事故報告做視覺／範圍對照。

因此目前標記為：**相關的 SAR／Taiwan application precedent，但不是已證實的 source-only target-blind benchmark。** Methods 證實 training 使用 extended MKLab，卻仍未提供足以排除台灣事件重疊、target tuning 或 event leakage 的 provenance。

## 九步分析摘要（Abstract／Introduction／Methods）

1. **領域地景**：SAR 油污語意分割；重點是 oil spill／lookalike 的辨識與輸出完整性。
2. **矛盾偵測**：作者把 label smoothing 描述為不平衡處理，但它本身不改變類別比例，需看 loss 設計。
3. **引用鏈**：傳統 SAR polarimetric／statistical methods → CNN／U-Net → FA-MobileUNet（CBAM＋ASPP＋full-scale aggregation）。
4. **研究缺口**：既有模型的 mask 破碎與孔洞妨礙污染範圍描述。
5. **方法審核**：MAM 是 low-level feature 上、作者描述為可在訓練中調整的 closing attention；但 parameterization、dataset 事件切分與 Taiwan protocol 不透明。
6. **假設殺手**：較平滑、完整的形狀不必然更接近真實 oil boundary，可能只是過度 closing。
7. **知識地圖**：屬 SAR 相鄰證據，與本專案 S2/L8/L9 光學成像機制不同。
8. **文獻綜合**：提出形態注意力以修補 SAR 分割碎裂，並用台灣事故作應用展示；跨域 protocol 尚未成立。
9. **So What**：可借 shape-aware early feature 與 explicit lookalike 設計；更重要的是以嚴格 source/target contract 補上它未回答的跨域問題。

## 下一階段閱讀問題

1. 2015–2022 擴充影像的逐事件／海域清單是什麼，是否包含台灣？
2. 1129／110 split 是否按 incident 分組？
3. Taiwan incidents 有沒有 labels，還是只做 inference 與面積對照？
4. Taiwan 結果是否進入 model／MAM／label-smoothing 選擇？
5. mIoU 84.55% 在哪個 test set 計算？
6. 「符合事故報告」採用什麼定量或空間判準？

## 相關頁面

- [[跨海域_source-only_zero-shot油污偵測]]
- [[dataset_split_strategy]]
- [[分割損失函數與類別不平衡]]
