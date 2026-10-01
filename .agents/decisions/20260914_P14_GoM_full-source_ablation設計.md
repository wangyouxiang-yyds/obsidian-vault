# 2026-09-14 P14 GoM full-source ablation 設計

議題：延續 P13 source-group sensitivity 的提問，但改用「每個海域分組的全部可用 source scenes」（不做 P13 式的 budget-matched 配額），比較 GoM-only 與 NonGoM-only 兩個 full-source 訓練臂在同一台灣 holdout 上的表現差異，並以描述性門檻 δ=0.02 判定差異是否達到值得記錄的量級。

專案：延續 P13，同一 repo `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand`（沿用 P13 路徑與 Taiwan holdout，本次交付未重新確認路徑本身，若有變動請另行更正）。

協商狀態：Claude × Codex 三回合收斂，最終雙方 `ACCEPT / BLOCKING=[]`。

**執行狀態：僅完成設計協商，未啟動訓練、推論或任何 GPU job。**

> 本次交付僅提供三輪 STANCE/BLOCKING 的結構性演變（各輪 blocking 編號與數量），未附 B1–B7 各項的具體議題文字。本紀錄不臆測補齊這些內容；若需逐項覆核，須調閱 session `cdb9a6bd-f24f-4197-bf0c-eb8e6dc7ea41` 原始記錄。

## 1. 定位：與 P13、P12 的關係

- P13（`decisions/20260911_...` 與 `decisions/20260913_...`）用 budget-matched A/B×G1/G2/G3/Mixed 8 arms 描述固定配額下的 source-group 表現；P14 改為兩臂、每臂使用該海域分組**全部可用**的 source scenes/patches，是不同 estimand 的平行設計，不是 P13 的延伸重跑。
- **P12 僅作歷史參考**：P12 的既有結果或結論不構成本設計的證據基礎、不作為本次比較的先驗或校準依據，僅用於背景脈絡陳述。

## 2. 兩臂設計與資料量（最終定案）

| Arm | Train scenes | Train oil patches | Train BG patches | Val scenes隱含統計 | Val oil | Val BG |
|---|---:|---:|---:|---|---:|---:|
| GoM-only | 156 | 1,188 | 1,550 | — | 312 | 390 |
| NonGoM-only | 120 | 584 | 1,180 | — | 132 | 310 |

- 兩臂皆為 full-source（各自海域分組全部可用 scenes），未做跨臂配額配平；GoM 臂 train 規模（scenes/oil/BG）皆大於 NonGoM 臂。
- Val 為各臂自身 source-domain 的獨立驗證集（非 Taiwan），供 checkpoint selection 使用（見第 4 節）。

## 3. 訓練與 Taiwan test 協定（凍結，與 P13 對齊之處已標註）

- 模型：Prithvi-EO-2.0-300M + DeepLabV3+（與 P13 相同）。
- Loss：Tversky `alpha=0.3` / `beta=0.7`（與 P13 相同）。
- batch size 16，最多 300 epochs，patience 50（early stopping，與 P13 相同）。
- Checkpoint selection：**best source-Val**（見第 4 節；優於 P13 的 non-independent train-proxy）。
- seed = 42，單次 run／每臂（single seed，與 P13 相同限制）。
- Taiwan test：137 patches／9 scenes／2 events（與 P13 完全相同的固定 holdout）。

## 4. Checkpoint selection：best source-Val

- 與 P13 不同，P14 兩臂各自擁有獨立的 source-domain Val 集（GoM-Val 312 oil/390 BG；NonGoM-Val 132 oil/310 BG），checkpoint 依各臂在自身 source-Val 上的最佳表現選取，**不使用 Taiwan 資料做模型選擇**，避免 Taiwan-side 洩漏進入 checkpoint 選取。
- 此為相對 P13 train-proxy 選模的方法論改進，但兩臂 Val 規模不同（312/390 vs 132/310），選模穩定性仍可能因 Val 集大小差異而不對稱，不視為完全消除的差異來源。

## 5. δ=0.02 描述性判定

