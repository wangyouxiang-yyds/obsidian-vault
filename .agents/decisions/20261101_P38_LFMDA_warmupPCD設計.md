# P38 LFMDA warmup（DINOv2 教師＋PCD）對 P37 source-only：事前卡草稿

狀態：**FROZEN v7（2026-11-02）**：Claude × Codex R5 收斂（ACCEPT／BLOCKING=[]）；程式與依賴 SHA 已固定於「凍結 SHA」節，正本存 vault `.agents/decisions/20261101_P38_LFMDA_warmupPCD設計.md`（凍結卡 SHA 見 repo `FROZEN_CARD.sha256`）。未 launch；launch 與讀臺灣 GT 各需使用者單獨授權。凍結後任何影響模型／資料／前處理／loss 的改動須重新凍結並依第 6 步重跑規則處理。

使用者 2026-11-01 已定的三項設計：(1) 只做作者流程 warmup 階段，不含自訓練／SAM；(2) PCD 的 target 無標註影像＝原 19 景 270 個測試窗（transductive）；新增五景只作測試；(3) P38 與 P37 對照組各 3 seed。

**實驗編號**：P38　**凍結日期**：2026-11-02　**事後補寫?**：否　**取代哪張卡**：無（以 P37 v2 卡 `9e5de5b1…` 為對照基線）

## 1. Objective

P37 已確認作者 SegFormer-B2 source-only 在臺灣 19 景可當基線（final pooled OilIoU 0.3563，support）。本卡檢驗 LFMDA 的第一個跨域元件——由視覺基礎模型 DINOv2 當教師、以原型對比蒸餾（PCD）傳給學生——能否在不使用臺灣標註的條件下，讓 GoM→臺灣的油污分割優於 source-only。

## 2. Problem

P37 的誤差以漏判為主（precision 0.814、recall 0.388），天使輪 0801 L9 與 taiwan_file 0522 L8 景特別低。本專案先前的 target 端方法：P29/P30 對抗與全景 entropy 會塌縮、P32/P33 固定偽標籤自訓練大量漏判、P36 判別器改看含油範本 not_improved；這些都在 Prithvi 表徵上操作 target 輸出或判別器。LFMDA 主張改用與任務無關、泛化較好的基礎模型特徵當蒸餾目標，這個訊號類型本專案沒有在完整訓練中測過（P35 只測了凍結 Prithvi 原型的可分性，沒有訓練 PCD，也沒有用 DINOv2）。

## 3. Research Question

在使用同一 GoM 標註集、相同學生架構與初始化、相同 optimizer 類型與 base LR、10,000 optimizer steps 的條件下（T 組照作者 warmup 原碼另有：lr poly 以 15,000 步計、無梯度裁剪、混合 batch、target 增強與截斷；source 曝光、FLOPs 與 LR 軌跡皆不相等），加上作者 warmup 階段的 DINOv2 教師＋PCD（target 影像＝19 景 270 窗，只讀影像），學生 final checkpoint 在臺灣 19 景的 coverage-union pooled OilIoU 是否比 P37 source-only 高出具實務幅度的量？

**已否定路線檢查（R6-1）**：是，有相近路線。
- P29/P30（對抗／entropy，target 全景）塌縮、P36（判別器含油範本）not_improved：本卡**不開**作者 `--align-domain` 對抗分支（作者預設也關閉），因此不是同型方法。
- P32/P33（固定偽標籤自訓練）rejects：本卡的偽標籤只用在原型更新與特徵層對比，不直接當學生的分割監督；學生的分割 loss 仍只有 GoM 標註。
- **結構性改變**：新監督訊號＝凍結 DINOv2 ViT-L/14 特徵空間的類別原型（新表徵），以特徵層對比 loss 監督學生，不是重調既有方法的超參數。

## 4. Hypothesis

