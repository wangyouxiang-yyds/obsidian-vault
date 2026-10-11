---
date: 2026-10-11
type: experiment
stage: supervised
status: completed
tags: [dads-sam, gom-only, taiwan-holdout, sam-decoder, experiment-card, inconclusive, single-seed, system-comparison]
---

# DADS-SAM GoM-only 正式 C 與 B：兩者皆不可判定

> 本頁為「實驗九步驟卡」制度（`.agents/institution/templates/experiment_card.md`）下的結案卡，只填第 7–9 步。第 1–6 步（開跑前凍結：目的、問題、研究問題與 preflight、假說、方法、事前數字門檻）正本為 `.agents/decisions/20261010_DADS_SAM_GoM_C_B正式run設計.md`（vault 根目錄下，1–6 已凍結，本頁不重複抄錄、不修改該卡）。

**實驗編號**：無 P 編號（DADS-SAM GoM-only B／C）
**凍結日期**：2026-10-10（1–6，見上引設計決議）
**結案日期**：2026-10-11
**事後補寫**：C＝否；B＝部分事後（凍結前已看過 B 100-epoch pilot 的臺灣結果，9 景平均 0.3179，見第 8 步）
**取代哪張卡**：無

三組定義：A＝既有 Prithvi＋DeepLabV3 基線（`MS6_P14_GoM_only_full_R1`，不重跑）；B＝公開 DADS-SAM 六波段適配；C＝Prithvi encoder＋SAM decoder。正式 run 於 2026-10-10 11:37 啟動（C 先 B 後，同一個 process）。

---

## 前半：開跑前凍結（1-6）

見正本 `.agents/decisions/20261010_DADS_SAM_GoM_C_B正式run設計.md`（vault 根目錄下），本頁不重複抄錄。

---

## 後半：結案才填（7-9）

### 7. Evidence / Result（實際結果）

`_kind` = `recon(gt_aware)`；checkpoint＝best.pt；臺灣每個 run 只評一次。主指標＝臺灣 9 景各算 Oil IoU 再平均；R＝A 設定四次 run 的 9 景平均＝0.3862（四次：0.3657［＝A］、0.4111、0.4050、0.3632）。

來源：
- 報表：repo `analysis/dads_sam_gomonly_20261005/endpoint_report.json`（GT_expand commit `b530e45`）
- 各 run 的 `results.csv`、`run_metrics.json`、`run_record.json`（路徑見文末）

**三組主表**

| 組 | GoM val mIoU | GoM val Oil IoU | 臺灣 9 景平均 Oil IoU（X） | X−R（R＝0.3862） | 判定 |
|---|---:|---:|---:|---:|---|
| A（基線，不重跑） | 0.7303 | 0.4934（第 46 epoch） | 0.3657 | —（A 為 R 的組成之一） | —（基準） |
| C（Prithvi＋SAM decoder） | 0.7394 | 0.5091（第 50 epoch） | 0.3596 | −0.0266 | 不可判定 |
| B（公開 DADS-SAM 六波段） | 0.7394 | 0.5081（第 83 epoch） | 0.3732 | −0.0130 | 不可判定 |

判定規則（卡 §6，事前凍結）：X−R ≥ +0.05 偏高、≤ −0.05 偏低、其餘不可判定。

**訓練紀錄**

| 組 | 早停 epoch（patience 50） | best.pt epoch | 訓練時間 | 備註 |
|---|---:|---:|---:|---|
| C | 100 | 50 | 88.9 分鐘 | 10-10 11:37–13:06 |
| B | 133 | 83 | 700.1 分鐘 | 10-10 13:07 至 10-11 00:47；results.csv 133 列全為有限值；run_record status trained、error null；git_head `f3b1001`；整個序列 EXIT=0（10-11 00:49） |

**描述性指標（不作判定）**

| 組 | X−A（A＝0.3657） | 兩事件平均 | 9 景像素加總 | leave-one-scene-out 的 X−R 範圍 |
|---|---:|---:|---:|---|
| A | — | 0.4101 | 0.4187 | 未填（待使用者） |
| C | −0.0061 | 0.3659 | 0.3373 | −0.045～−0.013（不改判定） |
| B | +0.0076 | 0.3958 | 0.4434 | −0.023～+0.031（不改判定） |

A 設定四次 run 的範圍：9 景平均 0.363–0.411；兩事件平均 0.379–0.461；像素加總 0.370–0.481。

**逐景 Oil IoU**

| 景 | B | C | A | A 設定四次平均 | B pilot（100 epoch） |
|---|---:|---:|---:|---:|---:|
| 0801_L9_50QRK | 0.410 | 0.292 | 0.303 | 0.340 | 0.336 |
| 0801_L9_50QRL | 0.327 | 0.216 | 0.250 | 0.277 | 0.255 |
| 0912_S2_50QRL | 0.141 | 0.626 | 0.444 | 0.510 | 0.033 |
| 0918_L9_50QRK | 0.478 | 0.492 | 0.462 | 0.481 | 0.474 |
| 0918_L9_50QRL | 0.542 | 0.553 | 0.526 | 0.524 | 0.540 |
| 0922_S2_50QRK | 0.260 | 0.204 | 0.181 | 0.208 | 0.193 |
| 0922_S2_50QRL | 0.245 | 0.164 | 0.170 | 0.189 | 0.136 |
| 0522_L8_51QTD | 0.477 | 0.345 | 0.474 | 0.470 | 0.446 |
| 0522_L8_51QUD | 0.478 | 0.344 | 0.481 | 0.478 | 0.448 |

