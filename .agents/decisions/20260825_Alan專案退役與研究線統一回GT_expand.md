# 2026-08-25 Alan 專案退役與研究線統一回 GT_expand

## 裁決

2026-08-25 使用者裁決：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_Alan` 全線退役，所有油汙研究工作統一回 `OIL_PROJECT_MutiBand_GT_expand`。使用者原話理由：「專案設太多飛來飛去的 有點麻煩」「不如我這邊先統一下還是繼續在GT_expand上做工」。

## 本裁決撤銷什麼

撤銷 `_Alan` 自己 2026-08-12 的重新定位（該 repo 當時把「大圖 full-scene 主線」退役、改成 patch-based overseas-source → Taiwan-target transfer 主專案）。`_Alan/README.md`、`CLAUDE.md`、`AGENTS.md` 內所有「Alan is the main project」敘述即日失效。

## 明確放棄（使用者原話「那個就別理了」）

- `alan_patch_source_sapp_v0`：2026-08-25 三折剛跑完，Prithvi-EO-V2-300M + SAPP（PPM bins 1/2/3/6、H/V strip、SE reduction16），source patch test Oil IoU = 0.3840 / 0.3147 / 0.5026，fold-macro mean±sample-SD = 0.4004±0.0950，mIoU 0.6614±0.0381
- 其成對 ASPP 對照 `alan_patch_source_erm_v0.yaml`：**不啟動**。SAPP-vs-ASPP 是擱置，不是待辦
- 海外 source → 台灣 target 的 transfer 研究軸（source_only_zero_shot / UDA / few_shot 三 protocol）一併擱置

## 不刪任何檔

`_Alan/` 全樹原地保留為 frozen legacy，包含 `docs/transfer_evaluation_contract_v0.md`、`docs/project_architecture_v2.md`、`configs/protocols/`、`manifests/splits/source_event_group_holdout/v0/`、`artifacts/experiments/alan_patch_source_sapp_v0/`。已在 `_Alan/CLAUDE.md` 與 `AGENTS.md` 頂端加退役橫幅（原檔備份為 `*.bak-20260825b`）。

## 必須帶走、不可隨退役遺失的一項知識

`_Alan` 2026-08-12 的「比例口徑修正」對 GT_expand 現行 BG 配額工作同樣成立：**`patch 張數 × 65,536` 是 optimizer exposure，重疊 patch 會重複累計同一像素，不得稱為唯一資料像素比例**。GT patch 是 GT bbox 中心 ±128、大 bbox 用 256-grid tile 覆蓋，重疊必然存在。GT_expand 現有數字（總 patch 6,403、總像素 419,627,008、油像素 17,470,827、整體油:背景 1:23.02、bin 別 1:277.2 / 1:70.9 / 1:13.2）**全部是 exposure 口徑，報告中須標為訓練曝光量**；要報唯一像素比例須另算 patch 空間聯集，作法參照 `_Alan/analysis/data_balance/compute_dataset_balance.py`。

## 影響與未決

- GT_expand 內既有的 P5 台灣 holdout 結論不受本裁決影響
- `_Alan/CLAUDE.md` 的那組 hard rules（Taiwan sealed-test、UDA/few-shot 命名、incident/acquisition 分割…）隨退役失效，不再是跨 repo 通則
- 使用者表示 BG 配額這條線「等這個部分用完我們再去討論一下實驗的細節」，即實驗設計細節暫緩

## 根因教訓

`_Alan` 在 institution harness 與 Claude memory 裡**完全隱形**：`institution/README.md`（7/18 後未動）與 `40_harness_continuity.md` 零次提及 Alan，Claude memory 15 條也零條提及。後果是同一個「油汙:背景比例」問題在兩個 repo 各算一次，且 GT_expand 這邊用了錯誤的 exposure 口徑而不自知。**制度含意：新增長期存在的 repo 必須同步進調度層（institution 權威地圖／memory 索引），否則會產生看不見的重複工作與口徑分歧。**

## 正本交叉索引

- institution：`/home/alanyh/.agents/institution/research/oil_spill_project_status.md` §⑦ 2026-08-25 條（已寫入，備份 `.bak-20260825b`）
- Claude memory：`project_alan_retired_consolidate_gt_expand`、`project_bg_quota_ratio_workline`（已寫入）
