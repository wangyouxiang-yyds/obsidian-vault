# 2026-09-02 GeoAgent／A2C 尺度配置實驗設計與建碼

議題：在 GT_expand 的 Prithvi-EO-2.0 + DeepLabV3 油污分割中，如何把 Liu et al.（ICCV 2023）GeoAgent 的 global–local 尺度選擇改造成可驗證的 fixed／supervised／RL 實驗，並先完成不干擾 P10 的程式骨架。

主持方：Codex。對手方：Claude Code（正式方法協商使用 Opus 5，effort high）。

**狀態：方法設計六輪收斂 `ACCEPT / BLOCKING=[]`；程式初審 `REVISE` 後修正，Claude 聚焦複審最終 `VERDICT: ACCEPT / BLOCKING: NONE`。程式已寫入隔離工作樹，但預註冊仍是 `DRAFT_NOT_FROZEN`，不得啟動真實資料或 GPU 階段。**

## 1. 執行邊界

- 唯一開發位置：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_geoagent_dev`。
- 以 detached sibling worktree 隔離；原 repo 與正在執行的 P10 程式、config、log、result 均未修改。
- 本次未讀真實 raster／mask、未執行真實 CPU data/geometry audit、未載入 Prithvi checkpoint、未使用 GPU、未訓練或推論。
- 未 commit、未 push。
- draft config 同時以 `status: DRAFT_NOT_AUTHORIZED`、`run: false`、未填 launch token 與 gate file fail closed；runner 對 CPU audit、真資料階段及 CUDA 階段另設獨立授權檢查。

## 2. 為何不是只做一個 A2C arm

單純讓 agent 對每張 patch 選 local／3×／5×，若三個 action 的輸出都可事後計算、又沒有成本限制，實際上是 full-information routing，不構成需要序列決策的 MDP。六輪協商因此把問題拆成兩條：

1. **Track A：尺度訊息是否有用、尺度異質性是否可預測。** 不宣稱 RL。
2. **Track B：在每景 context 預算有限時，如何把昂貴尺度分配給最值得的 patch。** hard budget 會改變後續可用 action，才形成真正的 sequential decision problem。

這個拆法讓 A2C 必須同時打贏成本相同的 fixed、random 與 supervised allocator，避免把「有更多 context」誤寫成「RL 有效」。

## 3. 資料生命週期

- development corpus 固定為 **NOAA319**。
- 36 個具名事件保持 sealed：不得加入訓練、early stopping、gate、超參數選擇或方法修改。
- NOAA319 內再作 scene-disjoint 的 `inner_train / val_es / val_gate`，以 region × frozen oil-size 分層；不得冒稱 event-disjoint。
- Stage-1 的 GT 與 BG 都只保留 `NOAA319 ∩ inner_train`；BG 政策固定為每景最多 10 張。實際 eligible GT、BG 與 realized ratio 必須等獨立授權的 CPU audit 後寫入，**不繼承 P10 的 ρ 候選值**。
- raw `255` 是未標註，tested mask 固定為 GT-aware footprint 且 `raw != 255`；context GT 永遠不讀。
- `final_corpus_mode=PENDING_USER_AUTHORIZATION`：未來是否打開 sealed 36，必須由使用者另行裁決。

## 4. 輸入、架構與分割輸出

### 輸入

| 視野 | 原始讀窗 | 物理範圍 | 送入 encoder 前 |
|---|---:|---:|---|
| local | 256 × 256 | 2.56 × 2.56 km | 保持 256 × 256 |
| context3 | 768 × 768 | 7.68 × 7.68 km | valid-area mean 至 256 × 256 |
| context5 | 1280 × 1280 | 12.8 × 12.8 km | valid-area mean 至 256 × 256 |

- 讀取同一份 8-band VRT，Prithvi 實際使用 B02／B03／B04／B8A／B11／B12 六 band。
- 越界以 training per-band mean 填補；下採樣只平均有效像素。
- Albumentations `ReplayCompose` 同步 local／context／GT 的 crop、flip、rotation 與 photometric draw；GT 只屬於 local。
- `RandomResizedCrop` 的 crop coordinates 會換算為 context feature 中對應 local crop 的 ROI。

### shared global–local segmenter

```text
local 256 ───────┐
context3 768→256 ├─ shared Prithvi tap23 ─→ 1024×16×16 ─→ 尺度對齊／gated add
context5 1280→256┘                                      ─→ 原 DeepLabV3 ASPP(6,12,18)
                                                         ─→ 256×256 Oil probability
