---
date: 2026-08-31
type: experiment
stage: training
status: completed
tags: [background-patch, pixel-ratio, quota-policy, sensitivity-analysis, prereg, gt-expand, oil-detection, prithvi, deeplabv3, tversky, evaluation-contract, event-clustering, inconclusive]
---

# P9 油背像素比例 bracket 敏感度實驗：inconclusive 與凹性發現

> 延續 [[20260827_P7背景patch配額政策實驗_r12_null與機制發現]] 的槓桿系列，第六項槓桿測試。P7 改的是背景 patch 在各景之間的**分佈**（總張數不變），P9 第一次動**全域水位**——訓練語料油汙:背景像素比例 ρ。判定結果同樣是 inconclusive，但產出兩項統計顯著但未達實質門檻的發現，以及一組可長期保留的機制知識（三臂失敗模式方向相反）。

## 1. 實驗背景

觸發問題：使用者質問「我們怎麼知道 1:23.2 就是最好的油汙背景比例？」以及「那這樣的話要怎麼去說明不是 1:20 或 1:25 比較好啊？」

核心事實：現行訓練語料的油汙:背景像素比例 ρ₀ = 23.0187 **不是選出來的**，是「每景 10 張背景」這條 2024 年便利規則的算術殘值（3,510 張背景 / 2,893 張油汙 patch），從未被比較過、從未被驗證過。且 P7（唯一做過的背景操作實驗）總張數仍約 3,510，改的是背景在各景之間的**分佈**，不是全域**水位**——全域水位至今從未變動過。

決議來源：`.agents/decisions/20260828_油背像素比例敏感度研究設計.md`（Claude × Codex 4 回合收斂，STANCE: ACCEPT / BLOCKING: []）。

## 2. 方法架構 / Pipeline 流程

- **選點**：ρ ∈ {15, 23.0187, 30}，以 ln ρ 建模的**不等距** bracket（ln 23.0187/15 = 0.428、ln 30/23.0187 = 0.265）。**包住 1:20 與 1:25 但不直接測。**
- **ρ 定義**：全語料**非油像素 ÷ 油像素**，非油 = 油污 patch 內的非油像素（h_f = 172,124,821）+ 背景 patch 數 × 65,536；油像素 o_f = 17,470,827。
- **嚴格巢狀 A₁₅ ⊂ A₂₃ ⊂ A₃₀**，以現行 3,510 張為不可變軸心：
  - 下臂（rho15）= 逐景取正典有序序列的**前 B₁₅,ₛ 張**（純前綴，append_count 總和 = 0）
  - 中臂（rho23）= 正典本身原封不動 → **重用正典三折，零 GPU 成本**
  - 上臂（rho30）= 正典 10 張全留 + 從 P8 凍結的 k22 候選池 extra 段依凍結順序追加（prefix 3,510 + append 1,861）
- **配額政策**：`analysis/p8_bg_pool_k22/bg_quota_policy_px.py` 給出 T* = round((ρ·Σo_s − Σh_s)/65,536)；自適應保底 f* = max{f: Σmin(C_s,f) ≤ T*}；**餘數按 C_s（可用海域容量）配水，不按油量**——按油量正是 P7 的失敗模式。
- **逐臂實得**：rho15 T*=1372, f*=3，逐景 B_s 為 3 張×32 景 / 4 張×319 景；rho23 T*=3510, f*=10，每景 10 張；rho30 T*=5371, f*=15，15 張×245 景 / 16 張×106 景。共 351 景（另 4 景無背景候選，三臂皆配額 0）。
- 三臂皆 Prithvi_EO_V2_300M + DeepLabV3+ + Tversky(α=0.3, β=0.7)、seed 42、patience 25、epochs 上限 300、batch 16。唯一變因 = `dataset.bg_coord_txt`。
- **禁用 `random.sample(k)` 做巢狀**（決議明文）：Python 依 k/n 比例切換演算法，`sample(c,15)` 前 10 個一般不等於 `sample(c,10)`，直接實作巢狀性會喪失。改用 domain-separated stable hash 排序追加候選。

