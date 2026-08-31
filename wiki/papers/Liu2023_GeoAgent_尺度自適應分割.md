---
type: paper
title: "Seeing Beyond the Patch: Scale-Adaptive Semantic Segmentation of High-resolution Remote Sensing Imagery based on Reinforcement Learning"
authors: Yinhe Liu, Sunan Shi, Junjue Wang, Yanfei Zhong
year: 2023
tags: [GeoAgent, reinforcement-learning, scale-adaptive, global-local, semantic-segmentation, remote-sensing, ICCV]
---

# Seeing Beyond the Patch（Liu et al., 2023）

> 來源：`raw/papers/Liu_Seeing_Beyond_the_Patch_Scale-Adaptive_Semantic_Segmentation_of_High-resolution_Remote_ICCV_2023_paper.pdf`
> 發表：ICCV 2023，pp. 16868–16875

---

## 核心貢獻

GeoAgent 將高解析遙測影像的滑窗分割建模成尺度選擇問題：Scale Control Agent（SCA）先依整景縮圖與目前 patch 位置選擇 context 倍率，再由共享權重的 local/context 雙分支分割網路融合細節與 patch 外資訊。其重點不是讓 reinforcement learning（RL）直接預測像素類別，而是讓 RL 替每個 patch 選擇合適的觀察尺度，免除人工標註「最佳尺度」的需求；分割網路與 reward 仍需要像素級 GT。（PDF p.2–5，§3.1–3.4）

---

## 關鍵方法

### State、action 與 episode

- 一張完整 HSR 影像是一個 episode，固定滑窗切出的每個 patch 是一個 timestep；action 完成後滑窗按既定順序前往下一個 patch。（PDF p.3–4，§3.1–3.2.1）
- state 由整張影像降採樣成的 global thumbnail（例：512×512）與目前 patch 在 thumbnail 上的 binary position mask 組成。（PDF p.4，§3.2.1）
- thumbnail 先經 CNN；position mask 遮掉目前位置以外的特徵，再取 3×3 區域並 global-average-pool，送入 MLP actor head 輸出尺度 action 機率。critic 與 actor 共用 CNN backbone，但有獨立 value head。（PDF p.4，§3.2.1–3.2.2）
- action `a=1` 表示只使用 local branch；`a>1` 表示以目前 patch 為中心擴大 `a` 倍範圍，再降採樣回固定輸入尺寸。正文未明列完整 action set；§4.3 的消融使用 2–6× context，故「1–6×」只能視為由實驗推知。（PDF p.4、p.7）

### 雙分支 segmentation network

```text
local patch（原解析度 h×w） ─→ shared segmentation network ─→ f_local

context（a 倍範圍）
  └→ downsample 回 h×w ─────→ shared segmentation network ─→ f_context
                                                           │
             依地理位置取出 local 對應範圍 ←───────────────┘
                              ↓
                      concat + 3×3 conv
                              ↓
                       1×1 conv + softmax
```

兩個分支的所有網路層共享權重；`a=1` 時 context branch 關閉。context feature 依地理座標裁出 local footprint，與 local feature 串接後用 3×3 convolution 融合。local/context 兩支另設 auxiliary segmentation heads 協助訓練。（PDF p.4–5，§3.3）

### Reward 與最佳化

Patch immediate reward 是所選尺度相對 local-only 的分割改善：

\[
R_{patch}(a,y)=Score(y,\hat y(a))-Score(y,\hat y(1)),
\quad Score=mIoU+mF1
\]

整景完成後再給 terminal mapping reward：

\[
R_{map}=T\,[Score(Y_i,\hat Y_i)-Score(Y_i,\hat Y_i^{1})]
\]

其中 `T` 是該景 patch 數，`Y_i^1` 是整景 local-only 結果。（PDF p.4，Eq. 1–2）

作者採用 A2C，以 advantage `A=Q−V` 更新 actor，critic 以 TD target 的 MSE 更新；另以 DQN、PPO 做敏感度分析。分割網路本身仍以 pixel-wise cross-entropy 與 auxiliary losses 做監督式訓練。（PDF p.5，Eq. 5–7；p.7–8，§4.4）

### 訓練流程

1. 先以 random action 預訓練 segmentation network。
2. 再訓練 SCA 學習尺度政策。
3. 最後 joint learning；scale agent 與 segmentation network 每 100 steps 交替／非同步更新，以降低同時最佳化的不穩定。（PDF p.5–6，§3.4、§4.1）

實作使用 ResNet-18 policy、ResNet-50 segmentation backbone、512×512 patch、batch 16、SGD lr 0.001；GID/WUSU 訓練 20k steps，FBP 60k，前置 segmentation pretraining 分別為 10k/10k/20k steps。（PDF p.5–6，§4.1）

