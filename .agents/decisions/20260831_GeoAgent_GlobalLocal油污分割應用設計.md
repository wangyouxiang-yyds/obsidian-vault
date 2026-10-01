# 2026-08-31 GeoAgent Global–Local 油污分割應用設計

議題：Liu et al.（ICCV 2023）GeoAgent 的 scale-adaptive global–local segmentation，是否及如何應用到 GT_expand 的 Prithvi-EO-2.0 + DeepLabV3 油污分割。

主持方：Codex。對手方：Claude Code（Opus 5，effort high）。

回合數：3。收斂：雙方 `STANCE: ACCEPT`、`BLOCKING: []`。

**執行狀態：僅完成概念設計；未改 code、未建立 prereg、未跑 CPU/GPU、未啟動訓練或推論。任何後續執行仍須使用者明確授權。**

## 問題與現行證據

- 現行模型：`256×256 @ 10m`、Prithvi-EO-2.0-300M final tap `16×16×1024`、DeepLabV3 ASPP `(6,12,18)`。
- 決定性基線：mean pooled Oil IoU `0.4399`；Evaluation Contract grouped `S=0.4606`。
- 主要失敗型態：large diffuse oil 的 high-confidence FN／under-segmentation。
- P3 已否定縮小 ASPP rates `(2,4,6)`；loss、threshold 等既有槓桿亦已關閉。
- 本案要檢驗的是「patch 外新像素是否有用」，不是再調 patch 內 receptive field，也不是把 segmentation 改成 RL。

## 逐輪候選稿演變

| 回合 | 候選稿與主要分歧 | Claude verdict |
|---|---|---|
| 1（盲提案） | Claude 獨立提出 shared local/context、fixed `k=3`、中央 ROI 融合；拒絕照搬 whole-scene thumbnail、context-only 與 A2C。初始阻礙為舊 track、證據取用、positive-area 欠採樣競爭解釋、記憶體／comparator。 | `REVISE` |
| 2 | Codex 指出 `concat(2048)→3×3→1024` 約新增 `18.9M` 參數，形成容量混淆；禁止用 outer-test OOF GT 選設計；30m 降格為 physical-scale prior；補同步 augmentation、padding、sensor 與 comparator 契約。Claude 接受並把 fusion 收斂為 context-only 1×1，但留下 B1–B3。 | `REVISE` |
| 3 | Codex 再縮為 projection-free 1024-channel gated add；Stage 0 拆 train/val-only 幾何 audit、工程 gate 與 validation-only 推論；development fold 由不碰 outer-test 的 GT 幾何規則選。Claude 撤回 projection 主張並接受。 | **`ACCEPT / BLOCKING=[]`** |

首輪實際返工：fusion `18.9M 3×3` → `1.05M context-only 1×1` → **1024-parameter projection-free gate**；Stage 0 由 outer-test OOF 分析改為 train/validation-only；`roi_align` 改列為 deterministic `FixedROIResample` 的對照驗證，而非預設實作。

## 最終方法判斷

GeoAgent 對本專案最可能有價值的是 **shared global–local information path**：local branch 保留 10m 邊界，context branch 真正讀取 patch 外像素。整景 thumbnail、RL reward、A2C 與 2–6× dynamic action set 不直接適用：GT_expand 是 GT-centric patch/reconstruction，不具原論文的完整滑窗 episode；只有約 12 個事件群，scene thumbnail、經緯度、日期或 tile 也很容易成為事件指紋。

因此：

1. **fixed scale 先行**；先回答 context 本身是否有用。
2. 只有 fixed scale 有效且 train-only nested oracle 顯示額外、可預測的尺度異質 headroom，才考慮 supervised router。
3. contextual bandit／A2C 預設 **out of scope**。只有真正回到 sliding-window episode、supervised router 仍取不回 oracle gain、whole-scene reward 又有獨立增益時，才重新立案。

## 最小可證偽首臂