## 3. 關鍵參數

| 項目 | 值 |
|---|---|
| 唯一變因 | `dataset.bg_coord_txt` |
| ρ bracket | {15, 23.0187, 30}（ln 尺度不等距） |
| 巢狀關係 | A₁₅（1,372）⊂ A₂₃（3,510）⊂ A₃₀（5,371） |
| 主終點 | Contract S=(M1+M2)/2（Evaluation Contract v1.0） |
| bootstrap 設定 | 12 事件群 paired cluster bootstrap，B=9,999，seed=20260718 |
| family-wise 臨界值 | sup-t c = 2.875（4 個對比） |
| 等價邊界 | ±0.02 |
| size 三分位邊界（P7 凍結） | tiny ≤3,200 px < mid ≤14,147 px < large |
| 成本硬上限 | 45 GPU-h |

## 4. 建構閘（六道全過）

| 閘 | 結果 |
|---|---|
| 1. scene block 連續 | PASS |
| 2. k22 池對正典為 append-only strict superset（extra 4,210） | PASS |
| 3. 逐景單調 0 ≤ B₁₅ ≤ B₂₃ ≤ B₃₀ ≤ C_s（355 景全成立） | PASS |
| 4. **中臂身分閘**：builder 產出的 ρ=23.0187 與正典 3,510 列逐列逐值完全一致（含列順序） | PASS → 正典三折可重用（省約 20 GPU-h） |
| 5. 嚴格巢狀 1,372 ⊂ 3,510 ⊂ 5,371、無重複座標 | PASS |
| 6. 由座標數反推實際 ρ | **14.9987 / 23.0187 / 29.9996**，誤差 −0.0013 / 0 / −0.0004，PASS |

## 5. 執行

2026-08-28 08:15:41Z 啟動 → 2026-08-29 20:22:45Z 完成，rc=0，未中斷。約 29.6 GPU-h（硬上限 45 h）。

逐折訓練時間：rho15 203.6 / 242.6 / 265.2 分；rho30 383.8 / 405.9 / 272.5 分。每臂啟動前都跑磁碟 gate（兩次皆 PASS，914–915G 可用）。

## 6. Exposure audit（決議 §7 必做，是詮釋閘門）

**判定 NOT_CENSORED**：九折皆未觸及 300 epoch 上限（最大 stop epoch = 88），且 best checkpoint 皆非最後一次 evaluation（stop − best = 25 = patience，九折全部乾淨早停）→ **比例機制／曲率／等價詮釋未被禁止**。

逐臂平均：patches 2,259 / 3,394 / 4,380；steps/epoch 142 / 212 / 274；**steps-to-best 5,294 / 9,128 / 8,632**。

⚠ **rho15 只拿到 rho23 的 58% optimizer 更新（−42%），rho30 只差 −5%。** 故 estimand = **完整政策包敏感度，不是純 ρ 因果效應**，偏誤方向不預設。

## 7. 主終點結果

| arm | ρ | M1 | M2 | S | 逐折 pooled 再平均 |
|---|---|---|---|---|---|
| rho15 | 14.9987 | 0.4301 | 0.4647 | 0.4474 | 0.4257 |
| rho23 | 23.0187 | 0.4439 | 0.4773 | **0.4606** | 0.4399 |
| rho30 | 29.9996 | 0.4238 | 0.4572 | 0.4405 | 0.4362 |

對比家族（12 事件群配對 cluster bootstrap，B=9,999，seed 20260718；sup-t family-wise 臨界值 c = 2.875）：

| 對比 | 估計 | SE | simultaneous CI95 | 排除 0 | 判定 |
|---|---|---|---|---|---|
| S₁₅ − S₂₃ | −0.0132 | 0.00484 | [−0.0271, +0.0007] | 否 | inconclusive |
| S₃₀ − S₂₃ | −0.0201 | 0.00443 | [−0.0328, −0.0074] | **是** | inconclusive |
| S₃₀ − S₁₅ | −0.0069 | 0.00526 | [−0.0220, +0.0082] | 否 | inconclusive |
| 曲率 C | **+0.0174** | 0.00382 | [**+0.0065**, +0.0284] | **是** | inconclusive |

