---
date: 2026-11-11
type: summary
tags: [p51, gom, roi, experiment-design]
---

> **凍結設計卡的 vault 唯讀鏡像。** 原件：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p51_gom_only_roi_20261111/P51_CARD.md`（本次複製時 SHA-256 `cc817684130fe90b6f3f8c0be6844eae6d0cf2f10554cb83b10bca23142dffb0`）。以下原文保持凍結時狀態；其「未 launch」是當時的歷史敘述。11 月 11 日 09:54 已啟動的記錄見 `.agents/handoff.md` 與 institution `research/oil_spill_project_status.md`；本頁不代替即時 process 查核。

# P51：GoM-only 資料制度（TRAIN＋VAL 皆 GoM、固定步數；P44 自然比例 ROI 流程）→ 臺灣

狀態：**FROZEN v3（2026-11-11）**，Codex R3 ACCEPT／BLOCKING＝[]（R1 REVISE 7 項、R2 REVISE 4 項，均已修正或由使用者裁決）。未 launch；launch 需使用者授權。
v1 曾於 commit efb62f6 凍結；使用者隨後要求送 Codex 審。

使用者裁決（2026-11-11）：
- 「我們已經試過了全球的資料了，接下來我想要試試看 GoM的資料 測試在台灣」；
- VAL 只用 GoM 25 景；
- 跑 3 個 seed（42／43／44）；
- 「可以 給codex審」。
- 讀臺灣 GT 沿用既有授權（「讀台灣GT的部分我這邊授權你直接就是幫我跑完」）。

## 1. 問題
baseline 是自然比例，也就是 P44 流程：全球 TRAIN 298 景＋全球 VAL 37 景。
P51 換成 GoM-only 的整體資料制度：TRAIN 和選模用的 VAL 都只用 GoM，總訓練步數固定，模型、ROI、臺灣推論方式都相同。
問題是：臺灣「大林」與「其他 22 張」的表現和穩定性會不會不同？

這是整體制度的比較，不是「訓練地理範圍」的單一因果效果。TRAIN 範圍、VAL 選模、單窗曝光量、正負窗比例四者同時改變（見 §2）。

## 2. 改動：資料範圍
清單由 `p51.py lists` 從 P44 凍結清單篩出（先核對 P44 `FROZEN_LISTS.sha256`）。每一列、每個窗都和 P44 相同，只是取子集。

| | P44 baseline（全球） | P51（只用 GoM） |
|---|---|---|
| TRAIN 景／ROI | 298／324 | 158／181 |
| TRAIN 256 窗（剔除無效後） | 265,496 | 148,684（56%） |
| 含油窗 | 7,123 | 4,511（63%） |
| 含油窗:背景窗 | 1:36 | 1:32 |
| VAL 景／ROI／窗 | 37／40／8,724 | 25／26／5,850（含油窗 182） |

**ROI 規則與大小不變**
- L_MAX＝3,430 來自 GoM 景 `NOAA_Gulf_of_Mexico_High_Confidence_20190428_S2_15RYM`，它本身就在 GoM TRAIN 裡。程式有 assert，並已用 `p44_roi.components` 重算出 3,430。
- 因此 ROI 3,840、每個 ROI 的位置都和 P44 相同。

**VAL 只用 GoM**
- 取 P44 VAL 裡的 25 景 GoM，其中 16RCS 佔 16 景。
- P44 移除的 12 景洩漏景維持移除。

**設定檔**
- `p44_config.yaml` 只改 `train.seed` 和 `paths.lists`；`p51.py configs` 會檢查除這兩項外完全相同。
- 三份實際訓練用的 YAML（`$W/configs/p44_gom_S{42,43,44}.yaml`）列在 `FROZEN_LISTS.sha256` 裡（路徑 `../configs/...`），launch 時與清單一起核對。

**其他一律不變**
- 模型：Prithvi-EO-2.0 300M＋DLV3，6 波段。
- loss：Tversky 0.3／0.7；增強與 AdamW 同 P44。
- 預算：40 ve × 4,000 步，patience 8。
- VAL 運算子：步距 256、無 TTA。臺灣運算子：步距 128＋TTA、fp32。
- 程式：沿用 P44 凍結的 `train_p44.py`、`p44_common.py`，透過 `--lists` 指定清單，不改程式。

**揭露、不修的事項**
- 曝光量：總步數相同、資料少 44%，每個窗被看到的次數約為 P44 的 1.8 倍。
- wandb：run 名稱沿用凍結程式的 `P44_S{seed}`，改用 tag `P51_GoM_only` 與 `wandb_run.json` 區分。
- 來源核對：`train_p44.py` 在 train 模式仍會讀 P44 全部 359 景（TRAIN 298＋VAL 37＋臺灣 24）的 source inventory 並核 hash（`p44_sources.verify`）。這些來源檔只用於核對，非 GoM 景不會進入訓練。
- 清單的 `build_manifest.json` 記錄的是 v1 `p51.py` 的 SHA。v2 沒有改 `cmd_lists`，清單內容由 `FROZEN_LISTS.sha256` 鎖定。
- workbook：專案規則（AGENTS.md）要求接共用 Excel 入口；**使用者 2026-11-11 裁決比照 P44～P50 豁免**（「比照 P44～P50 豁免」）。P51 以 `eval_result.json`／`eval_per_scene.csv` 為準；之後若要補，會對整條 P44 流程統一補 Excel。

## 3. 執行
`launch.sh` 的流程：
1. **核對凍結**：repo 乾淨、P51 `FROZEN.sha256`、P44 `FROZEN_CODE`、P44 清單、P51 清單與三份 YAML，以及 `eval_result.json` 尚不存在。
2. **取得共用 GPU 鎖**：開鎖檔或 `flock` 失敗就直接中止（fail-closed）。
3. **依序跑 S42 → S43 → S44**，每個 seed：
   - **訓練**：完成後用 `p51.py verify` 核對。
   - **續跑**：只有通過 `verify` 的目錄才會跳過；`verify` 會核 `train_complete` ok／mode＝train、seed、完整 resolved config＝該 seed 的 YAML、lists 路徑與 `build_manifest` SHA、P44 四支訓練程式的 SHA、best.pt SHA。不通過就中止，要求把舊目錄改名成 `_failed_attempt<N>`，重跑需使用者授權。
   - **臺灣 24 張預測**（只讀影像）：完成後用 `verify --pred` 核對；既有的預測目錄也必須通過 validator 才會跳過。不通過就中止，要求把目錄改名成 `_failed_attempt<N>`，重跑需使用者授權。
4. **執行 `p51.py evaluate --allow-target-gt`**，分兩段：
   - **讀 GT 像素並計分前**，先做六組 artifact 的 image-only preflight（P51 三組＋baseline 三組）。之後只有 mask 檔 SHA 核對會開啟 GT 檔，接著才裁切 GT 計分。
     - roster：核 `analysis/taiwan_test_roster_current.json` 的 SHA（釘在 `p51_config.yaml`）和它指向的 registry SHA，景集合恰為 24 景且各一次。
     - 每個 NPZ：SHA＝manifest、scene 一致；欄位恰為 scene／origin／pred／prob_oil／valid／union；origin 為 2 個整數；四個陣列同一 2-D shape；valid／union 為 bool、pred 為 uint8；ROI 聯集內 pred 只能是油(0)／背景(1)，聯集外必須是 255；計分範圍非空且機率全為有限值。
     - 每組預測：`rois_taiwan_sha256`、operator＝P44 test（128／TTA／fp32／32），`p44_common`、`p44_roi` 的 SHA 和現行相同。
     - baseline：三份 YAML 的 SHA 釘在 `p51_config.yaml`（S42＝凍結的 `p44_config.yaml`），且把 seed 改回 42 後必須和 `p44_config.yaml` 完全相同；三個訓練目錄同樣經 `verify_train` 核對 seed、YAML、P44 清單與程式。P51 預測另核 manifest 內 `p51.py` SHA。
     - 各景的 origin，以及 valid／union 的 dtype、shape、內容 SHA，在六組之間必須完全相同。
   - **讀 GT 後**：
     - 核對兩欄的 GT 和計分像素數在六組之間相同（預期大林 849,893、其他 610,036）。
     - 先寫暫存檔再 atomic rename；JSON 以 `allow_nan=False` 輸出，NaN 改寫成 null。

產物放在 `/home/alanyh/oil_dataset/new/_p51_gom_only_roi_20261111/`。
預估時間：baseline 三次分別約 3.5 h、4.2 h、1.2 h（早停），VAL 變小會稍快；每次預測約 10 分鐘。三個 seed 合計約 6–17 h。

**已驗證**（除了 evaluate 路徑測試會讀 baseline 的臺灣 GT，其他都不讀 GT）：
- smoke：`train_p44.py smoke --lists <P51>`，64 窗、2 景 GoM VAL、5 步，ok。
- 負向測試全部正確擋下：
  - baseline 目錄配 P51 清單／YAML；
  - seed 錯；
  - YAML 錯；
  - 預測的 checkpoint SHA 錯；
  - operator 錯；
  - 訓練目錄不存在（`verify` 回傳 exit 1）。
  NaN→null 也已測過。
- evaluate 正向測試：用 baseline 的三組預測代替 P51，六組都通過 preflight。
  - 大林 0.342／0.427／0.007，其他 0.267／0.280／0.089，和先前手算一致。
  - S44 被標為塌縮。
  - 分母：大林 849,893／29,491,200；其他 610,036／340,016,658。

## 4. 報告內容
依 2026-11-11 起的呈現規則，臺灣只分兩欄：**大林**（中油大林 2 張）、**其他 22 張**。每欄在 run 內做像素加總，列出：
- 逐 seed 的 Oil IoU、P、R、F1、BG IoU、mIoU、pred/gt（含 TP／FP／FN）；
- 三 seed 平均 ± SD（全部指標，`summary`）；
- 不計塌縮 run 的平均 ± SD，標為 sensitivity，只作描述（`sensitivity_excluding_collapsed`）；欄位固定，全部塌縮時每個指標為 mean／sd＝null、n＝0；
- 逐 seed 配對 Δ（P51 S42−baseline S42，依此類推）；
- 每個 run 的 best ve 和 VAL 值（P51 用 GoM 25 景、baseline 用 37 景，不可直接比較）；
- 塌縮旗標：臺灣 24 張合計 pred/gt < 0.2，或只輸出單一類別。

對照組是自然比例 baseline 的三個 seed：P44 S42、P48 S43、P48 S44。

## 5. 判讀（讀 GT 前寫定；啟發式規則，不是顯著性檢定）
大林和其他 22 張**分別**判讀：
- Δ＝mean(P51) − mean(baseline)，各取 3 個 seed 的 Oil IoU（含塌縮 run）。
- se＝√(sd₁²/3 + sd₀²/3)，採獨立樣本形式，現在凍結，看到結果後不換成配對形式。

| 條件 | 標籤 |
|---|---|
| Δ ≥ 0.03 且 Δ > 2·se | `GoM_higher` |
| Δ ≤ −0.03 且 −Δ > 2·se | `GoM_lower` |
| 其餘 | `no_decisive_difference`（依本規則未達方向性門檻） |

- 這些標籤是操作性的，不代表統計顯著。n＝3 下 2·se 不是可靠的 95% 檢定，兩欄也沒有做多重比較校正。
- 主要平均包含塌縮的 run；「不計塌縮」只作 sensitivity 描述，不改標籤。
- 逐 seed 配對 Δ 另外列出，但不參與判讀。

## 6. 限制
- 只有 3 個 seed；baseline 的 se 被 S44 塌縮撐大，在這個門檻下不容易判出差異。
- 這是整體資料制度的比較：TRAIN、VAL 選模、曝光量、正負窗比例同時改變，不能歸因到單一因素。
- 臺灣 ROI 由 GT 決定位置，ROI 外的誤報不計分；這是重複使用的基準，不是盲測。

## 7. Codex 審查紀錄
- R1（session `01a1c83d-e1f4-7c73-bdd8-25136f43164f`，gpt-5.6-sol）：REVISE，BLOCKING 7 項。
  1. GPU 鎖 fail-closed；
  2. completed train 的 provenance validator；
  3. 凍結三份 YAML；
  4. 釘住 Taiwan roster；
  5. evaluate 的嚴格 preflight 與共同計分範圍；
  6. summaries／sensitivity／NaN；
  7. 判讀標籤與 SE 形式。

  已逐項修正如上；non-blocking 項（問題重述、atomic 寫入、source inventory 說明、workbook hook）也已處理或說明。原文在 `consensus/`。
- R2：REVISE，BLOCKING 4 項。
  1. NPZ schema 移到讀 GT 前的 preflight；
  2. baseline YAML 的信任錨；
  3. 全塌縮時的 sensitivity 欄位；
  4. workbook。

  前三項已修正並加測試；第 4 項由使用者裁決豁免。
- R3：**ACCEPT，BLOCKING＝[]**，CONFIDENCE 0.98。可選強化（未做）：prob_oil 必須是浮點數；核對 manifest 的 n_scenes／artifact／seed；另外釘住 baseline 的 best_sha256（目前已透過 train_complete 和 prediction manifest 的 checkpoint SHA 間接核對）。