```text
Local:   256×256 @10m ─→ shared Prithvi final tap ─→ F_local [1024,16,16] ─┐
                                                                                ├→ F_local + γ⊙C_aligned
Context: 768×768 @10m ─area→ 256×256 ─→ shared Prithvi final tap                │
                                      └→ FixedROIResample(center 1/3) ─→ C_aligned

F_fused ─→ 原封不動 DeepLabHead / ASPP(6,12,18) ─→ 256×256 oil logits
```

- `γ`：1024-channel、fp32、zero-init、no weight decay、歸 head LR group。
- 不設 projection、bias、global pooled token、aux head 或新 loss。
- 新增可學參數只有 1024；`γ=0` 時嚴格退化為現行 baseline，沒有新的 random initialization。
- local/context 同中心、同六個 Prithvi bands、同一組同步 spatial augmentation。
- context 以 area downsample；越界維持中心不 shift，以各 band 的 train mean padding，使 normalized padding 為 0，另記 validity fraction。
- 3×/30m 只稱 **physical-scale prior**，不稱 HLS distribution alignment。S2 10m→30m 與 HLS PSF／processing 不等價；L8/L9 已經 Sen2Like fusion/upscale 後再平均，更不能當原生 HLS。
- sensor 分層只作描述性，並申報 sensor 與 event/collection 的共線風險。

### Null 的範圍

Projection-free gated add 偏向最小、可歸因的訊息路徑測試，但表達力有限。因此 scientific null 只能否證：

> additive tap-23 / fixed k=3 / center-ROI-only context injection

不得外推成「所有 global–local、其他 tap、其他 k、含 projection 或調變式融合皆無效」。

`non-executed` 與 scientific null 必須分開：以 `||γ⊙C_aligned|| / ||F_local||` 與非平凡 γ 通道比例判斷，門檻在 prereg 凍結。只有 `non-executed` 才可在未來 prereg 事前允許一次 context-only `1×1` projection 升級；若 γ 已有非平凡貢獻但結果為 scientific null，則結案，不換 k 或融合形式重試。

## 驗證階梯

### Stage 0a：train+validation-only 幾何／物理 audit

- HF-1：large-interior patch 的 k=3 annulus 是否實際新增有效非油水體；門檻須在執行前凍結。
- SE-2：×3 area downsample 後，同場景 oil-vs-water 相對訊號是否存活。
- 不讀 outer-test GT，不選 k、threshold 或 router。
- 只有 HF-1「幾乎沒有新增有效 context」可作科學停止條件；SE-2 只是 soft evidence。

### Stage 0b-i：工程 gate

- dataset 同中心讀窗、padding、validity、同步 augmentation。
- `FixedROIResample` 的幾何往返、shape、參數量、baseline-degenerate initialization。
- 雙跑位元決定性；與 bilinear ROIAlign 的數值等價只作驗證，不預設使用 CUDA `roi_align`。

### Stage 0b-ii：validation-only baseline 推論

- 用 frozen baseline checkpoint 對各 fold validation scenes 產生 SE-1。
- 同場景內依離 oil/water 邊界距離拆 recall／confidence；只作 soft evidence。
- 現存 outer-test OOF probabilities 不得用於設計本實驗。

### Stage 1：單一 development fold feasibility

fold 選取規則只用各 fold validation GT 幾何與 valid-water mask：計算符合 `large-bin ∧ local oil fraction≥0.8 ∧ k=3 annulus 新增有效非油水體≥凍結門檻` 的 patch 數，取最多者；tie 取 fold id 最小。這是刻意富集機制訊號的工程／機制 pilot，不作效能 claim。

檢查 OOM、訓練穩定性、γ 是否實際執行，以及 validation large FN 是否有方向性變化。若 Stage 1 後改任何 config，Stage 2 三 fold 全重跑。

### Stage 2：frozen three-fold evaluation

因同 12 個事件群已被歷次研究反覆使用，本階段只稱 **protocol-conditional development evidence**，不稱 independent confirmatory 或 OOD。

必報：

- 計畫任務 pooled Oil IoU。
- Evaluation Contract 的 `S=(M1+M2)/2`、M1、M2 與 12-event paired cluster bootstrap。
- small／mid／large 的 Oil IoU、FP、FN、TP。
- pooled 與 S 一律並報，方向衝突不得擇優。

