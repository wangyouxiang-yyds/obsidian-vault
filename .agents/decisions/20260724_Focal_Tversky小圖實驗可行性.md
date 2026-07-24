# 2026-07-24 Focal Tversky 小圖實驗可行性

## 議題

是否把 Focal Tversky Loss（FTL）列為 GT-centric 256 px 小圖 DeepLabV3 主軌的下一個受控實驗；以及如何在不把 augmentation RNG 舊污染、hard-mask 診斷與 soft-loss 機制混為一談的前提下，決定是否值得投入。

## 協商與模型 provenance

- 主持方：Codex；對手方：Claude。
- Claude CLI：2.1.218。
- requested model：`claude-opus-4-8`；effort：high。
- Claude session id：`2e986bba-b561-4ab2-90b2-d6c8684f1b8b`；兩輪沿用同一 session。
- actual model：兩輪 substantive response 均為 Claude Opus 4.8；第一輪 JSON 另出現一筆小型 Haiku tool-use accounting entry，不影響實質回答的 model provenance。
- fallback：無。
- 收斂輪數：2；Codex 主持方亦接受最終候選稿。

## 逐輪演變與 blocking

- **R1 Claude 盲提案 — conditional worth，但不是下一個實驗**：先完成 canonical deterministic plain Tversky `α_FP=0.3, β_FN=0.7`。Blocking：**[1]** 必須先有 plain Tversky 正典結果；**[2]** FTL 必須以 plain Tversky 為直接 reference，而非只對 Focal CE。
- R1 的解析：令 `u=1-TI`，FTL 為 `u^q`、`q=0.75`；相對 plain Tversky 的梯度倍率為 `q·u^(q-1)=0.75·u^-0.25`，交點 `TI≈0.684`。因此它抑制低 TI／高誤差批次，放大高 TI 後期殘差；目前 TI 是 batch aggregate，故這不是 pixel-level hard mining。
- **Codex 修正**：hard patch TTA precision/recall 不能精確反推 soft batch TI；任何由該診斷推算的「約 15–18% attenuation」均未驗證，正式候選稿只保留解析交點。若要主張機制，必須實際記錄 batch soft TI。
- **R2 最終稿 — ACCEPT**：把 FTL 降為「一張 gated coupon」，先由 canonical plain Tversky 對 deterministic Focal baseline 通過前置 gate，再做唯一變因 `q=0.75` 的 plain-Tversky 對照。Claude 最終 `ACCEPT / BLOCKING: [] / CONFIDENCE: HIGH`。

## 停止原因

第 2 回合雙方已接受同一候選稿，blocking 由 `[1,2]` 清為 `[]`，且機制主張、比較 reference、驗收線與停損線均已具體化；未觸發停滯、5 回合檢查點或 10 回合上限。

## 最終決議

### 1. 前置資格：FTL 不是現在立刻開跑

canonical deterministic plain Tversky `0.3/0.7` 必須先相對 deterministic Focal baseline 同時通過：

- `ΔM1 ≥ +0.008`
- `ΔM2 ≥ 0`
- large-oil recall drop `≤ 0.02`
- GoM FP density `≤ 1.10×`

任一未過，FTL 不取得實驗 coupon。

### 2. 唯一合法候選

只把既有

`L = 1 - TI`

改為

`L = (1 - TI)^q`，其中 `q=0.75`

其餘固定：`α_FP=0.3`、`β_FN=0.7`、`smooth=1`，資料、split、seed、optimizer、schedule、checkpoint rule、augmentation、推論與評估全部凍結；直接 comparator 是同代 plain Tversky，不是 Focal CE。

### 3. 實作 gate

進 GPU pilot 前須全部通過：

- `q=1` 的 forward 與對 logits 的 gradient 等同現有 Tversky。
- exponent curve 測試。
- ignore-only、empty-oil、mixed-batch 測試。
- AMP finite-gradient 測試。
- 最小 determinism 測試。

### 4. Fold 1 pilot 與停損

PASS 必須同時滿足：

- `ΔS ≥ +0.005`
- 各 `ΔM ≥ -0.010`
- large-oil recall drop `≤ 0.02`
- GoM FP density `≤ 1.10×`

失敗即記為 controlled negative，停止且不做 `q` sweep。Fold 1 是 baseline 最高、headroom 最小的一折，因此 NO-GO 是保守的局部證據，不外推為 FTL 在所有資料與設定下皆無效。

### 5. 三折 screen

Fold 1 PASS 才准補完三折；總體 PASS 必須同時滿足：

- `ΔM1 ≥ +0.008`
- `ΔM2 ≥ 0`
- event bootstrap `Pr(ΔS>0) ≥ 0.80`
- large-oil recall 與 GoM FP guardrail 均通過
- 報告 12 groups 與 LOGO

即使通過，也只能稱為 **single deterministic seed screen**，不能宣稱 seed-robust superiority。

### 6. 評估與 artifact 對稱

- FTL 與 plain Tversky 的 artifact、推論、threshold 與評估流程必須完全對稱。
- threshold 必須固定或事前登記且兩組共用，禁止看過 FTL prediction 後調門檻。
- 若 canonical Tversky checkpoint 缺必要 raw artifact，可做 post-hoc raw inference；不得為了補 reference 而重訓。

### 7. 暫緩的其他 loss family

- `Focal CE + Tversky` compound loss 在機制上合理，但屬另一個 loss family；日後若開，只能用一個 preregistered `λ`。
- per-image Tversky 也屬另一個實驗，不得混入本次 FTL coupon。

## 最強支持與反對理由

- **最強支持**：plain Tversky 已對 recall/overlap 問題展現合理先驗；FTL 只增一個固定 exponent，能以低成本、乾淨單變因檢查後期 residual gradient reshaping 是否再帶來 Contract 指標增益。
- **最強反對**：對 batch-level TI 做 focal exponent 不會在 batch 內定位 hard pixels；`q=0.75` 還會壓低低-TI／高誤差批次的相對梯度，因此「更聚焦困難小油汙」不是公式保證。若 plain Tversky 本身未先站住，FTL 缺乏值得投注的核心機制。

## 殘餘風險與反證條件

- hard-mask P/R 不能替代 soft batch TI；若未記 soft TI，不得把結果解讀成已驗證 gradient-regime 機制。
- Fold 1 的保守 NO-GO 可能是假陰性；本案接受此停損以控制 adaptive search，但不得寫成普遍否定。
- 12-group bootstrap 與單 seed 不涵蓋 training-seed variance；screen PASS 仍可能無法跨 seed 重現。
- 任一比較若 threshold、raw artifact、checkpoint selection、augmentation stream 或 evaluator 不對稱，結果失去因果解讀資格。
- 若 `q=1` parity、determinism 或有限梯度測試失敗，立即停止，不得啟動訓練。
- 若 canonical plain Tversky 未通過前置資格，或 FTL 未通過 fold1／三折任一 gate，則本案假說在目前協定下被反證並停止，不掃其他 `q`。

## 執行狀態

本案只完成方法論協商與預註冊式決策；未修改實驗程式、未啟動或變更任何訓練。