C = S₂₃ − (0.3825·S₁₅ + 0.6175·S₃₀)。**總判定：inconclusive**（沒有任一對比的 CI 完整落在 ±0.02 內，也沒有任一完整落在 ±0.02 外）。

## 8. 兩條「統計顯著但未達實質門檻」的發現（本篇重點）

1. **S₃₀ − S₂₃ 的 simultaneous CI 完全在 0 以下** → family-wise 校正後，把背景提高到 1:30 確實比現行 1:23 差；但 CI 跨過 −0.02，不能宣稱差距達實質門檻。
2. **曲率 C = +0.0174，CI 完全在 0 以上** → 反應面在 bracket 內**是凹的（inverted-U）**，ρ=23 落在 log-linear 內插線**之上**，峰值在區間內部，**不是單調關係**。但 |C| < 0.02 且 CI 跨過 +0.02 → 曲率為 inconclusive，**且不得作內插主張**（不得聲稱 1:20 或 1:25 等價）。

## 9. 機制（三臂的失敗模式相反 —— 這是最可長期保留的知識）

全語料四格（三臂 GT 相同 → TP+FN、FP+TN 恆定，**四格僅 2 個自由度**）：

| 指標 | rho15 | rho23 | rho30 | Δ15−23 | Δ30−23 |
|---|---|---|---|---|---|
| TP | 6,137,852 | 6,202,065 | 6,050,105 | −1.0% | −2.5% |
| FP | 3,412,047 | 3,190,247 | 3,069,079 | **+7.0%** | −3.8% |
| FN | 4,980,333 | 4,916,120 | 5,068,080 | +1.3% | **+3.1%** |
| Precision | 0.6427 | 0.6603 | 0.6634 | −0.0176 | +0.0031 |
| Recall | 0.5521 | 0.5578 | 0.5442 | −0.0058 | −0.0137 |
| F1 | 0.5939 | 0.6048 | 0.5979 | −0.0108 | −0.0069 |
| FPR | 0.0285 | 0.0267 | 0.0257 | +0.0019 | −0.0010 |

- **背景太少（1:15）→ 虛警爆掉**（FP +7.0%，FPR 最差）；**背景太多（1:30）→ 漏檢爆掉**（FN +3.1%，TP −2.5%）。
- **Precision 與 FPR 隨背景單調變好**（0.6427→0.6603→0.6634 / 0.0285→0.0267→0.0257），**但 recall 在 rho23 達峰**。IoU 是兩者合成 → 峰在中間。**這就是曲率 C>0 的像素層成因，不是統計假象。**
- **rho15 被 rho23 完全 Pareto 支配**（precision 與 recall 同時更差）；**rho30 則是壞交易**（precision 換到 +0.0031，recall 賠掉 −0.0137）。
- ⚠ 但 rho15 的全面劣勢**與其少拿 42% optimizer 更新混淆**，不可單獨歸因於比例；反之 rho30 與 rho23 的 steps 只差 5%，故「1:30 有害」是三個對比中混淆最小、最可信的一個。

## 10. 次終點（決議 §5 禁止只報全域）

逐 size 用 P7 凍結三分位（tiny ≤ 3,200 px < mid ≤ 14,147 px < large），逐景平均 Oil IoU：

| size | n | rho15 | rho23 | rho30 | 峰值 |
|---|---|---|---|---|---|
| tiny | 111 | 0.3843 | 0.3883 | 0.3743 | rho23 |
| mid | 120 | 0.4795 | 0.5026 | 0.4743 | rho23 |
| large | 124 | 0.5224 | 0.5324 | 0.5147 | rho23 |

ΔS vs rho23（rho15 / rho30）：tiny −0.0018 / −0.0140；mid −0.0228 / −0.0280；large −0.0146 / −0.0180。

**三個 size 全部峰值在 rho23，無方向衝突**（故不觸發決議「size 層方向衝突 → 描述為政策取捨」的分支）。

