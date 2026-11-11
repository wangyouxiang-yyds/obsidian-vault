# P32 Phase C 延後補充卡（addendum）：與 P33 聯合評估

狀態：FROZEN（2026-10-29）。隸屬 [[20261029_P32_光學EUDA_GTROI全流程設計]]；該卡與 [[20261029_P32_StageIII凍結addendum]] 未改動。

## 1. 決定

使用者於 2026-10-29 選擇方案 2：P32 phase C（凍結 registry、讀臺灣 GT）延後，等 P33（IntraDA-style entropy 圖對照組，見 [[20261029_P33_光學IntraDA對照組設計共識]]、[[20261029_P33_光學IntraDA對照組事前卡]]）三 seed 全部跑完並封存後，P32＋P33 用單一聯合 registry、單一 evaluator、臺灣 GT 只讀一次。

## 2. 本卡只改變兩件事

- (a) P32 的 GT 開封時間點延後。
- (b) registry 由 P32 單獨改為 P32＋P33 聯合。

P32 的主指標、門檻（±.03、≥2/3 seed）、method failure 定義、比較對象（matched P29 source-only S1）、checkpoint（Stage III last）全部不變。

## 3. 聯合 registry 角色

- source_only_s1（P29）
- uncertainty_s1／uncertainty_s3（P32）
- intrada_entropy_s1／intrada_entropy_s3（P33）
- fullgrid_adv_s1（P29 歷史參照）

每角色記錄訓練 commit 與訓練程式 sha（以 git show 核對）。聯合 evaluator 需另有 regression test 證明 P32 判定邏輯不變，並在讀 GT 前另行審查、凍結。

## 4. 理由

兩組都在未看結果下完成，P32−P33（+U 這一步）比較才乾淨；避免 P33 在看過 P32 結果後才設計／執行。

## 5. 現況

P32 phase B 已於 2026-10-29 約 15:09 完成（三 seed 5000 步、guard 無越界），臺灣 GT 至今未讀。
