# 2026-10-01 GoM→台灣遷移優勢的機制分析計畫骨架與 DG survey 方向（Claude×Codex 共識）

## 議題
使用者（論文 survey 階段）：實驗顯示只用 GoM 訓練的模型在台灣影像表現較好，但實驗本身不是直接證明；想找 (A) 把台灣海域特性與 GoM 海域特性關聯起來的分析方法、(B) 量化關聯性／domain shift 的方法、(C) 只用海外資料訓練、套用到沒有目標訓練資料的新海域的方法（domain generalization）。使用者明確要求與 Codex 討論。

## 參與與 provenance
- 主持方：Claude Code，claude-opus-5-5
- 對手方：Codex CLI 0.159.2，requested=actual `gpt-5.6-sol`（三輪 header 皆確認），`model_reasoning_effort=high`，web_search 開啟
- Codex session：`01a0f50c-fe5d-7430-b6bb-7d75c002d19b`（R1 首發，R2/R3 以 session id resume 並釘死 `-c model="gpt-5.6-sol"`）
- tokens used（Codex 自報）：R1 113,806／R2 161,462／R3 174,658
- **降級事件**：codex 0.159.2 下即使帶 `-c features.use_legacy_landlock=true` + `< /dev/null`，Codex 自述「sandbox 初始化失敗、無法讀取本機檔案」（三輪皆未讀檔）。本案背景事實已全文內嵌 prompt，不影響結論；但 2026-08-24 定案的 landlock 修法在 0.159.2 已失效，需另案重測。

## 逐輪演變
- **R1（盲提案）**：Claude 在看到 Codex 回覆前先寫下獨立立場（把「海域」拆成共變數；目標端模式檢查＋來源端逐景混合模型歸因＋介入；Prithvi embedding 距離；DG 方法族含 TTA）。Codex 獨立提案：研究問題應改寫為「解釋 GoM 對 05-22 的 recall 優勢、對天使輪無改善」；資料生成鏈 Z→X→H→Ŷ（另有標註 A、取樣 S）；P16 exact manifest；class-conditional 距離＋coverage；標註 erosion/dilation 檢驗；LOEO pseudo-target；DG 五層（強 ERM→物理增強→多來源不變性→meta-learning→FM+PEFT）。STANCE REVISE，BLOCKING [1,2,3]（未明文）。
- **R2（候選稿 v1）**：Claude 合併雙方提案。Codex REVISE，blocking 三項（R3 澄清）：①主要距離尚未完全操作化；②Phase 2c 的獨立單位與可識別性未定；③Phase 2b 介入參照不當（只把 NonGoM 配成 GoM）。另提 8 項修改（retrospective prereg 定位、H1 改名、Cox–Munk 降為 exposure proxy、2.6 降 supplement、共同支持域 Q、supergroup、TTA 獨立範式、outline 三層）。
- **R3（候選稿 v2）**：Claude 全數採納並新增 C1（Phase 2c 可行性：P5 算術已證明除 GoM 約 2,006、NOAA Atlantic 約 445 含油 patch 外，其他海域合計僅 11–166）與 C2（本共識只是 survey＋分析骨架，不是 launch 授權）。Codex **ACCEPT，BLOCKING []**，追加兩條 Phase 2c stop rules 與 confirmatory 範圍區分。Claude ACCEPT。
- 收斂：3 回合；停止原因＝雙方 ACCEPT 且 BLOCKING=[]。

## 最終決議（候選稿 v2 final）

### 0. 研究問題（改寫）
在固定 P16 協定下，哪些可觀測的資料生成因素能解釋 GoM 訓練集對 2026-05-22 事件的大幅 recall 優勢、卻沒改善天使輪？這些因素能否在其他獨立事件上預測 transfer performance？「GoM／台灣」只是代理標籤。

