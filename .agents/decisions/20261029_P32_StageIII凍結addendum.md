# P32 Stage III 凍結補充卡（addendum）

狀態：FROZEN P32 Stage III addendum（2026-10-29）；開跑已由使用者授權（P32 A–C 一併授權）。第 7–9 步結案才填。

## 1. 隸屬與範圍

- 隸屬 [[20261029_P32_光學EUDA_GTROI全流程設計]] 第 5 節 Stage III（該卡未改動，其 sha 被 launch 腳本核對）。
- 由 `analysis/euda_gtroi_pipeline_20261029/build_stage3_config.py` 依共識產生 `main/experiments_EUDA_stage3_gtroi_20261029.yaml`。
- 本卡只登錄 phase A 實際產物，設計不變。

## 2. Phase A 結果（無臺灣 GT）

Stage I：三 seed 皆 trained、target_retries 0。GoM source-val 數字：

| seed | best_epoch | epoch100 mIoU | epoch100 oil |
|---|---|---|---|
| S42 | 91 | .7192 | .4711 |
| S43 | 64 | .7022 | .4379 |
| S44 | 79 | .6921 | .4191 |

Stage II：三 seed 皆 strong 180／weak 90，tie_rate 0。

| seed | cut 處 entropy | strong 預測油像素比例 | weak 預測油像素比例 |
|---|---|---|---|
| S42 | .0047 | .029 | .116 |
| S43 | .0049 | .033 | .118 |
| S44 | .0040 | .029 | .126 |

油像素比例為 Stage I 模型預測，非 GT。

## 3. Stage III 每 seed 綁定

| seed | Stage I best.pt sha256 | Stage II input_provenance sha256 |
|---|---|---|
| S42 | `29ef00becd0db19c40cf03877fa7d3db6b29c7b5455a0fe335f5ea47b42fa041` | `173016b4853fc2a56383174225b0d06f69608df1983501f2b077d248a805123d` |
| S43 | `7b279470f569474556f7b388fdc31587f2fc282055d6fce25a14f83697e3bf28` | `5867ab00536263d5c2c73b2f864bc34565e729cdd4d8af45130c78599f6e86ca` |
| S44 | `b517e5782ab73ea2f693ffdf6c1b48d1fbfaef2dcdc5a020695f81e50c7bf0d6` | `8478d11602614727b139fc6aac270e9fd4de62b0edb6af57d567d32334f4684d` |

（正本為 yaml 的 `stage3_plr.checkpoint_sha256` / `input_provenance_sha256`。）

## 4. 設定

| 項目 | 值 |
|---|---|
| method | `fixed_stage1_argmax` |
| EMA teacher | .99，只記錄 |
| λ_adv | .001 |
| D lr | 1e-4 |
| 優化器 | AdamW，lr0 5e-5（encoder ×`backbone_lr_mult`），wd .001，cosine eta_min ×.001 |
| batch | 16＋16 |
| 步數／終點 | 5000 步，取 last |
| 精度 | AMP |
| guard | q/q0 ∈ [.25, 4]，連兩次越界即中止；檢查步 100, 200, 500, 1000, …, 5000 |
| post_tests | 空（訓練不讀臺灣 GT） |

## 5. 開跑前

隔離 10 步 GPU smoke：`analysis/euda_gtroi_pipeline_20261029/gpu_smoke_stage3.py`，輸出至 `_smoke`，不讀 GT、權重不使用。

## 6. 註記

10-29 P33 共識（[[20261029_P33_光學IntraDA對照組設計共識]]）提議延後 P32 phase C、與 P33 合併單次讀臺灣 GT；**尚待使用者決定**，phase B 不受影響。

## 7. 結果

PENDING（結案再填）

## 8. 結論

PENDING（結案再填）

## 9. 後續

PENDING（結案再填）
