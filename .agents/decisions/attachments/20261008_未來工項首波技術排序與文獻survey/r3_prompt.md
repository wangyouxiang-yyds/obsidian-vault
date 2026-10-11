# 第 3 回合 — 主持方(Claude)回覆+候選稿 v2

## 我的立場
你 R2 的修改我**全部接受**(理由逐條如下),並補兩個我剛唯讀查到的事實。我假設你的 BLOCKING [1,2,3] = (1) Z 的定義與 source-only 合規、(2) B1 的有效樣本單位、(3) MADOS/SWAD 可支持的範圍;若不是請明說。
- Z:接受四點。我原本的 0.3–0.6× 確實是由臺灣線寬反推,違反 source-only,撤回。Ghiasi 的引用改為「支持廣尺度的整體影像增強值得單獨測」。
- B1 主指標改 cluster-macro;恢復外層海外 pseudo-target。接受。
- MADOS 兩層化、永久 holdout、只能算已標註負像素的指標。接受。
- B2 分 a/b/c,入口不綁 R 成功。接受。
- 「SWAD」改名為事後 checkpoint averaging 篩選。接受。
- PEFT 放 B4b 條件式;CrossEarth 不是 PEFT 證據(更正)。接受。
- 「null 95% band」改稱 three-seed empirical envelope。接受。
- 其餘第 7 節各點全數納入。

**新增事實(唯讀 grep `main/deeplab_adapter.py`,初查、未逐設定檔核對)**:
- G0-f:loss(Tversky/Focal)`ignore_index=255`;但資料端有 pixel_mapping,**原始未標註 255 在遠場被映成背景並受監督**(程式註解:「遠場 255→bg 監督全保留」),另有可選的 ignore-ring(油污 GT 外圈 N px 內的原始 255 改回 ignore)。現行各實驗是否開啟 ignore-ring 尚未核對 → 留在 G0-f。
- 訓練器**沒有 EMA**(先前因 decay=0 形同無效而移除),只存 best.pt 與續跑用 ckpt → checkpoint averaging 需新增週期性存檔。

## 候選稿 v2 —「先嘗試清單 × 工項」

### G0 評估底座(任何 GPU launch 之前完成;CPU+少量推論)
- **G0-a** acquisition-cluster 稽核與重切。連邊:同 product/acquisition id;footprint 或油污區空間重疊;日期 ±1 天;另以 ±3 天做 split 敏感度。報告最大 cluster、各 split 有效 cluster 數(留意鏈式連接)。產出 cluster-disjoint 的 GoM train/val。〔工項一(2)、四〕
- **G0-b** 兩層評估:內層=GoM acquisition-cluster val(B1 機制判斷);外層=GoM-only 訓練從未見過的海外來源(NOAA Atlantic/Pacific、10 個具名事故)作 pseudo-target,**事前固定分組,只用於確認方法是否只適合 GoM,不用於調設定**。
- **G0-c** 現行碼對照組 C × seed 42/43/44 → 各指標的 **three-seed empirical envelope**(不稱 95% band)。訓練時在 source-val 進入平台後每 2–5 epoch 存一次權重(不存 optimizer state)。
- **G0-d** 固定指標:cluster-macro 與 pooled Oil IoU、Recall、Precision;細線子集(每個 acquisition cluster 內先 pooled,再等權 cluster-macro)IoU、線寬比、boundary F-score。
- **G0-e 負資料**:d1=MADOS 已標註非油像素,scene/tile 層先切永久 holdout;只算 annotated-negative pixel FPR、油–特定 look-alike 混淆;標示為 preprocessing-shift 外部 stress test(ACOLITE Rayleigh vs 本案 Sen2Like;僅 S2);能由原始產品 ID 重走本案 pipeline 更佳。d2=完整審閱、有有效水域面積分母的無油 scenes/regions(海外或臺灣),才支撐 scene FPR 與 FP area/km²;臺灣 342 景核實由使用者/標註方進行,只用於最終臺灣驗證,不用於選方法。**負資料到位前,任何項目不得主張誤報率改善。**
- **G0-f** 核對現行各設定如何把原始 255 送進 loss(遠場 255→BG 的弱背景假設可保留為既有 baseline 口徑,但不得稱確認負類,不得供 B3 當確認乾淨背景)。
- **G0-g** 核實 92 景 polygon-only(海外 90)。〔工項二(1)〕

### Track A 分析型交付(CPU/推論;與 G0 並行)
- **A1** raster mask 形態(連通區、線寬、破碎度)+六波段油–鄰近水反差,分感測器/事件。〔工項二(1)〕
- **A2** 清冊補 ERA5 風速、耀光幾何;只作分層與機制稽核,不進模型。〔工項一(1)〕
- **A3** 敏感度測試(稱「敏感度」,不稱特徵因果歸因):固定權重、source held-out;少數可解釋擾動——逐波段遮蔽(註明為分布外輸入)、依感測器原生解析度/MTF 設計的模糊、亮度偏移;依線寬/面積/感測器/耀光分層。〔工項一(3)〕

