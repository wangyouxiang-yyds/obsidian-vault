---
title: DADS-SAM GoM-only 正式 run（C 先、B 後）設計與九步驟卡 1–6
date: 2026-10-10
type: decision
status: frozen-1-6
tags: [decision, experiment-card, dads-sam, gom-only, taiwan-holdout, consensus]
---

# DADS-SAM GoM-only 正式 run（C 先、B 後）設計

**實驗編號**：無 P 編號（DADS-SAM GoM-only B／C）　**凍結日期**：2026-10-10
**事後補寫？**：C＝否（凍結時 C 沒有任何訓練或臺灣結果）。B＝**部分事後**：凍結前已看過 B 100-epoch pilot 的臺灣結果（9 景平均 Oil IoU 0.3179），見 §6 揭露。
**取代哪張卡**：無（先前沒有凍結的卡；`analysis/dads_sam_gomonly_20261005/PREFLIGHT.md` 是準備紀錄，不是卡）。

## 協商紀錄

- 議題：使用者 2026-10-10 指示「跟 codex 討論一下 C 的做法，你們兩個看過都 OK 就你這邊先測試跑看看，如果 OK 的話就正式執行，先跑 C 再 B，都調成 300 epoch 比照 A」。
- 主持：Claude Code（使用者當下入口）。對手：Codex headless，同一 session 四回合。
- Codex：requested／actual model `gpt-5.6-sol`、reasoning effort high、codex-cli 0.162.1、session `01a123c8-cf1f-7c52-b34e-8b21dcf9bbaf`、sandbox read-only（背景與程式全文內嵌，未讀本機檔）。累計 tokens used：R1 34,254／R2 66,330／R3 81,391／R4 88,245。無降級事件。
- Claude：Opus 5.5（主對話）。
- 逐輪：
  - R1（盲提案，11:08–11:13）：Codex 獨立提出 C 的介面、凍結／lr 政策、5-epoch preflight 九項判準、±0.05 門檻。STANCE REVISE，BLOCKING 5 項（鎖定 neck 規格、對照實作、啟動前鎖定 config 與門檻、preflight、正式 B 從頭並揭露 pilot）。**首輪分歧點**：(1) neck——既有實作只有 1×1 conv＋bilinear，Codex 要 1×1 conv→LayerNorm2d→bilinear→3×3 conv→LayerNorm2d；(2) 門檻中心——Codex 用 A 單次 0.3657。
  - R2（候選稿 v1＋程式全文，11:20–11:22）：Claude 採納 neck（已改程式並重跑接線檢查與梯度探針）、採納 preflight 與 δ；主張門檻中心改用 A 設定四次平均 R＝0.3862。Codex 接受 R，STANCE REVISE，BLOCKING 2 項（preflight checkpoint 不得自動刪除；同一 process 進 B 前須確認 C 的 CUDA 配置已釋放）。
  - R3（候選稿 v2，11:26）：兩項皆處理（checkpoint 保留並記 SHA256；runner 加紀錄，同 process C→B smoke 實測清理後殘留 64 MiB、B 完整跑完）。Codex **STANCE ACCEPT、BLOCKING []**，RESTATE 準確。Claude 同為 ACCEPT。
  - R4（preflight 結果，11:33–11:34）：preflight 12 項檢查 11 項通過，**原判準 2（GradScaler 零跳步）未通過**（跳了 2 步）。Claude 未啟動、回到協商，附上既有 A／A 設定／B run 的 scaler 結束狀態（全部低於初值）。Codex 同意原判準訂錯、接受替代的 2a–2c、認為不需重跑、維持 **STANCE ACCEPT、BLOCKING []**。詳見文末「Preflight 結果與判準 2 的事後修訂」。
