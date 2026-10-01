# 第 2 回合:候選稿 v1(Claude 主持,合併 R1 雙方提案)

先回報:你 R1 說本地 sandbox 無法初始化、讀不到檔案;本輪也不需讀檔,所有內容都在下面。

我在看到你的 R1 前已獨立寫下立場(框架:把「海域」拆成可量測共變數;目標端模式檢查 + 來源端歸因 + 介入;Prithvi embedding 距離;陷阱:事後挑變數、標註來源干擾、GT 中心 patch)。和你的 R1 大方向一致。你補上而我採納的:P16 exact manifest、class-conditional 距離、coverage、標註 erosion/dilation 檢驗、LOEO pseudo-target 才能把「距離↔表現」連起來、source-only 嚴格定義、Prithvi 預訓練可能看過台灣、DomainBed 式強 ERM 基準。

## 候選稿 v1

### 0. 研究問題重述(採你的 R1)
在固定 P16 協定下,哪些可觀測的資料生成因素能解釋 GoM 訓練集對 2026-05-22 事件產生大幅 recall 優勢、卻沒改善天使輪?這些因素能否在其他獨立事件上預測 transfer performance?
「GoM/台灣」只是代理標籤;機制鏈 = 事件與環境 Z → 影像 X → 表徵 H → 預測,加上標註 A → Y、取樣 S。

### 1. 主張邊界(採你的 R1 可/不可主張清單,全文照用)並加一條:論文稱「downstream 未使用台灣資料」,不稱「模型從未看過台灣影像」。

### 2. Phase 1(不訓練;只用 CPU + 推論)= P16 exact-manifest audit
- 2.1 單位:事件 > 同日 mosaic;patch/pixel 是巢狀重複量測。所有置換/重抽以日期或事件為 block。
- 2.2 **共變數清單在計算任何台灣數值前凍結**:
  A 海氣:ERA5 成像時刻 10 m 風速/風向、Hs(前 6/24 h 摘要為次要);
  B 耀光幾何:太陽天頂、觀測天頂、相對方位 → 鏡面反射角;以 ERA5 風速代入 Cox–Munk 得 glint 機率(次要);
  C 水體背景:local-water 六波段反射率分位數(主要);MODIS/OLCI Kd490/Chl、GEBCO 水深(次要);
  D oil–local-water 逐波段條件式反差、spectral angle(主要);
  E 油膜形態/patch occupancy;
  F 標註尺度(mask erosion/dilation、boundary-tolerant 評估)。
- 2.3 距離:**主要距離只指定一個**= frozen Prithvi 最後一層 encoder embedding、source-only PCA、class-conditional(oil 與 local water 分開)MMD;另報 kNN coverage。次要:共變數的 energy distance、domain-classifier AUC(事件分組 CV)。
- 2.4 錯誤稽核:P16 GoM vs NonGoM 兩 checkpoint,FN 依 oil interior/boundary、component 大小、D 反差分層。
- 2.5 可證偽假設(事前寫死):
  H1 光譜/背景支持:05-22 的主要距離 d(GoM) < d(NonGoM);天使輪不設方向。**主要距離方向不符即拒絕 H1**(改掉你 R1 的「五個距離大多數」,避免事後挑距離)。
  H2 形態/occupancy:GoM 優勢集中於高 oil fraction、大連通區、低邊界比;分層後若在小淡油膜也同等改善 → 拒絕。
  H3 標註:erosion/dilation 系統性改變 GoM–NonGoM 差距;若排名完全穩定 → 不支持。
- 2.6 (次要、免費)來源端難度歸因:用既有 347 景 OOF 逐景 recall ~ A–E,事件/tile random effect。只回答「哪些條件讓油難分」,不回答遷移;因姊妹景洩漏,僅作支持性證據。

### 3. Phase 2(GPU,依 Phase 1 結果才啟動,每步需使用者授權)
- 2a paired multi-seed P16(GoM/NonGoM 各補 S43/S44),確認 05-22 差距超出 paired seed 變異(單臂 seed SD 不可替代)。
- 2b 介入:依 Phase 1 指認的共變數,把 NonGoM 重新加權/配對成 GoM 的分布,看差距是否縮小。
- 2c LOEO pseudo-target(探索性):海外事件輪流 held-out,先 frozen encoder + 輕量 head;event-level Spearman/permutation。⚠ GoM、NOAA Atlantic 各是 collection 而非單一事件,需先決定是否以 tile×日期 union-find 子聚類(禁用模型輸出),否則 n≈10 且極不平衡。

### 4. (C) survey 地圖
(i) 強 ERM + 公平 protocol(DomainBed、SWAD)— 第一基準;
(ii) 物理合理的 randomization/style augmentation(MixStyle、RobustNet、WildNet、CCDR TGRS 2024)— 最值得先試;
(iii) 多來源不變性/worst-group(GroupDRO、Fishr);
(iv) meta-learning(MLDG)低優先;
(v) foundation model + PEFT(Prithvi-EO-2.0、SatMAE、Luo 2025 ISPRS、CrossEarth);
(vi) **相鄰但非嚴格 DG:test-time adaptation**——只用待測影像本身、無標註、不需再訓練資料集;使用者原話「沒有再訓練資料集的方法」可能正指這類,納入 survey 但論文中必須標成 TTA,不得稱 source-only。
油污領域錨點:MORP-Synth(arXiv 2512.02290,SAR 地中海→秘魯)、Yan et al. 2026 Frontiers(多來源 SAR → 中國沿海)、Cui et al. 2025 JAG(跨事件 UDA)、Sun & Hu 2016 GRL(glint 門檻)、MPB 2024 S2/L8/L9 光學 DL(glint 當特徵)。

### 5. 論文 outline 修改建議
Objective 加第二條「分析來源資料特性與遷移表現的關聯(機制性、探索性)」;Input 拆成「訓練資料=海外標註影像」與「推論輸入=台灣事件影像(依通報位置取範圍)」;GT 中心 patch 寫進限制;source-only 定義(台灣不參與任何選擇)與 Prithvi 預訓練 caveat。

### 6. 第一步
Phase 1(2.2 清單先凍結 → 2.3/2.4/2.5)。

## 請回覆
1. 你 R1 的 BLOCKING [1,2,3] 分別是什麼?候選稿 v1 是否已解決?
2. 對 v1 的修改意見(尤其:2.3 單一主要距離的選法是否合理、2.6 是否該保留、2c 子聚類、(vi) TTA 是否該納入)。
3. 結尾附 VERDICT(STANCE 針對候選稿 v1)。
---VERDICT---
STANCE: ACCEPT | REVISE | OBJECT
BLOCKING: [...]
RESTATE: <一句話複述 Claude 目前最強論點>
CONFIDENCE: HIGH|MEDIUM|LOW
---END---