- **H1（主要，臺灣）**：作者 warmup 套件使臺灣 19 景 seed 平均 pooled OilIoU 比 source-only 高出超過 0.03，且超過組內合併 SD。理由：P37 漏判為主（P .814／R .388），DINOv2 特徵若較能泛化，原型對比可望補回漏判。
- **H2（GoM 保留）**：GoM source-val final OilIoU 的 seed 平均下降不超過 0.03。
- **H3（跨景廣度，解釋用，不取代 H1）**：改善不只來自大林兩景（約占油像素 60%）。
- **H4（新增五景，探索）**：已標油 micro recall 往正向變化。
- 反向風險：DINOv2 為自然影像預訓練，對反射率量化影像上的油膜不一定可分；臺灣窗上信心 ≥0.9 的偽標籤可能以背景為主，原型被背景主導，PCD 反而壓低油類（接近 P30 塌縮方向）。

## 5. Method

**共用（兩組相同，沿用 P37 `baseline.py` 與 P37 v2 卡）**：作者 SegFormer MiT-B2 學生、ImageNet MiT-B2 初始化 SHA `ced22617…`；GoM R1 train 2,738／val 702；B04/B03/B02 DN/10000→uint8、GoM mean/std；raw0→oil、其餘→BG；source batch 8、10,000 步、AdamW lr 6e-5（作者 param groups，head ×10）、wd 0.01、linear warmup 1,500、poly power 0.9、CE；256 原生窗、source 增強 `OneOf(HFlip,VFlip,Rot90) p=0.75`；每 1,000 步以 GoM source-val mIoU 選 best（與 P37 相同用 `>=`，同分取最晚 step）。

**兩組不同的優化設定（v6，照各自作者入口原碼）**：
- C 組＝作者 `train_sourceonly.py` 語義（即 P37）：poly 分母 `NUM_STEPS=10,001`（第 10,000 步 lr≈0）、`clip_grad_norm_(max_norm=32)`。
- T 組＝作者 `train_warmup.py` 語義：poly 分母 `NUM_STEPS=1.5×10,000=15,000`（第 10,000 步 lr＝(1/3)^0.9≈0.372×base）、**無梯度裁剪**（原碼 import 但未呼叫）；教師與學生用同一排程。

