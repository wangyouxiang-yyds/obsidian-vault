# 第 2 回合 — 主持方(Claude)回覆+單一候選稿 v1

先揭露我在讀你 R1 之前寫下的獨立提案(盲提案對照),再給整合後的候選稿。請逐點審,結尾附 VERDICT。

## A. Claude R1 獨立提案摘要(寫於讀你回覆之前)
- T0 評估底座:GoM val 細油污子集(P23 的 IoU_thin)+海外留一來源 pseudo-target(Gulrajani & Lopez-Paz ICLR 2021;Ploton 2020、Meyer & Pebesma 2022 Nat. Commun.)。
- T1 **尺度/線寬平衡增強(zoom-out)**:現行增強 scale 僅 0.9–1.0;把既有油污縮到 0.3–0.6× 得到帶真實光譜的 5–10 px 油污。依據 Ghiasi CVPR 2021(大尺度抖動是 copy-paste 增益主要來源)。我自己找到的反證:Deng et al. WACV 2021 主張單靠隨機縮放不足以解 GSD 不匹配(UDA 脈絡,二線會議)。
- T2 P23(入口 2×),與 T1 共用對照;ViT-Adapter(ICLR 2023)/UNet 型 decoder 為後備(Marti-Escofet 2025,arXiv 2504.17397,未見正式出版,建議 UNet decoder)。
- T3 mask-based copy-paste(MADOS/MariNeXt ISPRS JPRS 2024 用 VSCP;Ghiasi 2021;SatSynth CVPR 2024 與 Mayr 2024 CVPRW 列第二階段);分「海外油×海外背景」與「海外油×臺灣背景(UDA 類)」兩組;貼油–鄰近水反差而非原始反射率。
- T4 敏感度測試+清冊補 ERA5 風/耀光角(工項一(1)(3),CPU/推論)。
- T5 PEFT/LoRA 微調 Prithvi(Rein CVPR 2024;CrossEarth TPAMI 2025;Marti-Escofet arXiv)。
- T6 DADS-SAM:不先做。不建議:細結構 loss、TTA、形態硬分組、現在上擴散生成。

## B. 兩份提案的一致與分歧
一致:評估底座先行;P23;copy-paste 需先盤點且設合理性閘;DADS-SAM/clDice/UDA/擴散/loss 再掃 都不先做;形態硬分組不當主貢獻;P24 只能描述。
分歧/各自獨有:
1. 你沒有列「zoom-out 尺度增強」;我沒有列 SWAD、look-alike 多類輔助、校準閾值、PointRend/SegFix。
2. 你把解析度(P23→decoder)放資料分布之前且為序列;我主張 zoom-out 與 P23 **同批、共用對照**。
3. 你把負景/look-alike 當第 3 項模型技術;我只把「核實負景」當前置交付。
4. 我列了 PEFT,你沒列。

## C. 針對你 BLOCKING [1,2] 的新資料(我假設=你「關鍵缺失」第 1、2 點;若不是請明說)
- **Blocker 1(lineage)**:我剛做了唯讀快查(P14 GoM R1 清單,景名解析日期/感測器/tile):GoM train 156 景/135 個日期、val 39 景/35 個日期;**val 39 景中 16 景與至少一個 train 景同一取得日期**,±1 天內 22 景,±3 天內 25 景。尚未查空間重疊,所以「同日」不等於「同一片油」,但足以證明現行 GoM val 不是 acquisition-disjoint。另:P23 的 52 張細線 patch 來自 14 景、其中兩景佔 36 張。→ 你的 blocker 成立,候選稿把「acquisition-cluster 稽核+重切」列為 G0-a,且**任何 GPU 實驗前必須完成**。
- **Blocker 2(確認負景)**:海外 438 景**全部**是含油景(348+90),背景 patch 取自含油景的未標註區(訓練口徑 255→BG);所以海外確認負景同樣是 0。這個缺口不能靠討論解除,候選稿的處理是:(i) 在確認負資料存在前,**所有項目不得主張誤報率改善**;(ii) 把負資料建置列為獨立工作包 G0-d。

## D. 單一候選稿 v1 —「先嘗試清單 × 工項」

### G0 評估底座(前置;CPU+少量推論)
- G0-a GoM/NOAA acquisition-cluster 稽核(日期±1 天+空間重疊,union-find)→ cluster-disjoint 的 source train/val/held-out;報告細線子集在各 cluster 的分布。〔工項一(2)、四〕
- G0-b 現行碼 baseline × seed 42/43/44(=後面 B1 的對照組 C)→ 每個指標的 same-config null 95% band。訓練時**密集存 checkpoint**(供 B4 事後用)。
- G0-c 固定指標:event/cluster-macro 與 pooled Oil IoU、Recall、Precision、細碎組 IoU、預測/GT 線寬比、boundary F-score;負資料到位後加 FP area/km²、scene FPR。〔工項四、五〕
- G0-d 負資料工作包:(i) MADOS 的非油類別(海水、濁水、浪/船跡、藻、泡沫等稀疏標註)先做輻射相容性檢查,通過才作為**像素級確認非油**的 source-only 評估集;(ii) 臺灣 342 景的無油核實由使用者/標註方進行,**只用於最終臺灣驗證報告,不用於選方法**。〔工項一(1)(2)、二(1)、四〕
- G0-e 核實 92 景 polygon-only(海外 90)能否成為可用正樣本。〔工項二(1)〕

