# P39 LFMDA 第二階段（自訓練＋SAM 局部區域＋LRH）：事前卡草稿

狀態：**FROZEN v4（2026-11-03）**：Claude × Codex R4 收斂（ACCEPT／BLOCKING=[]）；smoke 通過；程式、SAM 區域與依賴 SHA 固定於「凍結 SHA」節；正本存 vault（SHA 見 repo `FROZEN_CARD.sha256`）。launch 與讀臺灣 GT 各需使用者授權：未凍結、未 launch。凍結版須經 `obsidian-vault-manager` 存 vault `.agents/decisions/20261103_P39_LFMDA_stage2_SAM_LRH設計.md` 並固定 SHA。

使用者 2026-11-03 方針：多階段方法先每步完整走一次，再回頭改善。P39 是 walk-through：照作者第二階段把 SAM 與 LRH 啟用跑一次，**承接的是已判 not improved 的 P38 warmup**（19 景 Δ̄ −0.104、GoM H2 未保留），不是因為 P38 有效才延伸。

**實驗編號**：P39　**凍結日期**：2026-11-03　**事後補寫?**：否　**取代哪張卡**：無（P38 凍結卡 `700d4883…` 與 P37 v2 卡為對照基線）

## 1. Objective

把 LFMDA（StuLiu/LFMDA，commit 88d9f2c）作者完整流程在 GoM→臺灣上跑完：P38 已跑 stage 1（DINOv2 教師＋PCD warmup），本卡跑 stage 2（教師產生 target 偽標籤的自訓練、原型＋預測加權修正、SAM 局部區域均質化 LRH），看完整流程相對 source-only 與相對 warmup 的表現。

## 2. Problem

P38 warmup 後學生漏判更多（19 景 recall 0.18–0.27 vs C 0.27–0.39）；機制紀錄顯示教師 target 高信心偽標籤中油只佔 1.3–1.4%。作者第二階段的主張是：用 SAM 切出的局部區域把區域內多數偽標籤擴散到整區（LRH），讓偽標籤邊界貼齊物件、補回區域內缺漏。本專案過去的固定偽標籤自訓練（P32/P33）大量漏判；P39 與它們的差別是 SAM 區域約束與線上原型／預測加權修正。

## 3. Research Question

以 P38 T 各 seed 的 warmup 產物為起點，跑「作者 stage 2 的本案適配／設計意圖修正版」（作者 `tools/train_ssl_lfmda.py --sam-refine --percent 0.95` 語義，6,000 步）後，學生 final 在臺灣原 19 景的 coverage-union pooled OilIoU：(a) 是否高於 source-only C 組；(b) 相對同一 seed 的 P38 T 學生 final 變化多少？

**Estimand 聲明**：訓練 target＝原 19 景 270 窗（與 H1 評估的影像是同一批，只用影像、不用標註），所以 H1 是 **transductive UDA estimand**，不是未見臺灣場景的泛化；新增五景 36 窗完全不進訓練、偽標籤、SAM 或選模，是唯一的 test-only 探索。結論只能估計「經本案適配與設計意圖修正後的完整 P38+P39 continuation package」，不能歸因於 SAM、LRH 或自訓練單項（沒有 no-LRH 對照，也沒有「同 P38 checkpoint 再做 6,000 步 source-only」對照）。

**已否定路線檢查（R6-1）**：是，相近路線被否定過。
- P32/P33（固定偽標籤自訓練）rejects、大量漏判；P38 warmup not improved。
- 結構性改變：SAM ViT-H 自動遮罩產生的局部區域（與任務無關的分割基礎模型）約束偽標籤；偽標籤由 DINOv2 教師產生、兩次重產生，並以原型＋預測加權線上修正。
- P39 承接 P38 的失敗起點且同樣依賴 target 偽標籤，失敗風險高；本卡定位為 walk-through，不預期勝出。

## 4. Hypothesis

