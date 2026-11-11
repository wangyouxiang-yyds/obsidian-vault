# 2026-11-01 P38 事前卡協商紀錄（LFMDA warmup + PCD，Claude×Codex）

> 本檔是**協商紀錄**，不是凍結事前卡。凍結卡之後另建 `20261101_P38_LFMDA_warmupPCD設計.md`（填入程式 SHA 後），本次未建。

## 議題
P38 事前卡：LFMDA 作者 warmup 階段（DINOv2 ViT-L/14 教師＋PCD，不開 align-domain、不含自訓練／SAM）對 P37 SegFormer-B2 source-only。target 無標註影像＝臺灣原 19 景 270 測試窗（transductive）；新增五景只測試；兩組各 3 seed {2333,2334,2335}，P37 S2333 沿用既有。使用者 2026-11-01 已定上述三項範圍。

## 參與與 provenance
- 主持：Claude（Opus 5.5）
- 對手：Codex gpt-5.6-sol、reasoning high（requested=actual），codex-cli 0.166.0
- Codex session：`01a1971c-846b-75d3-8552-182635b6959c`，四輪同 session resume
- Codex 自述四輪皆未讀本機檔，全文內嵌 prompt。

## 逐輪
| 輪 | Codex 結論 | BLOCKING | 要點 |
|---|---|---|---|
| R1 | ACCEPT | [] | 盲提案；新增 H2 GoM 保留、H4 五景探索、近塌縮、機制紀錄、sign-flip、工程無效清單 |
| R2 | REVISE | [1,2,3,4] | 三分法未互斥；洩漏條件與既有 P37 知識矛盾；secondary 封存不足；教師輸入轉換未寫死 |
| R3 | REVISE | [1,2,3] | 執行期查看無標註 target 輸出＝sequential target tuning；target sampler／增強未寫死；修正後重跑混版本 |
| R4 | ACCEPT | [] | 收斂 |

blocking 數 0→4→3→0，未達 5 回合檢查點。

## 最終決議要點
- 主指標＝學生 step10000 final 臺灣 19 景 coverage-union pooled OilIoU（`_kind` patch）。
- Δ̄＝T 三 run 平均 − C 三 run 平均；s_p＝sqrt((2s_T²+2s_C²)/4)。
- 否定先判：≥2 個 T run 近塌縮（coverage-union 去重後預測油比例 <1% 或 >99%）或 IoU ≤ floor 0.1582；或 Δ̄≤0 且 ≥2 個 T run ≤ C 平均。
- 支持：非否定，且 Δ̄>0.03、Δ̄>s_p、≥2 個 T run > C 平均。其餘＝不可判定。
- H2 GoM 保留：Δ̄_GoM ≥ −0.03。整體四分支。
- 執行期盲化與固定 run 順序：C2334→C2335→T2333→T2334→T2335；不按 seed 編號配對。
- 非單一變因：只稱「本案 GoM-normalized LFMDA warmup 實作」套件效果，不得稱 PCD 單獨有效或 zero-shot。

## 少數意見
無未解決分歧。Codex 最強論點（RESTATE）：依序查看無標註 target 預測或偽標籤摘要仍會形成 target tuning；已以 write-only、禁止依機制紀錄改動、固定順序、封存後才揭露近塌縮處理。

## 殘餘風險
- 三 seed 對約 0.09 的重跑波動偏少。
- 多元套件不可歸因到 PCD 單項。

## 後續驗證點
- 程式實作後 smoke：作者 warmup 語義核對、np.rint bit-identical、原型有效像素與範數、PCD 梯度到 mapper、教師 backbone 無梯度、target 不觸及 mask。
- smoke 若發現語義與卡不符 → 回協商。
- 填入程式 SHA 後才凍結卡；launch 需使用者授權。

## 試行評估附錄
- 首輪分歧點：seed 配對方式與門檻互斥。
- 收斂輪數：4。
- 實際返工：卡片三次修訂，無程式返工。

## 正本
GT_expand `analysis/p38_lfmda_warmup_pcd_20261101/P38_CARD_DRAFT.md`（v5，commit 925dc46，本機未 push）；四輪 prompt／回覆存同目錄 `consensus/`。

## 2026-11-02 v6→v7：作者實際 warmup 設定（R5）

- 實作前逐行核對作者 LFMDA 原碼（commit 88d9f2c），觸發卡上反證條件「作者 warmup 語義與卡不符 → 回協商」。
- 作者執行腳本 `runs/lfmda/2vaihingen_segformer.sh` 用 config `st.lfmda.2vaihingen_segformer`，其 TARGET_DATA_CONFIG 覆寫 `configs/ToVaihingen.py`（R3 引錯檔）：target batch 4、HFlip/VFlip/Rot90 各 0.5 獨立、正規化後 clamp(max=1.0)。
- `tools/train_warmup.py`：poly 分母 1.5×STAGE1_STEPS（第 10,000 步 lr≈0.372×base）、沒有梯度裁剪。
- 教師 256→280 與作者 1.1× 比例相符（作者 get_model 寫死 512→560）。
- 使用者 2026-11-02 裁決：T 組改照作者實際設定；C 組維持 P37（poly 分母 10,001、clip 32）；GoM mean/std 的既有差異保留。
- Codex（gpt-5.6-sol/high，同 session 01a1971c-846b-75d3-8552-182635b6959c）R5：ACCEPT／BLOCKING=[]。非 blocking 全採：
  - 結論對稱限制：不得推「PCD 單獨有效」，也不得推「PCD 單獨無效」。
  - clamp 為單側上限、train–inference mismatch；GoM 尺度約對應 DN/10000 > R .3611／G .3521／B .3335，加 write-only 飽和紀錄。
  - 每 seed 原型檔獨立封存。
  - torch_scatter stub fail-fast。
  - read_sup I/O 由 smoke 證明。
  - DownscaleLabel 稀疏不得作改門檻理由。
- 已知機制風險：學生 1/32 下 GoM train 1,188 含油窗只有 518 個留下 ≥1 油格（5,461 油格對 162,415 背景格）。
- 版本：repo 卡 v7 commit 8e1fa87；程式 commit 176c409（warmup.py SHA 364158a8…、sourceonly_seed.py b707d58e…、predict.py 0e1df608…、check_inputs.py 4639b279…）。
- 驗證：CPU 檢查與 GPU 2 步 smoke（T S2333、C S2334）通過，未讀臺灣 GT。
- 狀態：正式凍結卡（`.agents/decisions/20261101_P38_LFMDA_warmupPCD設計.md`）尚未建立，launch 尚未授權。
