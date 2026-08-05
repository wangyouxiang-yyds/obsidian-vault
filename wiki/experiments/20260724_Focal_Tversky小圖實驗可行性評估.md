---
date: 2026-07-24
type: experiment
stage: training
status: completed
tags: [focal-tversky, tversky-loss, oil-detection, gt-expand, deeplabv3, experiment-design]
---

# Focal Tversky 小圖實驗可行性評估

> **狀態：已結案（2026-07-28 判定 null，保留 Tversky）。** 本篇原始第 1–6 節記錄的是啟動前的可行性協商與 gate 設計，內容保留不動；實際執行與判定結果見文末「判定結果（2026-07-27～07-28）」章節。

## 1. 實驗背景

目前工作位於分支 B（[[GT_expand_pipeline]]）的 GT-centric 小圖主軌：以 256×256 patch 訓練油汙語意分割模型，正式結論用重建後的 M1/M2，而 patch precision/recall 只作診斷。這條線的瓶頸長期偏向漏抓，尤其是大面積、瀰漫油膜的 under-segmentation；量化脈絡見 [[20260709_baseline錯誤分析_GT過寬與方向二降級]]。

| 項目 | 現行設定 |
|---|---|
| 任務 | 二類別油汙語意分割 |
| 資料 | 355 scenes，3-fold stratified v2 |
| Patch | GT-centric 256×256，另加入同 fold 場景的 background patch |
| 輸入 | 8-band Sen2Like VRT |
| 模型 | torchvision `deeplabv3_resnet50`，ImageNet pretrained、differential learning rate |
| 決定性 baseline loss | pixel-level Focal CE，`gamma=2` |
| Train oil-pixel ratio | fold1/2/3 = 0.0586 / 0.0775 / 0.0680 |

2026-07-23 至 07-24 完成的 deterministic Focal baseline，其 recon pooled Oil IoU 為：

| Fold | recon pooled Oil IoU | Patch Precision(Oil) | Patch Recall(Oil) |
|---|---:|---:|---:|
| fold1 | 0.3915 | 0.6325 | 0.4713 |
| fold2 | 0.3512 | 0.7149 | 0.4231 |
| fold3 | 0.2915 | 0.7268 | 0.3444 |
| **平均** | **0.3447** | — | — |

Patch 診斷仍呈現 precision 高於 recall，但這是 threshold 後的 hard-mask 統計，不能直接代表 loss 計算時的 soft batch Tversky Index。

舊執行世代中，plain Tversky `0.3/0.7` 相對舊 Focal baseline 曾有約 `+0.06` pooled IoU 增益；但當時 Albumentations internal RNG 未固定，兩組都是單次抽樣，故已降為 legacy evidence，不能用來宣告正式優勢。協商當下，新的 canonical deterministic plain Tversky `0.3/0.7` 正在重跑；Focal Tversky 尚未開始。

## 2. 方法架構 / Loss 定義

令 `p_i` 為有效像素的 oil soft probability、`y_i∈{0,1}` 為 oil target；忽略 `ignore_index=255` 後，在目前實作中以整個 batch 聚合：

$$
TP=\sum_i p_i y_i,\qquad
FP=\sum_i p_i(1-y_i),\qquad
FN=\sum_i(1-p_i)y_i
$$

plain Tversky 的設定為：

$$
TI=\frac{TP+1}{TP+0.3FP+0.7FN+1}
$$

$$
L_T=1-TI
$$

本次唯一允許的 Focal Tversky 候選為：

$$
L_{FT}=(1-TI)^q,\qquad q=0.75
$$

令 `u=1-TI`，Focal Tversky 相對 plain Tversky 的梯度倍率為：

$$
\frac{\partial u^{0.75}/\partial u}{\partial u/\partial u}
=0.75u^{-0.25}
$$

倍率等於 1 的交點是 `u=0.75^4≈0.3164`，亦即 `TI≈0.6836`：