**事件別 Oil IoU**

| 事件 | B | C | A |
|---|---:|---:|---:|
| 2026-05-22 | 0.477 | 0.342 | 0.474 |
| 天使輪 2023 | 0.314 | 0.389 | 0.346 |

**正式 B 的臺灣 post-test 完整性**：9/9 景；prediction manifest complete（ok 9／failed 0／skipped 0）；報表 P12 parity PASS。

### 8. Answer / Finding（結論）

**不可判定**（C 與 B 皆是；每組只有一次 run，power 不足）。

- C：X＝0.3596，X−R＝−0.0266，落在 ±0.05 之內。九個 leave-one-scene-out 的 X−R 介於 −0.045～−0.013，皆不改判定區。
- B：X＝0.3732，X−R＝−0.0130，落在 ±0.05 之內。九個 leave-one-scene-out 的 X−R 介於 −0.023～+0.031，皆不改判定區。
- 與第 6 步門檻的對照：兩者都落在「不可判定」區間，沒有落在「支持（高於 A 設定）」或「否定（低於 A 設定）」側。B（0.3732）在 A 設定四次 run 的範圍內，C（0.3596）略低於最低的一次（0.3632）。

**B 非盲測**：100-epoch pilot 的臺灣結果（9 景平均 0.3179，X−R −0.0683＝偏低，拿掉 0912_S2_50QRL 就改判）在規則凍結前已看過；pilot 與正式 run 並列報告、不合併。

**觀察（描述性；每組只有一次 run）**
- 與 A 設定四次平均相比，B 有四景（0801 兩景、0922 兩景）高出 0.05–0.07，另四景相差 0.02 以內；唯一明顯偏低的是 0912_S2_50QRL（整景只有 1 張 patch）。相對於 A 設定四次平均，這一景讓 B 的 9 景平均少約 0.04。
- C 的差異集中在兩處且方向相反：05-22 兩景低約 0.13；0912_S2_50QRL 高 0.18。
- 整體：正式 C、B 都落在不可判定區。

**不可主張**：B 或 C 與 A 等效；B、C 的排名；差異來自特定元件（SAM encoder、LoRA、decoder）。

**缺什麼 / 限制**
- 每組單一 seed。
- B 非盲測（見上）。
- 測試只有 9 景、兩個事件。
- 公開碼的 DSCA 不在計算圖內（沿襲公開碼），所以 B 測的是「公開碼的實際行為」，不是論文描述的完整方法；論文原文未逐段對過。

### 9. Decision / Next（下一步）

- **這條路線**（續跑／收線／改設計）：未填（待使用者）。
- **未被此實驗排除的替代解釋**：
  - 單一 seed、單次 run 的雜訊（每組只有一次 run，無法與 A 設定四次 run 的範圍區分）。
  - B 的結果受公開碼實作行為影響（DSCA 不在計算圖內），不等於論文描述的完整方法。
  - 九景中單景的影響：0912_S2_50QRL 只有 1 張 patch 卻占 1/9 權重，B 與 C 在該景方向相反。
- **未解**：B 在 0912_S2_50QRL 偏低的原因（pilot 也偏低）。
- **下一張卡**：未填（待使用者）。
- **排程狀態（非判定內容）**：排在 DADS-SAM 之後的 P12 seed 擴充（tmux `p12_seeds`）於 2026-10-11 00:50:15 自動啟動時被 launch 閘門拒絕（`M analysis/dads_sam_gomonly_20261005/endpoint_report.json`，EXIT=2），沒有在跑。原因是 DADS-SAM session 在 00:50 跑 build_report.py 改寫了這個已追蹤檔，約 20 秒後才 commit。之後沒有啟動任何 process；GPU 自 00:50 起閒置；重新啟動待使用者決定。

---

## 實驗檔案位置

| 類型 | 絕對路徑 |
|---|---|
| 專案 repo | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand` |
| 設計決議（1-6 正本） | `.agents/decisions/20261010_DADS_SAM_GoM_C_B正式run設計.md`（vault 根目錄下） |
| 報表 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/dads_sam_gomonly_20261005/endpoint_report.json`（commit `b530e45`） |
| 正式 C run 目錄 | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly/20261010-113744_MS6_DADS_SAM_C_prithvi_samdec_GoM_R1/` |
| 正式 B run 目錄 | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly/20261010-130708_MS6_DADS_SAM_B_public6_GoM_R1/` |
| 序列 log（結束時末行 `EXIT=`） | `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_formal_C_then_B_main_runner.log` |

## 相關頁面

- [[20260915_P14_GoM_vs_NonGoM_full_ablation_主端點未計]] — A＝`MS6_P14_GoM_only_full_R1` 的來源實驗
- [[20261005_台灣預測線寬量測與2x放大推論試探]] — 設計卡 §2 引用的臺灣預測線寬量測