**T 組＝P38（作者 `tools/train_warmup.py --loss-kd PrototypeContrastiveLoss`，不開 `--align-domain`，不進 Stage 2、不用 SAM）**：
- 教師：作者 `VitSeg`＋DINOv2 ViT-L/14（權重 SHA `d5383ea8…`），backbone 凍結（訓練前後 state hash 必須相同）、decoder 依作者原樣從頭與學生同步以 GoM CE 訓練；256 輸入內部放大到 280（20×20 token、1024 維）。
- **輸入轉換（學生與教師對同一樣本收到完全相同張量；source 的學生、教師與原型初始化共用相同基礎轉換；target 訓練另加單側 clamp；照作者語義用資料集統計而非 ImageNet 統計）**：VRT 八槽取 B04/B03/B02 → channel 順序 (R,G,B)=(B04,B03,B02)；`np.rint(clip(DN/10000,0,1)×255)` 再轉 uint8（與 P37 `baseline.py`／`infer_target.py` 相同；unit test 驗證新 T／C 路徑與 P37 既有前處理 bit-identical）；再 `(x/255 − mean)/std`，mean `[.1433,.1342,.1087]`、std `[.2178,.2179,.2248]`（GoM R1 三通道）；source 走 P37 `baseline.py` 的轉換，target 走 P37 `infer_target.tensor`，兩者數值語義一致（smoke 在 identity 幾何變換下驗證同一窗兩路徑逐像素相等）。**訓練用 target 另照作者 config 在正規化後 `clamp(max=1.0)`**（單側上限、負值不設下限；GoM 統計下約等於把 DN/10000 高於 R .3611／G .3521／B .3335 的值壓平；作者只對 target 截斷，source CE、原型初始化與正式推論皆不截斷，構成明確的 train–inference mismatch，屬 warmup 政策的一部分；clamp 錯套到 source、原型初始化或推論才是工程無效）；教師與學生收到同一個已截斷張量。教師內部再以 bilinear 把 256 放大到 280（作者 `get_model` 寫死 `img_size=512`→560；本案依作者 `vit_in_img_size=int(1.1×img_size)//14×14` 的比例，對 256 原生窗取 280，與 2026-10-31 作者輸入 smoke 相同）。
- 原型初始化：依 `init_prototypes_dg.py`，以凍結 DINOv2 特徵在 GoM train 的類別平均（`update_avg`／`init_avg` 為單純平均，傳入的 decay 0.996 未使用）；照作者走 source train loader 一個 epoch（2,738//8＝342 batch，含 source 增強），特徵 bilinear 到 1/16、標籤 1/16 降採樣；每個 T run 先以該 run seed 重設 RNG 做初始化、再以同 seed 重設後開始 warmup（作者兩支程式皆 `seed_torch(2333)`）。
- 每步：教師與學生各在 GoM batch 上算 CE；另取 GoM 前 4 張＋target 4 張組混合 batch，target 偽標籤取教師 softmax 信心 ≥0.9，其餘忽略；原型以 EMA 0.99 更新；學生特徵經作者 mapper 投影後與原型算 `PrototypeContrastiveLoss`（溫度 0.3）。作者其餘設定照原樣。
- **target 抽樣與增強（v6 改照作者實際執行 config `configs/st/lfmda/2vaihingen_segformer.py` 的 `TARGET_DATA_CONFIG`；它覆寫 `configs/ToVaihingen.py` 的同名設定，v5 誤引後者）與 warmup 迴圈語義**：`RandomSampler`（每 epoch 不重複、epoch 間重新洗牌）、**batch 4**、`drop_last=True`（作者 `DALoader` 寫死；270 窗→每 epoch 67 batch，每 epoch 丟 2 窗、各 epoch 不同）；每步取一個 target batch，作者 `images_t[:bs//2]`（bs＝source batch 8）＝4 張全用；loader 耗盡即重建 iterator 繼續（語義同作者 `ever` Iterator）；window-uniform，不做 scene-balanced。增強照作者 `lfmda.aug.augmentation`（張量上依序）：`RandomCrop(512)` 對 256 原生窗不適用，略過（事前尺寸適配，結果屬 GoM-normalized、256-window 實作）；`RandomHorizontalFlip(0.5)`、`RandomVerticalFlip(0.5)`、`RandomRotate90(0.5)`（`torch.rot90(k=1, dims=[1,2])`）三者**各自獨立**；再正規化並 `clamp(max=1.0)`。教師與學生收到同一個已增強張量，因此偽標籤與學生特徵共享完全相同的幾何。`num_workers=0`；target sampler 與增強的 RNG 由 run seed 決定（與 source loader 分開的 generator，seed 固定為 run seed+1）。每 run target 使用＝10,000×4＝40,000 張，每窗約 148 次；每 run 記錄各窗實際使用次數。**與作者的差異**：唯一預定的演算法／輸入語義差異是 target 也用 GoM mean/std（作者用 target 自身統計；截斷門檻 1.0 因此作用在 GoM 標準化尺度上）；理由是維持與 P37 相同的輸入慣例、不把 target normalization 另納入 treatment（本案已是 transductive，用 target 無標註統計本身不算標註洩漏）。因此結論須稱「本案 GoM-normalized LFMDA warmup 實作」，不稱完全等同作者設定。其餘（`num_workers=0`、自製 iterator 取代 `ever`、獨立 RNG 流、作者 `read_sup=True` 讀的 SAM 區域在 warmup 未使用故不讀——smoke 以檔案開啟紀錄證明 target 路徑未開啟任何 mask／support 檔）為可重現性實作差異，預期語義等價。`torch_scatter` 以「被呼叫即報錯」的 stub 匯入（封存 stub SHA；smoke 確認 forward/backward 未呼叫；正式 run 若呼叫即中止＝工程無效）。
- **已知機制風險（GoM source 統計，非 target 資訊）**：作者 `DownscaleLabel(min_ratio=0.75)` 把標籤降到學生 1/32（8×8 格，每格須 ≥75% 同類才算數）；GoM train 1,188 個含油窗中只有 518 個留下 ≥1 個油格（全部 5,461 油格對 162,415 背景格、7,356 忽略格），原型初始化的 1/16 為 987 窗／28,201 油格。細油污窗幾乎不提供 PCD 油類訊號；照作者原樣，不改門檻。
- target loader：只暴露 19 景 270 窗的影像與**不含標籤欄位**的 manifest，前處理同 P37 `infer_target.tensor`；loader 程式碼路徑不可達 mask 目錄，並在 smoke 時驗證。新增五景 36 窗不進原型初始化、訓練、調參或選模。
- 機制紀錄（每 1,000 步，只記錄，不用於選模或重跑）：target 信心像素覆蓋率、偽標籤油／背景比例、兩類原型範數、PCD loss；clamp 前每通道 >1.0 的像素比例與超出量的平均／最大值。這些與其他 target 語義摘要一樣，到全部新 run 封存後才揭露；不得因飽和比例改 clamp。
- 原型檔：每個 T run 各自存 `init_prototypes.pth`（及結束時 `warmup_prototypes.pth`）於該 run 目錄，記錄 SHA、初始化 seed、source manifest 與載入它的 run ID，不跨 seed 共用檔名。

