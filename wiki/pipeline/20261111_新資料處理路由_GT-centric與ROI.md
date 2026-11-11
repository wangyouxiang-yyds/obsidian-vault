---
date: 2026-11-11
type: pipeline
tags: [new-data, sen2like, gt-centric, roi, data-quality]
---

# 新資料處理路由：Sen2Like → GT-centric／ROI

本頁是**判斷新資料進哪條研究流程的入口**，不改現有 split、mask、模型或已凍結實驗。使用者所稱 data-centric 在這裡指以標註和資料品質為中心整理的 **GT-centric 256 窗**路線；若日後要把 data-centric 定義成另一套處理，需再明確定義。

## 共用入庫（兩條路線都先做）

1. 保存來源產品 ID、衛星、拍攝 UTC、tile／path-row、Sen2Like 真正輸入與處理 route 的證據。若僅日期／tile 命中，登錄為**候選**，不填已確認事故 ID。
2. 由 Sen2Like 產生八波段單波段 TIFF；以 B02 格網建 VRT，檢查波段順序、CRS、transform、尺寸、有效像素及 DN/scale/offset。來源：[Sen2Like](sen2like.md)、[VRT 前處理](VRT_pipeline_01_前處理.md)。
3. 保存 QGIS GPKG 或 LabelMe JSON 原始標記，對齊 VRT 後產生 mask；檢查油、已標背景、未知、無效像素語意與 mask hash。來源：[標記流程](annotation_workflow.md)。
4. 更新 `full_band/manifest.csv`，記 scene ID、檔案存在性、dirty／blur、來源與 QA；風／海流四欄與 ERA5 分開登錄且標明方向、時間、位置和配對旗標。來源：[資料集清冊](../datasets/20261111_資料集現況與物理標記清冊.md)。
5. 先決定該景是 source train 候選、source val/test 候選或臺灣等外部測試；檢查同地點、同日 footprint、同來源報告和 ROI／256 窗重疊，**不要因新增了檔案就自動加入任一正式 split**。封存測試資料尤其不可用於選模。[來源分折稽核](../datasets/20261111_來源分折與封存測試稽核.md)。

## 選哪條派生路線

| 情境 | GT-centric／資料中心路線 | ROI 3840 路線 |
|---|---|---|
| 用途 | 檢查標記、油區形態、含油與背景取樣，以及與舊 P12/P14/P15 可比的歷史設定；不是目前跨海域主基線。 | 目前的自然比例 baseline 與後續模型／資料政策研究；評估更大範圍的背景誤判、漏判。 |
| 取窗 | 油連通區 bbox：小區中心 256，大區 256 格鋪磚；另由**指定版本**規則取背景窗。 | 由油區分組錨定 3,840×3,840 ROI；ROI 內 256 窗，TRAIN 步距128。P44 不挑背景；其他配比實驗另有卡。 |
| 依賴標記 | 取窗與評估範圍都由 GT 決定。 | ROI 位置仍由 GT 決定，故也不是全景盲選區。 |
| 新資料進入條件 | 完整標記、來源角色及 split 決議後，重建該版本 patch 座標並驗證 mask hash。 | 同上，且要驗證 ROI 分組、邊界、覆蓋、窗數、各來源 TRAIN/VAL 清單及 purge；不能直接沿用 GT-centric 座標。 |
| 報告方式 | 明列 256 GT-aware 覆蓋、測試名冊、seed、聚合方式。 | 明列 ROI 口徑、seed、checkpoint；臺灣分大林2張與其他22張。與舊 GT-centric 數值不能直接相減。 |

## 新資料的實際決策

- **只有影像、沒有可靠油標記**：先收來源／QA 與資料品質，不產生 GT-derived 訓練窗或 ROI，也不放進既有 test 名冊。
- **有標記，但來源、時間或地理對位不明**：先做 QA 和人工核對；標記可供診斷，是否入 TRAIN/VAL/TEST 另定，避免同源或空間重疊污染。
- **有合格標記與明確資料角色**：按研究問題產生 GT-centric 診斷清單、ROI 清單，或兩者；每份清單記腳本版本、mask/split hash、場景數、窗數及可回溯路徑。若只要進現行主線，優先按 P44 ROI 規格準備，但**新 ROI 尺寸、背景抽樣或分折更動都需新設計卡**。
- **外部測試景**：保持 test-only。任何後續 ROI/patch 生成只為凍結後評估，不能用其結果回頭選訓練政策或 checkpoint。

舊 [新增 Sen2Like 資料](add_new_data.md)保留轉檔操作史；[GT_expand pipeline](GT_expand_pipeline.md)保留 GT-centric 實作史；P44 規格與結果見 [P44](../experiments/20261106_P44_ROI自然比例基線_結果.md)。腳本和凍結清單以主線 repo 原件為準，本頁不是 launch 指令。
