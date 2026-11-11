---
date: 2026-11-04
type: experiment
stage: unsupervised
status: 完成：not improved（臺灣24 0.479 vs T3 0.491，單 seed）
tags: [p42, lfmda, stage2, self-training, sam, lrh, segformer-b2, prithvi-teacher, execution-record, wandb]
---

# P42 LFMDA stage 2（自訓練＋SAM＋LRH，接在 P41 T3 之後）執行紀錄：完成，not improved（臺灣24 0.479 vs T3 0.491，單 seed，描述性）

> 本頁為執行紀錄。事前凍結卡見 [[20261104_P42_LFMDA_stage2_from_P41_T3設計]]（`.agents/decisions/`，FROZEN v1，SHA-256 `2fde70f6…`，逐位元複製自 repo `P42_CARD_DRAFT.md`，**本頁未修改該卡**）；上一階段見 [[20261104_P41_LFMDA_PCD_固定P15teacher_執行紀錄]]，原 stage 2 設定見 [[20261103_P39_LFMDA_stage2_SAM_LRH設計]]。設計、判準與評估規則不在此重抄。訓練已完成並封存（見文末「2026-11-04 21:44 完成」）；2026-11-05 使用者授權讀臺灣 GT 後完成評估（見文末「2026-11-05 臺灣 24 張 GT 評估」）：臺灣 24 張 pooled OilIoU 0.479，低於起點 T3 的 0.491（−0.012），not improved。

## 1. 實驗背景

P41（固定 P15 Prithvi＋DeepLabV3 teacher 的 PCD warmup）只走了 LFMDA stage 1，臺灣 24 張 T3 student final 0.491。2026-11-04 使用者裁決：新線只走了 stage 1，**先照原架構的 SAM（P39 設定）跑 stage 2 一次**，之後再討論 SAM 是否適合本資料（候選：DADS-SAM）。

已知風險（見 [[20261103_P39_LFMDA_stage2_SAM_LRH_結果]]）：P39 舊線 stage 2 失敗——SAM 多數窗只切出 1 區或整窗一塊，LRH 把油擦成背景。本次刻意不修 SAM，預期可能重演。

## 2. 方法架構 / Pipeline 流程

起點 P41 T3_S2333 `student_final` ＋ `warmup_prototypes`（硬 pin SHA，strict 載入）→ 固定 P15 S42 Prithvi＋DeepLabV3 當 teacher（eval、不更新，無 teacher 訓練）→ P39 封存 ViT-H SAM 區域（manifest `05fa132b…`，不重跑）→ 自訓練＋LRH，其餘照 P39 凍結卡 v4。流程：訓練 → 2 組預測 → image-only seal。細節見凍結卡 §2。

## 3. 關鍵參數

| 項目 | 內容 |
|---|---|
| 起點 | P41 T3_S2333 student_final ＋ warmup_prototypes（硬 pin SHA） |
| teacher | 固定 P15 GoM-only Original S42 Prithvi＋DeepLabV3（無 teacher 訓練） |
| SAM 區域 | P39 封存 ViT-H，manifest `05fa132b…`，不重跑 |
| 其餘設定 | 照 P39 凍結卡 v4 |
| seed | 單一 S2333 |
| Codex 協商 | session `01a1a6ab-daa4-7f93-80ef-0afc004b39c9`：R1 REVISE（pin 起點／SAM manifest／P39 程式）→ R2 ACCEPT |
| repo commit | `c962d03` |
| launch | 2026-11-04 21:14（順序：訓練→2 組預測→image-only seal），預計約 40 分 |
| wandb | run `P42_S2333`：https://wandb.ai/tree8454-national-central-university/oil-gt-expand/runs/xwt27bfz |

## 4. 量化結果

臺灣 24 張 GT 評估結果見文末「2026-11-05 臺灣 24 張 GT 評估」（P42 final 0.4786，T3 起點 0.4906，−0.012）；GoM 驗證與機制見文末「2026-11-04 21:44 完成」。封存前 Claude 未看臺灣預測與 wandb `mechanism/`。

對照（P41 封存結果，臺灣 24 張 pooled OilIoU）：

| 項目 | OilIoU |
|---|---|
| T3 final（P42 起點） | 0.491 |
| C6（P40，無 teacher） | 0.463 |
| P39 舊線 stage 2 | 失敗（SAM 1 區／整窗 → LRH 擦油） |
| **P42 final（=best，step 6000）** | **0.479**（2026-11-05 評估，見文末） |

## 5. 觀察與結論

P42 臺灣 24 張 pooled OilIoU 0.479，低於起點 T3 final 0.491（−0.012）、高於 C6 final 0.463（+0.016）；FP 增加約 49k、TP 幾乎不變，24 張中 20 張下降。與已知 SAM/LRH failure mode 相容，stage 2 未帶來增益，判定 not improved（單 seed，描述性）。詳見文末「2026-11-05 臺灣 24 張 GT 評估」。

## 6. 後續改進方向

- [x] 訓練跑完、預測與 image-only seal 完成（2026-11-04 21:44）
- [x] 使用者另授權後讀臺灣 GT，跑 `evaluate_p42.py`（24 張主報）（2026-11-05 完成）
- [ ] 討論 SAM 替代：以 DADS-SAM encoder＋原 SAM decoder 點網格自動切區塊（A 方案），先做不讀 GT 的區域統計；需另立卡＋使用者授權

## 7. 實驗檔案位置

- 程式與凍結卡草稿：`/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p42_prithvi_stage2_sam_lrh_20261104/`
- 凍結卡：`/home/alanyh/obsidian-vault/.agents/decisions/20261104_P42_LFMDA_stage2_from_P41_T3設計.md`