### Track A 分析型交付(CPU/推論;可與 G0 並行,不需等 GPU)
- A1 raster mask 形態(連通區、線寬、破碎度)+六波段「油–鄰近水」反差統計,分感測器/事件。〔工項二(1)〕
- A2 清冊補成像時刻 ERA5 風速與耀光幾何;只作分層與機制稽核,不進模型。〔工項一(1)〕
- A3 敏感度測試:固定權重,在 source held-out 上做逐波段遮蔽、模糊(模擬原生 20/30 m)、亮度偏移;依線寬/面積/感測器/耀光分層。〔工項一(3)〕

### Track B 模型實驗(GPU;依序)
- **B1 細油污三組同批實驗(共用對照)**〔工項四優化、工項二(2)、工項一(3)〕
  - C:現行碼 scale 1(=G0-b)
  - Z:zoom-out 尺度增強(讀 2× 視窗降採樣回 256,倍率範圍與機率事前凍結)→ 檢驗「資料缺細樣本」
  - R:P23 入口 2× 重訓 → 檢驗「16 px token/輸出步距太粗」
  - 各 3 seeds。成本:C、Z 每 run 約 5–14 小時;R 依 P23 卡每 run 約 20–86 小時 → 先跑 C、Z,再跑 R。
  - 判準:沿用已收斂的 P23 設計卡 §6(ΔIoU_thin 三 seed 中位 ≥ +0.040 且三個皆 > 0、ΔW_thin ≤ −2.0 px、Δrecall ≥ −0.030、Δ全體 val Oil IoU ≥ −0.020 等),Z 與 R 各自對 C 配對;門檻取 max(卡上數值, G0-b null 95% band);在 G0-a 重切後的 source val 上判定。臺灣只描述。
  - 我主張 Z 併入而非等 P23 結果的理由:(1) 兩假說不互斥,序列做會把「R 失敗」誤讀成「細油污沒救」;(2) Z 只多 3 個便宜 run;(3) Z 是論文候選貢獻(形態平衡)唯一的直接檢驗,R 是架構/解析度變更,不是資料策略;(4) 10-06 的結論正是「細樣本太少/Tversky 偏召回/token 解析度三者分不開,要重訓才分得開」。
- **B2 高解析局部支路 decoder**(合併你的 #2 與 #6):Prithvi 保留,加 1/4 解析度的淺層 CNN stem(直接吃六波段=局部光譜/紋理支路)+skip,必要時加 PointRend 式邊界細化。條件:B1 的 R 成立 → B2 是較省算力的可部署形式;R、Z 都不成立 → 回 A1/A3 找原因再議。光譜反差特徵只在 A1 顯示低反差 strata 重要時才加。文獻:PointRend(CVPR 2020)、SegFix(ECCV 2020)、ViT-Adapter(ICLR 2023)。〔工項四優化〕
- **B3 有條件的 mask copy-paste**(Z 的延伸:把縮放後的油污貼到別的背景)。前置:A1、G0-e。合理性閘採你提的「synthetic–real 六波段局部油–水反差 MMD 不超過 real–real null 95%」。兩組:海外油×海外背景(論文可用)/海外油×臺灣背景(計畫交付,標為使用目標域影像,不進論文 source-only 主張)。〔工項二(2)〕
- **B4 SWAD/權重平均**:不另訓練,用 G0-b/C 組密集 checkpoint 事後評估;判準採你 R1 的(跨 seed SD −30%、worst held-out cluster IoU +0.03、cluster-macro 不降超過 0.01)。〔工項四、五〕
- **B5 look-alike 多類輔助訓練**:待 G0-d 量到可觀的誤報問題後才做(我方語料內證據是大油污欠分割、51 張大圖僅 2 張 over-fire;細油污的 precision 問題是畫胖而非 look-alike)。〔工項四、五〕
- **B6 source-only 校準與固定操作閾值**:待負資料到位;CPU-only。〔工項五〕
- **B7 PEFT/LoRA**:第二波候選;請你表態(放哪、或不做)。

### 不做/延後
形態 SM/L 硬分組(日後若重訪改連續加權/批次配額);DADS-SAM B/C;clDice/Skeleton Recall(目標錯位:我們是畫太胖);Tversky/class weight/閾值再掃;UDA/pseudo-label/TTA;擴散/GAN 生成(OSS-Diff 等列為 B3 之後);環境資料當模型輸入;把 P24 當證據。

### 論文 outline 建議(給使用者,非定案)
候選貢獻由「形態感知的資料**選擇**」改述為「source 資料中稀有細碎油污的表示/解析偏差之診斷,與經多光譜合理性約束的形態(尺度)平衡擴充」;在 B1/B3 有 source 端正證據前,不寫成既定貢獻。

## E. 請你回答
1. Z 與 P23 同批(共用對照)是否接受?若反對,請給出「必須序列」的理由。
2. #3 拆成 G0-d(評估)+B5(後做)是否接受?MADOS 作為像素級確認非油評估集,你認為主要風險是什麼?
3. #2 與 #6 合併為 B2 是否接受?
4. SWAD 以 piggyback 方式做是否接受?
5. PEFT 的位置。
6. G0-a 與「不得主張誤報率」的處理是否解除你的 blocker 1、2?若否,還缺什麼。
7. 候選稿還有哪裡你認為會讓使用者做白工。

(更正你 R1 兩處事實:P23 不是從頭訓練,是 Prithvi 預訓練權重微調;P23 卡估 6 個 run 合計 3–12 天單卡,不是 30–84 GPU-hours。)

---VERDICT---
STANCE: ACCEPT | REVISE | OBJECT
BLOCKING: [編號清單,空=[]]
RESTATE: <一句話複述對方目前最強論點>
CONFIDENCE: HIGH|MEDIUM|LOW
---END---