- 停止原因：收斂（R3 收斂；R4 處理 preflight 未過的判準後再次收斂）。少數意見：無。反方終審：不適用（非不可逆決策）。
- 實際返工：有——C 的 neck 改寫（commit `6a59b58`）；新增梯度探針與 preflight 腳本（`79c4432`）；preflight 判準 2 事後修訂。
- Codex 列的失效條件（任一出現即本共識與其預定判定停止引用）：preflight 任一判準未過；正式 YAML／commit／資料清單／prompt／lr 分組／scheduler／類別語意／評估口徑與本卡不一致；正式 C 用了 preflight checkpoint 或 B 接續 pilot；臺灣資料在正式 post-test 前被用於調參、選 checkpoint 或設計 prompt；9/9 景、prediction manifest、重建 parity、類別順序或主指標聚合出錯；四個 A run 並非同清單／架構／評估口徑；C 的共用故障可能污染 B 卻仍引用 B；非有限訓練、未揭露的中斷續跑、輸出覆寫或跨 run 狀態殘留。
- 原文：repo `analysis/dads_sam_gomonly_20261005/consensus_C_20261010/`（`round{1,2,3,4}_prompt.txt`、`round{1,2,3,4}_stdout.txt`、`candidate_v{1,2}.md`）。

# 前半：開跑前凍結（1–6）

## 1. Objective
評估「把 SAM 系解碼端／整套公開 DADS-SAM 用在六波段光學油污分割」在 GoM-only 訓練、臺灣封存測試下，相對現行 Prithvi＋DeepLabV3 是否有值得後續投入的差異。屬使用者 2026-10-10 指示的 system comparison（10-08 首波技術排序曾把 DADS-SAM B/C 列為延後；本次依使用者當日明確指示執行，G0 評估底座尚未重建，列為限制）。

## 2. Problem
A 在臺灣的細油帶普遍圈得太胖（10-05/06 量測：預測線寬下限約 13–17 px，接近 16 px token 格距）。DADS-SAM（TGRS 2025）宣稱對油污分割有效，但原文是 RGB；能否搬到六波段、以及只換解碼端有沒有用，都沒有實測。B 的 100-epoch pilot 只證明「學得起來、GoM val 與 A 同量級」。

## 3. Research Question 與 preflight
RQ：在固定 P14 GoM R1 清單、seed 42、單次 run 下，(C) 把 A 的 DeepLabV3 head 換成「neck＋凍結 SAM decoder＋公開 class head」，或 (B) 整套換成公開 DADS-SAM 六波段適配，其臺灣 9 景平均 Oil IoU 與 A 設定相比，差異是否超出單次 run 的雜訊尺度？

已否定路線檢查（R6-1）：現況檔③的六條（無監督異常偵測、指數通道、波段子集、舊 ignore_to_bg、P7、P9）沒有同型方法。否。

### C 的 preflight（Claude 執行；只用 GoM train／val，不讀臺灣）
做法：`analysis/dads_sam_gomonly_20261005/preflight_C.py` 直接呼叫正式 trainer 入口 `training_module.train_model`（main_runner 呼叫的同一個函式），設定取自正式 C YAML，`epochs` 改 5、拿掉 post_tests。只包兩層不改 trainer 的包裝：
- (a) 建 scheduler 時把 `T_max` 釘在正式值 300（trainer 的 T_max 綁 `epochs`，不包就會變成 5）。以 results.csv 的 lr 欄對照正式公式逐 epoch 驗證（容差 1e-9）。
- (b) 記住 trainer 建出來的 adapter 物件，在訓練前複製一份初始 state，供訓練後逐 tensor 比對。
preflight 的 checkpoint **不刪除**，留在 `_dads_sam_runs/_preflight_C/<時間戳>/weights/`，best.pt 與 last.pt 的 SHA256、results.csv、逐項判準結果、設定來源都寫進 `preflight_C.json`；這些 checkpoint 不得再使用，正式 C 從官方預訓練權重、epoch 0 重新開始（正式 run 另開時間戳目錄，trainer 的自動續跑碰不到它）。