### 1. 主張邊界
- 可主張：固定 P16、seed42、兩台灣事件、GT 中心含油 patch 上 GoM event-macro 較高；差異高度事件依賴、主要來自 05-22 recall；未過事前雙事件 ≥+0.02 判準 → 跨台灣事件穩健優勢 inconclusive；單臂 seed SD 不能取代 paired multi-seed。
- 不可主張：GoM 與台灣海域整體相似；GoM-only 普遍較能泛化；海況／季節／感測器／大片油「造成」優勢；可用於全景搜尋；137 patch = 137 次獨立證據；NonGoM 較差 = 地理多樣性有害。
- 論文寫「downstream 未使用台灣資料」，不寫「模型從未看過台灣影像」（Prithvi 預訓練可能涵蓋台灣）。

### 2. Phase 1（不訓練；CPU＋推論）＝ retrospective preregistered audit（鎖定式事後分析）
- 定位：P16 結果已看過，故不是獨立確認性證據；只有新事件或事前鎖定的 pseudo-target 才算 confirmatory。
- 單位：事件 > 同日 mosaic；patch/pixel 為巢狀重複量測；不做 token/patch permutation p 值。
- 共變數清單在計算任何台灣數值前凍結：A 海氣（ERA5 成像時刻 10 m 風、Hs）；B 耀光幾何（太陽/觀測天頂、相對方位→鏡面角；Cox–Munk 改稱 water-surface normalized glint proxy，只作 exposure proxy）；C 水體背景（local-water 六波段分位數為主，MODIS/OLCI、GEBCO 次要）；D oil–local-water 逐波段反差（光學機制的次要分析）；E 形態／occupancy；F 標註尺度（erosion/dilation、boundary-tolerant）。
- **Primary 距離（唯一）**：D = ½D_oil + ½D_water。同一未 fine-tune 的 Prithvi pretrained checkpoint；最後層 spatial feature 依 class mask 加權 pooling（每 patch 每 class ≤1 向量）；local water＝油 mask 外擴 16 px 環帶、扣內側 2 px 緩衝，排除其他油、陸地、雲/陰影、255，有效水像素 <200 則排除該 water 向量；PCA 只在 P16 GoM+NonGoM source 上 fit、固定 32 維；multi-kernel RBF MMD，bandwidth {0.5m, m, 2m}（m＝source pairwise median）等權；source acquisition-cluster ↔ target-date 先算再日期等權、事件等權；component 只報告不得推翻 primary；另報 kNN coverage。
- 假設：H1（Prithvi representation-support）D(GoM,T0522) < D(NonGoM,T0522)，方向不符即拒絕，天使輪只作異質性對照；H2 形態／occupancy；H3 標註。
- 錯誤稽核：P16 GoM vs NonGoM checkpoint 的 FN 依 interior/boundary、component 大小、反差分層。
- Supplement：「既有 OOF 預測的影像條件—錯誤關聯稽核」（350/355 姊妹景洩漏，非 transfer 證據、不作 Phase 2 gate、報 sibling exposure）。

### 3. Phase 2（GPU；依 Phase 1 結果，且每步需使用者授權）
- 2a paired multi-seed P16（GoM/NonGoM 各補 seeds）。
- 2b 介入：主要＝GoM 與 NonGoM 都配到共同支持域 Q（固定 patch 數、update、ESS）；次要＝配到 05-22 共變數分布，標 target-aware diagnostic；報 ESS、最大權重、丟棄比例；overlap 不足 → 「無法識別」。
- 2c（探索性，預期不可行）：primary 分組＝provenance supergroup；union-find acquisition cluster 只作敏感度；優先 frozen encoder＋shallow head 的 source→target transfer matrix。Stop rules：cell-level（target ≥30 含油 patch 且不全來自同一重疊 acquisition；30 是評估穩定度門檻，不是獨立樣本數）；relation-level（<3 個可訓練 source supergroup 或 eligible target 太少 → 只報描述性矩陣，不算 Spearman/迴歸、不作距離—表現主張）。
- Confirmatory 範圍：新增獨立台灣事件 → 可確認／反駁「對台灣事件的遷移」；新增其他海域事件 → 可檢驗一般性 representation distance–transfer 機制，但不能單獨確認台灣主張。