- **小油污最怕背景太多**：rho30 讓 tiny 的 FN +16.4%、recall 0.7338→0.6901（−0.0437）。反過來 rho15 對 tiny 幾乎無傷（−0.0039，M1 甚至 +0.0003）。
- **mid 是 rho23 優勢最大的一層**，也是唯一 FPR 非單調的一層（0.0257 → 0.0232 → 0.0240，rho23 最低）。
- 與 P7 的對照很重要：**P7 改分佈傷 small；P9 改水位傷 mid。兩者是不同的軸。**

## 11. ⚠ 逐折不穩定（必寫入限制）

ΔS vs rho23 逐折：rho15 = −0.0191 / −0.0255 / **+0.0015**；rho30 = **+0.0217** / −0.0534 / −0.0326。

**fold1 的 rho30 反而贏過 rho23**（IoU_pooled +0.0079、precision +0.0748、FPR 0.0296→0.0193）。逐折全距 0.075，是點估計 −0.0201 的 **3.7 倍**。

預註冊 bootstrap 對 **12 個事件群**重抽、不對折重抽，**故 fold 層異質性不在該 CI 內**。

## 12. ⚠ 預先議定的論文段落失效（本篇必須醒目記錄）

決議 §論文措辭 的等價版前提為「all pairwise differences in S were contained within the ±0.02 practical-equivalence margin」與「no material departure from log-linear interpolation」。**兩個前提都不成立**：S₃₀−S₂₃ 的 CI 下界 −0.0328 已在 −0.02 之外；曲率 CI 上界 +0.0284 也在 +0.02 之外。**該段不得使用，須改寫為 inconclusive 版本。** 依決議末條，此結局仍須如實寫入，不得只在等價時報告。

## 13. 處置

1. 依預註冊判準，bracket {15, 23.02, 30} 內主判定 **inconclusive**。
2. 現行 ρ=23.0187 在三個受測水位中 S 最高，且 1:30 顯著較差。
3. 依決議 §8，此為 inconclusive，現行值**因 protocol continuity 暫留**，**不得稱經驗最佳、不得稱全域 optimum**。
4. **不得作 1:20 / 1:25 內插主張**。
5. **不得事後追加水位、seed、單臂延長或重跑**；任何追加須重新取得使用者授權。

## 14. 必寫入 Limitations（決議 §反方終審 E1，雙方同意成立）

> Your equivalence intervals resample only the 12 event groups, whereas each treatment arm was trained once at seed 42 and received a different number of optimizer updates under epoch-based early stopping; therefore, the analysis does not establish robustness to training stochasticity or isolate the effect of the class ratio itself.

所有結論限定 **conditional on the frozen deterministic protocol (seed 42)**。

## 15. 一個必須明載的外部依賴

rho30 的追加座標來自 **P8 實驗留下的凍結候選池** `patch_level_gt_expand_bg_k22/bg_coords.tsv`（正典 3,510 append-only 保序 + extra 4,210，全部通過與 runtime `_get_coord_vrt_item` 相同的 NaN 預檢）。**P8 的訓練結果已依使用者裁決刪除，但其資料產物被 P9 沿用**——此依賴須在論文 Methods 明載。

## 16. 一個誠實的建構限制

rho15 不是「隨機少抽」而是「取前綴」。實測正典序列**不是 raster order**（0/351 景座標呈單調排列），但前 4 張相對後 6 張 x 座標平均偏左 244 px（t 檢定 p = 0.0069，SD 1,678，效應量 d ≈ 0.15）。統計上偵測得到、實務上可忽略，但嚴格說 rho15 的背景**不是無偏子樣本**。改用隨機重抽會破壞巢狀性，而巢狀是為了消除更大的「內容差異」混淆——這是刻意的取捨。

## 17. 一個給後人的製圖陷阱