- 兩臂在 Taiwan holdout 上的表現差（GoM-only 與 NonGoM-only 的 Oil IoU 差值）以 δ=0.02 作為**描述性**門檻：|Δ| < 0.02 時記錄為「未觀察到值得記錄的量級差異」，|Δ| ≥ 0.02 時記錄為「觀察到量級差異」。
- δ=0.02 是**描述性**分類門檻，不是統計檢定的顯著性水準、不附信賴區間或 p-value，也不代表該量級具有實務或因果意義上的門檻效力。

## 6. 可以與不可以聲稱的結論

### 可以
- 描述「在這一個 seed、凍結協定、固定台灣 holdout 下，GoM-only 與 NonGoM-only 兩個 full-source 訓練臂的 Taiwan Oil IoU 差值是否達到 δ=0.02 描述性門檻」。
- 將結果與 P13 的 source-group budget-matched 結果並列比較（不同 estimand，須註明差異）。

### 不可以（本次協商明確禁止）
- **不得宣稱海域因果效果**：GoM/NonGoM 的表現差異不得歸因為海域本身的因果效應，兩臂在 scenes／oil／BG 數量上本就不對等（full-source、非配平），任何差異都無法排除資料量、場景多樣性、content composition 等混淆。
- **不得宣稱統計顯著**：single seed、無跨 seed 變異估計、無顯著性檢定；δ=0.02 僅為描述性分類，不得包裝成統計判定。
- 不得將 P12 的既有結論或數字當作本次比較的基準或先驗。
- 不得外推至其他海域、季節、sensor 或非本 9 scenes/2 events 的台灣資料。

## 7. 協商歷程

同一對手方 headless session（Claude，見第 8 節 provenance），三回合收斂：

- **R1**：`STANCE: REVISE`，`BLOCKING: [B1, B2, B3, B4, B5, B6, B7]`（七項）。
- **R2**：`BLOCKING` 縮減為僅剩 `[B7]`（六項於本輪之前解決）。
- **R3**：雙方 `STANCE: ACCEPT`、`BLOCKING: []`，正式收斂。

各輪 B1–B7 逐項具體內容未隨本次交付提供，本紀錄不代為補述或臆測；已知的兩項硬性結論（不得宣稱海域因果、不得宣稱統計顯著）已寫入第 6 節作為協商後的定案條款。

少數意見：未提供，本紀錄不臆測。

## 8. Harness provenance

- requested/actual model = `claude-opus-5`，effort = `high`，無降級事件（依交付資訊，未見 fallback 觸發紀錄）。
- Claude CLI：`2.1.270`。
- session id = `cdb9a6bd-f24f-4197-bf0c-eb8e6dc7ea41`。
- usage/cost：未隨本次交付提供，本紀錄不臆測補齊。

## 9. 殘餘限制

- GoM／NonGoM 兩臂 scenes／oil／BG 數量皆不對等（full-source、非配平設計），無法排除資料量或場景多樣性混淆，因此結果只能是 source-group 表現的關聯描述，不是海域效應的因果推論（見第 6 節）。
- single seed=42，無跨 seed 穩健性估計；δ=0.02 描述性門檻不承載統計顯著性。
- checkpoint 依各臂 source-Val 選取，但兩臂 Val 規模不同（312/390 vs 132/310），選模穩定性未必對稱。
- P12 僅為歷史背景陳述，未經本次設計重新驗證，不得作為本次比較的校準基準。
- 純背景 false alarm、全景評估、事件層級拆解（2 events 各自的表現分佈）未在本設計範圍內。

## 10. 啟動閘門與未啟動狀態

**本案僅完成設計協商，兩臂均未啟動訓練、推論或任何 GPU job。** 使用者未另行明確授權前，不得啟動任一臂。

## 11. 後續驗證點

- 多 seed 重跑以估計跨 seed 變異，脫離 single-seed 限制。
- 補做 budget-matched 版本（比照 P13 A/B 配額邏輯）作為 full-source 版本的對照，區分「資料量差異」與「source-group 本身」的貢獻。
- 事件層級拆解（2026/2023 或對應 P13 事件命名）以檢查差異是否集中於特定 event，避免以聚合數字掩蓋事件條件性。
- 加入純背景 false alarm 與全景（非僅 GT-aware patches）評估。
- 若後續要討論海域因果或統計顯著性，須另行設計具備 confounder 控制與變異估計的實驗，本設計本身不足以支撐這類主張。
