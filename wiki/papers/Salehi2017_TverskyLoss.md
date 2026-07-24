---
type: paper
title: "Tversky loss function for image segmentation using 3D fully convolutional deep networks"
authors: Seyed Sadegh Mohseni Salehi, Deniz Erdogmus, Ali Gholipour
year: 2017
tags: [loss-function, class-imbalance, Tversky-index, Dice-loss, segmentation, medical-imaging, 3D-U-Net]
---

# Tversky Loss Function for Image Segmentation（Salehi et al., 2017）

> 來源：`raw/papers/Tverskyloss.pdf`（arXiv:1706.05721v1）
> 發表：International Workshop on Machine Learning in Medical Imaging (MLMI) 2017

---

## 核心貢獻

提出以 **Tversky index**（Tversky, 1977，原為心理學相似度指標）取代 Dice 係數作為分割任務的損失函數。Dice loss（Milletari et al., V-Net, 2016）對 FP（偽陽性）與 FN（偽陰性）加權相同；Tversky loss 引入兩個超參數 α、β 分別控制 FP 與 FN 的懲罰權重，讓損失函數本身就能直接調節 precision/recall 的取捨，**不需要額外做 class re-weighting 或 balanced sampling**。在 3D U-Net 架構、multiple sclerosis（MS）病灶分割任務上，α=0.3/β=0.7（加重 FN 懲罰、偏向召回率）全面優於對稱的 Dice loss（α=β=0.5）。

---

## 關鍵方法

**Tversky index 定義**：

```
S(P, G; α, β) = |P∩G| / (|P∩G| + α|P\G| + β|G\P|)
```

- P\G ≈ FP（預測有但 GT 沒有），G\P ≈ FN（GT 有但預測沒有）
- α=β=0.5 時，Tversky index 退化為 Dice 係數（= F1 score）
- α=β=1 時，退化為 Tanimoto coefficient
- α+β=1 時，產生一般化的 F-score 家族
- **α 越大 → 越加重 FN 的懲罰 → 訓練後模型偏向高召回率（recall），犧牲 precision**

**損失函數**（voxel-wise softmax 機率形式）：

```
T(α, β) = Σp0·g0 / [ Σp0·g0 + α·Σp0·g1 + β·Σp1·g0 ]
```

其中 p0/p1 是 softmax 輸出的病灶/非病灶機率，g0/g1 是對應的 one-hot GT。論文推導了對 p0、p1 的解析梯度，可直接嵌入標準反向傳播。

**架構**：3D U-Net（基於作者自己先前的 Auto-Net），contracting path + expanding path，3×3×3 卷積 + ReLU，2×2×2 max pooling，skip concatenation。損失函數是唯一變動項，架構固定，用來做乾淨的消融對照。

**實驗設計**：15 位受試者的 T1/T2/FLAIR MRI，2-fold cross-validation，掃了 5 組 (α, β) 組合：(0.5,0.5) 對稱 Dice、(0.4,0.6)、(0.3,0.7)、(0.2,0.8)、(0.1,0.9)，全部滿足 α+β=1。

---

## 重要數據/結果

Table 1（test set 表現，數字為百分比）：

| α, β | DSC | Sensitivity(Recall) | Specificity | F2 | APR |
|---|---|---|---|---|---|
| 0.5, 0.5（= Dice loss） | 53.42 | 49.85 | 99.93 | 51.77 | 52.57 |
| 0.4, 0.6 | 54.57 | 55.85 | 99.91 | 55.47 | 54.34 |
| **0.3, 0.7（最佳）** | **56.42** | 56.85 | 99.93 | **57.32** | **56.04** |
| 0.2, 0.8 | 48.57 | 61.00 | 99.89 | 54.53 | 53.31 |
| 0.1, 0.9 | 46.42 | 65.57 | 99.87 | 56.11 | 51.65 |

- α 越小（β 越大、越加重 FN）→ sensitivity 單調上升（49.85→65.57），但 DSC/F2/APR 在 α=0.3 之後開始下降，代表**過度加重 FN 懲罰會讓 precision 崩壞到拖累整體指標**，並非「β 越大越好」。
- 綜合表現最佳點在 α=0.3/β=0.7，而非文獻中常直覺選用的極端值。
- Dice loss（α=β=0.5，等同 F1）在此高度不平衡任務上是**五組中表現最差**的設定。

---

## 與本專案的關聯

**這不是一個未嘗試的假設性方向——本專案 GT_expand 分支（見 [[GT_expand_pipeline.md]] 第三-c 節）已經在實測這篇論文的確切設定，且結果已經勝出**：

