# 2026-09-08 P11 GeoAgent attempt1 非執行判定與 attempt2 啟動

議題：P11 GeoAgent Track A（fold1）attempt1 的 Stage-1 訓練完成後，判定為 `non-executed`（非 scientific null），定位根因、修復後於 2026-09-08 經使用者授權啟動 attempt2。

工作樹：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand_geoagent_dev`。

**狀態：attempt1 判定為 non-executed，artifacts 原地保留未刪；修復（val 組成 + checkpoint role 標記）已完成；attempt2 於 2026-09-08 13:43 啟動 Stage-1（PID 3020744），epoch 1 完成，cuBLAS 警告 0 次；此前 13:25 的首次啟動（PID 3014469）因漏帶決定性環境變數已中止。**

## 1. 根因：val dataset 漏 BG、loss 口徑誤判

`main/geoagent/workflow.py` 的 Stage-1 val dataset 只放 GT patch、漏了 BG patch，與正典（`deeplab_adapter.py:1211-1214` 的 `bg_scene_filter=val_scenes`）不一致，是單行偏差。

此處先前錯過三次、最後由 Codex 糾正的損失口徑：`TverskyLoss`（`deeplab_adapter.py:61`）以 `targets != ignore_index(255)` 判定有效像素，**不是** `tested_mask`。BG patch 從不讀 mask，`data.py:257-262` 回傳 `np.ones((256,256))` 當 target、`np.zeros(...)` 當 tested，故每張 BG patch 貢獻全部 65,536 個 loss-valid 像素。

修正後查得的數字：

- 訓練端 loss-valid 油比：**5.2479%**
- attempt1 的 val 端（漏 BG）：**83.9259%**
- 兩端相差：**15.99×**
- 補回 BG 後的 val 端：**13.1139%**
- 殘差：**2.499×**

Tversky 是比值型損失、無絕對尺度，油密度直接決定最佳操作點；β=0.7 在高油密度的 val 端獎勵 recall，與訓練端的 precision 取向相反，early stopping 因此凍結了過度預測的 epoch 1。

**歸因用詞的上限**：只能說 model selection 是 **strongly implicated**，不能說「已證明」——epoch 26 的權重從未存檔，且 gamma 量級與 `architecture_null_diagnostic`（`evaluation.py:132-146`，計算 `mean|P_context − P_local|`）是完全不同的量，不可互相取代作為證據。

## 2. 修法（僅二項，研究設計零變動）

1. val_es 依同規則補回 BG（`bg_filter: NOAA319_INTERSECT_SPLIT_SCENES`），inner_train 與 val_es 各自依自身場景取 BG，per-scene quota 10。
2. checkpoint 加 `selection_role`（`early_stop_best` / `diagnostic_last`），雙層 fail-closed 拒絕 diagnostic checkpoint 進入 cache/gate 階段。

**未動的部分**：四臂（fixed_local/fixed_3x/fixed_5x/full_information_router）、三視野（256²/768²→256/1280²→256）、`F_local + γ_k ⊙ Ctx_aligned` 的 1024-channel zero-init gamma、M1/M2/S、G1≥0.05、G2≥0.02、architecture_null≥1e-3、split（1234/494/772，319 景）、Tversky α=0.3 β=0.7、ASPP (6,12,18)、seed 42、epochs 300／patience 25。

## 3. Codex 三輪協商中抓到的實質錯誤

- 我最初用 mask 標註面積（95.97%）計算 BG 油比 —— BG 根本不讀 mask，口徑錯。
- `materialize --verify` 對缺檔會假 PASS：`verify()` 比對 live vs cached，但 `data.py:306` 在 cache miss 時靜默回退讀 live VRT，於是缺檔變成 live-vs-live 恆真。已改為先做全清單完整性檢查、再分層抽樣。
- gate containment 的 `.parent.parent` bug：gate JSON 就放在 attempt root 內，`.parent` 才是 root；`.parent.parent` 會**拒絕所有合法的 attempt2 gate**。我先前的測試只測 helper、沒測 runner 內聯檢查，所以漏掉。
- 我宣稱「回歸正典」，被糾正為過度延伸：正典以 `val_mIoU` 選模並把 255 映為背景，P11 用 pooled soft Tversky 且保留 255 為 ignore。只能說「恢復正典的 val dataset 組成」。
- 我宣稱 commit attempt2 會「順帶補上 attempt1 的 provenance 缺口」，被糾正：attempt1 的原始碼位元已不存在，無法回溯。

## 4. 正對照：修改未動訓練路徑

attempt1 用 `steps=6`，我最初測試用 `steps=16`，導致 parity 的 train_loss 看似不同（0.8691459894 vs 0.8675667271）。以 attempt2 重跑 `steps=6`：train_loss **完全相同 0.8691459894**，340 個 tensor `max_abs_delta = 0.0`。

範圍限制：此對照僅涵蓋 6 個 optimizer step，不涵蓋完整 epoch 或 300 epoch 尺度。

## 5. 決定性缺陷（本次新發現並修復）：漏帶 CUBLAS_WORKSPACE_CONFIG

attempt2 首次啟動（13:25，PID 3014469）**漏帶 `CUBLAS_WORKSPACE_CONFIG=:4096:8`**。正典要求它由啟動指令的環境變數帶入（`main_runner.py:66`），而 `runner.py` 用 `torch.use_deterministic_algorithms(True, warn_only=True)`，非決定性 cuBLAS op 會照跑不報錯。此為該缺陷第二次咬到 P11。

實測佐證：兩次啟動 epoch 1 的 `train` 同為 0.698744，但 `val` 為 **0.410345 vs 0.410343**（差 2e-6）。

處置：該次執行於 epoch 1 後中止，未產生任何 checkpoint 或 metrics，log 保留為 `ABORTED_no_cublas_config_stage1_20260908_132534.log`；在 `runner.set_global_seed` 加入 fail-closed 檢查（缺席或值不符即 raise）。

另記：原本想用 `--phase router` 當 preflight 驗此 guard，**驗不到**——router 的 artifact 檢查排在 `set_global_seed` 之前，需直接呼叫 `set_global_seed` 才測得到。

## 6. 已登記但刻意不修的限制（寫入預註冊）

- **per-size-stratum BG quota 不均**：以 P11 的 size 分層定義（scene-level `oil_px` 取自 `manifest_445.csv`，tertiles `(3200, 14147]`，凍結於 `config.py:33`）計，tiny 級距 BG:GT 像素比 129.27 vs large 4.92，跨級距 exposure 差約 **37×**。這是設計選擇，且與既有的 P8 像素口徑工作線重疊，同一輪內修會混淆歸因。
- **2.499× 殘差**：集中在 large 級距——val_es 的 13 個 large 景有 416 張 GT patch（32/景），訓練端為 16.3/景。
- **全 255 語料 patch**：train 14.5% / val 25.2% / test 15.0%，是既有語料性質，在 `ignore_index` 下對訓練為 no-op，影響範圍遠超 P11。
- ES 指標維持 pooled soft Tversky（未改為 S）；是否改為 S 待 fold1 結束後協商。
- `TRAINING_BUDGET_CENSORED` 規則：300/25 的上限保留，因為調低上限只在模型仍在進步時生效，那正是會污染負面結論的預算截斷情形。

## 7. 版控與授權

- `381541c` 契約凍結（38 檔；`result-seg/` 16GB 已 gitignore）。
- `54356c9` 使用者授權（`status: PREREG_FROZEN_USER_AUTHORIZED` / `run: true` / `launch_token: USER_EXPLICITLY_AUTHORIZED_P11_LAUNCH`）。
- `3ab43de` CUBLAS guard。
- 分支 `p11-geoagent-attempt2`（先前 HEAD 為 detached，已開分支避免 dangling commit）。
- attempt1 artifacts 原地保留未刪，見 `result-seg/P11_geoagent_track_a/ATTEMPT1_NON_EXECUTED.md`；attempt2 使用獨立 root `result-seg/P11_geoagent_track_a_attempt2`。
- 預註冊 `analysis/p11_geoagent_rl/preregistration.md` 由 335 → 479 行。

## 8. 現況

2026-09-08 13:43 啟動 attempt2 Stage-1（PID 3020744），epoch 1 `train 0.698744 / val 0.410343`，cuBLAS 警告 0 次。7.2 min/epoch，patience 25，100 epoch 約 12 小時、跑滿 300 約 36 小時。

## 9. 未決

- folds 2/3 的 confirmatory 門檻須在跑之前登記。
- 本次算出的像素油背比 1:17.34 是否與既有 `bg_quota_policy_px.py` 的口徑（記錄為「1:9.85 硬地板」）一致，尚未查核。
- 本地 git commit 非不可竄改的外部時間戳。

## 10. 與前案的關係

本案是 [[20260902_GeoAgent_A2C尺度配置實驗設計與建碼]] 完成建碼後的第一次 Stage-1 真實執行嘗試；attempt1 的 non-executed 判定與 attempt2 的修復、啟動記錄於此，不回頭改動 20260902 決議中已凍結的方法設計（四臂、三視野、gated add、split、loss、ASPP、seed、epoch/patience 等）。
