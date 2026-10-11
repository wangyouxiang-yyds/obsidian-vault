# Claude 端 survey(2026-10-08;讀取層級:S=搜尋摘要、A=摘要頁、M=沿用既有 vault/memory 筆記)

| # | 文獻 | Venue(等級) | 與本案關係 | 層級 |
|---|---|---|---|---|
| 1 | Ghiasi et al. 2021, Simple Copy-Paste | CVPR 2021 oral(頂會) | copy-paste+大尺度抖動(LSJ);資料效率至多 2× | S |
| 2 | Kikaki et al. 2024, MADOS/MariNeXt | ISPRS JPRS(頂刊) | S2 多光譜海洋污染(含油污)174 景/1.5M 像素;用 VSCP copy-paste、EMA、TTA;Zenodo 10.5281/zenodo.10664073 | S(全文 403) |
| 3 | Toker et al. 2024, SatSynth | CVPR 2024(頂會) | 擴散模型聯合生成影像–mask 當增強 | S |
| 4 | Mayr et al. 2024 | CVPR Workshop(非主會) | mask→TIR 合成,全合成訓練在未見資料集 +7.1 | M |
| 5 | Juarez et al. 2025, MORP-Synth | arXiv 2512.02290(preprint,未見正式出版) | SAR;形態擾動 mask+label-to-SAR;地中海→秘魯 油 IoU +10.8 | A |
| 6 | Kim et al., DAKD | arXiv 2412.08116(preprint) | SAR 油污擴散式增強+蒸餾 | S |
| 7 | Wei et al. 2024, Rein | CVPR 2024(頂會) | 凍結 VFM+~1% 可訓練 token,DG 分割勝全量微調 | S |
| 8 | Gong et al., CrossEarth | arXiv 2410.22629;memory 記為 TPAMI 2025 | 遙感 DG 分割基礎模型,32 個跨域設定 | S/M |
| 9 | Marti-Escofet et al. 2025, PEFT GeoFM | arXiv 2504.17397(未見正式出版) | PEFT 提升未見地理區泛化;建議 UNet decoder、不用 metadata | A |
| 10 | Chen et al. 2023, ViT-Adapter | ICLR 2023(頂會) | plain ViT 加空間先驗分支做 dense prediction | S |
| 11 | Kirchhoff et al. 2024, Skeleton Recall Loss | ECCV 2024(頂會) | 細管狀結構 loss(保連通/召回) | S |
| 12 | Hu, Lu, Sun, Liu 2021, Optical RS of oil spills: what is really possible? | Journal of Remote Sensing(Science 夥伴刊,非傳統頂刊) | 光學油污的光學原理;反差取決於耀光 | S |
| 13 | Sun & Hu 2016 | GRL(頂刊) | 偵測油膜所需耀光門檻 | M |
| 14 | Jiao, Lu, Hu 2024 | RSE 308:114205(頂刊) | HY-1C/D CZI;DL+光譜–空間–幾何特徵做油污類型 | S(全文 403) |
| 15 | Sun et al. 2024 | Mar. Pollut. Bull.(領域刊) | S2/L8/L9 六波段油污分割;最近感測器 baseline | M |
| 16 | Gulrajani & Lopez-Paz 2021 | ICLR 2021(頂會) | DG 的 model selection 不得用 target;leave-one-domain-out | M |
| 17 | Ploton et al. 2020;Meyer & Pebesma 2022 | Nature Communications(頂刊) | 非空間切分會高估泛化;須空間/群組驗證 | S |
| 18 | Höhl et al. 2024, XAI in RS review | IEEE GRSM 12(4)(頂刊) | 敏感度/可解釋方法盤點(遮蔽、擾動) | S |
| 19 | Deng et al. 2021, Scale Aware Adaptation | WACV 2021(二線會議) | **反證**:主張單靠隨機縮放不足以解尺度不匹配 | S |
| 20 | Du et al. 2025, MS3OSD | JSTARS(二線刊) | 中心光譜分支+大範圍空間分支 | M |