- **H1（主要，對 source-only）**：P39 學生 final 在 19 景的 run 平均 pooled OilIoU 比 C 組平均高 >0.03 且 >s_p。事前預期：多半否定（起點 P38 T 平均 0.2103，比 C 低 0.104）。
- **H1b（stage 2 continuation package 的 post–pre change）**：d_s＝P39_s − P38T_s（同 seed 學生 final，同一軌跡延續所以配對）。不是 target adaptation／SAM／LRH 的因果增益。
- **H2（GoM 保留）**：GoM val final OilIoU 相對 C 平均下降不超過 0.03。
- **H4（新增五景，探索）**：已標油 micro recall 相對 C 的變化。
- 反向風險：偽標籤油比例低，LRH 只擴散區域內多數類別，可能進一步壓低油；SAM 面積門檻 1,024 px 在 256 窗上佔 1.56% 面積（作者 512 crop 只佔 0.39%），會系統性排除細長或小油區。

## 5. Method

**起點（每個 seed s∈{2333,2334,2335}）**：P38 `T_S{s}` 的 `student_final.pth`（SHA 前綴 2333 `5bbc102c…`／2334 `575c0280…`／2335 `223b8e81…`）、`teacher_final.pth`（`b3897b38…`／`97dd9c22…`／`f9ffe7ed…`）、`warmup_prototypes.pth`（`01a7f32d…`／`fc136e51…`／`62f90662…`），全部以 strict 載入並核對 P38 `train_complete.json` 的 SHA。作者同樣只載模型與原型：**stage 2 的 optimizer 與 LR 排程重新初始化，不承接 stage 1 的 AdamW moments**。

**凍結的作者 CLI 值**：`--gen 1`、`--tea-gen 1`、`--refine-label 1`、`--refine-mode all`、`--refine-temp 2.0`、`--sam-refine`（True）、`--percent 0.95`；config `st.lfmda.2vaihingen_segformer`：`STAGE2_STEPS 6000`、`CUTOFF_TOP 0.8`、`CUTOFF_LOW 0.6`、`LEARNING_RATE 6e-5`、`WEIGHT_DECAY 0.01`、`OPTIMIZER AdamW`、`EVAL_EVERY 1000`；腳本內 `NUM_STEPS = 6000×1.05 = 6300`、`PREHEAT_STEPS = 1500`、`POWER 0.9`。無梯度裁剪。

**每步流程（照作者主迴圈順序）**：
1. 兩個 optimizer 依作者 `adjust_learning_rate`（linear warmup 1,500 → poly 分母 6,300；head 群組 ×10）。
2. **偽標籤產生**：`i_iter == 0` 與 `i_iter == 3000` 時，教師 `eval()`，對 270 窗以 `pre_slide(tile 256, overlap 1/2, tta=True)` 推論。作者兩個模型在 eval 模式的 forward 都是 `return logit.softmax(dim=1)`（`vitseg.py:136`、`segformer.py:24`），所以 `tta_predict`（水平翻轉 × Rot90{0,90} 共 4 組）平均的是**機率**，存下的是機率 simplex，不另外補 softmax；`DATASETS.SIZE` 固定 (256,256)，`interpolate` 到同尺寸為恆等。以 **float32** 存為 run 目錄內 `pseudo_probs_step0.pt`／`pseudo_probs_step3000.pt`（270×2×256×256）並記 SHA；檔案為 dict：`probs`（依 dataset index＝Taiwan19 manifest 列序逐窗產生，不經 loader、不洗牌）、`window_keys`（每列 [scene_id, x, y]）、`target_manifest_sha256`、`step`、`teacher_state_sha256`、`class_index`（0＝oil、1＝background）。**該輪訓練唯一使用的偽標籤＝從這份已 hash 檔案讀回的 float32 值**；讀回時逐列 assert `window_keys` 與 target dataset 相同、manifest SHA 與 step 相符，之後才建 RandomSampler／target loader。
3. source batch 8（GoM，同 P37/P38 增強）；target batch 4（270 窗，RandomSampler、drop_last），每張帶「偽標籤機率 (2,256,256)」與「SAM 區域圖 (1,256,256)」；增強照作者 `mag`：HFlip、VFlip、Rot90 各 0.5 獨立，三者同步套在影像／機率／區域圖；影像正規化後依作者 `mag.Normalize(clamp=True)` 原碼 `torch.clamp(image, max=1.0)`。
4. 教師與學生都在 `train()` 模式（教師 backbone 參數凍結、不更新，但整個教師維持 train mode）分別前向 source 與 target；教師特徵 bilinear 到 1/32（8×8）。
5. **線上修正 `label_refine(None, feat_t_tea, [pred_t_tea1, pred_t_tea2], soft, refine=True, mode='all', temp=2.0)`**，逐式：
   - 原型權重 P＝softmax_c(1 / PearsonDist(feat, prototypes))（temp 1），上採樣到 256（bilinear, align_corners=True）後除以 max_c P；
   - 預測權重 Q＝[softmax_c(pred1/2.0) + softmax_c(pred2/2.0)]/2（pred 為教師訓練態 logits，先上採樣），除以 max_c Q；
   - weight＝P＋Q（第一參數 None → 超像素分支不執行）；
   - soft'＝(weight × soft) / (Σ_c(weight × soft) + 1e-7)。