**C 組＝P37 source-only**：既有 S2333（checkpoint SHA `39947952…`）保留不重跑；以同一程式與設定補跑 2 seed。
**Seed**：兩組共用 {2333, 2334, 2335}，開跑前寫死。**不可撤銷的既有背景資訊**：P37 S2333 的臺灣 19 景指標、逐景結果、all-oil floor 0.1582、新增五景 recall，以及窗由已知事件位置選出的事實，皆已公開且列入本卡；這些資訊不得用來選 seed、改設定、決定重跑或選 P38 checkpoint。

**對照組與 estimand（R6-2）**
- 主指標：臺灣原 19 景／270 窗 coverage-union pooled OilIoU，學生 **step 10,000 final**，`_kind: patch`；評估邏輯同 P37 `evaluate_target.py`（raw0=oil、其餘=BG、重疊後列覆蓋），預測採 0=oil 編碼。
- 聚合：每 run 內 19 景 TP/FP/FN 加總算一次 IoU；Δ̄ = mean(T 三 run) − mean(C 三 run)。
- **不按 seed 編號配對**（兩組程式路徑不同，同 seed 不代表同隨機性）；以「T 各 run 是否高於 C 三 run 平均」計數方向一致性。
- 同時報：六個 run 原值；s_p = sqrt((2·s_T² + 2·s_C²)/4)（s_T、s_C 為各組三 run 的樣本 SD，ddof=1）；逐景差（每景「T 三 run 平均 − C 三 run 平均」）、scene-macro 差、正差景數，並以這 19 個逐景差做 one-sided exact sign-flip 檢定（統計量＝19 個逐景差的算術平均；零差保留為零、只對非零差翻號；窮舉全部 2^k 種翻號）（19 景是固定、事件選出的有限 panel，p<0.05 只佐證改善是否跨景分布，不推論到一般臺灣場景）；9 個跨組比較中 T_i > C_j 的比例；P/R/F1、TP/FP/FN、臺灣預測油比例；GoM val OilIoU 差；GoM-val-best 學生同一指標；教師自身臺灣 IoU（只當機制解釋線索；教師佳而學生差支持「傳遞不足」，其他組合可能混有原型、mapper 與學生 source CE 的交互作用，不能唯一診斷原因）；新增五景已標油 micro／逐景 recall（P37 Lane B 口徑）。
- **非單一變因**：T 同時多了 DINOv2 教師、原型、混合 batch（每步 source 曝光與 FLOPs 不同）、target 影像，且 v6 起 LR 軌跡（poly 分母 15,000 vs 10,001）與梯度裁剪（無 vs 32）也不同（學生 mapper 兩組都有，P37 學生的 decode head 本來就接在 mapper 後）。本卡只檢驗「作者 warmup 套件 vs source-only」兩個 10,000 步訓練政策的總效果；不得寫成「PCD 單獨有效」或 zero-shot 泛化；同樣地，否定結果也不得寫成「PCD 單獨無效」——無論方向，只能判斷本案 GoM-normalized warmup 完整政策是否勝過 P37 source-only 政策。若要分離，另立卡加入「同教師／mapper／混合路徑但移除 target PCD」對照。

