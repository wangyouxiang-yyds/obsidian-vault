---
date: 2026-07-09
type: experiment
stage: evaluation
status: completed
tags: [error-analysis, gt-quality, oil-detection, baseline, ssl4eo, extent-polygon-gt, false-negative, gt-expand, precision-recall, under-segmentation, correction]
---

# Baseline 錯誤分析：GT 過寬是 Pooled IoU 天花板的主因，方向二 Cascade 降級

> ⚠ **本篇原結論（GT 過寬為主因）已於同日被 precision/recall 量化實驗推翻，詳見文末〔2026-07-09 修正〕一節。**

## 1. 實驗背景

SSL4EO 預訓練三個配置（norm10k / frozen / bandfix）全部跟 from-scratch baseline 打平，沒有任何一個明顯超越（詳見 [[deeplabv3plus_358clean_overfitting_改善方向]] 中的 SSL4EO-S12 路線討論，以及 [[20260708_JM距離判別髒資料可行性診斷]] 附帶發現的 SSL4EO band_map 錯位 bug）。使用者提問「還能往哪 push、是不是已經到頂了」，於是先做一輪 error analysis，判斷目前 pooled Oil IoU ≈ 0.33 的瓶頸屬於「資料極限」還是「方法極限」，再決定下一步方向，避免盲目換模型或加大訓練量。

分支延續 B（[[GT_expand_pipeline]]，OIL_PROJECT_MutiBand_GT_expand），全程唯讀分析，不動訓練資料、不重跑訓練。

## 2. 方法架構

對 baseline（from-scratch，2026-07-01/02 三 fold，共 355 個測試 scene 的 gt_aware recon 結果，見 [[20260702_CV_358clean_gt_expand_進行中]]）做兩層分析：

```
第一層：per-scene 三角交叉分析
    per-scene oil IoU × 從 mask 算出的油汙像素數 × dirty 旗標 × 感測器（S2/L8/L9）
    → 找出 pooled 指標偏低的量化成因
第二層：目視 overlay 覆核
    挑 6 張油汙面積最大的 scene，看 overlay（青=GT / 黃=pred）
    → 判斷誤差主因是 FP（濫抓）還是 FN（漏抓），以及漏抓背後的具體原因
```

## 3. 分析範圍與分桶定義

| 項目 | 內容 |
|---|---|
| 資料來源 | baseline 3-fold gt_aware recon，測試集共 355 scene |
| 感測器分組 | S2 / L8 / L9 |
| 油汙面積分桶 | tiny (<500px) / small (0.5k–5k) / medium (5k–50k) / large (>50k) |
| dirty 旗標來源 | `dirty.txt` / `blur.txt`（見 [[20260708_JM距離判別髒資料可行性診斷]] 第 1 節） |
| 目視覆核張數 | 6 張最大油汙 scene（5 張差、1 張好對照） |

## 4. 量化結果

### 4.1 dirty 命中率

355 張測試 scene 中 **dirty 命中 = 0**。358clean split 在建立時已把手標髒 scene 全部排除（見 [[20260708_JM距離判別髒資料可行性診斷]] 第 1 節），因此目前的 0.33 天花板**跟手標髒資料無關**——問題出在乾淨資料本身。

### 4.2 感測器別 IoU

| 感測器 | Oil IoU |
|---|---|
| S2 | 0.421 |
| L8 | 0.417 |
| L9 | 0.396 |

三者接近，感測器不是主因。

### 4.3 油汙面積分桶 vs IoU（倒 U 形）

| 面積分桶 | 像素數範圍 | Oil IoU |
|---|---|---|
| tiny | <500px | 0.322 |
| small | 500–5,000px | 0.394 |
| medium | 5,000–50,000px | **0.456**（最佳） |
| large | >50,000px | 0.382（又掉） |

模型在中等面積油汙上表現最好；面積太小或太大都會掉。

### 4.4 pooled 0.33 vs per-scene 0.42 的落差來源

落差幾乎全由 **~51 張大油汙 scene** 造成。前 12 大油汙 scene 的 IoU 只有 0.04–0.15；其中一張油汙像素數達 100 萬、IoU=0.07 的 scene，單獨一張就能壓垮 pooled 分母（pooled 指標用總像素數加權，大 scene 的低 IoU 會不成比例地拖累整體）。

## 5. 目視覆核（6 張大 scene overlay）

瓶頸是 **FN（漏抓）**，而漏抓主要有兩種成因：

**(b) GT 過寬 extent-polygon（主因）**：GT 是人工粗略包絡多邊形，把周圍沒有油汙光譜訊號的水域也框進「油」的範圍；模型正確地只圈出可見油膜，因此被灌爆 FN。

決定性對照組：

| Scene | GT 緊鬆 | Oil IoU | 說明 |
|---|---|---|---|
| Gulf_20190428 | GT 緊貼可見油膜 | **0.753** | 標註緊，模型與 GT 高度吻合 |
| Gulf_20210607 | GT 比可見油膜寬 2–3 倍 | **0.134** | 模型精準描出 sun-glint 亮油膜，但 GT 框了大片沒訊號的水 |

兩張都是大油汙 scene，模型行為一致（只圈可見訊號），IoU 好壞差在**標註緊鬆**，不在模型能力。

**(a) 真的瀰漫薄油沒抓滿（次因，少數案例）**：例如 Gulf_20190907，IoU=0.040，屬於模型確實漏抓瀰漫薄油的情況，與 GT 過寬無關。