---

## 重要數據／結果

Table 1 中，將 GeoAgent 套在 DeepLabV3+ 後的結果如下（指標為多類別 mean IoU／mean F1，不是 Oil IoU）：

| Dataset | DeepLabV3+ IoU | GeoAgent-DLV3+ IoU | ΔIoU | DeepLabV3+ F1 | GeoAgent-DLV3+ F1 | ΔF1 |
|---|---:|---:|---:|---:|---:|---:|
| WUSU | 63.39 | 76.19 | +12.80 | 69.97 | 80.56 | +10.59 |
| GID | 64.89 | 75.51 | +10.62 | 68.65 | 76.07 | +7.42 |
| FBP | 57.08 | 62.25 | +5.17 | 59.46 | 63.21 | +3.75 |

（PDF p.6，Table 1）

尺度政策消融顯示：context-only 隨降採樣倍率增加而明顯掉分；fixed context 有時會傷害結果，且最佳固定尺度依 dataset 改變；random scale 優於 local-only，而 GeoAgent 最佳。不過 Fig. 4 只有長條圖，正文沒有提供各消融臂的精確數字。（PDF p.7，§4.3、Fig. 4）

A2C 收斂較快，但 DQN／PPO／A2C 最終 mapping improvement 差異不大，表示框架未必高度依賴特定 RL 演算法。Table 2 的「accuracy improvement」沒有清楚說明是 IoU、F1 或複合 score，不能過度解讀。（PDF p.7–8，§4.4、Table 2）

---

## 九步系統化分析

### 1. 領域地景（Domain Map）

本研究屬於高解析遙測語意分割、multi-scale／global-local context 與 dynamic network routing 的交集。它要解決的是：無論 ASPP、FPN 或 self-attention 如何設計，模型都不能看見輸入 patch 邊界外的真實像素；固定 context 又可能不適合所有地物尺度。主要競爭方法包含 GLNet、WiCoNet、CascadePSP、MagNet 與 RAZN。（PDF p.2–3，§2）

### 2. 矛盾偵測（Contradiction Detector）

論文顯示「更大 context 不一定更好」：context-only 會因降採樣破壞細節，某些 fixed scale 也會傷害結果。這與本專案 P3 的結果並不真正矛盾：P3 證明 ASPP 的大 receptive field 對大型 diffuse 油膜仍有用，但 ASPP 只聚合 patch 內特徵；GeoAgent 則讀取 patch 外新像素。兩者應視為互補，而非用 global-local 取代 ASPP。

### 3. 引用鏈追蹤（Citation Chain）

- GLNet／WiCoNet：提供 fixed global-local 雙分支先例。
- CascadePSP／MagNet：提供 coarse-to-fine、多階段跨尺度推論先例。
- RAZN：以 RL 判斷 whole-slide image 是否需要 zoom-in，是 SCA 的直接動態路由上游。
- A2C：提供 actor–critic 與 advantage-based policy update。

GeoAgent 的主要新意是把這些想法組合成「依 patch 狀態選 context 倍率」的遙測分割框架。（PDF p.2–3，§2；p.5，§3.4）

### 4. 研究缺口掃描（Gap Scanner）

論文未交代 action 數 `N`、discount factor、Stable-Baselines3 版本與多項 A2C 超參數，也未說明 sliding-window overlap、stitching、border padding、FBP/WUSU split 細節。沒有 FLOPs、latency、memory、seed variance 或 confidence interval。跨資料集部分只展示 action maps，沒有 unseen-dataset segmentation IoU，不能視為準確率泛化證據。（PDF p.6–8）

### 5. 方法審核（Method Audit）

Table 1 在三個資料集、五種 FCN 上皆呈現改善，且包含 fixed/random/context-only 消融，支持「動態尺度有訊號」。但比較缺乏重複實驗與計算量控制；GeoAgent 對 `a>1` 通常須跑 local/context 兩次 encoder，與單分支 baseline 的算力並不等價。大型增益可信為值得驗證的方向，但不能直接外推為本專案可得到相同幅度。

### 6. 假設殺手（Assumption Killer）

方法依賴三項關鍵前提：global thumbnail 必須保留足以選尺度的訊號；不同 patch 的最佳尺度必須真的不同；scale policy 必須能跨場景泛化。若整景縮圖後油膜只剩數個像素，或所有油膜都偏好同一 context，SCA 就沒有存在必要。此外，action 不改變下一個 state（滑窗仍固定前進），因此除 terminal whole-map reward 外，其結構更像 contextual bandit，而非強序列依賴的 MDP；完整 A2C 可能是過度複雜化。

### 7. 知識地圖（Knowledge Map）