各臂 S 自己的 bootstrap SE 是 **0.033**，但**配對**對比的 SE 只有 0.0038–0.0053——共同的事件效應在配對時被抵銷。若在三個點上畫各自的邊際 CI（±0.064），會把 0.013–0.020 的差距完全蓋掉，視覺上等於把結論畫掉。圖上一律畫**以 rho23 為基準的配對 simultaneous CI**。

另：`pd.factorize` 依首次出現指派事件群編號，**建表列順序一變、同一 seed 抽到的就是不同群組合**，會導致圖與 `primary_results.json` 的 CI 對不上（曾實際發生：[+0.0063,+0.0286] vs [+0.0065,+0.0284]），已修正為兩者共用 manifest 列順序。

## 18. 觀察與結論

- ρ bracket {15, 23.02, 30} 內的敏感度檢驗，主判定為 inconclusive，但兩個對比（S₃₀−S₂₃、曲率 C）達統計顯著卻未達 ±0.02 實質門檻，是「顯著≠實質」教訓的乾淨案例。
- 反應面呈凹形（inverted-U），現行 ρ=23.02 落在峰值附近，機制上對應「precision 隨背景單調變好、recall 在中點達峰」的合成效應。
- 三臂失敗模式方向相反（1:15 虛警爆、1:30 漏檢爆），且逐 size 拆解顯示 P7（分佈）與 P9（水位）傷害的分層不同（P7 傷 small，P9 傷 mid），兩者是互補而非重疊的知識。
- 已關閉槓桿總表（延續 [[20260806_P4操作點決策閾值_實質null]] 第 9 節、[[20260827_P7背景patch配額政策實驗_r12_null與機制發現]] 第 9 節）新增一列，見該篇 §9c 更新。

## 19. 實驗檔案位置

| 類型 | 絕對路徑 |
|---|---|
| 專案 repo | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand` |
| 決議 | `.agents/decisions/20260828_油背像素比例敏感度研究設計.md`（vault 根目錄下） |
| 建臂器 | `analysis/p9_rho_bracket/build_rho_arms.py`（配額政策 `analysis/p8_bg_pool_k22/bg_quota_policy_px.py`） |
| 執行腳本／log | `analysis/p9_rho_bracket/run_p9_chain.sh`、`logs/p9_rho_bracket.log` |
| 結果 | `analysis/p9_rho_bracket/results.md`、`primary_results.json`、`exposure_audit.md`、`exposure_audit.csv`、`arm_comparison.md` |
| 分析程式 | `analysis/p9_rho_bracket/primary_analysis.py`、`exposure_audit.py`、`arm_comparison.py`、`make_figures.py` |
| 圖 | `analysis/p9_rho_bracket/p9_curvature.png`（主圖：S vs ln ρ + 內插線 + 曲率）、`p9_dashboard.png`（六格） |
| 結果目錄 | `result-seg/P9_rho15/`、`result-seg/P9_rho30/`；中臂 atoms 重用 `result-seg/P7_baseline_replay/` |
| P8 凍結候選池（外部依賴） | `/mnt/backup/oil_dataset/new/full_band/data_split/3_fold_stratified_v2/patch_level_gt_expand_bg_k22/bg_coords.tsv` |

## 相關頁面

- [[20260827_P7背景patch配額政策實驗_r12_null與機制發現]] — 前一項槓桿（背景配額分佈），本篇是同系列對「全域水位」的延伸測試，兩者機制互補（P7 傷 small，P9 傷 mid）
- [[20260806_P4操作點決策閾值_實質null]] — 已關閉槓桿總表格式的起點
- [[20260716_資料溯源洩漏與評估合約v1]] — Evaluation Contract v1.0（M1/M2/S、12 事件群 cluster bootstrap 定義），本篇判定端點與 bootstrap seed 皆沿用
- [[20260731_ASPP_rate適配實驗]] — 計畫任務基線 0.4399（逐折 pooled 再平均）出處
- [[GT_expand_pipeline]] — 背景 patch 原始機制（每景固定 10 張）設計章節，本篇即是對其全域水位的首次驗證
- [[20260718_訓練不可重現根因與決定性修復]] — 本篇訓練管線沿用的決定性修復基礎