6. **選取 `pseudo_selection(soft', cutoff_top 0.8, cutoff_low 0.6)`**：每張影像每類門檻 τ_c＝max(0.8 × max_pixels soft'_c, 0.6)；像素在恰好一類 soft'_c > τ_c（嚴格大於）時給該類，否則忽略。
7. **LRH**：每個 SAM 區域 r>0 內，計算已選取（非忽略）標籤的各類數量；若最大類 / (總數 + 1e-5) ≥ 0.95，整區（含原本忽略與少數類）改為該類；未達門檻與區域 0 保留第 6 步結果。
8. 原型 EMA（decay 0.99）先以教師 source 特徵＋source GT、再以教師 target 特徵＋LRH 後偽標籤更新；stage 2 沒有 PCD loss，原型只用於下一步的 label_refine。
9. loss：教師 CE(source GT)＋CE(target LRH 偽標籤)、學生同樣兩項（作者 `CrossEntropy(reduction='none')` 後對全部像素取平均，忽略像素以 0 計入）；兩者各自 backward，再各自 step。
10. **評估時點照作者**：更新後的 step 1，以及 step 1000、2000、…、6000 以 GoM source-val 評估學生；best＝GoM val **mIoU** 最大（`>=`，同分取最晚，含 step 1）。報告地位：**headline＝step 6000 學生 final**；GoM-val best 為次要。

**SAM 局部區域（作者 `lfmda/utils/local_region_homog.py`）**：作者 `get_all_regs` 用 `SAM()` 預設 **ViT-H**（`sam_vit_h_4b8939.pth`，官方 dl.fbaipublicfiles.com，md5 `4b8939a8…`、SHA-256 `a7bf3b02f3ebf1267aba913ff637d9a2d5c33d3173bb679e46d9f338c26f262e`）；`SamAutomaticMaskGenerator(points_per_side 32, pred_iou_thresh 0.90, stability_score_thresh 0.95, crop_n_layers 1, crop_n_points_downscale_factor 2)`；面積 ≥1,024 px 的遮罩依產生順序寫入編號 i+1（後者覆蓋前者），其餘 0；存無損 int32 `.npy`。對 270 窗各跑一次（開跑前一次性產生、封存 SHA）；不跑新增五景。headless、不輸出 overlay；不得依區域統計修改任何設定。
**SAM 輸入**：由同一窗另行渲染的 **SAM RGB proxy**：B04/B03/B02 `np.rint(clip(DN/10000,0,1)×255)` uint8、無效像素 0、不逐張拉伸（與 P37 `infer_target.tensor` 正規化前的量化相同，但不是模型的正規化浮點輸入；smoke 驗證 channel 順序與 `tensor()` 反算一致）。作者讀的是一般 8-bit PNG；本案海面在固定量化下偏暗，區域可能偏少。

**Material adaptations（全部事前列出）**：
1. 資料與類別：GoM→臺灣、二類（oil／background），非作者 ISPRS 六類；source batch 8、target batch 4；256 原生窗（作者 512 crop），`RandomCrop(512)` 不適用。
2. 正規化用 GoM R1 RGB 統計（同 P37/P38；作者用 target 統計）：mean (R,G,B)＝(0.1433, 0.1342, 0.1087)、std＝(0.2178, 0.2179, 0.2248)（四位小數為顯示；實際值由 P37 `baseline.RGBSource` 從 `main/experiments_EUDA_stage1_20261025.yaml` 讀取，該檔 SHA 已記於 P37 run_manifest `source_config_sha256`，P39 run_manifest 重記）。
3. `pre_slide` tile 改 256：作者 tile 512 在 256 窗上 `tile_rows = ceil((256−512)/256)+1 = 0`，0 個 tile、除以 0；改 256 即恰一個 tile、count map 全 1、不 padding（smoke 驗證）。
4. **偽標籤保留機率（使用者 2026-11-03 裁決）**：作者 `basedata.py` 讀回 `label_type='prob'` 的 `.pt` 後回傳 `dict(cls=mask.long())`，把機率截成整數，只有恰為 1.0 的像素留下非零值；與 config `label_type='prob'`、`label_refine` 的機率運算及 0.8／0.6 門檻設計不符，判定為疑似 bug；本案保留浮點機率。
5. SAM 面積門檻照字面 1,024 px（在 256 窗上的相對尺度是作者的 4 倍，見反向風險）；SAM 輸入為固定 8-bit RGB proxy。
6. `torch_scatter` 未安裝：LRH 的 `scatter(reduce='sum')` 改原生 `scatter_add_`，以單元測試對照逐區域參考實作（含不連續區域 ID、區域 0、全忽略、平手、多 batch）完全相同；label_refine 的超像素分支作者未啟用。
7. `segment-anything 1.0` 以 `pip --target` 裝到 `/home/alanyh/oil_dataset/new/_p39_lfmda_stage2_20261103/deps`。
8. 評估／選模用 GoM val（作者用 target val）。
9. 偽標籤存檔保留供稽核（作者訓練後刪除）。
因此結論須稱「作者 stage 2 的本案適配／設計意圖修正版」，不稱照作者原樣執行。

**資料清單（凍結）**：訓練 target＝Taiwan19 image-only manifest `analysis/euda_gtroi_pipeline_20261029/stage1_target_manifest.tsv` SHA `855a2307d229a3f436aabcc205fa50b3c278f325c35f7b40dfbaa1a440ea5162`（19 景 270 窗，欄位只有 scene_id/sensor/vrt/x/y/sampled_invalid_fraction）；評估 roster＝Taiwan24 image-only manifest `analysis/taiwan24_external_test_20261101/taiwan24_image_only.tsv` SHA `f8a5fda852e946a429f0063cd9007a610853533d99a8b3e1e43389333bac6075`（前 270 列＝上述 19 景，後 36 列＝新增五景）。程式只讀前者訓練、SAM 也只跑前者；smoke 驗證新增五景的 VRT 從未被訓練路徑開啟。專案正本清單 `analysis/taiwan_test_roster_current.json` SHA `302ef181395fd2cd3215cf7540fd48048d5589c8a20bcca037fce04136501ab2`（version taiwan24_20261101、24 景／306 窗；其內 image-only manifest SHA＝上述 `f8a5fda8…`、registry SHA＝`8effb392a1b76c15441ccd8ff1e42820993d0c4de46bbc2a959dfa8b8f923571`，2026-11-03 已核對一致）。
**評估口徑來源（凍結）**：19 景＝P37 `evaluate_target.py`（SHA `3a2f5869…`）的 `evaluate`：GT 讀 `full_band/mask/{scene}.tif`，raw 0＝油、其餘＝背景，評估窗 coverage-union、重疊後列覆蓋；新增五景＝P37 `evaluate_p37_extension.py`（SHA `729c68c6…`）的 `audit_invalid`／`evaluate_role`：油標註檔路徑與 SHA 由上述 registry 固定，無效像素由 VRT 依 TSV 的 `sampled_invalid_fraction` 稽核後排除；19 景 GT 檔的 SHA 在使用者授權的評估當下記錄（凍結前不開 GT 檔）。P39 評估器將沿用 P38 `evaluate_p38.py`（SHA `c7db9837…`）的結構另寫並在讀 GT 前封存。

**機制紀錄（write-only，三個 run 全部封存後才揭露）**：每次偽標籤產生（對完整 270 窗）：油機率平均、油機率 >0.5／>0.9 比例；每 1,000 步（**該 1,000 步內所有 target batch 的累積計數**；原型範數取該步當下值）：選取後（LRH 前）油／背景像素數、LRH 後油／背景像素數、忽略→油、忽略→背景、油→背景、背景→油的像素數、兩類原型範數；SAM 區域（manifest 內）：每窗區域數、區域覆蓋比例。

**執行期盲化**：程式與設定 SHA 凍結後，到三個 run 全部封存為止：target 預測與語義摘要只寫不看；**不執行也不開啟任何臺灣 GT evaluator**；機制紀錄不得用來改程式、設定、seed、run 順序或重跑；run 順序固定 S2333→S2334→S2335。

**開跑前工程 smoke（不讀 GT）**：P38 checkpoint strict 載入、SHA 相符；教師 backbone 無梯度且 hash 不變；`pre_slide` 恰一個 tile、count 全 1；偽標籤 270 窗形狀 (2,256,256)、有限、機率和與 1 相差 ≤1e-4；target batch 的影像／機率／區域三者幾何同步（以合成座標 tensor 驗證）；label_refine 輸出非負、和與 1 相差 ≤1e-4（`_logits_norm` 分母加 1e-7，嚴格和略小於 1）；選取與 LRH 輸出值域 ⊆ {−1,0,1}；LRH 原生實作與參考實作逐像素相同；偽標籤檔讀回後首、末與固定抽查列（0、1、134、268、269）的 window key 與 manifest 相符；2 步 loss 有限、教師 decoder 與學生有梯度；lr 在首步、warmup 邊界、最後一步等於 6,300 分母公式值；SAM proxy channel 與 `tensor()` 反算一致；target 路徑只開 270 窗 VRT、SAM 區域檔、偽標籤檔，不開任何 mask，也不開新增五景 VRT。

### 凍結 SHA（2026-11-03）

P39 程式（`analysis/p39_lfmda_stage2_sam_lrh_20261103/`）：
- stage2_code_sha256: e3b8486b37254d73b8a068d351704b7d075ea0c5457c0def36577c11297c466c（`stage2.py`）
- sam_regions_code_sha256: 4198ed46c5860263bdc4fe234f7121c48f992405a1dddda639b10ae3ef491fd3（`sam_regions.py`；與 SAM 區域 manifest 內記錄相同）
- predict_code_sha256: c4b9c2ed9b207c8b3214d9ecc612fb8b7ebf1cafcbe8c5127276dc23b6b4f404（`predict_p39.py`）
- check_code_sha256: cf46a5ffcabc8a08056e557c3164f3c70b5d7516ec75fd0079c9ea669f6f643b（`check_p39.py`）；test_homogenize 95649472fe2e206ef31913f63e5beab68476e92758876ad81efa6d88d33946ac

沿用（不得改動）：P38 `warmup.py` 364158a8…、P38 `predict.py` 0e1df608…、P37 `baseline.py` a0091733…、P37 `infer_target.py` f8cf151d…（P38 凍結卡已列完整值）；作者原碼 commit 88d9f2c 未改，另用到 `lfmda/gast/pseudo_generation.py`（`pseudo_selection`）、`lfmda/utils/tools.py`（`pre_slide`／`tta_predict`）與 P38 卡列出的檔案。

SAM：權重 `sam_vit_h_4b8939.pth` SHA-256 a7bf3b02f3ebf1267aba913ff637d9a2d5c33d3173bb679e46d9f338c26f262e；`segment_anything` 1.0（`automatic_mask_generator.py` c818960c61c4f51d3f5a2286034d4b5f491e05ab1e84f3155756d9d031375d01）；**270 窗區域 manifest** `/home/alanyh/oil_dataset/new/_p39_lfmda_stage2_20261103/sam_regions/manifest.json` SHA 05fa132bd23de4580199442ecaba8eeab22042a705f21000928050e61cf0235f（270 筆，每窗 int32 `.npy` 各有 SHA）。

輸出根目錄 `/home/alanyh/oil_dataset/new/_p39_lfmda_stage2_20261103/`（run 目錄 `P39_S2333`、`P39_S2334`、`P39_S2335`；`smoke_*` 不計入結果）。

Smoke 證據（不讀 GT）：`check_p39.py` 全過（lr、SAM proxy 0 差、幾何同步 8 種、target 只開 19 個 VRT／270 次且未開新增五景）；`test_homogenize` 通過；`smoke_P39_S2333/smoke_result.json`：P38 checkpoint strict 載入、`pre_slide` 恰一 tile（4 次 TTA forward）、偽標籤檔 window key 抽查 0/1/134/268/269 相符、label_refine 和與 1 最大差 1.8e-7、選取與 LRH 值域 ⊆ {−1,0,1}、教師 backbone 無梯度且 hash 不變、教師 decoder 與學生有梯度、4 項 loss 有限。

## 6. Expected Evidence

**工程無效**：起點 SHA 不符或 strict 載入失敗；教師 backbone hash 改變；SAM 權重 SHA 不符或區域檔缺漏／SHA 不符；偽標籤非有限、形狀錯、與 1 的最大差異 >1e-4，或 `window_keys`／manifest SHA 與 target dataset 不符；LRH 等價測試失敗；target GT 被讀或新增五景進入訓練；loss／原型／預測非有限、缺 checkpoint 或缺預測；訓練未跑完。修正影響模型／資料／loss 的問題 → 重新凍結並重跑全部受影響 run；純外部中斷且程式未改 → 可單獨重跑同 seed。接線正確但偽標籤油為零、LRH 幾乎不改變像素、或結果極差 → 有效的科學結果，保留並解釋，不算工程無效。

**基線原值（凍結；表列四位小數為顯示，判定計算使用封存結果檔的完整精度：P38 `eval_result.json` SHA `7b18bd53…`、P37 Lane A `ff36cb9d…`、Lane B `d7c95df1…`）**：C 組 19 景 pooled OilIoU（學生 final）＝P37 S2333 0.3563、P38 C2334 0.3349、P38 C2335 0.2530（平均 0.3147）；C 組 GoM val final OilIoU＝0.3763、0.3906、0.3556（平均 0.3742）；P38 T 學生 final 19 景＝S2333 0.2538、S2334 0.2017、S2335 0.1754。C 組新增五景 micro recall（final）＝0.2516、0.5287、0.4184。

**H1（只用學生 final 的 19 景 pooled OilIoU；三個 P39 run 都有效才計算）**：
- 定義：x_i＝P39 三 run，c_j＝C 三 run；Δ̄＝mean(x)−mean(c)；s_p＝sqrt((2·s_x² + 2·s_c²)/4)，s_x、s_c 為各組三值的樣本 SD（ddof=1）。
- run 層失敗旗標（每個 P39 run 各自判，該 run 仍計入平均，不算工程無效）：(i) 19 景 coverage-union 去重後預測為油的像素比例 <1% 或 >99%；(ii) pooled OilIoU ≤ 0.1582（同覆蓋 all-oil floor；與 P38 相同，0.1582 為刻意凍結的操作性常數，照此四位數使用）。
- 判定依序、互斥：
  - **否定**：(a) 三個 P39 run 中 ≥2 個觸發任一旗標；或 (b) Δ̄ ≤ 0 且 ≥2 個 P39 run ≤ mean(c)。
  - **支持**：不符否定，且 Δ̄ > 0.03，且 Δ̄ > s_p，且 ≥2 個 P39 run > mean(c)。另滿足下列三項 → 「跨景支持」，否則「像素加權支持、效果集中於少數景」：逐景差 e_k＝(P39 三 run 該景 IoU 平均) − (C 三 run 該景 IoU 平均)，k＝1..19；scene-macro＝mean_k e_k > 0；改善景數＝#{e_k > 0} ≥ 10；one-sided exact sign-flip（統計量＝mean_k e_k，零差保留、只翻非零差的號、窮舉 2^m，p＝P(翻號統計量 ≥ 觀察值)）p < 0.05。
  - **不可判定**：其餘。
- 同時報：六個原值、9 個跨組比較中 x_i > c_j 的比例、P/R/F1、TP/FP/FN、預測油比例、逐景差、教師 final 19 景 IoU（機制線索）、GoM-val best 學生同一指標（次要）。

**H1b**：d_s＝P39_s − P38T_s（s＝2333/2334/2335）；mean(d) > 0.03 且 ≥2 個 d_s > 0 → 「continuation package 有正向 post–pre change」；mean(d) ≤ 0 且 ≥2 個 d_s ≤ 0 → 「無正向 change」；其餘不可判定。不推翻 H1。

**H2（GoM 保留）**：g_i＝P39 GoM val final OilIoU；c̄_GoM＝C 三個封存值（P37 `train_log.jsonl` 與 P38 C 兩 run `train_log.jsonl` 最後一列）的完整精度平均（顯示約 0.3742）；Δ̄_GoM＝mean(g) − c̄_GoM。Δ̄_GoM ≥ −0.03 → 保留；Δ̄_GoM < −0.03 且 ≥2 個 g_i < c̄_GoM → 未保留；其餘不可判定。

**整體（四分支）**：H1 否定 → not improved／失敗（H2 不改變）；H1 支持且 H2 保留 → 較佳替代方案；H1 支持且 H2 未保留 → trade-off，不宣稱勝出；H1 支持但 H2 不可判定、或 H1 不可判定 → 整體不可判定。

**H4（新增五景，探索，不推翻 H1）**：每個 run 的 micro recall＝Σ景（評估窗 coverage-union 內、有效像素中，GT 已標油且預測為油的像素）／Σ景（評估窗 coverage-union 內、有效像素中 GT 已標油的像素）；油 polygon 以外全部視為 unknown，不計 FP，不算 precision／F1／IoU。ΔR＝mean(P39 三 run) − mean(C 三 run)；逐景差＝P39 三 run 平均 − C 三 run 平均。ΔR > 0.03 且 ≥3/5 景逐景差 ≥0 → 正向訊號；ΔR ≤ 0 且 ≥3/5 景逐景差 ≤0 → 無正向訊號；其餘不可判定。必須同時報預測油比例（分母＝五景評估窗 coverage-union 內的有效像素；只作機制描述，polygon 外的預測油不算 FP）。

三 seed 對 C 組自身 0.25–0.36 的波動仍偏少；報告附全部 run 原值。

**成本估計**：SAM ViT-H 270 窗約 15–30 分鐘（smoke 2 窗約 11 秒含載入）；每個 run 6,000 步加兩次偽標籤產生，估約 1 小時（未實測）；3 run 約 3–4 小時。

---

## 後半（7–9）：結案才填

（未填）

---

## 協商紀錄

- R1（Codex gpt-5.6-sol／high，新 session）：REVISE，blocking：eval 輸出與 softmax／TTA 次序、label_refine 精確公式、H1/H2/overall 自足化、訓練／評估 manifest 與 transductive 聲明、凍結 CLI、step-1 評估語義。v2 處理：作者兩模型 eval forward 即 softmax（vitseg.py:136、segformer.py:24），TTA 平均機率、不另補 softmax；label_refine 逐式寫入；判定式、s_p、旗標層級、sign-flip 單位、基線原值全部寫入；manifest SHA 與 transductive estimand 明列；CLI 值凍結；評估改為 step 1＋每 1,000 步，best 用 GoM mIoU 為次要。非 blocking 全採（material adaptations 改名並補齊、SAM 面積尺度、SAM RGB proxy 稱呼、optimizer 重新初始化、教師 train 模式、H1b 改稱 post–pre change、評估器在三 run 封存前不執行、SAM headless）。
- R2（同 session resume）：REVISE，blocking：偽標籤 authoritative dtype／路徑唯一化；pin `taiwan_test_roster_current.json` 與可重建評估口徑的 manifest。v3 處理：偽標籤改 float32 存檔、訓練只讀回同一份已 hash 檔（程式已改）；pin roster JSON `302ef181…`、registry `8effb392…`、兩個 P37 評估器 SHA，寫明 GT 讀取與無效像素口徑來源。小修全採（和為 1 的容差、GoM 統計數值與來源、基線完整精度來源、機制紀錄為 1,000 步累積、刪除 dropout 斷言、H4 預測油比例分母）。
- R3（同 session）：REVISE，blocking：單一 270-window 偽標籤 tensor 未固定逐列 window identity。v4 處理：偽標籤檔改存 dict（probs＋window_keys＋manifest SHA＋step＋教師 state SHA＋class map），讀回逐列核對後才建 loader（程式已改）；smoke 抽查固定列。非 blocking 全採（H2 用完整精度 c̄_GoM、0.1582 註明為凍結操作常數、工程無效改用 1e-4 容差措辭）。
- R4（同 session）：**ACCEPT／BLOCKING=[]**。非 blocking：讀回偽標籤時另 assert shape 與 class_index → 採納（程式已加）。Codex RESTATE：「P39 是承接失敗 P38 起點、在原 19 景無標籤影像上執行的 transductive LFMDA stage-2 walk-through；主要結論只針對經本案適配與設計意圖修正後的完整 continuation package，headline 為 step-6000 學生 final，不能歸因於 SAM、LRH 或自訓練單項。」Codex 明言此判定不等同授權 launch。
- 收斂於 R4；模型 gpt-5.6-sol／reasoning high，codex-cli 0.166.0，session `01a19f1f-9f4c-7790-821d-19aa98fa64ac`；四輪皆全文內嵌。
