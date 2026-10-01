# 2026-09-11 P13 patch-balanced 台灣 source sensitivity 實驗設計

議題：在既有 pixel-balanced 支線之外，建立一條平行的 patch-balanced 支線，於固定每 epoch patch budget 下描述 G1、G2、G3 與等額 Mixed 對固定台灣 holdout 的表現。

專案：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand`

協商狀態：Claude × Codex 第 4 回合收旂，雙方 `ACCEPT / BLOCKING=[]`。

**執行狀態：設計與輸入已建立，但 8 arms 全部 `run: false`；未啟動訓練、推論或 GPU job。任何啟動仍需使用者新的明確授權。**

> 本次交接未附 requested/actual model、CLI version 或 session id；本紀錄不臆測補齊這些 provenance 欄位。

## 1. 實驗位置與 estimand

- patch-balanced 與既有 pixel-balanced **平行保留**，前者固定 patch 數，後者固定 pixel-level 曝光；兩者 estimand 不同。
- 本支線只能描述「在這一個 seed、凍結訓練協定與固定台灣 holdout 下」的 source-group sensitivity。
- 不得把 patch-balanced 與 pixel-balanced 的分數差，直接歸因為單一 balancing factor 的效果。

## 2. 八個 arms 與凍結配額

8 arms 為 `A/B × G1/G2/G3/Mixed`，每個 arm 僅跑 seed 42 的 R1。

| Level | Oil slots/epoch | Pure-BG slots/epoch | Total patches/epoch | Mixed oil（G1/G2/G3） | Mixed BG（G1/G2/G3） |
|---|---:|---:|---:|---:|---:|
| A | 164 | 180 | 344 | 55 / 55 / 54 | 60 / 60 / 60 |
| B | 1,658 | 1,820 | 3,478 | 553 / 553 / 552 | 607 / 607 / 606 |

- A/B 都以 patch budget 為控制軸，不控制油污大小、場景數或 pixel-level oil:background ratio。
- 不足配額以 deterministic repetition 補齊；重複 patch 不是新的獨立資料。

## 3. 凍結訓練協定

- seed 42，單次 R1。
- Prithvi-EO-V2-300M + DeepLabV3+；input 256×256。
- Tversky loss `alpha=0.3` / `beta=0.7`。
- batch 16，最多 300 epochs，patience 50，保存 best checkpoint。
- checkpoint 依 source-matched train-proxy mIoU 選取。
- 固定台灣 holdout：137 patches／9 scenes／2 events。

## 4. Checkpoint selection 限制

可用 source 資料無法在保留 G2 的 A 層下限（164 oil + 180 BG）時再切出獨立驗證集，因此本支線使用 **source-matched、non-independent train proxy** 進行 early stopping 與 checkpoint selection。

- train-proxy 分數不是泛化結果；唯一比較終點是凍結的台灣 holdout。
- A/B 的 steps per epoch 不同，epoch-based early stopping 不等於固定 optimizer exposure。
- B 層 oil unique pool 大小因 group 而異，造成 train-proxy oil coverage/multiplicity 不同；不得聲稱 B 層的 validation exposure 或 checkpoint-selection leakage 已完全固定。
- runner 已增加 `epoch_to_best`、`updates_to_best`、`stop_epoch` 紀錄，B 層台灣結果必須與停止時點同表呈現。

## 5. 可以與不可以聲稱的結論

### 可以

- 描述固定 patch budget 下，G1/G2/G3/Mixed 在 A/B exposure regime 的台灣 holdout 表現與排序是否改變。
- 將 patch-balanced 與 pixel-balanced 當作不同 estimand 的平行敏感度分析。

### 不可以

- 不得宣稱對 training stochasticity 穩健：每個 arm 只有 seed 42。
- 不得把 train proxy 稱為獨立 validation，也不得用其分數主張泛化。
- 不得忽略 B 層的 oil validation exposure 差異或把重複 patch 當作新資料。
- 不得作海域本身的因果效果、統計顯著差異或跨 seed 穩健性聲稱。

## 6. 已建立的 repo 檔案（本次僅登錄，未修改）

- `analysis/p13_taiwan_source_sensitivity/build_patch_balanced_design.py`
- `analysis/p13_taiwan_source_sensitivity/PATCH_BALANCED_DESIGN.md`
- `analysis/p13_taiwan_source_sensitivity/generated_patch_balanced/`
- `main/experiments_P13_taiwan_source_sensitivity_patch_balanced_A.yaml`
- `main/experiments_P13_taiwan_source_sensitivity_patch_balanced_B.yaml`
- runner 紀錄擴充：`epoch_to_best`、`updates_to_best`、`stop_epoch`。

## 7. 啟動閘門

兩份 patch-balanced YAML 內的 8 個 experiment entries 全部保持 `run: false`。本案只代表「建碼與設計已完成」，**不構成 launch 授權**；使用者未再次明確確認前，不得啟動任何 arm。
