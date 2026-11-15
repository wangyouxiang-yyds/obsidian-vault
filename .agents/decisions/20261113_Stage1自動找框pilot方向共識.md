---
date: 2026-11-13
type: summary
tags: [oil-spill, optical-imagery, stage1, object-detection, roi, consensus]
validity: current
---

# 光學油污 Stage 1 自動找框 pilot：方法與觀察方向共識

**狀態**：Codex 與 Claude 收斂的**零訓練稽核及後續 pilot 方向**。本頁不是九步驟實驗卡，也未凍結資料、超參數、驗收門檻或授權 launch。承接 [[20261112_光學油污Stage1自動ROI方法方向]]；該頁記首次方向，本頁細化其待定的切圖、截段標註及觀察方式。

**議題**：使用者要求「先安排 Stage 1」，由 Codex 與 Claude 討論怎麼在新來的完整 Sentinel-2／Landsat 光學影像上，自動找出多處油污候選框，轉成多個 3,840×3,840 ROI，交給已訓練的 P44 Stage 2 分割器，並討論如何觀察成敗。

## 已核對的邊界

- P44 source TRAIN 298 景、VAL 37 景。主持方唯讀開啟兩者 335 張 VRT 的 metadata：全部為 10 m、10,980×10,980 的 Sen2Like 像素網格，分為 S2 270、L8 48、L9 17 景；VAL 中 S2 33、L8 3、L9 1，Landsat 分層僅 4 景，不能下強結論。L8／L9 原生資訊較粗，重採至 10 m 不代表資訊解析度等同 S2。TRAIN 最大已見 8 鄰接油區外框為 2,782×3,430 px；這不是未來油污的尺寸上界。來源：repo `analysis/p44_roi_sliding_global_20261106/P44_CARD_DRAFT.md`、`/home/alanyh/oil_dataset/new/_p44_roi_sliding_20261106/lists/` 與本次 VRT metadata 唯讀盤點。
- P44 既有 3,840 ROI 由 GT mask 產生，屬 oracle 定位，不能用來推論新影像。Stage 2 使用六波段原生 256×256、stride 128 視窗，本 pilot 方向保持其 checkpoint 與算子固定。P44 VAL 曾用於 Stage 2 選模，只能提供工程診斷；臺灣 24 景已反覆查看，不用來調 Stage 1。來源：[[20261112_光學油污Stage1自動ROI方法方向]]。
- raw mask 0 為油；P44 曾把 raw 255 映成分割背景，但 raw 255 的全景標註完整性未核實，不能據此認定所有無油視窗都可作 Stage 1 的真負例。來源：repo `analysis/p44_roi_sliding_global_20261106/train_p44.py` 與 institution `research/oil_spill_project_status.md`。此處的「負例未核實」是本案訓練前風險，不追溯改動 P44 歷史設定。

## 三輪協商與收斂

| 有效回合 | Claude 立場與阻擋 | 候選稿如何改變 |
|---|---|---|
| R1 | `REVISE`，3 項：誤認 S2／Landsat 需先統一 GSD；負樣本標註完整性；需要群組隔離的內部 dev。 | 初提六波段與「只標完整元件、截斷標 ignore」；另提 8,192 大 tile、無油景 FP 與 ROI 成本門檻。 |
| R2 | `REVISE`，4 項轉為 raw 255、抽查完整性、寬度／降採可見性、非學習基線等稽核事項。 | Codex 提供 335 張 VRT 皆為 10 m 的實查；Claude 撤回 GSD 阻擋及 stock YOLOv8 可直接 ignore 的假設。改為 tile 內可見截段標局部框，粗 4,096／stride 512 縮至 1,024，細 1,024／stride 512 原生；承認原 ROI 成本門檻自相矛盾。 |
| R3 | `ACCEPT`、`BLOCKING=[]`、confidence `HIGH`；Codex 也接受方向。 | 撤回先驗硬門檻 raw 255≤1%、mask 降採存活≥0.90、截段框 16 px、框間隙 256 px；把稽核列成實作前置，不當作方法方向阻擋。固定 RGB 首版、跨尺度 NMS 後依預定規則覆蓋框、同 AOI 比較端到端，成本須實測。 |

**停止原因**：R3 雙方接受同一設計方向，Claude 無 blocking，且準確複述 Codex 對標註、降採與成本的主要反對及處置；未出現需使用者仲裁的方向分歧。首次較長的 Claude 呼叫 240 秒逾時且無輸出，不算有效回合；之後 READY probe 成功，在同一 session 完成上述三輪。

## 共同候選：先稽核，再做一個 YOLOv8 pilot

1. **零訓練前置稽核**：盤點 TRAIN／VAL mask 原值及影像有效性交叉表，按來源、感測器、油區大小抽查標註完整性並列出人工確認的可靠負區；不足則補確認過的無油景。用 TRAIN mask 建可見截段框，統計元件尺寸、局部寬度、細長比、油像素／bbox 面積占比（稀疏度）、貼邊與截段大小；GT mask 降採存活率只作幾何診斷，不當模型召回上界。抽樣檢視實際縮放的光學 tile。另建 TRAIN 內部 proxy-group dev（來源／地點代理群，不稱已確認事件隔離），計算固定 RGB 拉伸常數、核對 split 與資料雜湊。
2. **首個訓練候選**：固定 B04／B03／B02，拉伸統計只取 TRAIN。從 mask 在每個 tile 的**可見油污截段**產生局部 bbox，包括穿越 tile 邊的截段；非常小或不可靠者先盤點，不得悄悄當負例。候選為 YOLOv8n、輸入 1,024；粗 tile 原景 4,096×4,096、stride 512、縮 4 倍，細 tile 原景 1,024×1,024、stride 512、不縮放。10,980 方景約 225＋441＝666 個推論 tile。粗尺度 `stride≤tile邊長−3,430`（512≤666）且末端貼邊，故對已見最大邊長的框，幾何上至少有一窗可完整包含；新影像超過 3,430 px 時仍需走截段與多 ROI。六波段偵測器留作後續受控對照，不與首版 RGB 同時改動。
3. **部署串接候選**：兩尺度框映回整景座標、全景去重，再用事前固定且決定性的 ROI 覆蓋規則，以 3,840×3,840 覆蓋預測框；128 px 邊界餘裕是待稽核的候選值。框任一軸超過單 ROI 可容納範圍時用二維網格鋪多 ROI，貼影像邊界只平移不縮小；相距很遠的油污可產生多個 ROI。最後交給固定 P44 Stage 2。
4. **SinGAN**：只作首版可行後的擴增消融；須固定真實正負資料量、負例數與模型設定，並檢查合成圖真實性及框標註。此頁不啟動 SinGAN。

