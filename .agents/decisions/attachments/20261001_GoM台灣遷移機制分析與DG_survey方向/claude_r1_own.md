# Claude 第 1 回合獨立立場(Codex 回覆前寫下,防止被錨定)
1. 框架:把「海域」拆成可量測共變數;三層證據:
   (i) 目標端模式檢查:05-22 / 天使輪 在 GoM vs NonGoM 共變數/特徵分布中的位置(可證偽預測:05-22 應較近 GoM、天使輪不應有差);
   (ii) 來源端歸因:347 景 OOF 逐景 Oil IoU/recall ~ 共變數,事件/tile random effect 混合模型(有統計力的唯一地方);
   (iii) 介入:依共變數重新加權/配對 NonGoM,看差距是否消失(需 GPU,最後做)。
2. 共變數:glint 幾何(metadata 太陽/觀測角 + Cox–Munk 用 ERA5 風速)、ERA5 風/波、背景水體反射率(影像自身 + MODIS/OLCI Kd490/Chl)、油像素逐波段對比、形態、標註特徵、感測器。
3. 距離:影像統計、Prithvi embedding(MMD / OTDD / domain classifier proxy-A)、patch 級 kNN 到 GoM vs NonGoM 的距離 ↔ 台灣 patch 級錯誤。
4. 主張:n=2 事件只能說「與…一致」的機制性描述;不得稱海域因果;事件≠海域。
5. DG 方法族:多來源混合/來源加權、風格/光譜增強(含物理 glint 模擬)、正規化類(IBN/whitening)、特徵統計擾動(MixStyle/DSU)、foundation model DG(CrossEarth、Rein)、test-time adaptation(只用待測影像本身,不需再訓練資料)。
6. 陷阱:看過台灣後才挑共變數(需事前列清單)、標註來源干擾、seed 雜訊、測試 patch 以 GT 為中心。
7. 第一步:CPU 共變數表 + 推論取 embedding。
