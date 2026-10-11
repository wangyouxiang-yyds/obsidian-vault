# Claude R1 獨立提案(寫於讀 Codex R1 之前,2026-10-08)

## 排序
T0(前置,不是技術但所有技術都靠它):**不靠臺灣的評估場** = (a) GoM val 細油污子集(P23 已設計的 IoU_thin,52 patch)+(b) 海外留一來源 pseudo-target(GoM 訓練→Atlantic 156 景/具名事故)。依據:Gulrajani & Lopez-Paz ICLR 2021(DG 的 model selection 不能用 target;leave-one-domain-out)。工項四/五。
T1 **尺度/線寬平衡增強(zoom-out scale jitter,必要時細樣本過取樣)**。工項二(2)+論文貢獻。依據:Ghiasi CVPR 2021(LSJ 是 copy-paste 的主要增益來源);我方:增強 scale 僅 0.9–1.0、細碎 patch 3.1%、5 px 被畫成 11–14 px、05-22(17–21 px≈訓練中位 17.6)表現好。反證:Deng WACV 2021 稱單純隨機縮放不足以解 GSD mismatch(UDA 脈絡)。判準:GoM val IoU_thin,3 seed 配對。
T2 **解析度:P23(入口 2×)**,與 T1 共用對照;ViT-Adapter(ICLR 2023)/UNet 型 decoder 為後備。Marti-Escofet 2025(arXiv,未見正式出版)建議 UNet decoder。
T3 **mask-based copy-paste 融合**(工項二(2)必交)。依據:MADOS/MariNeXt(ISPRS JPRS 2024)用 VSCP;Ghiasi CVPR 2021;SatSynth CVPR 2024(擴散式影像-mask 聯合生成,列為第二階段);Mayr 2024 CVPRW;MORP-Synth(arXiv preprint,SAR)。分兩組:海外油×海外背景(source-only)/海外油×臺灣背景(UDA 類,另稱)。貼的是「油–鄰近海水反差」而非原始反射率(耀光/海況一致性,Hu 2021 J. Remote Sens.;Sun & Hu 2016 GRL)。
T4 **敏感度測試+條件欄(ERA5 風、耀光角)**:工項一(1)(3)。CPU/推論,必交。
T5 **PEFT(LoRA)微調 Prithvi**:Rein CVPR 2024、CrossEarth TPAMI 2025、Marti-Escofet 2025(arXiv)。DG 文獻一致指出全量微調傷 OOD。成本中、需改訓練路徑;排在 T1–T3 後。
T6 **以通報位置為提示的 SAM 類(DADS-SAM B/C)**:工項四敘事最貼,但零學習證據。
不建議:細結構 loss(問題是畫胖/precision,非斷線);TTA(非 source-only);再做形態「分組」訓練(P19/P22 無訊號);擴散生成現在就上(12 事件群、成本高)。
缺的資訊:92 景能否用;臺灣確認無油景;P23 null 校準(IoU_thin 的 seed 雜訊)。