**封存與讀 GT 時序**：每個新 run 結束即封存 config、log、程式 commit、dependency 版本清單、CUDA／PyTorch 版本、GPU 型號與 deterministic 旗標（C2333 為歷史 run，環境差異只能稽核、無法消除） 與下列 checkpoint 的 SHA 及其對 19 景 270 窗＋新增五景 36 窗的預測 manifest：C 組新 run 的 final 與 GoM-val-best 學生；T 組的 final 與 GoM-val-best 學生、final 教師。所有新產物封存後，經使用者單獨授權一次批次評估。P37 S2333 是既有已評估的例外：其 final/best 預測與結果沿用已封存版本（勘誤後 0=oil），不重新推論。重跑只限事前定義的工程無效（見第 6 步），不得因臺灣結果重跑。

**執行期盲化（smoke 結束、程式與設定 SHA 凍結後，到所有新 run 封存為止）**：
- target 預測圖與語義性 target 摘要（預測油比例、信心覆蓋率、偽標籤油／背景比例等）只寫入、不查看；執行期間只允許自動檢查檔案存在、shape、finite、數量與第 6 步事前列出的工程無效條件。
- 機制紀錄照寫入，但不得因其數值改程式、設定、門檻、seed、run 順序或重跑。
- 近塌縮旗標所需的預測油比例在全部新 run 封存後才揭露並判定。
- run 順序事前固定：C2334 → C2335 → T2333 → T2334 → T2335。

**開跑前工程 smoke（不讀 GT）**：作者 warmup 語義核對（source CE、混合 batch 切法、原型更新來源、PCD 涵蓋 source+target、mapper 輸入層、ignore mask、學生實際收到的 loss 項）；2 步 loss 有限、原型形狀 (2,1024)、兩類原型各自的有效 source 像素數 > 0 且範數有限非零、PCD 梯度確實到學生 mapper、教師 backbone 無梯度、target batch 未觸及 mask；v6 另驗：T 組 lr 依迴圈實際使用的零基 iteration index，在首步、warmup 邊界與最後一次 update 等於 15,000 分母公式值、優化步未呼叫梯度裁剪、target batch 為 4 且最大值 ≤1.0、三種翻轉各自獨立出現；C 組補 seed smoke 確認與 S2333 只差 seed。

### 凍結 SHA（2026-11-02，repo commit 176c409）

P38 程式（`analysis/p38_lfmda_warmup_pcd_20261101/`）：
- warmup_code_sha256: 364158a88aafeef5cfb7bbebf542f458f4efd7e8118047c6c7c8b464200da316（`warmup.py`，T 組）
- sourceonly_seed_code_sha256: b707d58ee0a739b87b3f62a80391ec780049322479792bd4a1d69c2a68bf5410（`sourceonly_seed.py`，C 組）
- predict_code_sha256: 0e1df60826ec3d665ca5bcc3606e577cd56ee54064217b5af870e64dda0a4b10（`predict.py`）
- check_inputs_code_sha256: 4639b27921c573582f2b6806ce9a5dbad7210a8cf6ab2c1bab7eb8ca58e8bb7e（`check_inputs.py`）

沿用的既有程式（不得改動）：
- P37 `baseline.py` a0091733422440c8a732006349ed6cc2ec7c183a0574893ff8436400e1b61ae6（＝P37 S2333 run_manifest script_sha256）
- P37 `infer_target.py` f8cf151de183f297ae22fc61ba71b63cec231668d9412b4284a86febc08ad04f
- `taiwan24_external_test_20261101/infer_p37_extension.py` 779685ad3914ff7c654b6353a90620853d21bcaf08b9b2a7c1872038deaa3b0e（只用 `MANIFEST`、`encode_class0_oil`）