## 6. 觀察與結論

1. **方向二（detect-then-verify cascade）是錯的工具，判定降級**：cascade 的設計目標是砍假陽性（FP），但這裡的瓶頸是 FN + GT 過寬，硬做只會把 recall 再砍低，方向錯誤。除非後續在小/中油汙子集另外證實有 FP 問題，否則此方向擱置。
2. **pooled 0.33 天花板大部分是標註品質／指標假象，不是模型能力不足**：模型對「可見油汙」其實做得不差，per-scene 0.42 是相對誠實的數字（且它也仍被同一批鬆 GT 壓著，實際能力可能更高）。
3. **可直接寫成論文 finding**：optical 油汙 benchmark 的 extent-polygon GT 會系統性低估精準偵測器；pooled IoU 這類像素數加權指標，會被少數大事件、鬆標註的 scene 誤導，錯判模型品質。

## 7. 誠實揭露 / 侷限

目視覆核只看了 6 張（5 差 + 1 好對照），模式一致、好壞對照乾淨，但仍屬**質性判讀**，樣本數小。下方第 8 節「待跑硬化」是把這個質性結論量化驗證的待辦，尚未執行。

## 8. 後續改進方向

- [ ] 改報 per-scene 為主指標，或在「精標子集」上評估（避免鬆 GT 誤導 pooled 指標）
- [ ] 標註稽核：把最鬆的 NOAA High Confidence 大包絡 GT 挑出來收緊，或改為分層報告
- [ ] 真正 (a) 瀰漫薄油子集：才需要 recall 導向訓練（油汙加權 loss、更大 context）
- [ ] **待跑硬化**：對全 51 張大油汙 scene 算 pred precision（pred∩GT / pred 面積），若普遍很高，可用數字證明 (b) GT 過寬是主因，把第 7 節的質性判讀變成量化證據

## 9. 實驗檔案位置

| 類型 | 路徑 |
|---|---|
| Repo | `/mnt/backup/alanyh/oil_IR_Fullband/OIL_PROJECT_MutiBand_GT_expand` |
| Baseline gt_aware recon（3 fold） | `result-seg/CV_358clean_gt_expand/*/post_test_*_reconstruction/gt_aware_recon/`（per-scene 指標 + `overlays/` 子目錄） |

## 相關頁面

- [[20260708_JM距離判別髒資料可行性診斷]] — 同屬資料／標註品質診斷這條線；附帶發現的 SSL4EO band_map bug 是本次分析的直接動機之一
- [[deeplabv3plus_358clean_overfitting_改善方向]] — SSL4EO-S12 預訓路線討論，三配置打平 baseline 是本次 error analysis 的觸發點
- [[20260702_CV_358clean_gt_expand_進行中]] — baseline 3-fold 訓練紀錄，本次分析的資料來源
- [[20260702_波段選擇消融實驗規劃]] — 同一 baseline（組0）在波段消融實驗中的角色與指標定義
- [[GT_expand_pipeline]] — 本次分析所在的分支 B pipeline

## 10. 〔2026-07-09 修正〕結論推翻

**推翻的舊結論**：原筆記依 6 張 overlay 目視，判大油汙瓶頸主因是「(b) GT 過寬 extent-polygon，把無油的水框成油，pooled 0.33 天花板大半是標註/指標假象」。

**新證據（硬化量化，已跑完）**：對全 51 張大油汙 scene（oil_px>50000）重跑 baseline inference（每 scene 用所屬 fold 的 best.pt；sanity check：重算 IoU vs 原 csv 最大誤差 1e-4，完全重現），計算 oil 的 precision / recall：

- precision mean 0.80 / median 0.85
- recall mean 0.47 / median 0.42
- 37/51 precision>0.7、28/51 recall<0.5、23/51 高準低召（典型 under-segmentation）、僅 2/51 over-fire
- 極端例：Pacific_20211005 P0.99/R0.008、Gulf_20210607 P0.99/R0.134（模型只咬住最厚/最明顯的油，大片 diffuse 放掉）

**修正後結論**：

1. 大油汙 IoU 低完全是 recall 拖的，不是模型亂噴——是真實的 under-segmentation（漏抓）。
2. 結合使用者標註證詞（大油汙一律保守框、只框看得見的明顯油 → 未被預測的 GT 是真油），判定為真漏抓 (a)，不是 (b) GT 過寬。
3. 誠實界線：precision/recall 本身分不出 (a)/(b)（兩者都給高準低召）；(a) 的判定是靠使用者證詞，不是靠這組數字。
4. 方向二 cascade 擱置的證據更硬（51 張僅 2 張 over-fire，砍 FP 幾乎無事可做）。
5. 「pooled 0.33 是標註/指標假象、模型對可見油不差」一句作廢——那是真實 recall 天花板。
6. 「extent-polygon GT 低估精準偵測器」的論文 finding 收回（與保守標註證詞矛盾）。
7. 下一步 = recall 導向（更大 context/多尺度、沿大 slick 邊界取樣、油汙加權 loss）；Prithvi-EO-2.0（ViT 全域注意力，對大片均質 slick 的長距 context 對口）升為 live 候選。

資料來源：主線 session 2026-07-09，腳本輸出 `large_scene_pr.csv`（51 scene 的 precision/recall/IoU）。