實質門檻、HF-1 門檻、`non-executed` 門檻、稀釋算術與完整結局集須另行 prereg，不從 P3/P4 靜默繼承。

### Stage 3：router gate

只有 fixed k=3 已有效，且 train-only nested/cross-fitted oracle 顯示相對 fixed scale 還有足夠異質 headroom，才訓練 supervised router。首版 state 僅限 local image-derived features；不含 event ID、經緯度、日期、tile 或 scene thumbnail。若 supervised router 已取回大部分 oracle gain，即停止，不進 RL。

## 機制診斷

- validation 為主做同 scene 的 location-shuffled context：保留場景統計、破壞幾何對應。
- 做 context annulus perturbation，檢查 patch 外像素是否真正影響中央 token。
- 兩者皆為次要機制診斷，不作主端點；outer-test 若做，只能是預註冊的描述性結果，不參與任何選取。

## 不能宣稱

- 不能外推原論文 `+5.17～+12.80` mIoU 到本專案。
- 不能宣稱 3× context 真正對齊 Prithvi/HLS 的資料分布。
- 不能宣稱 OOD、獨立 confirmatory 或可外推 full-scene sliding deployment。
- 不能因使用雙 branch 就稱 reinforcement learning。
- 不能以一個 projection-free null 否定 global–local 方法家族。

## 競爭解釋與殘餘意見

Positive-area 欠採樣與 patch 外 context 不足對 large FN 給出相近預測。兩者須分開立案，不得綁在同一 arm；哪一個先花 GPU 預算，本輪不替使用者決定。

Claude 對「本設計是目前最小且可歸因的可證偽形式」信心高，但對實驗會陽性信心低；最可能的 assumption killers 是：

1. k=3 annulus 其實沒有新增非油水體。
2. 油膜訊號撐不過降採樣。
3. 二元油污任務的最佳尺度同質，沒有 router／RL 的存在價值。

## Launch gates

1. **LG-1**：Stage 0b-i 驗證 `FixedROIResample` 雙跑位元決定性與 bilinear ROIAlign 數值等價；解除後才可進 Stage 1。
2. **LG-2**：雙 Prithvi forward 的記憶體、effective batch 與 comparator 有效性。優先 gradient checkpointing；若改 microbatch／BN 行為，舊 `_dlv3_det` comparator 失效，需另立 paired control。
3. **LG-3**：manifest 的 per-scene sensor provenance；缺少時不得承諾 sensor 分層。
4. **LG-4**：舊 track budget 與使用者明確 launch 授權。
5. **LG-5**：HF-1 與 fold 選取共用門檻須先凍結。
6. **LG-6**：Stage 0b-i 工程 gate 與 Stage 0b-ii validation-only 推論各自取得授權。

## 協商 provenance

- Claude CLI：`2.1.251`
- requested/actual primary model：`claude-opus-5`；effort `high`；無 fallback
- same session：`7757d8cf-0c33-4bbd-9db1-c5df92ac3573`
- 回合成本：R1 `$1.4568225`、R2 `$0.8404490`、R3 `$0.7316210`；合計 `$3.0288925`
- R1 的工具內部顯示 Haiku 4.5 helper 共 21 output tokens，但沒有 spawn subagent；正式判斷均由 Opus 5 完成。
- 首輪證據限制：Claude 無權直接讀 repo code／PDF extraction 工具；Codex 主持方直接核對 `prithvi_deeplab.py` 與論文逐頁擷取筆記後，在 R2 明示補齊，後續回合無 permission denial。
- 本案僅概念設計，沒有不可逆執行或論文定論，因此未觸發不可逆 devil's-advocate 終審。

## 試行評估附錄

- 首輪主要分歧：過大的 fusion 容量、outer-test 設計層重用、30m 過度詮釋、ROI／augmentation／comparator 契約。
- 收斂輪數：3。
- 實際返工：有；核心架構與資料使用規則均被實質修正，見逐輪表。