通過判準（全部滿足；門檻即 Codex 第 1 回合所提，以下標出兩處量測替代）：
1. 5 個 epoch 的 train loss、val loss 與所有 val 指標皆有限（trainer 把每個 batch 的 `loss.item()` 累加，任何一個 batch 非有限，該 epoch 平均就非有限）。
2. GradScaler **零次**跳步：last.pt 的 `scaler_state` 必須是 `scale==65536`、`_growth_tracker==855`、`growth_interval>855`（比「最多 5 次、epoch 2–5 不得再跳」更嚴）。證據是 scaler state：跳步會把 scale 減半並把 tracker 歸零，而 scale 每 `growth_interval`（預設 2000）個乾淨 step 才會加倍，總步數只有 855，所以 scale 仍是預設初值 65536 且 tracker 等於總步數，才代表一次都沒跳。`global_optimizer_step==855` 只確認步數，本身不能證明沒跳步（跳步時計數照加）。inf/NaN 梯度會觸發跳步，所以這一項同時涵蓋「梯度無 NaN/Inf」。
3. 梯度可達性：**替代量測**——trainer 不記逐 step 逐群組梯度，我不為此改 trainer；改用 `wiring_check.py --arm C --grad-probe`（與 trainer 相同的 fp16 autocast＋GradScaler＋unscale＋clip 1.0，兩張真實 GoM patch，4 個 optimizer step）。已執行，結果見附錄 D：第 1 步只有 class head 第二層有梯度（零初始化所致），第 2–4 步四個群組梯度範數皆有限且 >0，無跳步。
4. 訓練後相對初始 state：encoder、neck、class head 第一層、第二層四個群組各至少一個 tensor 改變；所有 `requires_grad=False` 的參數與 buffer 逐 tensor 不變。
5. epoch 5 的平均 train loss ≤ epoch 1 的 95%。
6. 固定看 epoch 5：GoM val Oil IoU ≥ 0.15 且 val mIoU ≥ 0.50。
7. 回載：把 last.pt 的 `model_state` strict 載入另一個新建的模型，eval、fp32、同一固定 minibatch 上與記憶體中訓練完的模型比 logits，最大絕對差 ≤ 1e-5；另以 `DeepLabAdapter(best.pt)`（評估／重建實際走的路徑）回載，所有 tensor 與檔案相等。**替代**：固定 minibatch 用 `gom_fixture_8ch.npz` 的兩張真實 GoM train patch，而不是 val batch（trainer 不對外提供 val loader；回載保真度與 batch 來源無關）。
8. `torch.cuda.max_memory_allocated()` < 30 GiB（10-10 改 neck 後 5-step 檢查為 9.8 GiB）。
9. 每 epoch 時間：以「整段 wall time ÷ 5」報告（含建模與 loader 啟動，是上界）；> 8 分鐘則暫停回報使用者，不自行改 batch。
失敗處置照 Codex 所提：工程錯誤→修正後原門檻重跑；無工程錯誤而 5／6 不過→停止、回到協商，不得輪流試 neck／解凍／prompt／lr。

## 4. Hypothesis
- C：Prithvi 的有效特徵格距仍是 16 px，與 A 相同；凍結的 SAM decoder 沒有新增空間資訊。預期 C 落在 A 設定的雜訊範圍內（不可判定）。
- B：SAM encoder 在 1024 輸入上的格距相當於原圖 4 px，理論上有機會把細油帶畫細；但 pilot 的錯誤型態與 A 幾乎相同。預期正式 B 也落在不可判定。
兩者都沒有強方向性預期；本實驗的價值在於排除或發現大幅差異。