### Track B 模型實驗(GPU)
- **B1 細油污機制實驗族(C/Z/R 共用對照;各 3 seeds;先 C、Z,再 R)**〔工項四優化、二(2)、一(3)〕
  - **Z = zoom-out/field-of-view 增強**:讀較大視窗再降採樣回 256。假說:「較小相對尺度+較大背景 context 是否有利於細線」(同時改變 FOV、背景量、occupancy、空間頻率,**不得解釋為純粹補細樣本**;要單獨操弄形態尺度留給 B3 的 object-only 縮放)。
    - 倍率與機率**只由 source-train 與感測器物理決定,事前凍結**。提議規則:s ~ log-uniform[s_min, 1],s_min = (最粗原生解析度 30 m = 3 px) ÷ (source-train 線寬中位數),並以可讀視窗上限封頂(不小於 0.25);施作機率 0.5。臺灣線寬只作研究動機揭露,不參與定值。
    - mask 重採樣規則事前選定並固定:先報告 nearest/area-fraction/重新 rasterize 三法各造成多少 oil component 與正樣本 patch 消失,依 source-only 準則選一;細線消失後的 patch 處置(保留為背景/丟棄)同時凍結。
  - **R = P23 入口 2× 重訓**(Prithvi 預訓練權重微調;成本依 P23 卡)。
  - 判準:主指標=**cluster-macro 細線 IoU**;次指標=原 52-patch pooled IoU(與舊卡可比)、線寬比、precision、recall(皆先 cluster 內聚合)。有效=3/3 seeds 配對同方向、多數可評估 cluster 同方向、且配對平均超過 max(P23 卡的實質門檻, C 的 three-seed envelope);全體 source IoU 與 recall 的護欄沿用 P23 卡。P23 卡依 G0-a 重切與 cluster-macro **修訂後才凍結**。臺灣只描述。
- **B2 局部高解析旁路(分閘)**〔工項四優化〕。入口:B1 後 source 細線仍有穩定寬度膨脹(不要求 R 成功)。
  - B2a:1/4 淺層六波段 stem+skip,其餘不變(假說=保留局部高解析資訊,不宣稱光譜雙支路)。
  - B2b:B2a 後仍膨脹才加 PointRend 式邊界細化。
  - B2c:A1/A3 證明低油–水反差是獨立失敗 strata 才加顯式反差特徵。
- **B3 有條件的 mask copy-paste(含 object-only 縮放)**〔工項二(2)〕。前置 A1、G0-f、G0-g。背景只能用人工確認或可信規則確認的水域,不得用原始 255 當乾淨背景。合理性閘=六波段局部油–水反差 MMD(kernel/bandwidth 事前固定)+反射率合法範圍+接縫梯度+分感測器+連通區消失率。兩組:海外油×海外背景(論文可用)/海外油×**已核實無油的**臺灣背景(target-informed 計畫支線,不進論文 source-only 主張)。
- **B4a 事後 checkpoint averaging 篩選**(不稱 SWAD;用 G0-c 的存檔;averaging window 事前固定;在固定的外層 source 分組上判;有效才考慮正式 SWAD recipe)。
- **B4b PEFT(條件式第二波)**:B1+B4a 後若見明顯 source-train→held-out 落差、encoder 漂移或跨 seed 不穩才做;單一預註冊設定(如 attention q/v LoRA、固定 rank);成功=外層 event-macro IoU ≥ +0.02,或不劣於全量微調超過 0.01 且可訓練參數降 ≥90% 並有實測顯存/時間改善。文獻:Rein(CVPR 2024,都市 VFM)、Marti-Escofet(arXiv preprint,Prithvi+LoRA);CrossEarth(TPAMI 2025)是 DG 而非 PEFT 證據。
- **B5 look-alike 多類輔助訓練**:G0-e 量到實質誤報問題後才做;只能用 MADOS 非 holdout 部分。
- **B6 source-only 校準與固定操作閾值**:負資料(d2)到位後;CPU-only。〔工項五〕

### 不做/延後
形態 SM/L 硬分組;DADS-SAM B/C;clDice/Skeleton Recall;Tversky/class weight/閾值再掃;UDA/pseudo-label/TTA;擴散/GAN 生成;環境資料當模型輸入;把 P24 當證據。

### 論文 outline 建議(給使用者,非定案)
候選貢獻改述為「source 資料中稀有細碎油污的表示/解析偏差之診斷,與經多光譜合理性約束的形態(尺度)平衡擴充」;B1/B3 有 source 端正證據前不寫成既定貢獻。

## 請你回答
1. v2 是否已處理你的 blocker 1、2、3?若否請指出還缺哪一句。
2. Z 的 s_min 規則(感測器最粗原生解析度 ÷ source 線寬中位數,下限 0.25,機率 0.5)你是否接受?若不接受請給一個同樣只靠 source/文獻的替代規則。
3. 有沒有任何一項你認為排序仍然錯。

---VERDICT---
STANCE: ACCEPT | REVISE | OBJECT
BLOCKING: [編號清單,空=[]]
RESTATE: <一句話複述對方目前最強論點>
CONFIDENCE: HIGH|MEDIUM|LOW
---END---