作者原碼（StuLiu/LFMDA commit 88d9f2cda2e0c666611a0a8857240aaeb39a972b，clone 未改）：
- `lfmda/gast/alignment.py` faef3c4f134ad999c3a7667ae03a9bd380c3de91b6b02cc2009fa4664a56f352
- `lfmda/loss.py` e44b6398e70d662997d92e43ad1b2b7f2bb3ec1bd5cf9b297c7fd24a15b4cbfd
- `lfmda/models/vitseg.py` 1b31a7d2924a9df84a33020da616dd10183a6d3cc9d385e336cc3e4be2ebe726
- `lfmda/models/segformer.py` bac70ee0e88ba08963a21cce046519be035dec60b390646b68bb27db4a9d4111
- `lfmda/utils/tools.py` a585a8e536cc6b158d847995780ff510a20f716f236b9826931da6d39a5534c2
- `lfmda/gast/balance.py` 2c06e7faa7c3fbd643ba4c44e7cddfa5a90672280e8649185e4e1fe100951197

資料與權重：
- Taiwan19 image-only manifest 855a2307d229a3f436aabcc205fa50b3c278f325c35f7b40dfbaa1a440ea5162；Taiwan24 image-only manifest f8a5fda852e946a429f0063cd9007a610853533d99a8b3e1e43389333bac6075
- DINOv2 ViT-L/14 d5383ea8f4877b2472eb973e0fd72d557c7da5d3611bd527ceeb1d7162cbf428；MiT-B2 ced22617efb7bae3c34ad0a80f20a9b8afb4d27368cb0835a23456baa9d0e092
- 輸出根目錄 `/home/alanyh/oil_dataset/new/_p38_lfmda_warmup_20261102/`（run 目錄 `C_S2334`、`C_S2335`、`T_S2333`、`T_S2334`、`T_S2335`；smoke 目錄 `smoke_*` 不計入結果）

Smoke 證據（不讀 GT）：`smoke_T_S2333/smoke_result.json`、`smoke_C_S2334/smoke_result.json`；CPU 檢查 `check_inputs.py` 全過。

## 6. Expected Evidence（R6-3）

**工程無效（不屬科學判定）**：DINOv2 權重 hash 不符或凍結 backbone state hash 改變；新 run 的訓練、選模或預測程式讀取 target GT／mask，或在新 run 預測封存前產生／查看任何新 run 的 target-GT 衍生指標；標籤映射、ignore index、normalization、256→280→20×20 路徑、T 組 LR 排程／無裁剪／target batch 4／獨立翻轉／截斷不符 v6；loss／原型／預測非有限、缺 checkpoint 或缺預測；學生、教師 decoder 或 mapper 應更新卻沒有梯度，或 target PCD 因接線錯誤從未進入 loss；訓練未跑完。
**工程無效後的重跑規則**：若修正任何影響模型、資料、前處理或 loss 的問題，程式 SHA 改變 → 重新凍結 code／config SHA，並重跑**所有受影響組別**的全部受影響 run（錯誤若在共用前處理，兩組都要重跑）（不得保留舊版本 run 與新版混合）；只有純外部中斷（如斷電、OOM kill、GPU 失聯）且程式與狀態未改時，才可單獨重跑同 seed。重跑紀錄全部保留。
接線正確但最後 1,000 步 target 信心像素率 <0.1% 或油類偽標籤為零 → **不是無效**，是有效的機制失敗結果，保留並解釋。
P38 任一 run GoM val final OilIoU < 0.10 → 先查工程；確認接線正確則照常計入。