## 5. Method
- A：既有 `MS6_P14_GoM_only_full_R1`（不重跑）。
- C：Prithvi-EO-2.0-300M（同 A 的建構與預訓練權重，tap 23 經 terminal norm，B×1024×16×16）→ neck（§0）→ 凍結 SAM prompt encoder（固定全圖 box）＋凍結 SAM mask decoder → 公開 class head → B×2×256×256 logits。可訓練 304,756,736（encoder 303,886,336；neck 852,992；class head 17,408）、凍結 4,064,560。
- B：與 100-epoch pilot 完全相同的模型與設定，只有 epochs 100→300（T_max 100→300）、臺灣 post-test 開啟、名稱與輸出目錄不同。從官方 SAM 權重重新開始，不接續 pilot。
- 共同（比照 A）：P14 GoM R1 清單、seed 42、batch 16、最多 300 epoch、patience 50、GoM val mIoU 嚴格大於才更新 best.pt、Tversky 0.3/0.7、AdamW wd 1e-3、lr0 5e-5、cosine T_max=300／eta_min 5e-7、fp16＋GradScaler、clip_grad_norm 1.0、同增強。300 是上限，早停也算完成。
- 評估：訓練結束後同一 process 以 best.pt 對臺灣 137 張含油 patch 做 GT-aware 重建、argmax。臺灣每個 run 只評這一次。
- 對照與 estimand（R6-2）：主指標＝**臺灣 9 景各算 Oil IoU 再平均**（`_kind` recon(gt_aware)；`aggregate()` 的 `scene_macro_secondary.oil_iou`）。次要（只報告）：兩事件平均、9 景像素加總、逐景、precision／recall、GoM val best mIoU／Oil IoU。
- 與 A 不同的不只一項（C：head 換成 neck＋SAM decoder＋class head，且程式版本不同；B：encoder、預訓練來源、輸入放大、LoRA、解碼端全不同）→ system comparison，不是單因子消融。
- 已知限制：單 seed 單次；P14 GoM R1 的 val 有 16/39 景與 train 同取得日期（10-08 初查），checkpoint 選擇與 A 同受影響；臺灣僅 9 景 2 事件，0912_S2_50QRL 整景只有 1 張 patch 卻占 1/9 權重。

## 6. Expected Evidence（事前數字門檻）
參考分布：A 設定（同清單、同架構）四次既有 run 的臺灣 9 景平均 Oil IoU＝0.3657（P14 S42，即 A）、0.4111（P15 S42）、0.4050（P15 S43）、0.3632（P15 S44）；平均 **R＝0.3862**，SD 0.0253，全距 0.3632–0.4111。

對 X∈{C, 正式 B}，Δ＝X 的 9 景平均 Oil IoU − R，δ＝0.05：
- **支持「高於 A 設定」**：Δ ≥ +0.05（X ≥ 0.4362）。
- **否定（低於 A 設定）**：Δ ≤ −0.05（X ≤ 0.3362）。
- 其餘：**不可判定**。
用未四捨五入的數值計算。

為什麼用 R 而不是 A 單次的 0.3657：A 剛好落在同設定四次的低端。若以 0.3657 為基準，「支持」門檻是 0.4157，而 A 設定自己換個 seed／程式版本就有 0.4111 與 0.4050——等於 A 對自己幾乎就能過關，向上誤判的保護不足；向下則要 ≤0.3157 才算退步，比 A 設定最低一次還低 0.048，又太寬。以 R 為中心時，支持門檻比四次最大值高 0.025（約 1 SD），否定門檻比四次最小值低 0.027，兩側對稱。報表上 A 的數字仍照列 0.3657，並同時列出四次與 R。

報表必列：A 單次 0.3657、四次各自的數值與 R；`X−R`（卡片判定用）與描述性的 `X−0.3657`（不作判定）。四次混有不同 seed 與程式版本，不稱為 seed 分布，也不稱信賴區間。

必附註記（不改判定）：報告判定時須同列九個 leave-one-scene-out 的 8 景平均（X 與 A 設定四次各自同口徑）；若拿掉任一景會使 X 落到另一個判定區，判定旁必須寫明「由單一景主導」並指出是哪一景。

可主張：「在固定 P14 GoM R1、seed 42、此實作與臺灣 9 景上，X 這次 run 的 9 景平均 Oil IoU 比 A 設定四次平均高／低至少 0.05」。不可主張：統計顯著、可泛化、C–A 差異純由 decoder 造成、落在 ±0.05 內代表等效、以單次結果排名。

B 的揭露：本門檻訂於已看過 B 100-epoch pilot 的臺灣結果（9 景平均 0.3179）之後；正式 B 不是盲測，pilot 與正式 run 並列報告、不合併、不挑較好者。套用本規則，pilot 的 0.3179 會落在「否定」區且由 0912 單一景主導（拿掉該景後 8 景平均 0.3535，對 A 設定四次的 8 景平均 0.3708 為 −0.017）；這是事後套用，只作說明。

