# 協商議題(第 1 回合,盲提案):如何科學性地解釋/分析「GoM 訓練的油污分割模型遷移到台灣表現較好」,以及該 survey 哪些方法

你是 Claude Code 的對等夥伴,依 vault `.agents/consensus.md` v2.1 協商。本輪是**第 1 回合盲提案**:請獨立提出你的方案,我(Claude)暫不透露我的傾向。請用**繁體中文**回答。

## 使用者的問題(原話整理)
使用者正在寫論文、做 survey。實驗下來發現「只用墨西哥灣(GoM)資料訓練」的模型用在台灣影像上表現較好,但光靠實驗結果不是直接證明。使用者想找:
(A) 能把「台灣海域特性」與「GoM 海域特性」關聯起來的**分析方法**(海況、海面、海洋/影像特徵),用來科學性解釋為何 GoM 模型在台灣較好;
(B) 量化關聯性 / 量化 domain shift 的方法;
(C) 「只用海外資料訓練、套用到沒有任何目標訓練資料的新海域」這類 domain generalization 方法,值得 survey 的方向。

## 使用者論文 outline(摘要)
- Motivation:已通報油污事件只有大致位置與時間,需從衛星影像確認實際範圍;台灣可靠標註事件太少,不能只用台灣資料訓練;現有研究多在相近分布下訓練測試,好表現不代表能泛化到台灣;利用海外標註事件補足。挑戰:不同海域的海況、氣候、背景干擾、油污外觀造成 domain shift。
- Problem definition:Input = 統一波段對齊的海外標註多光譜油污影像(無台灣);Output = 台灣影像的像素級油污機率圖與二元 mask;Objective = 評估僅用海外事件訓練的模型能否遷移至台灣,以 Oil IoU 與 Recall 衡量。

## 技術背景(事實,已查證)
- 模型:Prithvi-EO 300M encoder + DeepLabV3 ASPP head,Tversky loss(α0.3/β0.7),256×256 patch。輸入 6 波段(Sentinel-2 B02/B03/B04/B8A/B11/B12 及 Landsat 對應波段;無熱紅外)。感測器混 S2 / L8 / L9。
- 語料 355 景、名目上只有 12 個事件群:NOAA Gulf_of_Mexico collection 195 景(S2 167、L8 28,月份 2–11 月,4–7 月最多);NOAA Atlantic 120 景(S2 83、L8 21、L9 16);其餘是 8 個具名船難事件(希臘 Agia Zoni II、所羅門、模里西斯、斯里蘭卡、美國喬治亞、秘魯、千里達、NOAA Pacific),每個 1–8 景。GoM 是單一 NOAA collection(單一標註來源);NonGoM 是 NOAA Atlantic + 多個小事件拼接。姊妹景洩漏嚴重(350/355 測試景在同折訓練有同群姊妹景)。
- 台灣固定測試集只有 **2 個事件**:
  - 天使輪 2023(高雄/台南外海,7 景、4 個日期 8–9 月、L9×4 + S2×3,小面積淡油膜,每景油像素中位數約 7k);
  - 2026-05-22 事件(2 景、L8×2、5 月,大片油膜約 114k–128k px;但兩景是相鄰 tile 拍到同一片油,油 90%/100% 落在重疊區 → 實質 1 次觀測)。
  - 主端點 = event-macro Oil IoU(同日 mosaic → 日期等權 → 兩事件等權)。測試 patch 是以 GT 為中心取的 137 個含油 patch。
- 關鍵實驗:
  - P14(各自全量):GoM-only(1188 oil patch)vs NonGoM-only(584 oil patch),台灣 pooled Oil IoU 0.4187 vs 0.2037(資料量不等)。
  - P16(patch 數對齊,兩組各 584 oil + 1180 BG,同協定 33,000 updates,seed42):event-macro GoM 0.4554 vs NonGoM 0.2812;**逐事件:05-22 +0.367、天使輪 −0.019**,依事前判準(兩事件各 ≥ +0.02)為 inconclusive。差異主要來自 05-22 大片油的 recall。
  - P15 seed 雜訊(GoM 同資料換 seed):05-22 事件 SD 0.048–0.069、天使輪 SD 0.015–0.022。
- Claude 粗查(景級,非 P16 實際清單):感測器不像主要干擾(05-22 是 L8;GoM L8 占 14%,NonGoM Landsat 占約 26%);季節也不像(兩者都集中 4–8 月)。NonGoM 也有大片油事件(希臘、NOAA Pacific 每景 >100k px),所以「GoM 大片油較多」並不明顯成立。
- 資源限制:單 GPU;每次訓練約 10–30 小時。研究者可取得的公開資料包含 S2/Landsat metadata(太陽/觀測角)、ERA5、MODIS/OLCI 水色產品、GEBCO 等。
- 相關檔案(可讀,唯讀;讀不到也沒關係,上面已足夠):
  - `/home/alanyh/.agents/institution/research/oil_spill_project_status.md`(P14/P16/P19 條目)
  - `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand/analysis/p5_taiwan_holdout/preregistration.md`

## 請你獨立提出(不要只列清單,要排優先順序並說明理由)
1. 針對 (A):你建議用什麼分析框架把「海域特性」和「遷移表現」連起來?具體要量哪些變數(物理/環境、影像光譜、油膜形態、標註、模型特徵空間)、從哪裡取得、怎麼分析?
2. 在台灣只有 2 個事件(其中一個實質 1 次觀測)的情況下,什麼樣的主張是科學上站得住的?什麼主張不能做?如何設計成可證偽的分析?
3. 針對 (B):量化 domain shift / 資料集距離並與遷移表現關聯的方法,哪些適合這個資料規模?
4. 針對 (C):domain generalization(source-only,不用任何目標資料)值得 survey 的方法族,尤其是遙測/多光譜/油污,以及跟 foundation model 有關的。
5. 文獻錨點:每個方向給 2–4 篇關鍵文獻。**請用 web search 核對**標題/年份/期刊並附連結;沒核對到的請標「未核對」。
6. 你認為最大的陷阱或我們最容易犯的錯是什麼?
7. 如果只能先做一件事(不需 GPU 優先),你會做什麼?

請在回覆最後附上:
---VERDICT---
STANCE: ACCEPT | REVISE | OBJECT   (對「這個問題的提法本身」:ACCEPT=問題可照此回答;REVISE=問題提法需修正;OBJECT=問題提法不成立)
BLOCKING: [編號清單,空=[]]
RESTATE: <一句話複述你理解的使用者核心需求>
CONFIDENCE: HIGH|MEDIUM|LOW
---END---