**H1（主要判定，只用學生 final 的 19 景 pooled OilIoU；需六個有效 run 齊全才計算 Δ̄ 與 s_p）**
先定義每個 T run 的兩種失敗旗標：
- 極端輸出／近塌縮：依與主指標相同的 coverage-union 去重後，19 景 covered pixels 中預測為油的比例 <1% 或 >99%；
- failure-floor：pooled OilIoU ≤ 同覆蓋 all-oil floor 0.1582。
判定（依序檢查，三類互斥且完整）：
- **否定**：(a) T 三 run 中至少 2 個觸發任一失敗旗標；或 (b) Δ̄ ≤ 0，且 T 三 run 中至少 2 個 ≤ C 三 run 平均。
- **支持**：不符合否定，且 Δ̄ > +0.03，且 Δ̄ > s_p，且 T 三 run 中至少 2 個高於 C 三 run 平均（此為 heuristic 一致性 gate，不是配對檢定）。
  - 另滿足 scene-macro 差 > 0、≥10/19 景改善、sign-flip p<0.05 → 可稱「跨景支持」；否則只稱「像素加權 pooled 支持，效果集中於少數景」。
- **不可判定**：其餘全部（如 0 < Δ̄ ≤ 0.03；Δ̄ > 0.03 但 Δ̄ ≤ s_p 或只有 1 個 T run 高於 C 平均；Δ̄ ≤ 0 但方向 gate 不成立）。
- 有效 run 不足：工程無效 run 依上述重跑規則補足三個有效 run；若無法補足，判不可判定，不得以不等樣本數套用原門檻。須寫明缺什麼。

**H2（GoM 保留）**：Δ̄_GoM ≥ −0.03 → 保留；Δ̄_GoM < −0.03 且 T 至少 2 run 低於 C 平均 → 未保留；其他不可判定。
**整體結論**：
- H1 否定 → warmup 套件 not improved／失敗，H2 不改變此結論；
- H1 支持且 H2 保留 → 較佳替代方案；
- H1 支持且 H2 未保留 → trade-off，不宣稱方法勝出；
- H1 支持但 H2 不可判定，或 H1 不可判定 → 整體不可判定。

**H4（新增五景，探索，不推翻 H1）**：ΔR_micro = mean_s(R^T_s) − mean_s(R^C_s)（各 run 先算 micro recall）；逐景以「T 三 run 平均 − C 三 run 平均」判斷。ΔR_micro > +0.03 且 ≥3/5 景不下降 → 正向訊號；差 ≤ 0 且 ≥3/5 景不改善 → 無正向訊號；其餘不可判定。不得報 IoU／precision。

三 seed 對約 0.09 量級的重跑波動仍偏少；0.03 是最低實務幅度，不代表不確定度已被精確估計，報告須附六個 run 原值。

---

## 後半（7–9）：結案才填

（未填）

---

## 協商紀錄

