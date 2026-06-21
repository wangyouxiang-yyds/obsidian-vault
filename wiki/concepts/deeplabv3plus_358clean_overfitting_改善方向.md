---
date: 2026-06-21
type: concept
tags: [deeplabv3plus, supervised, oil-detection, overfitting, pretrained-backbone, transfer-learning, sentinel-2, training-improvement]
related: [DeepLabV3+, deeprx_vae_架構與運作機制]
---

# DeepLabV3+ 358-clean 訓練改善方向統整 + A1（Pre-trained Backbone）細節

> 本頁統整 DeepLabV3+ 在 358 clean 場景（排除 82 dirty/blur）5-fold CV 的 overfitting 根本診斷，以及各改善方向的投入/效益分析。A1（Pre-trained Backbone）因投入最低、效果最大，另以獨立章節詳細說明實作細節。

---

## 1. 問題背景

- 資料集：Sentinel-2 8-band（B01/B02/B03/B04/B08/B8A/B11/B12），358 clean 場景，5-fold CV
- 模型：DeepLabV3+ ResNet-50，**從 scratch 訓**，imgsz=256
- 訓練設定：class_weights=[13.0, 1.0]、lr=5e-5、batch=16、patience=50、epochs=300、AMP
- 觀察現象：train loss 收斂，val loss 飄升 → overfitting；oil IoU 普遍很低、誤報率高
- 歷史背景：過去做過的 unsupervised 路線（v3 micro F1=0.0081、v9 micro F1=0.0085，差距是噪聲）已認定走不下去

---

## 2. 根本診斷

from-scratch 訓練 8-band DeepLabV3+ 在 358 場景 + 極端不平衡的設定下，train loss 收斂 + val loss 飄升幾乎是**必然現象**。

**根本原因不是 regularization 不夠，而是「初始化 + 資料量」配不上模型容量**：ResNet-50 約 25M 參數，要在 358 張噪聲很大的衛星影像上從頭學 ImageNet 級別的 low-level feature，根本沒有餘力學 oil-specific 的 semantic。

- val loss 飄升的本質：初始化點離真正的 minimum 太遠，模型只能 memorize train set
- Weight decay / dropout 對「資料與初始化錯位」這個主因幫助有限

---

## 3. 改善方向

### A. Overfitting 修正方向（依 ROI 排序）

| # | 方向 | 投入 | 預期效果 | 備註 |
|---|---|---|---|---|
| 1 | **Pre-trained backbone**（用 RGB 子集 B02/B03/B04 走 ImageNet 權重；剩 5 個 band 額外加 input conv 並 zero-init） | 1-2h | ★★★★★ | 最大槓桿。或試 SatlasPretrain / Prithvi-100M / SSL4EO 這類 S2-pretrained backbone |
| 2 | 強化資料增強（H/V flip、90° rotation、隨機 crop、per-band brightness jitter、加雜訊） | 2h | ★★★★ | 現行 augmentation 估計偏弱 |
| 3 | Loss 改成 Combo (CE + Dice/Tversky) | 1h | ★★★ | Tversky 可調 FP/FN trade-off |
| 4 | Hard-positive oversampling（含 oil pixel 的 patch 採樣機率拉到 ~50%） | 2h | ★★★ | 對應「誤報率高」 |
| 5 | 縮小 backbone（ResNet-50 → ResNet-18 / EfficientNet-b0） | 30min | ★★ | 但前提是不打算用 pre-trained，否則先做 #1 |
| 6 | EMA weights | 30min | ★★ | 便宜的 regularization |
| 7 | Stop on val IoU 而非 val loss、patience 縮短到 20-30 | 10min | ★ | patience=50 太鬆 |

> Weight decay / dropout 對「資料初始化錯位」這個主因幫助有限，故排後面。

### B. 監督式以外可以試的路線

| 路線 | 描述 | 評估 |
|---|---|---|
| B1. SSL pre-train → fine-tune | 把 358 + 82 dirty + 未標註 S2 拿去 SimCLR/MAE/SeCo 預訓，再 fine-tune | **最該做**；把過去 v9 努力轉化成 backbone 預訓 |
| B2. Semi-supervised (FixMatch / MeanTeacher) | 用模型對未標註資料生成 pseudo-label 同訓 | 中等 ROI、實作較複雜 |
| B3. 兩階段（scene/ROI classifier → segmentation） | 第一階粗篩有沒有油，第二階只對候選區段做分割 | **直擊「誤報率高」痛點** |
| B4. 物理特徵注入（NDWI、波段比值、Sun-glint mask） | concat 到 8 band 後當額外通道 | 中等 ROI、便宜但只是錦上添花 |
| B5. 多任務學習（oil + water + cloud） | 共享 backbone | 高 ROI，但成本取決於是否有對應 mask |
| B6. SegFormer / UperNet + Swin | 換 transformer-based | **不建議現在做**，更吃資料 |
| B7. 5-fold ensemble + TTA | 推論期票決 | 訓練問題沒解決時用處有限 |

### C. 推薦執行順序

1. 先做 A1（pre-trained backbone）+ A2（augmentation）+ A4（hard-positive oversampling）— 預估一輪 fold 就會看到 val 曲線變平
2. 觀察後再加 A3（loss combo）
3. 中期投入 B1（SSL pre-train）+ B3（兩階段）

---

## 4. A1 詳細細節：Pre-trained Backbone

