# DADS-SAM B 公開碼訓練設定：AdamW 同 weight decay 單因子實驗

**實驗編號**：無 P 編號　**凍結日期**：2026-10-18 Asia/Taipei　**狀態**：FROZEN_prelaunch
**事後補寫？**：否。本卡在兩組新實驗結果出現前凍結；第 7–9 步結案時另立紀錄，不回改本卡。
**取代哪張卡**：無。對照設計依據：[[20261013_DADS_SAM_B_公開碼訓練設定_設計]]、[[20261014_DADS_SAM_B_公開碼訓練設定_seed43_44_設計]]。
**授權與協商**：使用者於 2026-10-18 明確授權準備及啟動兩組新 DADS-SAM B 實驗；本卡只記錄事前設計，沒有訓練啟動或跨 harness 共識紀錄。

## 1. Objective（目的）

在 DADS-SAM B 六波段公開碼訓練設定下，隔離 Adam 與 AdamW 的 optimizer 規則差異對來源域 GoM 驗證表現的影響，為後續訓練設定選擇提供三 seed 配對證據。

## 2. Problem（問題）

既有公開碼設定與正式 B 同時差了 optimizer、weight decay 數值等多項設定，不能由原對比辨識 AdamW 本身的效果。公開碼設定三 seed 的臺灣結果已看過，不能把本次臺灣轉移評估稱為盲測或用它反向挑選 optimizer。

## 3. Research Question（研究問題）

其餘公開碼訓練設定固定時，只將 Adam 換成 AdamW，兩者均用 `weight_decay=1e-4` 和固定 `lr=1e-4`，是否使 seed 42／43／44 的 GoM 驗證 Oil IoU 在事前門檻下同向提高？

**已否定路線檢查（R6-1）**：已查 `/home/alanyh/.agents/institution/research/oil_spill_project_status.md` 的③「已否定路線清單」；其中沒有與本卡同型的 DADS-SAM B Adam 對 AdamW、同 weight decay 數值的單因子配對測試。這不是重開該清單中已否定的方法。

## 4. Hypothesis（假說）

AdamW 的 decoupled weight decay 可能讓此公開碼訓練設定取得較高的 GoM 來源驗證 Oil IoU。這是待測假說，不預設臺灣轉移會同步改善。

## 5. Method（方法）

**對照組與 estimand（R6-2）**

- 對照組：既有 DADS-SAM B 六波段公開碼訓練設定，seed 42／43／44，詳見上述兩張事前卡。三 seed 在「GoM 驗證 mIoU 最高 epoch」的 `_kind=patch` GoM val Oil IoU 依序為 **0.4333／0.4350／0.4237**。以同 seed 的新 run 減同 seed 的既有 run；不把正式 B 或只換 loss 組當作受控對照。
- 實驗組：同樣三 seed，各自從相同公開碼設定啟動新訓練；**唯一預定的訓練設定差異是 optimizer 從 Adam 換為 AdamW**。兩組均固定 `weight_decay=1e-4`、`lr=1e-4`，全程無 cosine 或其他 scheduler。各 seed 的資料、split、模型初始化規則、訓練和評估實作均須與對應控制 run 保持一致。
- 兩組共同設定：DADS-SAM B、相同六波段輸入及正規化、256→1024 放大、可訓練 3→6 patch embedding、二類 256×256 輸出、固定整張影像 box prompt `[0,0,1023,1023]`；loss 為 `0.7 DiceCE + 0.3 Focal`；batch 2；固定 100 epochs、不早停；不做資料增強、不做 gradient clipping；FP32。除 optimizer 外，不改 checkpoint、推論或評估政策。
- **解釋範圍**：本卡測試 AdamW 的 decoupled decay 相對 Adam 的規則差異，**不隔離 weight decay 數值大小**；也不隔離 batch size、epoch 數或資料增強。兩個 optimizer 共用的 `1e-4` 是相同輸入數值，不能解釋成相同數學更新規則。
- **主指標**：每個 seed 在 100 epochs 內取 **GoM validation mIoU 最高的 epoch**（若完全同分取最早 epoch），讀取該 epoch 的 GoM val Oil IoU，`_kind=patch`。主 estimand 為三個同 seed 配對差值 `Δ_s = OilIoU_new,s − OilIoU_control,s`；來源驗證指標不與臺灣重組指標混算。
- 每個 seed 同時報選中 epoch、該 epoch GoM val mIoU／Oil IoU，以及 epoch 100 的 GoM val mIoU／Oil IoU；控制組與新組的 Oil IoU、三個配對 Δ 均列出，並報三 seed 的平均與樣本標準差（`ddof=1`）。完整曲線保留供稽核，不事後改選 epoch。
- **臺灣轉移僅探索性**：`_kind=recon`；19 tiles = 原始 9 張快照的 137 patches + 新 9 張的 125 patches + Matsu 的 8 patches。報 19 tiles 與原始 9 張 subset，原始 9 張 subset 才與舊 run 作同範圍描述性比較。依原公開碼測試協定用 epoch 100 的 `last.pt`；若另做 GoM mIoU 選出的 `best.pt`，須標為事前指定敏感度分析，兩者分開全報。絕不按臺灣分數選 checkpoint、optimizer 或主結論。
- **已知非盲背景**：控制組 `last.pt` 的臺灣原始 9 張 pooled Oil IoU，seed 42／43／44 已觀察到 **0.1807／0.3709／0.3614**；其 `best.pt` 重測已觀察到 **0.3028／0.2571／0.3931**。這些數字不參與主判定；新臺灣結果也須標為非盲、探索性。GoM 來源驗證與臺灣域外轉移分開解讀。

## 6. Expected Evidence（預期證據）

- 假說成立時，預期三個同 seed 的主指標差值均為正，且各有實質幅度。
- **支持門檻**：三個 `Δ_s` 全部 `>= +0.03`。
- **否定門檻**：三個 `Δ_s` 全部 `<= -0.03`。
- 其他組合一律為**不可判定**，包括方向混合、幅度不足或缺少可驗證的配對結果；不得把均值為正或臺灣探索性改善改寫成支持。門檻只作用於上述 `_kind=patch` GoM 來源驗證主指標，並不檢驗臺灣轉移效果。