本論文連接 [[20260731_ASPP_rate適配實驗]] 的 patch 內 receptive field、[[Szwarcman2026_PrithviEO2]] 的 30m HLS 預訓練物理尺度，以及本專案大型 diffuse 油膜的高信心 FN。它與 iForest／DeepRX 的「異常是否為油」路線不同：GeoAgent 不改變 supervised oil classifier，而是補足其 patch 外空間資訊。

### 8. 文獻綜合（Synthesis）

GeoAgent 用 SCA 對每個 patch 選擇 context 倍率，再由共享權重的 local/context 雙分支完成分割，直接處理固定滑窗看不到外部資訊的問題。它在三個 1–4m GF-2 多類別 LULC 資料集上有一致增益，但實驗缺乏計算公平性、重複統計與跨域 segmentation accuracy。其核心思想與本專案的大型油膜 FN 對得上，但原版整景 thumbnail 與 A2C 不宜直接照搬。

### 9. So What 測試

本專案最直接的可行延伸是 `10m local + 20/30m regional context + shared Prithvi`：local 保留油膜邊界，30m context 則對齊 Prithvi 的 HLS 預訓練尺度。應先用 fixed-scale 與 oracle-scale 實驗確認 context headroom 及最佳尺度異質性；若固定 30m 已普遍最好，就不需要 RL；只有在不同 patch 的最佳尺度明顯不同時，才依序測 supervised router、contextual bandit，最後才是 A2C。

---

## 與本專案的具體關聯

### 建議的尺度對齊版本（本專案推論，非原論文方法）

| Action | VRT 讀取視窗 | resize 後輸入 | 等效解析度 | 地面覆蓋 |
|---|---:|---:|---:|---:|
| 1× | 256×256 | 256×256 | 10m | 2.56km |
| 2× | 512×512 | 256×256 | 20m | 5.12km |
| 3× | 768×768 | 256×256 | 30m | 7.68km |

現行 Prithvi branch 只取 `[B02,B03,B04,B8A,B11,B12]`，local/context 應使用完全相同的 band selection 與 shared encoder。context feature 中與 local footprint 對應的中心區域需以座標／ROI alignment 對齊到 local token grid，再與 `F_local` 融合，最後沿用現行 DeepLabV3 ASPP `(6,12,18)`；P3 已否定縮小 ASPP rates，global-local 是新增 patch 外資訊，不應順便改動 ASPP。

### Reward 與評估建議（本專案推論）

原文的 `mIoU+mF1` reward 容易被大量 background 主導，且與本專案 Evaluation Contract 不一致。訓練可先採相對 local-only 的 Tversky-loss 改善作 patch reward，整景再用 Oil IoU 改善作 terminal reward；正式模型選擇仍使用 `S=(M1+M2)/2`、12-event cluster bootstrap，並逐 small／mid／large 報告 Oil IoU、FP、FN。

### 建議實驗順序

1. local-only vs fixed 2× vs fixed 3× shared-Prithvi dual branch。
2. 在 training/validation 估計每個 patch 的 oracle-best scale，確認是否存在足夠 headroom 與尺度異質性。
3. 若固定尺度已足夠，停止 RL；若最佳尺度確實異質，先訓練 supervised router。
4. supervised router 仍無法利用整景 reward 時，才測 A2C；不得以 test labels 計 reward 或挑尺度。

---

## 研究限制與外推邊界

- 原任務為 1m/4m GF-2 多類別 LULC，不是 10m Sentinel-2/Landsat 的低對比稀有 binary oil。
- 本專案整景 10980×10980 若縮到 256/512，物理解析度約 429/214m，油膜訊號可能消失；應優先使用 regional context，而不是照搬 whole-scene thumbnail。
- 本專案僅約 12 個事件群，真實經緯度或事件 metadata 容易讓 policy 記住事件；state 僅應使用場景內相對位置。
- context 降採樣可能抹掉細長油膜，因此 local branch 必須始終保留，不能使用 context-only。
- 共享參數不等於共享計算量；dual branch 的 GPU memory／latency 必須另行量測。

---

## 相關頁面

- [[尺度自適應_Global-Local分割]] — GeoAgent 與本專案尺度對齊延伸的概念頁
- [[Szwarcman2026_PrithviEO2]] — 現行 encoder 的 30m HLS 預訓練來源
- [[20260731_ASPP_rate適配實驗]] — patch 內 receptive field 實驗；大尺度 context 有用但仍受 patch 邊界限制
- [[GT_expand_pipeline]] — 現行 256×256 GT-centric patch 與評估流程
- [[20260716_資料溯源洩漏與評估合約v1]] — 12-event cluster bootstrap 與正式主終點