- `TI<0.684`（低 TI／高誤差 batch）時，相對梯度倍率小於 1。
- `TI>0.684`（高 TI／低誤差 batch）時，相對梯度倍率大於 1。

因此 `q=0.75` 比較像「抑制早期高誤差 batch、加強後期殘餘修正」，不是一般 pixel-level focal loss 的 hard-pixel mining。因目前 TI 是整個 batch 的單一 scalar，它只能整體縮放該 batch 的梯度，不能在 batch 內指出哪個薄油邊界或像素比較困難。

Codex 在協商中修正了一個過度推論：hard patch TTA precision/recall 無法精確反推出 soft batch TI，所以先前可能提到的「約 15–18% 梯度 attenuation」沒有實證基礎，不能保留為機制結論。若未來要主張實際梯度落在哪個 regime，必須記錄訓練時的 batch soft TI 分布。

## 3. 關鍵參數與控制變因

| 項目 | 凍結值／規則 |
|---|---|
| Direct comparator | canonical deterministic plain Tversky |
| `alpha_FP` | 0.3 |
| `beta_FN` | 0.7 |
| `smooth` | 1 |
| FTL exponent `q` | 0.75，唯一候選 |
| 其他變因 | split、seed、資料、augmentation、optimizer、schedule、checkpoint rule、推論與 evaluator 全部凍結 |
| Threshold | 固定或事前登記，FTL 與 plain Tversky 共用 |
| Reference artifact | 可對 canonical Tversky checkpoint 做 post-hoc raw inference；不可為補 reference 而重訓 |

`q=0.75` 若失敗，不再掃其他 `q`。`Focal CE + Tversky` compound loss 雖有機制合理性，但屬另一個 loss family；日後若另開，最多只測一個事前登記的 mixing weight `λ`。per-image Tversky 同樣是另一個實驗，不得混入本次比較。

## 4. 量化現況與前置 Gate

### 4.1 指標定義

- **M1**：region（GoM / Atlantic / Other）× oil-size tertile 的 9-stratum 等權 macro Oil IoU。
- **M2**：355 scenes 等權的 per-scene mean Oil IoU。
- **S**：`S=(M1+M2)/2`，所以 `ΔS=(ΔM1+ΔM2)/2`。

### 4.2 FTL 取得實驗資格的前置 gate

canonical deterministic plain Tversky 必須先相對 deterministic Focal baseline 同時滿足：

| 條件 | 門檻 |
|---|---:|
| `ΔM1` | `≥ +0.008` |
| `ΔM2` | `≥ 0` |
| large-oil recall drop | `≤ 0.02` |
| GoM FP density | `≤ 1.10×` |

任一未過，Focal Tversky 不取得實驗資格。

### 4.3 Fold 1 pilot gate

只有前置 gate 通過才做 fold1。PASS 必須同時滿足：

| 條件 | 門檻 |
|---|---:|
| `ΔS` | `≥ +0.005` |
| 每一個 `ΔM` | `≥ -0.010` |
| large-oil recall drop | `≤ 0.02` |
| GoM FP density | `≤ 1.10×` |

Fold1 是 baseline 最高、headroom 最小的一折，因此 NO-GO 是保守的局部證據，不代表 FTL 在所有資料集或設定下必然無效；但本專案仍以此作停損：失敗即記為 controlled negative，不補三折、不做 `q` sweep。

### 4.4 三折 screen gate

Fold1 PASS 才能補完三折；總體 PASS 必須同時滿足：

| 條件 | 門檻 |
|---|---:|
| `ΔM1` | `≥ +0.008` |
| `ΔM2` | `≥ 0` |
| event bootstrap | `Pr(ΔS>0) ≥ 0.80` |
| Guardrails | large-oil recall 與 GoM FP 均通過 |
| 穩定性報告 | 12 groups 全列＋LOGO |

即使三折 screen 通過，也只能稱為 **single deterministic seed screen**；它沒有涵蓋 training-seed variance，不能宣稱 seed-robust superiority。