## 怎麼觀察與比較

| 層次 | 指標與讀法 |
|---|---|
| 零訓練幾何閘 | 以 GT 框直接走框→ROI 規則，檢查已標 GT 油像素在候選 ROI 的涵蓋率；pilot 前置目標候選 ≥99%。若不過，先修 ROI 幾何，不訓練偵測器來掩蓋。 |
| Stage 1 主觀察 | 完整有效影像／事前 AOI 中，已標 GT 油像素落入至少一個自動 ROI 的比例；pilot 門檻初稿 ≥95%。另報油區覆蓋召回，依 **TRAIN** 定義尺寸分箱、局部寬度、影像邊緣及 S2／L8／L9 分層，不只報 pooled 數字。預測框 AP 可附報，不能代替 ROI 覆蓋。 |
| 成本與誤報 | 主成本 proxy：自動 ROI 內 unique Stage 2 256／stride 128 視窗數 ÷ 全景掃描視窗數，初稿 ≤50%。另報每景 ROI 數／面積分布，以及 Stage 1、Stage 2、合計 wall-clock、I/O、峰值 VRAM；未實測總成本不得稱較省算力。另取獨立確認無油景報每景 false ROI；樣本不足時僅描述，不立主門檻。 |
| 端到端 | 自動 ROI 與 GT-oracle ROI 的 Stage 2 結果在**同一完整有效影像／固定 AOI**加總 TP／FP／FN，任何 ROI 外像素算預測背景；OilIoU 差距 ≥−0.03 僅作 pilot 描述性參考，不把 P44 VAL 當獨立泛化測試。新獨立資料與來源代理群重疊稽核才支撐強泛化主張。 |

閾值、負窗政策、極小截段處置、RGB 拉伸、框去重、ROI 覆蓋規則及上述數字**均為討論初稿**：資料稽核後、任何訓練前須另寫並凍結九步驟卡 1–6。內部 dev 只在 TRAIN 內建立，用於調偵測閾值與候選規則；P44 VAL 只作一次工程讀數，臺灣 24 景不得反推這些選擇。若主覆蓋不足，依序查 GT 框→ROI 幾何、tile 與縮放、偵測器；不能因一次 `τ→0` 失敗就直接否定 YOLO。

## 殘餘風險與反證

- **最大風險**：可信負區可能不足，訓練配置或無油景 FP 評估需另設計。完整性抽查只能發現問題，不能證明全體景皆完整。
- 兩尺度的中間尺寸可能有召回接縫；大 tile 下採會使細絲光譜證據變弱。按局部寬度與尺寸報漏判、看原圖與縮圖，必要時另立卡換尺度。
- 細長或破碎的連通油區可能形成大而極稀疏的 bbox，浪費 ROI 並引入大量背景；先量油像素／框面積比，再觀察此類框的覆蓋與成本。全景 Stage 2 掃描只作成本與品質參考，不能稱數學上的品質下界。
- proxy-group dev 仍不等於已確認事件隔離；P44 VAL 曾參與 Stage 2 選模，臺灣 24 景也非全新封存測試。任何單 seed／既有資料上的效果只屬探索或工程診斷。
- 多個低信心假框可能吃掉 ROI 預算；全景 NMS 與 ROI 規則須在 TRAIN 內部 dev 固定，並用確認無油景及分層 FP 圖反證「高覆蓋但不可用」。

**下一步**：做零訓練資料稽核及 GT 框→ROI 幾何驗收；根據實測負窗與 tile 成本，才凍結正式 pilot 卡。沒有改動資料、P44 程式、Stage 2 checkpoint 或任何訓練程序。

## 協商 provenance

- 主持：Codex；對手：Claude Code CLI 2.1.323。請求 `claude-opus-5`、`--effort high`；每輪 JSON `modelUsage` 均回報 canonical model `claude-opus-5`，未回報可單獨驗證的實際 effort，沒有觀察到降級。session `b7b5bc17-bcd4-4ced-8070-9e9f3c66b408`。
- CLI 逐次回報（美元／token，皆為各次值，非累計）：READY probe $0.218092，output 4；R1 $0.3770165，cache-read 30,409、output 5,516；R2 $0.6418565，cache-read 30,990、output 7,349；R3 $0.905992，cache-read 37,551、output 6,380。有效三輪總費用回報值合計 $1.924865；連同 READY probe $2.142957。首次長請求逾時無輸出，無可核實費用，不算一輪。
- 原始 CLI JSON 暫存於 `/tmp/stage1_claude_{probe,r1b,r2,r3}_20261113.json`；本頁摘要保留各輪立場、修訂及數字。Claude 沒有啟動另一個 harness、agent 或訓練。