### 4. Survey taxonomy（四欄）
1. 正式 DG 方法：強 ERM＋公平 protocol（DomainBed、SWAD）→ 物理合理的 randomization／style（MixStyle、RobustNet、WildNet、CCDR TGRS 2024）→ 多來源不變性／worst-group（GroupDRO、Fishr）→ meta-learning（MLDG，低優先）→ FM＋PEFT（Prithvi-EO-2.0、SatMAE、Luo 2025 ISPRS、CrossEarth）；綜述：AI Review 2024 DG for semantic segmentation、Neurocomputing 2026 RS DG survey。
2. 相鄰範式（不得稱 source-only）：test-time augmentation／episodic single-image TTA／online-continual TTA／SFDA（Tent、MEMO、Niloy ICASSP 2024）；警告：Prithvi 用 LayerNorm、稀少油像素 entropy 崩塌、同事件相鄰 patch 的 episodic vs continual 與事件間 reset。UDA：Cui et al. 2025 JAG 139:104479。
3. 油污跨區域實證：Yan et al. 2026 Frontiers Env. Sci. 14:1909612（SAR 多來源→中國沿海）；MORP-Synth arXiv 2512.02290（preprint，SAR 地中海→秘魯）。
4. 油污成像物理／特徵：Sun & Hu 2016 GRL（glint 門檻，MODIS/VIIRS）；Sun et al. 2024 MPB 206:116777（S2/L8/L9，glint 特徵）。
- 距離量化：Ben-David 2010（ML、AISTATS）、Gretton 2012 MMD、Székely & Rizzo 2013 energy distance；LEEP/LogME 僅次要。

### 5. 論文 outline 修改
- Objective 加第二條：分析來源資料特性與遷移表現的關聯（機制性、探索性）。
- Input 拆分：訓練資料＝海外標註影像；推論輸入＝台灣事件影像。
- 任務三層：operational target task（通報 ROI → 整個 ROI 機率圖）／current controlled evaluation（oracle GT-centered 含油 patch）／未驗證項（ROI 級搜尋、FP rate、無油景拒絕）。
- source-only 嚴格定義：台灣影像不參與訓練、增強設計、超參數、early stopping、source 選擇。

### 6. 第一步
Phase 1 設計卡（凍結共變數清單與 primary 距離規格）→ 動工。本共識**不授權**任何執行。

## 少數意見
無實質分歧。Codex 註明：R2 blocker②（目標端有效 n=2）只能管理、不能靠文字解決，唯有新增獨立事件能解除。

## 後續驗證點（Phase 1 設計卡動工前必須凍結）
MMD biased/unbiased estimator；每 block 最大 embedding 數與抽樣 seed；kNN coverage 的 k、reference radius、class balancing；water 向量缺失時日期/cluster 排除規則與缺失率報告；cloud/shadow/255 的實際資料語意驗證；acquisition-cluster union-find edge 規則。Phase 2 任何訓練需使用者另行授權。

## 試行評估附錄
- 首輪分歧點：Claude 原以 OOF 逐景混合模型為來源端主要歸因 → 降為 supplement（姊妹景洩漏）；Codex 原 H1「五個距離多數決」→ 改為單一 primary 距離。
- 收斂輪數：3；實際返工：無。
- 對話原文：R1–R3 prompt 與 Codex 回覆見附件目錄 `attachments/20261001_GoM台灣遷移機制分析與DG_survey方向/`（r1/r2/r3_prompt.md、r1/r2/r3_out.txt、claude_r1_own.md；路徑相對於本檔所在的 `.agents/decisions/`）。