```

- context ROI 使用 deterministic separable bilinear matrices：`Ay × F × Axᵀ`；禁止 `roi_align` 與 `grid_sample`。
- action feature：`local = F_local`；`3× = F_local + gamma3 × C3_aligned`；`5× = F_local + gamma5 × C5_aligned`。
- `gamma3/gamma5` 各 1024-channel、fp32、zero-init、no weight decay、head LR；起始時三個 action 與 local 嚴格相同。
- 不加 projection、global token、auxiliary loss；ASPP rates 保留 `(6,12,18)`。
- Stage-1 seed 42；每 sample 均勻抽一個 action，只對該 action 計 segmentation loss。val_es early stopping 後凍結 encoder、gates 與 head，再快取每個 patch 三種 action 的 Oil probability 與 policy features。

## 5. Track A：非 RL 尺度比較

- fixed local。
- fixed 3×。
- fixed 5×。
- supervised atom regressor：輸入 local GAP 1024、六 band mean/std 12、context quality 3，共 1039 維；預測 ownership-region 的 `delta TP / delta FP / delta FN`，再用 Dinkelbach + knapsack 做配置。
- full-information Track A 用來回答「context 是否有用」與「scale heterogeneity 是否可預測」，不得稱為 reinforcement learning。

## 6. Track B：hard-budget A2C

### Episode、state 與 action

- 一個 scene 是一個完整 episode；patch exact deduplicate 後依 `(y, x, patch_stem)` 排序。
- cache、reward、reconstruction 共用同一個 `EpisodeOrder` SHA256 與 tested-mask SHA256，不一致即停止。
- 每景 context budget：`B = ceil(0.30 × n_unique)`；local cost 0，3×／5× cost 1；budget exhausted 後 hard-mask context action。
- policy state 共 1073 維：local GAP 1024、band stats 12、context quality 3、remaining budget/steps 2、history embedding 32。
- history 的三張 16×16 map：all-local Oil probability、running selected Oil probability、coverage；用 `3→16→32` 兩層 3×3 conv + GELU + GAP。
- shared 256 trunk；actor 輸出 local／3×／5× logits，critic 輸出 `V(s)`；不輸入明示 latitude、longitude、date、event 或 tile。

### Exact potential reward

先用全 local 建立最後 stitched canvas `L*`，episode 起始 `C0=L*`，尚未處理處仍由 `L*` 補上：

```text
Phi(Ct) = Oil-IoU(Ct, frozen tested mask) - Oil-IoU(L*, frozen tested mask)
r_t     = Phi(Ct) - Phi(Ct-1)
```

`discount=1`、無 terminal bonus、無 reward normalization。reward telescopes 到最後相對 all-local 的 IoU 改變；後續 patch 覆寫重疊區時，也會扣回已不存在的 gain。若 prediction 與 GT 的 union 為空，Oil-IoU 固定記為 0，避免 ±1 reward cliff。

### A2C 與成本相同的 comparators

- A2C：Adam `3e-4`、value coefficient `0.5`、entropy `0.01` 在前 50% updates 線性降到 0、gradient clip `0.5`。
- 每 update 八個完整 scene episodes，不做 truncated rollout；最多 200 epochs、patience 20；policy seeds 42–46；val_es 以 deterministic argmax budgeted `S` early stop。
- 同 budget 比較：even-spread fixed schedule、random seeds 42–46、supervised atom-regressor + DP、exact ownership-atom oracle。
- always 3×／5× 僅為 unbudgeted upper anchors，不能拿來做公平成本比較。

## 7. 評估、輸出與 claim 上限

- 重建採 canonical last-write-wins，且只在 frozen tested footprint 計分。
- M1：region × frozen size cell 內先逐景平均，再跨 `n>=3` cells 等權 macro；小 cell 排除 macro 但逐格列出。
- M2：逐景 Oil-IoU 等權平均；`S=(M1+M2)/2`。
- G1：exact budget oracle `S` − best budget-matched fixed `S >= 0.05`。
- G2：supervised DP `S` − five-seed random scene-first mean `S >= 0.02`。
- architecture-null：fixed context 與 fixed local 的 mean absolute Oil-probability difference 必須至少 `1e-3`。
- 必輸出：config/revision/data accounting、order/tested-mask hashes、per-scene actions/budget、per-step reward/potential/value、gamma trajectory、history variance、三尺度 probabilities、stitched probability、confusion atoms、M1/M2/S 與 bootstrap。
- P11 v1 的 claim 僅限 **GT-aware patch-footprint、NOAA319 development evidence**；不能宣稱完整 10980² 場景、獨立 confirmatory 或未見事件 OOD 成效。

## 8. 已建置程式

工作樹新增 `main/geoagent/` 套件，共 19 個模組：

- `types.py / config.py / artifacts.py`：action、patch/episode 型別、draft config 與 provenance。
- `data.py / geometry.py / audit.py`：三尺度 VRT 資料、同步 replay、固定矩陣對齊、NOAA/sealed 邊界、data/geometry audit。
- `model.py`：shared encoder、zero-init gated segmenter、1073-D actor–critic。
- `cache.py / reconstruct.py / reward.py / environment.py`：三 action cache、hash 守門、last-write-wins、potential reward 與 budget episode。
- `policy.py / oracle.py / train.py / evaluation.py`：atom regressor、Dinkelbach/DP、Stage-1/A2C、Contract/gate/bootstrap。
- `workflow.py / runner.py / selftest.py`：各 phase 串接、授權 fail-closed 與 CPU synthetic contracts。

另新增：

- `main/experiments_P11_geoagent_router_draft.yaml`
- `main/experiments_P11_geoagent_a2c_draft.yaml`
- `analysis/p11_geoagent_rl/experiment_spec.md`
- `analysis/p11_geoagent_rl/preregistration.md`
- `analysis/p11_geoagent_rl/sealed_named_event_collections.txt`
- 七個 `main/test_geoagent_*.py` pytest-style 測試模組。

既有 P10 核心檔、P10 configs 與原本訓練路徑沒有修改。

## 9. 驗收結果與限制

- `py_compile`：通過。
- 標準函式庫 CPU selftest：**16/16 PASS**，涵蓋 masked-area downsample、真 ReplayCompose seed/replay、fixed-matrix forward/backward、crop/flip ROI、fake encoder 決定性、zero-init、Stage-1 單 action decode、reward telescoping/overwrite reversal、Dinkelbach 對 brute force、1073-D A2C backward、atom feasibility、budget env、cache/reconstruction hashes、global coordinates、bootstrap schema、NOAA319/sealed36 executable boundary。
- pytest-style test modules 已寫，但環境未安裝 `pytest`，因此**沒有執行 pytest suite**；不得誤寫成 pytest passed。
- 以上測試全是 synthetic／fake encoder；不構成真資料、Prithvi、GPU、記憶體或訓練可行性證據。

程式審查歷程：Claude 初審為 `REVISE`；Codex 修正後，Claude 聚焦複審最終 `VERDICT: ACCEPT / BLOCKING: NONE`。此 verdict 表示目前程式骨架無剩餘阻礙，不等於 launch gates 已通過。

## 10. 後續 launch gates

1. 等 P10 完成；P11 不得搶占或干擾現行程序。
2. 使用者另行明確授權真實資料 CPU data/geometry audit；填入 NOAA319 split、GT/BG/rho、geometry gates 與 hashes。
3. geometry gates、context-quality liveness、architecture-null 與兩個 fresh process trainer parity 通過。
4. 使用者裁決 final corpus mode。
5. 使用者另行授權 prereg freeze 與 commit。
6. 最後仍須使用者明確授權 GPU launch；前述任何授權不可相互推定。

## 11. Claude × Codex 協商 provenance

- 正式方法設計：六回合同一 Claude session，R1–R5 均為 `REVISE`，R6 `ACCEPT / BLOCKING=[]`。
- Claude CLI：`2.1.252`。
- requested/actual model：`claude-opus-5`；effort `high`；無 fallback。
- session id：`a577051d-49a2-43a3-9b20-fb6809408653`。
- R1–R5 可核成本：`$1.448627 + $1.183799 + $1.061761 + $0.751823 + $0.489760 = $4.935770`；R6 成本未在本紀錄取得，故不猜測總額。
- 程式初審與聚焦複審的正式 verdict 如第 9 節；其 session/cost 未在本紀錄取得，不補造 provenance。

## 12. 與 2026-08-31 決議的關係

[[20260831_GeoAgent_GlobalLocal油污分割應用設計]] 的 fixed 3× 最小首臂仍是本案的設計起點，但使用者後續明確要求「可以多方嘗試 RL，不用那麼保守」。本決議因此以 hard budget 建立 genuine MDP，同時保留 Track A 作必要比較；凡與前案「RL 預設 out of scope」衝突處，以本案使用者新指示與六輪共識為準。前案對 shared Prithvi、zero-init gated add、原 ASPP、禁止身份特徵與 claim 限制等要求則繼續有效。