## 2026-11-04 21:44 完成

訓練、預測與 image-only seal 全部完成；**尚未讀臺灣 GT**，以下皆為 GoM 驗證與封存後揭露的機制數字（未使用 GT）。

### 執行狀況

- `launch.log` 全部 EXIT 0：訓練 21:14:57–21:44:12（約 29 分），預測 final／best 各約 15 秒，image-only seal 21:44:43。
- `train_complete`：6000 步完成，`teacher_unchanged=true`，target 抽樣 24000 次，翻轉計數 [11953, 12009, 11897]。
- step 0 與 step 3000 的偽標籤 probs 張量完全相同（teacher 固定，符合預期）；兩個檔的 SHA 不同只是因為 step 欄位不同。

### GoM 驗證（Oil IoU／mIoU）

| step | Oil IoU | mIoU |
|---|---|---|
| 1 | 0.345 | 0.654 |
| 1000 | 0.261 | 0.611 |
| 2000 | 0.386 | 0.674 |
| 3000 | 0.378 | 0.670 |
| 4000 | 0.358 | 0.660 |
| 5000 | 0.363 | 0.663 |
| 6000 | 0.385 | 0.674（P 0.701／R 0.461） |

best＝final＝step 6000。參考：P41 T3 step 10000 是 0.353。

### 機制（封存後揭露，沒有用到 GT）

每 1000 步的量級：

| 項目 | 像素數 |
|---|---|
| 選中的油 | 約 35.4M px |
| LRH 油→背景 | 約 1.53–1.58M（約 4.4%） |
| 忽略→背景 | 約 1.1–1.25M |
| 忽略→油 | 約 7–8k |
| 背景→油 | 約 2k |

LRH 幾乎只把像素往背景推。teacher 偽標籤油機率 >0.5 的比例是 0.159。

### Seal

- 檔案：`/home/alanyh/oil_dataset/new/_p42_prithvi_stage2_20261104/eval_seal.json`
- SHA-256：`6cc54123d5bb9fc7294940b324c47d056b1547f07aa0352c309c78ddc44c7722`（已用 `sha256sum` 核對檔案）
- `target_mask_values_read=false`

### 下一步

等使用者授權讀臺灣 24 張 GT：`evaluate_p42.py evaluate --seal-sha256 <sha> --allow-target-gt`。授權前不看臺灣預測結果。

## 2026-11-05 臺灣 24 張 GT 評估

使用者 2026-11-05 授權讀 GT（原話「讀GT吧」）。結果檔 `/home/alanyh/oil_dataset/new/_p42_prithvi_stage2_20261104/eval_result.json`，SHA-256 `728f9e011695d1e4cab033f1c885426979b1fa3dc1fc384993baae20c99e1a97`；對應 seal SHA-256 `6cc54123d5bb9fc7294940b324c47d056b1547f07aa0352c309c78ddc44c7722`（見上節）。單 seed S2333、描述性。

### 量化結果

| 項目 | 臺灣24 pooled OilIoU | 19 張 | 新增5 | P | R |
|---|---|---|---|---|---|
| **P42 final（=best，同 step 6000）** | **0.4786** | 0.4785 | 0.4812 | 0.671 | 0.626 |
| P41 T3 final（起點） | 0.4906 | 0.4885 | 0.5375 | 0.695 | 0.625 |
| P40 C6 final | 0.4628 | 0.4607 | 0.4988 | 0.760 | 0.542 |
| P40 C6 best | 0.4799 | — | — | — | — |

REF P15 S42／S43／S44（臺灣24）：0.489／0.407／0.393。

- P42 − T3 final = −0.012；P42 − C6 final = +0.016。
- 混淆矩陣（24 張加總）：

| 項目 | P42 | T3 final | 差 |
|---|---|---|---|
| TP | 913,417 | 912,235 | 幾乎不變 |
| FP | 448,464 | 399,630 | +49k |
| FN | 546,512 | 547,694 | 幾乎不變 |

### 逐張

24 張中 20 張下降、3 張上升（>0.01）。

- 降最多：天使輪 20230918 L9 QRL −0.127、QRK −0.114；20230801 L9 QRK −0.126。原因是細線斷開、FN 增加。
- 上升：20260522 L8 51QTD +0.033、51QUD +0.029；馬祖 20220622 +0.068。20260522 兩張為大張，像素權重大，所以 pooled 只降 0.012。

### 判讀

- 與已知 SAM/LRH failure mode 相容：LRH 幾乎只往背景推（約 4.4% 選中油被擦），內部補油幾乎沒有。
- 大油污（20260522）內部仍有未填滿。
- 單 seed，不能拆 stage2／SAM／LRH 個別因果。

### 疊圖

`/home/alanyh/oil_dataset/new/_p42_prithvi_stage2_20261104/figures_final/`；renderer `GT_expand/analysis/p42_prithvi_stage2_sam_lrh_20261104/render_p42.py`（已 local commit）。

### 結論與下一步

- LFMDA 新線（P41→P42）兩階段皆已走完一次。
- 下一步：討論 SAM 替代，以 DADS-SAM encoder＋原 SAM decoder 點網格自動切區塊（A 方案），先做不讀 GT 的區域統計；需另立卡＋使用者授權。

## 相關頁面

- [[20261104_P42_LFMDA_stage2_from_P41_T3設計]]
- [[20261104_P41_LFMDA_PCD_固定P15teacher_執行紀錄]]
- [[20261103_P39_LFMDA_stage2_SAM_LRH設計]]
- [[20261103_P39_LFMDA_stage2_SAM_LRH_結果]]
