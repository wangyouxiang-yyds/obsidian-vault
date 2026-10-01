# 第 3 回合:候選稿 v2 = v1 + 以下修改(你 R2 的 8 項全部採納,另加 Claude 兩項)

先請你澄清:R2 的 `BLOCKING: [1,2,3]` 指的是 R1 原 blocker 1–3,還是 R2「v2 最小修改清單」的第 1–3 項?本輪請用**明確文字**列出 blocking 內容,不要只給編號。

## v2 相對 v1 的修改
M1 Phase 1 定位改為「retrospective preregistered audit(鎖定式事後分析)」;只有新台灣事件或事前鎖定的 overseas pseudo-target 結果才算 confirmatory。
M2 主要距離完整寫死(採你 R2 2.2 的 7 點):
  - Encoder = 同一個未經 GoM/NonGoM fine-tune 的 Prithvi pretrained checkpoint(fine-tuned encoder 距離列次要)。
  - Embedding = 最後層 spatial feature,依 class mask 加權 pooling,每 patch 每 class 至多一向量。
  - Local water = 油 mask 外擴 16 px 的環帶、扣除內側 2 px 緩衝;排除其他油 mask、陸地、雲/陰影、無效值(255);環帶有效水像素 < 200 則該 patch 的 water 向量排除,不臨時改環帶。數值是事前固定、非調參。
  - PCA 只在合併 P16 GoM+NonGoM source embeddings 上 fit,固定 32 維。
  - MMD = multi-kernel RBF,bandwidth = {0.5m, m, 2m}(m = source pairwise median distance)等權。
  - 階層:source acquisition-cluster ↔ target-date 先算,再日期等權、事件等權;patch 僅為計算樣本;不做 token/patch permutation p 值。
  - D = ½ D_oil + ½ D_water 為唯一 primary;component 只報告,不得推翻 primary。另報 kNN coverage。
M3 H1 改名「Prithvi representation-support hypothesis」;H1: D(GoM, T0522) < D(NonGoM, T0522),方向不符即拒絕;天使輪只作異質性對照。逐波段 oil–water contrast 是較接近光學機制的次要分析。
M4 Cox–Munk 改稱「water-surface normalized glint proxy」,僅作成像條件 exposure proxy,不入 primary 距離;註明 ERA5 尺度、油改變粗糙度、Sun & Hu 門檻來自 MODIS/VIIRS 不可直接套用。
M5 2.6 改名「既有 OOF 預測的影像條件—錯誤關聯稽核」,降為 supplement;不作 Phase 2 gate;報 sibling exposure;事件/acquisition-cluster 等權描述為主。
M6 Phase 2b 改為:主要 = GoM 與 NonGoM 都配到共同支持域 Q(固定 patch 數、update 數、ESS);次要 = 各自配到 05-22 共變數分布,標 target-aware diagnostic。報 ESS、最大權重、丟棄比例;overlap 不足 → 結論為「無法識別」。
M7 Phase 2c:primary 分組 = provenance supergroup(GoM collection、NOAA Atlantic collection、各具名船難事件);union-find acquisition cluster 只作敏感度;每 fold 固定 patch budget、取樣規則、update、感測器與前背景比例;優先 frozen encoder + shallow head 的 source→target transfer matrix,LOEO 作 robustness。
M8 TTA 獨立成「相鄰部署範式」,survey taxonomy 分 5 類:source-only DG / test-time augmentation / episodic single-image TTA / online-continual TTA / SFDA;附三警告(Prithvi 用 LayerNorm、稀少油像素的 entropy 崩塌、同事件相鄰 patch 的 episodic vs continual 與事件間 reset)。
M9 Survey 分四欄:正式 DG 方法 / 相鄰 UDA·TTA / 油污跨區域實證 / 油污成像物理;Cui 2025 歸 UDA、MORP-Synth 標 preprint、Yan 2026 歸跨海域實證、Sun & Hu 2016 與 Sun et al. 2024 MPB 206:116777 歸成像物理/特徵。
M10 Outline 拆三層:operational target task(通報 ROI → 整個 ROI 機率圖)/ current controlled evaluation(oracle GT-centered 含油 patch)/ 未驗證項(ROI 級搜尋、FP rate、無油景拒絕)。

## Claude 新增兩項
C1 **Phase 2c 可行性先講清楚**:專案 P5(2026-08-12)已算術證明此語料不存在可行的海域分法——訓練池 2,797 含油 patch 中 72% 在 GoM;除 GoM(約 2,006 含油 patch)與 NOAA Atlantic(約 445)外,其他海域依 basin 合計只有 11–166 含油 patch(太平洋 3 事件合計 166、千里達 122、希臘 47、印度洋 2 事件合計 11),拆成單一事件更少。故 transfer matrix 實際上只有 2 個夠量的 source supergroup,target 端小事件樣本也極少。→ 2c 在 v2 中明確標為「探索性、很可能不足以估計距離–表現關係」,不得作為論文主證據;論文的 confirmatory 路徑只剩「新增獨立台灣(或其他)事件」。設計時要寫最低樣本門檻(例如 target supergroup ≥ 30 含油 patch 才進矩陣)。
C2 **交付給使用者的定位**:本共識的產出是「survey 方向 + 分析計畫骨架」,不是 launch 授權;Phase 1 的設計卡(含 M2 數值)在動工前另行凍結,Phase 2 任何 GPU 步驟皆需使用者授權。

## 請回覆
1. 明文列出仍 blocking 的項目(若無則 BLOCKING: [])。
2. 對 C1、C2 的意見。
3. VERDICT(STANCE 針對 v2)。
---VERDICT---
STANCE: ACCEPT | REVISE | OBJECT
BLOCKING: [...]
RESTATE: <一句話複述 Claude 目前最強論點>
CONFIDENCE: HIGH|MEDIUM|LOW
---END---
