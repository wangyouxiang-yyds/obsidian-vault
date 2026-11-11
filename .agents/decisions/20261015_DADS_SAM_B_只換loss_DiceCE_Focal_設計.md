本檔為開跑前凍結的事前紀錄。正本：GT_expand repo `analysis/dads_sam_gomonly_20261005/B_DICECE_FOCAL.md`（commit `7eb2c3b`，在這組任何結果出現前寫成；事後不改）；以下正文逐字取自該檔。

**實驗編號**：無 P 編號　**凍結日期**：2026-10-15 13:35 Asia/Taipei
**事後補寫？**：否
**結案時**：第 7–9 步另存 `wiki/experiments/`。

---

# DADS-SAM B 只換 loss（0.7 DiceCE ＋ 0.3 Focal）— seed 42、43、44

日期：2026-10-15 Asia/Taipei。使用者先問：「那我想問問 這會不會是跟loss的有關? 或是batch size?」；我提議只把 B 的 loss 換成公開碼的 loss、其他不動、跑三個 seed，使用者確認：「你是說我的baseline 設定 把tversky 換成 原架構的loss?」→「好吧，那就幫我安排下去 跑這個實驗」。
協商狀態：未經 Claude × Codex 協商。本檔在這組任何結果出現前寫成；事後不改。B 與「公開碼訓練設定」兩組的結果都已看過。

## 目的與問題

公開碼訓練設定（`B_PUBLIC_RECIPE.md`、`B_PUBLIC_RECIPE_SEEDS.md`）三個 seed 的臺灣像素加總 Oil IoU（0.3043 ± 0.1071）都低於 B（0.4495 ± 0.0178）。那兩組同時差了九項訓練設定。本組只換 loss 一項，回答：在 B 的其他訓練設定下，把 Tversky 換成 0.7 DiceCE ＋ 0.3 Focal，結果會不會變低？

## 做什麼

`main/experiments_DADS_SAM_gomonly_B_dicece_focal.yaml` 的三個 block 是正式 B block（`MS6_DADS_SAM_B_public6_GoM_R1`）的複本，只改 loss、seed、實驗名、臺灣測試名，另加一句 `wandb.notes`（只影響 wandb 網頁）。`build_B_dicece_focal.py` 逐鍵比對：seed 42 對正式 B，seed 43／44 也對 B 的 seed 擴充 block。

| 項目 | B（現有三個 seed） | 本組 |
|---|---|---|
| Loss | Tversky（α 0.3、β 0.7） | **0.7 × DiceCE ＋ 0.3 × Focal（γ 2）** |
| 其餘 | AdamW lr 5e-5 cosine、batch 16、最多 300 epoch／patience 50、取驗證 mIoU 最高的 epoch、資料增強、clipping 1.0、fp16 混合精度、固定整張圖 box prompt | 同左 |

Loss 用 `B_PUBLIC_RECIPE.md` 那次加的 `dataset.loss.type: dice_ce_focal`（與 monai 1.3.2 比對一致），`train_recipe` 不設。自 B 的 seed 擴充以來 `main/` 只多了公開碼設定選項（未設時不影響）與 wandb 記錄。

## 開跑前的檢查

`smoke_B_dicece_focal.py`（seed 42 block，GoM 訓練 171 步＝batch 16 一個 epoch，加完整 GoM 驗證，fp16 混合精度，不讀臺灣資料，wandb 關閉）：PASS。loss 皆為有限值；顯存峰值 17.1 GiB（與 B 的 smoke 相同）；訓練加驗證 5.3 分鐘。171 步內 cosine 已降到最低學習率，驗證 Oil IoU 0.000（全部判成背景），這麼早不足以判斷學不學得起來。

## 怎麼報告（事前寫定）

主要看 GoM 驗證，臺灣照例報告但不拿來選 loss。

- **GoM 驗證（主要）**：每個 seed 取驗證 mIoU 最高的 epoch（即拿去測臺灣的權重），報 val mIoU 與 val Oil IoU，三 seed 平均 ± 樣本標準差（ddof=1）。對照（同一選法）：B 0.7394／0.7468／0.7247（Oil 0.5081／0.5231／0.4824，平均 0.5045）；公開碼訓練設定 0.6985／0.6989／0.6924（Oil 0.4333／0.4350／0.4237，平均 0.4307）。
- **臺灣**：9 景像素加總算一次的 Oil IoU／P／R／F1／BG IoU／mIoU，附混淆矩陣，三 seed 平均 ± 標準差；對照 B（0.4434／0.4695／0.4355，平均 0.4495 ± 0.0178）與公開碼訓練設定（0.1807／0.3709／0.3614，平均 0.3043 ± 0.1071）。
- 不訂判定門檻，只作描述，GoM 驗證 Oil IoU 與臺灣 Oil IoU 各寫一次：
  - 三個 seed 都低於 B 的最小值 → 「只換 loss 就低於 B 的範圍」；
  - 與 B 的範圍重疊 → 「只換 loss 時與 B 沒有可分辨的差異」；
  - 三個都高於 B 的最大值 → 「高於 B 的範圍」。
- 另附描述性的「loss 佔差距的比例」＝（B 平均 − 本組平均）／（B 平均 − 公開碼設定平均），GoM 驗證與臺灣各一個。公開碼設定的臺灣數字用的是最後一個 epoch 的權重、seed 間差異大，這個比例很不穩，不單獨據此下結論。
- 同列訓練停在第幾個 epoch、最佳 epoch、訓練時間。
- 10-10 凍結卡與前兩組的結論不因本組更改。

## 可以與不可以主張

- 可以：在 B 的其他訓練設定下，只換 loss 的影響（描述性、n=3）。
- 不可以：推到其他模型或公開碼設定下（loss 與 batch size、學習率等可能有交互作用）；batch size、資料增強、取權重方式等另外八項的影響本組沒有測。

## 限制

- 三個 seed；B 與公開碼設定的結果都已看過，不是盲測。
- DiceCE／Focal 的權重（0.7／0.3、γ 2）照公開碼，沒有調；Tversky 的 α／β 也是沿用 baseline 的，沒有為 DADS-SAM 調過。
- 早停以驗證 mIoU 為準，loss 不同可能讓停的時間不同。
- Prompt 與公開碼不同（固定整張圖 box）；臺灣只有 9 景、兩個事件。

## 成本

每個 epoch 約 5.3 分鐘。B 三個 seed 停在第 133／151／121 個 epoch（700／798／637 分鐘），估每個 seed 11–14 小時，三個依序約 1.5–2 天；若都跑滿 300 個 epoch，每個約 26.5 小時。launcher `launch_B_dicece_focal.sh`（tmux `dads_sam_b_dicece_focal`）；結果 `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_gomonly_B_dicece_focal`；log `/home/alanyh/oil_dataset/new/_dads_sam_runs/DADS_SAM_B_dicece_focal_main_runner.log`（末行 `EXIT=`）。