- R1（Codex 盲提案，session `01a1971c-846b-75d3-8552-182635b6959c`，gpt-5.6-sol／high）：ACCEPT／BLOCKING=[]；新增 H2 GoM 保留、H4 五景探索門檻、近塌縮定義、機制紀錄、sign-flip 跨景佐證、工程無效清單。Claude 併入候選稿 v2；改動：不按 seed 編號配對（改以「T run 高於 C 平均」計數）、支持另加 Δ̄ ≥ s_p、塌縮另含 IoU ≤ floor。
- R2（同 session resume，gpt-5.6-sol／high）：REVISE／BLOCKING=[1,2,3,4]。1 支持／否定未形式互斥 → 改為「否定先判、支持須排除否定」；2 洩漏條件與既有 P37 知識矛盾 → 改為限定新 run 並列出不可撤銷背景資訊；3 封存範圍未涵蓋 best 學生與 teacher → 補列；4 教師輸入轉換未寫死 → 依作者原碼核對（teacher/student 同一張量、資料集統計非 ImageNet）並寫死。非 blocking 5–10 全數採納（RQ 措辭、s_p 公式 ddof=1、H4 聚合、sign-flip 單位、整體結論四分支、教師 IoU 只作線索）；另加 9 個跨組比較勝出比例為報告項。
- R3（同 session）：REVISE／BLOCKING=[1,2,3]；R2 四項 blocking 確認已解決。1 執行期未禁止查看無標註 target 輸出（sequential target tuning）→ 加「執行期盲化」段與固定 run 順序；2 target 抽樣／增強未寫死 → 依作者 `configs/ToVaihingen.py` TARGET_DATA_CONFIG（RandomSampler、batch 8、drop_last、與 source 同增強）與 warmup 迴圈（只用前 bs//2）寫死，記錄與作者唯一差異（GoM 統計）；3 修正後重跑規則 → 影響模型／資料／loss 的修正須重跑該組全部受影響 run。非 blocking 4–7 採納（np.rint bit-identical unit test、H1 文字與門檻一致、best 同分取最晚 step 同 P37 `>=`、sign-flip 統計量）。
- R4（同 session）：**ACCEPT／BLOCKING=[]**，R3 三項 blocking 確認解決；非 blocking 1–5 全數採納為 v5（target 統計理由改寫與「GoM-normalized 實作」稱呼、差異分類、重跑涵蓋所有受影響組別、環境封存、近塌縮分母）。Codex RESTATE：「最強的反對意見是依序查看無標註 target 預測或偽標籤摘要仍會形成 target tuning；v4 已用 SHA 凍結後的 write-only artifacts、禁止依機制紀錄改動、固定 run 順序及全體封存後才揭露近塌縮指標妥善處理。」
- Claude 立場：ACCEPT；RESTATE Codex 最強論點＝不看 GT 但依序查看 target 輸出仍是 target tuning，已以執行期盲化處理。殘餘風險：三 seed 對 ~0.09 重跑波動仍偏少；P38 為多元套件，結果不可歸因到 PCD 單項。反證條件：smoke 發現作者 warmup 語義與卡上描述不符 → 回協商。
- **v6（2026-11-02，實作前原碼核對，觸發反證條件）**：作者 `runs/lfmda/2vaihingen_segformer.sh` 的 warmup 用 config `st.lfmda.2vaihingen_segformer`，其 `TARGET_DATA_CONFIG` 覆寫 `ToVaihingen.py`（R3 引錯檔）：target batch 4、作者自製 HFlip/VFlip/Rot90 各 0.5 獨立、正規化後 clamp(max=1.0)；`train_warmup.py` 的 poly 分母為 1.5×STAGE1_STEPS、未做梯度裁剪。教師 280 放大與作者 1.1× 比例相符（作者 `get_model` 寫死 512→560）。使用者裁決 T 組改照作者實際設定（C 組維持 P37）；GoM 統計正規化的既有差異保留。另發現 `torch_scatter` 未安裝：`Aligner` 只在 SAM 路徑用到 `scatter`，以會報錯的 stub 匯入；`ever` 在 P37 deps 目錄可用。
- R5（同 session resume，gpt-5.6-sol／high）：**ACCEPT／BLOCKING=[]**。(a) LR／裁剪差異已由 package estimand 涵蓋，建議補對稱限制（不得推「PCD 單獨無效」）→ 採納；lr smoke 以實際零基 index 核對首步／warmup 邊界／最後 update → 採納。(b) clamp 為單側上限、train–inference mismatch，須聲明門檻對應反射率並加 write-only 飽和紀錄 → 採納。(c) 非 blocking 1–6 全數採納（每 seed 原型檔獨立封存、輸入轉換措辭縮窄、RandomCrop 列為尺寸適配、stub fail-fast、read_sup I/O 由 smoke 證明、downscale 稀疏不得作改門檻理由）。Codex RESTATE：「T 與 C 連 LR 軌跡、梯度裁剪及 target clamp 都不同，任何正負結果都不能歸因於 PCD；v6 已將 estimand 限定為兩個完整 10,000-update 政策的總效果。」
- 收斂於 R5（v5 已於 R4 收斂、v6 因原碼核對重開一輪）（未達 5 回合檢查點）；模型 gpt-5.6-sol／reasoning high（requested=actual），codex-cli 0.166.0，session `01a1971c-846b-75d3-8552-182635b6959c`；四輪皆全文內嵌，Codex 自述未讀本機檔。