### 4.1 核心觀念

ImageNet 預訓權重把 low-level 視覺特徵（邊緣、紋理、梯度、色塊）封裝好了。這些特徵跨領域通用，**衛星影像也適用**。

From-scratch 訓練等於要從 358 張影像重新學這些特徵，模型把容量都耗在 low-level，根本沒餘力學 oil-specific 的 semantic。「val loss 飄升」這現象，大多時候不是「regularization 不夠」而是「初始化點離真正的 minimum 太遠，模型只能 memorize train set」。

### 4.2 三條可行路線

**路線 1：ImageNet transfer + input conv 改造**（成本最低，立刻可做）

- DeepLabV3+ 的 ResNet-50 第一層是 `Conv2d(3, 64, k=7, s=2)`
- 改造成 `Conv2d(8, 64, k=7, s=2)`，權重初始化策略：
  - B02（藍）/ B03（綠）/ B04（紅）對應 channel：複製 ImageNet R/G/B 權重
  - 其餘 5 個 NIR/SWIR band：用 ImageNet 三通道的**平均值**初始化（或 zero-init，但會收斂較慢）
- 其他層全部載入 ImageNet 權重
- Decoder 隨機初始化（本來就是任務特定）

**路線 2：Satellite-pretrained backbone**（效果最好但要適配模型）

| 資源 | 描述 |
|---|---|
| SatlasPretrain（Allen AI） | 多模態 Sentinel-1/2 |
| Prithvi-100M（NASA + IBM） | Sentinel-2 MAE 預訓，100M 參數 ViT |
| SSL4EO-S12（Wang et al.） | 多種 SSL 方法在 S2 上 |
| SeCo | Seasonal Contrast，S2 時序對比學習 |

多數直接吃 8+ band，省下 input conv 改造步驟。缺點：通常搭 UperNet/SegFormer decoder，要重接到 DeepLabV3+。

**路線 3：自己做 SSL 預訓**（中長期，與 B1 重疊）

- 358 + 82 dirty + 任何未標註 S2 場景丟給 MAE 或 SimCLR
- 把 v9 stage1 學到的「正常海面 prior」轉化成 backbone 表徵
- 再 fine-tune 到 358 labeled

### 4.3 關鍵技術細節

**Input conv 權重複製 pseudo-code**

```python
import torch.nn as nn

old = backbone.conv1                                  # (64, 3, 7, 7)
new = nn.Conv2d(8, 64, kernel_size=7, stride=2, padding=3, bias=False)
with torch.no_grad():
    # 假設 8 band 順序為 [B01, B02, B03, B04, B05, B08, B11, B12]
    # B02(blue)=ch1, B03(green)=ch2, B04(red)=ch3
    new.weight[:, 1] = old.weight[:, 2]               # blue ← B
    new.weight[:, 2] = old.weight[:, 1]               # green ← G
    new.weight[:, 3] = old.weight[:, 0]               # red ← R
    # 其餘 5 個 band 用 ImageNet 3 通道平均
    mean_w = old.weight.mean(dim=1, keepdim=True)     # (64, 1, 7, 7)
    for ch in [0, 4, 5, 6, 7]:
        new.weight[:, ch:ch+1] = mean_w
backbone.conv1 = new
```

**Differential learning rate（強烈建議）**

- Backbone lr = 主 lr × 0.1（已預訓好，只需微調）
- Decoder lr = 主 lr（隨機初始化要從頭學）

**兩階段 fine-tune（穩定性更好）**

- Stage 1（warm-up 5~10 epoch）：凍結 backbone，只訓 decoder + 改造後的 conv1
- Stage 2：解凍全部，backbone lr 用 0.1 倍

**Normalization 注意**

- 使用者已在 YAML 算好 8 band mean/std
- **不要混用 ImageNet 的 mean/std**，因為輸入分布跟自然影像差很多
- 保留自己算的 stats，讓模型微調吸收差異

**Decoder 初始化**

- DeepLabV3+ 的 ASPP + decoder 維持原本隨機初始化
- ImageNet weights 只在 backbone 層上有意義

### 4.4 預期效果

- val loss 飄升現象應該明顯緩解（train/val 差距從一倍以上縮到 30~50% 內）
- oil IoU 預期能上升 30~100%（from-scratch 改 transfer 在小資料集上常見的提升幅度）
- 訓練 epoch 數可從 300 降到 100 以內就收斂

### 4.5 風險與失敗模式

| 症狀 | 可能原因 | 對策 |
|---|---|---|
| 改完 conv1 後不收斂 | lr 太高 | 把 backbone lr 砍 0.1 倍 |
| oil IoU 還是 0 | hard-positive oversampling（A4）沒做 | 預訓只能解 backbone 問題，採樣問題要另外處理 |
| train loss 反而上不去 | input channel 順序對齊錯誤 | 確認 RGB band 對應位置寫對 |

### 4.6 參考資源關鍵字

- `torchvision DeepLabV3 modify input channels`
- `SatlasPretrain Sentinel-2`
- `Prithvi-100M huggingface`
- `SSL4EO-S12`
- `discriminative learning rate fine-tune segmentation`

---

## 5. 相關頁面

- [[DeepLabV3+]] — 工程紀錄：現行 ResNet-50 架構、8 通道設定、class_weights
- [[deeprx_vae_架構與運作機制]] — v3/v9 stage1 VAE 路線（本頁改善方向要超越的對象）