## 7. 執行與操作（不屬卡 1–6，但一併鎖定）
- 一份新 YAML `main/experiments_DADS_SAM_gomonly_formal_C_then_B.yaml`：`experiments` 依序為正式 C、正式 B（內容分別與現有兩份正式 YAML 的 experiment 區塊相同，僅 `run: true`），同一個 commit 鎖定，C 啟動前 B 即不可再改。每個 run 有自己的時間戳目錄（`<results_base_dir>/<timestamp>_<experiment_name>/`，含 weights、results.csv、run_metrics.json、args），共用的只有由各 run 的 json 重新產生的彙整 Excel。
- 兩個 run 在**同一個 main_runner process** 內依序執行（runner 逐實驗 `set_global_seed(42)`；`train_model` 訓練完即 `del model`＋gc＋`empty_cache`，runner 的 finally 再 gc＋`empty_cache`）。runner 已加一行紀錄：每個實驗清理後印出 `torch.cuda.memory_allocated()`。**實測**（10-10，`experiments_DADS_SAM_gomonly_smoke.yaml`，同一 process 先 C 後 B，各 3 個 optimizer step＋GoM val 子集重建，launch gate PASS、EXIT=0）：C 的訓練與重建結束、清理後仍配置 **64 MiB**，B 接著完整跑完，B 清理後同為 64 MiB。正式序列會留下同一行紀錄；若 B 仍因顯存失敗，會明確報錯，之後在新目錄從頭重跑並註記。
- C 若丟出例外，runner 會自動接著跑 B（既有的逐實驗 try／except）。共用故障多半會在 B 以同樣方式報錯；不報錯的共用故障在任何設計下都無法自動偵測。處置：C 一結束（不論成敗）我就檢查其 run_record、9/9 景、prediction manifest、報表 parity；若判斷是共用 runner／資料／評估故障，立即回報使用者（本專案規定停止訓練 process 須使用者明確授權，我不自行 kill），該序列的 B 結果在查清前一律隔離、不引用。原因：本容器在 host systemd reload（snapd 自動更新，10-07／10-09／10-10 早上各發生一次）後，新啟動的 process 會拿不到 GPU，已在跑的不受影響；分兩個 process 的話 B 很可能起不來。
- 中斷處理：trainer 的 last.pt 續跑不保證資料順序可重現，正式 run 若中斷一律在新目錄從頭重跑並註記，不接續。
- fp16 出現非有限 loss：回報使用者並請求停止該 run，不靜默改精度。
- 凍結的 prompt encoder＋decoder 內沒有 Dropout／BatchNorm（10-10 逐模組檢查：只有 Linear、LayerNorm、LayerNorm2d、Conv2d、ConvTranspose2d、Embedding、GELU、ReLU、Attention／MLP 容器），所以 decoder 留在 train mode 不影響數值。
- 監看只做故障偵測，不依訓練中的任何數字改設定。
- 結束後依序回寫現況檔與 vault，結案卡 7–9 另填。

# Preflight 結果與判準 2 的事後修訂（R4；**不屬於事前凍結的那批**）

執行：2026-10-10 11:27–11:31，`analysis/dads_sam_gomonly_20261005/preflight_C.py`（commit `79c4432`），RTX 5090；輸出 `/home/alanyh/oil_dataset/new/_dads_sam_runs/_preflight_C/20261010-112709/`。checkpoint 保留、不再使用：best.pt SHA256 `72cd32ae…b58cd`、last.pt `5da78a30…934f4`（全碼在 repo `preflight_C.json`）。

| epoch | train loss | val mIoU | val Oil IoU | val BG IoU |
|---|---:|---:|---:|---:|
| 1 | 0.7407 | 0.5602 | 0.2106 | 0.9098 |
| 2 | 0.4580 | 0.6411 | 0.3267 | 0.9556 |
| 3 | 0.3993 | 0.6483 | 0.3434 | 0.9533 |
| 4 | 0.3528 | 0.6600 | 0.3703 | 0.9497 |
| 5 | 0.3361 | 0.6759 | 0.3963 | 0.9555 |

