---
type: concept
tags: [scale-adaptive, global-local, dynamic-routing, reinforcement-learning, semantic-segmentation, Prithvi]
related: [Liu2023_GeoAgent_尺度自適應分割, Szwarcman2026_PrithviEO2, 20260731_ASPP_rate適配實驗, GT_expand_pipeline]
---

# 尺度自適應 Global–Local 分割

## 一句話定義

同時以原解析度 local patch 保存邊界細節、以較大範圍的低解析度 context patch補足 patch 外空間資訊，並以固定、監督式或 RL router 為每個位置決定最適觀察尺度。

## 為什麼 ASPP 還不夠

ASPP、FPN、self-attention 可以擴張或聚合輸入 patch 內的 receptive field，卻不能看到 patch 邊界外沒有被讀進來的像素。Global–local 方法改變的是輸入資料範圍：local branch 看細節，context branch 真正讀取更大的地面 footprint。

這與 [[20260731_ASPP_rate適配實驗]] 的結論互補。該實驗顯示大型 diffuse 油膜受益於大 receptive field，但現行 ASPP 仍受限於 256×256 patch；global–local branch 是在此基礎上新增 patch 外資訊，不代表應縮小或移除 ASPP。

## GeoAgent 的原始機制

[[Liu2023_GeoAgent_尺度自適應分割]] 將一張完整影像視為 episode，每個 sliding-window patch 是 timestep：

- state：global thumbnail + 目前 patch 的 position mask。
- action：選擇 context 倍率；`1×` 為 local-only，較大倍率讀取較大區域並縮回固定尺寸。
- segmentation：local/context 共享同一網路權重；context feature 依地理位置裁出 local footprint，再與 local feature 融合。
- reward：所選尺度相對 local-only 的 `mIoU+mF1` 改善，整景完成後再給 whole-map improvement。
- optimization：先 random-scale 預訓練 segmentation network，再訓練 Scale Control Agent，最後交替 joint fine-tuning。

RL 的作用是免除「每個 patch 最佳尺度」人工標註，不是免除 segmentation GT。

## RL、supervised router 與 contextual bandit

GeoAgent 稱其為 MDP/A2C，但 action 不會改變下一個 state：滑窗仍按固定順序移到下一個 patch。序列耦合主要來自 episode 結尾的整景 reward，因此其局部決策結構接近 contextual bandit。

可依複雜度由低到高選擇：

1. **Fixed scale**：所有 patch 使用同一 context，最容易確認 context 是否有用。
2. **Oracle analysis**：在 train/validation 比較各尺度，估計理想 router 的上限與最佳尺度異質性。
3. **Supervised router**：用 oracle-best scale 或 scale gain 當 pseudo-label，訓練輕量分類器。
4. **Contextual bandit**：每個 patch 獨立選 action，以即時 reward 更新。
5. **Actor–critic RL**：只有 whole-scene reward 確實提供額外價值時，才值得承擔較高變異與訓練成本。

如果所有 patch 都偏好同一尺度，dynamic routing 沒有必要；如果 oracle headroom 很小，RL 也無法憑空創造增益。

## 在本專案的尺度對齊延伸

> 以下是由 GeoAgent 與 Prithvi 預訓練尺度導出的研究假說，不是 Liu et al. 原論文方法或結果。

現行輸入為 256×256、10m Sen2Like patch；[[Szwarcman2026_PrithviEO2]] 則以 30m HLS 預訓練。可建立：

| Scale | 讀取範圍 | resize 後 | 等效解析度 | 用途 |
|---|---:|---:|---:|---|
| 1× local | 256×256 | 256×256 | 10m | 保存細薄油膜與邊界 |
| 2× context | 512×512 | 256×256 | 20m | 中尺度過渡 |
| 3× context | 768×768 | 256×256 | 30m | 對齊 Prithvi HLS 預訓練尺度 |

```text
Local 256 @10m ───────→ shared Prithvi ─→ F_local ─────────────┐
                                                               ├→ align + fusion → ASPP → Oil mask
Context k×256 → resize256 → shared Prithvi ─→ F_context → ROI ─┘
```

3× branch 的 16-pixel Prithvi token 對應約 480m，與 30m HLS 預訓練時的物理 token footprint 對齊；local branch 則保留現行 10m 邊界。這是「尺度對齊」的具體含義，不只是一般 multi-scale augmentation。

## 對油污任務的 reward 與 state

- 原文 `mIoU+mF1` 容易受 background 主導；patch reward 可先用 `TverskyLoss(local)−TverskyLoss(action)`。
- terminal reward 可用所選尺度相對 local-only 的 scene Oil IoU 改善。
- 正式成敗仍依 Evaluation Contract：`S=(M1+M2)/2`、12-event cluster bootstrap，並拆 small/mid/large 的 FP/FN。
- 不建議把整張 10980×10980 場景直接縮成 policy thumbnail：縮至 256/512 後約為 429/214m per pixel，細薄油膜可能消失。優先使用 regional thumbnail、pooled local/context features 與場景內相對位置。
- 不餵真實經緯度或 event ID，避免約 12 個事件群下的事件記憶／捷徑學習。

## 採用閘門

1. Fixed 2×／3× 是否改善大型 diffuse 油膜 FN，且未造成 tiny/mid 或 FP 明顯惡化？
2. Oracle-best scale 是否隨 patch／size bin 改變，且 headroom 達到預先定義的實質門檻？
3. Supervised router 是否已能取得大部分 oracle gain？若是，就不必上 RL。
4. 所有比較固定 split、seed、loss、sampling、ASPP rates 與 evaluator；不得用 test GT 產生 action label、reward 或選尺度。
5. 另報 dual-branch 的 memory、latency 與失敗邊界；共享 encoder 權重不代表計算量不增加。

## 主要風險

- 大倍率降採樣會消除細長、低對比油膜，因此 context-only 不可作主架構。
- 只有約 12 個事件群，RL reward 方差與 policy overfit 風險高。
- local/context 同時 fine-tune 300M Prithvi 可能增加顯存與訓練不穩定，需考慮交替 forward、gradient accumulation 或先凍結 router／encoder。
- 原論文只在 1m/4m GF-2 多類別 LULC 驗證，不能直接宣稱可提高 Sentinel-2/Landsat Oil IoU。

## 相關頁面

- [[Liu2023_GeoAgent_尺度自適應分割]] — 方法、結果與完整批判性分析
- [[Szwarcman2026_PrithviEO2]] — 30m HLS 預訓練與水體表徵假設
- [[20260731_ASPP_rate適配實驗]] — patch 內大 receptive field 的既有證據
- [[GT_expand_pipeline]] — 現行 256×256 patch、Prithvi+DeepLabV3 管線
- [[20260716_資料溯源洩漏與評估合約v1]] — 正式評估單位與外推限制
