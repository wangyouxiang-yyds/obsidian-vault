# P32 Stage III bug-fix 補充卡（addendum）

狀態：FROZEN bug-fix addendum（2026-10-29 14:13）；設定不變，只修程式。

## 1. 隸屬

- 隸屬 [[20261029_P32_StageIII凍結addendum]]（該卡與 [[20261029_P32_光學EUDA_GTROI全流程設計]] 均未改動，sha 被 launch 腳本核對）。
- Stage III config 凍結 commit `61034a1`（不變）。

## 2. 事件

隔離 GPU smoke 第一次在 Stage III 第一批 target 讀取失敗：`RasterioIOError: TIFFReadEncodedStrip() failed`（`20230918_L9_50QRL` B01）。同檔單獨整檔讀取正常，非檔案損毀。

失敗產物保留於 `/home/alanyh/oil_dataset/new/_euda_gtroi_pipeline_20261029/_smoke_failed_fdshare`；**不得列入正式 run discovery／registry**。

## 3. 根因

- P32 新增的 `load_probe` 在主程序透過 `deeplab_adapter` 模組層 rasterio 快取讀 270 窗，handle 留在快取。
- 之後 DataLoader 以 4 worker fork，繼承同一 GDAL dataset 與共享的檔案 offset，並行讀取互相干擾（也可能靜默讀錯）。
- 影響範圍：P31、Stage I 未在 fork 前於主程序開檔；Stage II 主程序 audit 用 `with rasterio.open`（已關閉）。**phase A 結果不受影響。**

## 4. 修正

- commit `ee9610f`，只動 `main/euda_stage3.py`（`load_probe` 讀完後關閉並清空快取）＋新測試（真 LZW GeoTIFF、快取為空、2 worker 讀到的影像與 probe 逐元素相等）。
- 全部回歸測試通過（27＋9＋12＋6）。
- Codex 程式審查 R4 APPROVE（.98）。
- `launch_files_stage3.sha256` 已重產並含於 `ee9610f`。

## 5. 重跑 smoke

位置 `/home/alanyh/oil_dataset/new/_euda_gtroi_pipeline_20261029/_smoke`，7 項檢查全過：

| 項目 | 結果 |
|---|---|
| q/q0 第 5 步 | 1.014 |
| q/q0 第 10 步 | 0.980 |
| cache | 180 窗 |
| 油比例 | .029 |
| 峰值顯存 | 11.5 GiB |

## 6. 開跑與登錄

- Phase B 於 2026-10-29 14:13 以 commit `ee9610f` 開跑（tmux `p32_phaseB`）。
- **registry 的 P32 Stage III 訓練碼須記為 `ee9610f`，不是 `61034a1`。**
- 設定（yaml、種子、步數、guard、post_tests）全部不變。