- `tversky` 實驗線用的正是這篇論文推薦的 **α=0.3 / β=0.7**（本論文 Table 1 綜合表現最佳點），三折跑完後 pooled oil IoU = **0.3915**，是三個實驗線（baseline 0.332 / cw31 0.394 / tversky 0.3915）中**唯一同時通過雙 gate 判準**（per-scene Wilcoxon p=9.5e-14）的設定，已被判定優於 `cw31`（class_weights=[3.0,1.0]，均值雖達 0.394 但 Wilcoxon 不顯著、且 tiny 油汙場景顯著變差）
- 目前（截至 handoff STATUS_AS_OF 2026-07-12）此 Tversky(0.3, 0.7) 設定已被**帶入下一輪架構消融**（`tversky_v3plus`，判準為須顯著優於 0.3915 才留用新架構），成為目前研究線的標竿 loss 設定
- 這代表本論文提出的超參數在完全不同的領域（MS 病灶 MRI vs. 海面油汙光學影像）、完全不同的不平衡比例下，都收斂到相近的最適 α/β，是一個獨立於本專案之外的文獻佐證，強化了「α=0.3/β=0.7 附近是這類極端不平衡分割任務的合理起點」這個結論的可信度，而非純屬本專案調參巧合

**與 [[DeepLabV3+.md]]/[[OSDMamba.md]] 現行設計的差異**：這兩個模型頁面記錄的仍是較早的 `class_weights=[13.0,1.0]` + FocalLoss / Dice+FocalLoss 設計，尚未反映 GT_expand 分支已經驗證過的 Tversky 結果——這是 wiki 內兩條記錄（models/ 舊設定 vs. pipeline/GT_expand 新實驗線）出現落差的地方，之後模型頁面若要更新為現行最佳設定，應以 [[GT_expand_pipeline.md]] 與 handoff 正本為準。

---

## 研究缺口與假設

**論文自承的局限**：僅在單一小型資料集（15 位受試者、2-fold CV）、單一架構（3D U-Net）上驗證，未與同期其他方法做直接跨資料集比較；作者明確說這是 proof-of-concept，未來需要在更大標準資料集上驗證。

**未明說的隱含假設（假設殺手）**：
1. **α/β 是全域固定值，套用在整張影像上**——但論文自己的 Fig.3（高密度病灶樣本）與 Fig.4（極低密度病灶樣本）顯示，不同影像的病灶密度差異很大，理論上最適 α 應該因影像而異，而非用同一組全域超參數。這正是本專案油汙偵測的真實情境：不同場景（大片浮油 vs. 零星油膜）的不平衡程度差異可能比 MS 病灶更懸殊，全域固定 α 是否最適，需要實測驗證。
2. 假設「FN 比 FP 更該被懲罰」在所有應用都成立——這是醫學診斷（漏診代價 > 誤診代價）的領域假設，需要確認是否適用於油汙偵測（漏抓一塊油汙 vs. 誤報一塊乾淨海面，代價結構是否類似，本專案目前用 class_weights=13 隱含地做了類似假設，但沒有像 Tversky 這樣系統性掃描過 trade-off 曲線）。
3. 論文用「α+β=1」的限制條件掃參數（產生 F-score 家族），未探索 α+β≠1 的一般化 Tversky index，也未探索「每像素動態調整懲罰權重」（此限制後續被 2018 年 Focal Tversky Loss 論文針對性解決，加入 γ 項聚焦難分類像素——與本專案已用的 FocalLoss 思路互補，值得對照）。

**與本研究的關聯**：GT_expand 分支的 tversky 實驗線已經用實測結果部分回答了假設 1——三個實驗線橫向比較顯示 cw31（全域固定 class_weight）在 tiny 油汙場景顯著變差，而 tversky 通過雙 gate，暗示「不對稱化損失函數」比「全域 class_weight」更能兼顧不同尺度的油汙場景，但 tversky 是否在**所有**場景尺度下都穩定，或是否仍受惠於更精細的 per-image 動態權重，尚待後續 FN 分解分析（見 handoff 下一步：threshold 校準+FN 分解）確認。

---

## 相關頁面

- [[GT_expand_pipeline.md]] — **本論文 α=0.3/β=0.7 設定已在此實測並勝出**（tversky 實驗線 pooled oil IoU=0.3915，唯一通過雙 gate），目前研究線標竿 loss 設定
- [[DeepLabV3+.md]] — 本專案 DeepLabV3+ 工程紀錄，記錄的是較早的 class_weights=[13.0,1.0]+FocalLoss 設計，尚未反映 GT_expand 分支的 Tversky 結果
- [[OSDMamba.md]] — 本專案 OSDMamba 工程紀錄，現行 Dice+FocalLoss 聯合損失，Dice 項可對照 Tversky(α,β) 的不對稱化設計
- [[deeplabv3plus_358clean_overfitting_改善方向.md]] — 現行改善方向清單中列有 Tversky 選項（第3項），已被 GT_expand 分支實測驗證
- [[hard_negative_mining.md]] — HNM 是資料層級處理不平衡的策略，與 Tversky loss（損失函數層級）互補而非競爭
- [[unsupervised_oil_detection.md]] — 本專案油汙偵測整體方法總覽