## 5. 觀察與結論

Claude×Codex 的共同結論是：**Focal Tversky 值得保留為一個低成本、單變因的條件式候選，但不應成為現在立刻執行的下一個實驗。**

最強支持理由是：plain Tversky 已直接對應目前的 overlap／recall 問題，而 FTL 只增加固定 exponent，若 plain Tversky 在乾淨正典重跑仍站得住，便可用一張受限實驗券檢查後期 residual gradient reshaping 是否有額外價值。

最強反對理由是：batch-level TI 的 focal exponent 並不等於 hard-pixel mining；`q=0.75` 甚至會相對壓低低-TI batch 的梯度。因此「它會更專注困難小油汙」不是公式保證。若 plain Tversky 本身沒有通過前置 gate，FTL 的核心假說就沒有先站住。

協商由 Codex 主持，Claude 使用 Opus 4.8（high effort），同一 session 共兩輪；最終為 `ACCEPT / BLOCKING: [] / CONFIDENCE: HIGH`，無 model fallback。完整逐輪 provenance 與 blocking 變化見[內部決策正本](../../.agents/decisions/20260724_Focal_Tversky小圖實驗可行性.md)。

## 6. 後續改進方向

- [ ] 等 canonical deterministic plain Tversky `0.3/0.7` 完成，先執行前置 gate
- [ ] 若取得實驗資格，先實作 `q=1` forward／logit-gradient parity
- [ ] 補 exponent curve、ignore-only、empty-oil、mixed-batch 測試
- [ ] 補 AMP finite-gradient 與最小 determinism 測試
- [ ] 只跑 fold1 pilot；未過即停止並記 controlled negative
- [ ] fold1 PASS 才補三折，報 M1/M2、event bootstrap、12 groups、LOGO 與兩項 guardrail
- [ ] 若要解讀機制，額外記錄 batch soft TI；不得用 hard patch P/R 代替
- [ ] compound `Focal CE + Tversky` 保留為獨立 family，不與本次 FTL 混跑

## 7. 實驗檔案位置

| 類型 | 絕對路徑 |
|---|---|
| 專案 repo | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand` |
| Loss 實作 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/main/deeplab_adapter.py` |
| Canonical rerun 預註冊 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p1_tversky_sweep/canonical_rerun_prereg.md` |
| Evaluation Contract v1.0 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/eval_contract/evaluation_contract.md` |
| Deterministic baseline 結果根目錄 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/result-seg/CV_358clean_gt_expand` |
| Canonical Tversky 結果根目錄 | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/result-seg/CV_358clean_gt_expand_tversky` |
| 內部決策正本 | `/home/alanyh/obsidian-vault/.agents/decisions/20260724_Focal_Tversky小圖實驗可行性.md` |

## 8. 判定結果（2026-07-27～07-28，null，保留 Tversky）

> 本節記錄第 1–7 節規劃通過前置 gate 之後的實際執行與判定；前述章節的可行性論述維持不動，僅補記後續結果。

2026-07-27 啟動，3 折乾淨跑完，code commit `f36e421`；run＝`result-seg/CV_358clean_gt_expand_ftl/20260727-104558, 20260727-170323, 20260728-021821`，per-fold recon pooled_oil_iou 0.4541/0.3760/0.3277，**mean 0.3859**。⚠此線為 **ResNet-50 線**（`deeplabv3_resnet50` + ImageNet），**非 Prithvi**；run_metrics.json 的 `architecture` 欄位兩線同為字串 `"deeplabv3+"` 會騙人，實際要看 `Experiment_name`／config yaml（本線為 `..._ftl_deeplabv3+_fold{1,2,3}`，不含 `prithvi300m`）。

2026-07-28 grouped replay 判定（`analysis/p2_focal_tversky/p2_grouped_replay.py`）：三 arm S＝baseline 0.3975 / tversky 0.4512 / ftl 0.4524。