（參考：A 的 val Oil IoU ep1 0.215／ep3 0.319／ep5 0.372；B pilot ep5 0.228。）

逐項（原判準）：scheduler T_max=300 ✅；1 紀錄值有限 ✅；**2 GradScaler 零跳步 ❌**；3 梯度探針 ✅（R2 時已過）；4 四群組皆更新（encoder 293/293、neck 6/6、class head 兩層各 1/1）且凍結 tensor 改變 0 ✅；5 train loss 0.336 ≤ 0.95×0.741 ✅；6 ep5 val Oil IoU 0.396 ≥ 0.15、val mIoU 0.676 ≥ 0.50 ✅；7 last.pt 回載 logits 差 0.0、best.pt 經 adapter 回載逐 tensor 相等 ✅；8 峰值配置 9,817 MiB ✅；9 每 epoch 上界 55.7 秒 ✅。

**原判準 2 未通過**：last.pt 的 scaler_state 為 scale 16384、`_growth_tracker` 581、總步數 855——scale 由 65536 減半兩次（跳過 2 個 optimizer step，約 0.23%），最後一次在第 274 步（epoch 2），其後連續 581 個乾淨 step。依 v1 Claude 收緊的寫法（零跳步）不過；依 Codex R1 的原始寫法（最多 5 次且 epoch 2–5 不得再跳）也不過。

**為什麼判定是判準訂錯而不是 C 有數值問題**：同一個 trainer 的既有 run 全部以低於初值 65536 的 scale 結束——

| run | optimizer steps | 結束時 scale | `_growth_tracker` |
|---|---:|---:|---:|
| A＝P14 GoM S42 | 16,245 | 32768 | 86 |
| A 設定 P15 Original S42 | 14,193 | 4096 | 1949 |
| A 設定 P15 Original S43 | 35,055 | 4096 | 274 |
| A 設定 P15 Original S44 | 11,970 | 4096 | 1979 |
| B pilot（100 epoch） | 17,100 | 4096 | 402 |
| C preflight（5 epoch） | 855 | 16384 | 581 |

GradScaler 的設計是每 2000 個乾淨 step 把 scale 加倍、放大後的 fp16 梯度溢位時跳過該步並減半；跳步反映的是 scale×梯度超出 fp16 範圍，不是模型梯度非有限。以「零跳步」為準，A 自己的四次 run 與 B pilot 也不會通過。（措辭限制，Codex R4：對超過 2000 步的長 run，結束時的 scale 與 tracker 只能定位最後一次 scale 變動，不能分辨那次是加倍還是減半；對 C，855 步內不可能加倍，所以兩次都確定是減半。）

**修訂後的判準 2**（在看到 preflight 的 scaler 結果**之後**訂定；依據只來自既有 A／B run 的 scaler 行為與 GradScaler 機制，沒有用到 C 的表現數字；學習判準 5、6 未放寬）：
- 2a 所有紀錄的 loss／指標有限（＝判準 1）。
- 2b epoch 5 的 model_state（441 個 tensor）與 optimizer state 全部有限。
- 2c scale 沒有塌陷：結束時 scale ≥ 4096 且結束前至少連續 171 個乾淨 step。這是**本專案依既有 A／B 行為訂的經驗門檻**，不是 GradScaler 的普遍安全下限。

同一次 preflight 在修訂後判準下**通過**（`preflight_C.py --recheck`，repo `preflight_C_recheck.json`：2b ✅、2c ✅（16384、581），其餘同上）。Codex R4：同意修訂、不需重跑（同 seed 同設定重跑不會讓修訂變成事前門檻）、維持 ACCEPT。

正式 run 的對應監看：出現非有限 loss／state，或 scale 持續下跌到 < 64（保守警報，須結合有限性與學習狀態判讀），回報使用者並請求裁決；不靜默改精度。

# 後半：結案才填（7–9）

（正式 C、B 完成後另填，存 `wiki/experiments/`。）
