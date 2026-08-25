# 全景移植 amendment 與論文最小誠實外部效度包

- 日期：2026-07-16
- 協商：Claude×Codex 三回合（R1 盲評 → R2 以實測 F1–F5 對抗 → R3 ACCEPT/BLOCKING=[]；R2 verdict=ACCEPT_WITH_AMENDMENTS conf 0.92）
- 觸發：使用者拍板「小圖維持 DeepLabV3、UPerNet 優勢留給大圖」後諮詢下一步；provenance 實測推翻 external-84 假設

## 實測事實（audit artifacts：scratchpad leak_check3.py / external84_check.py）
- F1 全語料 12 事件群，NOAA Atlantic 195 + GoM 120 = 89%；其餘 10 具名船難 1–8 景；含 S2/L8/L9 三感測器
- F2 350/355 test 景同 fold train 有姊妹景（332 同 tile 異日、53 同日）→ 355-scene Wilcoxon 獨立性違反，headline p 值高估；絕對 IoU 為群內內插
- F3 A-only 90 景零新事件 → 語料內不存在事件級 OOD
- F4 NOAA 集合非單一事件，需 tile×日期 union-find 子聚類（禁用模型輸出），真實獨立單位介於 12 與數百
- F5 相對比較仍 protocol 內有效，但姊妹洩漏對不同架構的加成未必對稱，「排名跨事件維持」未證明

## 收斂路線（RESTATE）
amendment → Stage 0（GPU）＋ provenance/grouped 再分析（CPU）並行 → manifest 凍結 → LOCO → 2026 prospective cohort 依 event/collection/sensor 軸定性。論文只先寫不變章節；DLV3 P/R 留 backlog；不跳過 Stage 0 直接 transplant。Stage 0/LOCO 的 GPU 啟動依 C2 待使用者明說授權。

## 外部效度包＝(a)+(b)
- (a) **grouped 再分析（必做，CPU）**：headline（Prithvi+UPerNet vs tversky）改以子聚類等權配對效應＋cluster bootstrap CI＋exact sign test 為主要 estimand，scene-weighted 降次要；感測器作 collection×sensor 分層
- (b) **LOCO 一次（GPU，待授權）**：預設 hold out NOAA GoM（source ~235 景較穩；codex conf 0.78，若使用者 2026 部署 target 為 Atlantic 則改 hold Atlantic，決定須在任何 LOCO 結果前落檔）。Final method 與 tversky comparator 各一 run，稱 **transport stress test**
- LOCO 前置凍結（blocking 條款）：子聚類只用 tile/footprint/日期/感測器；GoM 完全 target-blind（禁用於 early stopping/checkpoint 選擇/threshold/超參/hard-negative mining/augmentation 決策）；兩模型同 downstream 條件；單 seed 標 screening＋預寫 ambiguous-zone 加 seed 規則；無 size-matched control 不稱純 domain-shift penalty

## 措辭紅線（全域生效）
- external-84 / 單次 GoM holdout 永不稱一般 external / event-OOD validation
- 「ViT 全域 context 已確認」→「與全域 context 假說一致/支持」
- LOCO 差距不解讀為純 domain-shift penalty（混雜訓練量下降/姊妹景消失等）