**主對比 FTL vs Tversky：ΔS=+0.0011，95% CI [−0.0087,+0.0168]** → ΔS<0.02 且 CI 跨 0 → **保留 Tversky**。ΔM1 +0.0035 / ΔM2 −0.0012 皆跨 0；Wilcoxon p=0.891；win/loss/tie=177/169/9。整條 CI 夾在 ±0.02 內＝prereg 預寫的「**實質等價**」結局（是接近零，不是測不出）。

佐證：FTL vs baseline ΔS=+0.0549 顯著（95% CI [+0.0347,+0.0692]），與 Tversky 贏 baseline 同量級 → **能力全來自 Tversky 骨幹，加的指數 q 幾乎沒動針**。LOGO ΔM1∈[−0.0038,+0.0087]，無單一事件群主導；唯一受害群＝希臘 Agia Zoni（tiny-oil, n=4）−0.042。

**結論：loss 家族的槓桿在 Focal→Tversky（+0.058）已經吃完，Tversky 上再加 focal 指數空轉；loss 分支確認枯竭，真正槓桿不在 loss（→架構／資料／操作點）**。Phase 2 SAR 組合技（prereg 條件式）因 FTL 隔離非正向 → 不觸發。與同一時期的 [[20260731_ASPP_rate適配實驗|ASPP rate 適配實驗]]（rate 槓桿同樣判定 negative）合看：**loss 與 rate 兩條槓桿皆已枯竭，精進要換地方找**。

γ 陷阱備註：Abraham & Khan 2019 論文寫 `(1−TI)^(1/γ)`、γ=4/3 → 實際指數 0.75（倒數），作者 code 實作 0.75，後人常反寫成 4/3，兩者曲率相反。本專案一律用「直接指數 q」表述（`L_FT=(1-TI)^q`，不取倒數）。

**⚠️ 與第 2–3 節規劃的落差（需明確記錄）**：第 2、3 節（2026-07-24 可行性評估當時）討論的候選是 `q=0.75`（sub-linear，第 5 節據此推導出「抑制早期高誤差 batch」的解讀）。但 2026-07-27 正式凍結的 prereg（`analysis/p2_focal_tversky/preregistration.md`）改登記候選為 **`q=4/3=1.3333...`（direct exponent，super-linear，「focus on hard batch」）**，且 prereg 原文明寫「非 Abraham 倒數 1/γ」——即刻意選了與第 2–3 節不同的指數方向，並非疏漏。實際執行的 config（`main/experiments_CV_358clean_gt_expand_ftl.yaml` `loss.gamma: 1.3333333333333333`）與程式碼（`deeplab_adapter.py` `TverskyLoss.focal_gamma`）跟 07-27 prereg 一致，`main/test_focal_tversky.py` 5/5 PASS 亦針對 `q=4/3` 驗證。**上方「判定結果」的數字（mean 0.3859、ΔS=+0.0011 等）皆為 07-27 prereg 凍結的 `q=4/3` 執行結果，不是第 2–3 節（07-24）討論的 `q=0.75`**——兩者曲率方向相反（`q=4/3` 加強高誤差 batch 梯度，`q=0.75` 才是抑制），第 5 節的梯度交點推導僅適用於 `q=0.75` 這個未被採用的候選，不適用於實際執行的 `q=4/3`。

## 相關頁面

- [[GT_expand_pipeline]] — 小圖分支的資料、patch 與評估流程
- [[DeepLabV3+]] — 模型與 loss 工程背景；現行主軌實際使用 torchvision DeepLabV3
- [[20260709_baseline錯誤分析_GT過寬與方向二降級]] — recall／under-segmentation 問題的量化背景
- [[20260702_CV_358clean_gt_expand_進行中]] — 早期 GT_expand 三折訓練紀錄
- [[20260731_ASPP_rate適配實驗]] — 同時期的另一條槓桿測試（decoder rate 適配），同樣判定 negative
