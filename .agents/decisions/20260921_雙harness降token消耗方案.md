# 雙 harness 降低 token 消耗方案（Claude Code × Codex 正式協商，3 回合收斂）

- 日期：2026-09-21
- 主持方：Claude Code（claude-sonnet-5，effort 繼承全域 high）；對手：Codex（gpt-5.6-sol，effort=high 釘死，codex-cli 0.155.1，同一 session `01a0c185-aba8-7b03-8e37-802888cccbad` 三輪 resume）
- 觸發：使用者反映 token 消耗大、協商時一方額度掛掉、單一工具也很快耗盡；要求「開始做做看」並要主持方問 Codex「怎樣做才可以比較好」。屬「制度結構修改」協商觸發項。
- 狀態：**協商收斂（R3：雙方 ACCEPT／BLOCKING=[]／RESTATE 準確）。Phase 0 進行中；Phase 1（改 institution／vault 正本）尚未授權，未動任何制度檔。**
- 可逆性：全案皆可逆，故不強制反方終審。

## 一、問題與量到的事實（本機 Linux，2026-09-01 起；腳本見第七節，自測 11/11 PASS）
Claude 端（requestId 去重）：互動 cli 17 session 共 434.8M（94%）、headless sdk-cli 17 session 28.3M（6%）；最大兩個 session（a6e8146b 1161 步／182.3M、79598ffa 558 步／115.0M）占 Claude 總量 64%（占互動 69%）；平均 context 157K／206K 每步，兩者以 claude-opus-5 為主。
Codex 端（邊際用量，已扣 fork 繼承基線，合計約 682M）：desktop 61%、subagent 22%、tui 11%、**codex_exec（協商呼叫）11 個 session 45.7M = 7%**；最大 desktop session 150.6M（1474 步／152 任務，135 個 turn_context 全為 high effort）。
**結論**：主因是「長 session × 每步重讀累積 context × 工具迴圈 × 高 effort」，不是協商呼叫本身（兩邊 headless 皆 <10%）。
限制：僅本機；不含 Claude subagent 側鏈與 Windows 端；Claude 額度本機無欄位；raw token ≠ quota（Codex Sol 官方 credit：input 100／cached 10／output 500 每百萬）。

## 二、逐輪演變與 blocking 變化
- R1（盲提案）：Codex 獨立提案並挑戰主持方分析；指出 (1)「輪」實為 token_count 事件（≈8–10 步／任務）(2) fork/subagent 顯示值含父基線 (3) 主持方 codex_exec 視窗漏算。VERDICT REVISE，BLOCKING `[1,2]`（**未編號對應，屬格式錯誤**，R2 補說明：[1]=計量修正、[2]=raw token≠quota 且跨端觀測不全）。
- R2（候選稿 v1，C1–C9）：主持方先自行驗證並接受 R1 三項修正（codex_exec 實為 44.9M/7% 而非 5M）。Codex REVISE，BLOCKING `[C1,C3,C5,C7,C9]`（5 項）。
- R3（候選稿 v2，含 C10 與一項事實修正）：Codex 全條 ACCEPT，BLOCKING `[]`，RESTATE 準確。
- 事實修正（主持方→Codex）：**institution 目錄不是 git repo（vault 才是）**，故不可用 Git history 取代備份。
- 停止原因：收斂。停滯偵測未觸發（blocking 5→0）。分歧點與返工：首輪分歧＝數字口徑（步數/視窗/fork 基線）；產生返工＝主持方自身分析修正一次。

## 三、最終決議（候選稿 v2）
- **C1 session 邊界（試行值，由 C8 資料校準，兩邊同一規則）**：觸發＝work package 換／設計→實作→review 換階段／完成、blocked、等待外部結果／soft：8 人類回合或 25 步，hard：12 回合或 50 步（先到者）／context>50% 且下階段≥3 次工具呼叫／第一次 compact 後換階段（第二次 compact 視為硬停止）／協商收斂後執行另開 session 只帶 decision record／連續三步 input context ≥100K → 原子工作完成後 checkpoint。soft 到時進下一階段前必換 session。
- **C2 最小交接包**：目標 ≤2KB、hard cap 4KB，超出以路徑指向；欄位＝目標與狀態、AS_OF＋誰實查、已定決策與不可違反條件、changed files／產物路徑、已跑驗證與結果、live process（PID/tmux/log/checkpoint）、blocker＋下一個精確動作、授權／共識狀態＋quota reset（未知填 `UNKNOWN`，不算缺欄）、證據指標。
- **C3 分層**：`oil_spill_project_status.md` 保留路徑但 ≤10KB（router/current）；`research/status/{methods,negative-results,bugs,experiments}.md`；`research/archive/YYYY-MM.md` append-only；`handoff.md` ≤5KB；`handoff_archive/YYYY-MM.md`；共識歷史只留 decision 索引。**執行方式**：scratchpad dry-run →段落 mapping 100% → fresh-context retrieval test 10/10 → 模擬 failover 1/1 → 才換入。舊檔以 timestamp 存入 `institution/archive/`（**非常載目錄根**；institution 非 git），vault 檔靠 git；archive 保留與否交使用者。institution 檔由具該 root 權限且 parity 合格的主持 adapter 換入，vault 檔走 vault-manager。
- **C4 模型／effort 路由**：例行用 Sonnet 5／Codex low；Opus 5 high／Sol high 只留正式協商、高判斷、終審。Terra 未驗證前僅為建議值，不得列 fallback。不改共用 `~/.codex/config.toml`；使用者 Desktop effort 屬個人習慣，只建議不強制。
- **C5 額度偵測**：Codex 由唯讀腳本讀 `token_count.payload.rate_limits`（附資料時間、視窗已重置則標舊值）；門檻 70% 警示／85% 不開新長任務／90% checkpoint＋交接。Claude 額度本機不可讀：使用者回報 GREEN/YELLOW/RED 或實際 limit error，否則 `UNKNOWN`；不送 probe。正式協商前「查 Codex；Claude 未提供則標 UNKNOWN，主持方揭露風險後由使用者決定啟動或延後」。**不得在已知 Codex reserve<15%（且無使用者豁免）時開始正式協商**，豁免須留紀錄。
- **C6 協商回合**：保留同 session resume、盲提案、VERDICT、RESTATE。首輪只帶最小 decision packet；prompt 明列允許讀檔清單（deny-by-default，新增需先說理由）；第 2 輪起傳編號候選稿，之後只傳 clause diff＋blocking 變化＋新證據，接受前傳一次完整 canonical 版；decision record 落檔可由主持方／vault-manager 直接完成。實測：加「不要讀檔」限制後 Codex 輸出由 554KB 降至 23KB。
- **C7 補 Codex-unavailable 對稱程序（寫入 40_harness_continuity.md，Phase 1）**：CODEX_UNAVAILABLE 判定與單一模型不可用區分；Claude 接手檢查清單；可逆工作繼續、重大決策交使用者、不冒稱共識；恢復後 review＋parity audit＋清狀態。Codex 模型 fallback 不寫入未驗證項；Sol 缺席且無已驗證 fallback → 標 `CODEX_MODEL_UNAVAILABLE` 交使用者仲裁；Terra 需品質驗證＋使用者批准才可列 PROVISIONAL。
- **C8 用量報表**：Claude／Codex 算法分開（requestId 去重 vs fork inherited-base），不合成跨供應商 quota 百分比；驗收含人工 fixtures（resume、subagent fork、Desktop user fork、缺 rate_limits、跨 reset window）。
- **C9 分期與成功指標**：Phase0＝可逆、無制度改動；Phase1＝協商收斂＋使用者授權後改 00/10/40 與換入 C3；Phase2＝重跑 C8 比對。一週只判安全與遵守率，兩週且每類≥5 個 completed work packages 才判效益，不足標 `INSUFFICIENT_DATA`。指標：安全＝不得在已知低 reserve 時啟動協商、因 session 切換遺失授權／process ownership／active state＝0；集中度＝單一 session ≤30%、top-2 ≤50%（或較基線降 ≥20 個百分點）；效率＝同類 work package median marginal tokens −25%、median steps −20%；context＝每步 input tokens p90 −30%（分 routine／implementation／consensus）；品質＝共識必填欄 100%、假共識 0、因漏證據重開 decision 0；狀態層＝status≤10KB、handoff≤5KB、mapping 100%、retrieval 10/10、failover 1/1；執行率＝soft 後下一階段前換 session ≥80%、hard 100%。**使用者行為（Desktop effort／session 長度）為「報告但不強制」。**
- **C10 執行邊界**：Phase 0 現在就做（scratchpad／唯讀分析／自願遵守 C1、C2）；Phase 1 任何 institution／vault 正本改動須同時具備「本案收斂」與「使用者授權」。

## 四、基線（C9 比對用；2026-09-01～09-21，本機）
Claude：top-1 39%、top-2 64%（互動 69%）；headless 6%。Codex：top-1 22%、top-2 35%（**Codex 端集中度已達標，該指標對 Codex 無鑑別力，改看 median steps/tokens 與 p90 context**）；codex_exec 7%。

## 五、必須由使用者裁決（不可由兩個 harness 自行決定）
1. institution／vault 正本結構換入時機、archive 位置與是否保留額外備份。
2. Claude 額度 `UNKNOWN` 時是否仍啟動正式協商。
3. Codex fallback 型號與 PROVISIONAL 權限邊界。
4. Desktop 的模型／effort 習慣；15% reserve 是否值得犧牲當下產能。
5. 門檻試行後的正式定案；任何刪舊檔、commit/push 或 process 操作。

## 六、少數意見與殘餘風險
- 無反對意見。Codex 指出最大過度樂觀點：「段落無遺失＋一週數據」不足以證明 C3／C9 成功，真正風險是內容搬走後新 harness 找不到 → 已加 retrieval test 與 failover 演練。
- 誠實邊界：C1 數值（8/25、12/50、100K）為試行值，非已證實最佳點；Claude subagent、Windows 端、Claude 額度、cached token 對額度的實際折算皆未量。
- 協商成本：三輪 Codex 端自報 tokens used 為 146,029／173,095／180,291（主要為同 session 重送的 system／tool schema／既有 context，不是答案本身）；支持 C1「協商收斂後另開執行 session」。

## 七、Phase 0 產物與驗證點
- `harness_usage.py`（report／quota／selftest；自測 11/11 PASS；真實資料重跑與手算一致）目前位於 session scratchpad `/tmp/claude-0/-mnt-backup-alanyh/d716d1d5-968e-4eb3-9213-4d10099c27b4/scratchpad/harness_usage.py`（**暫存區，可能消失**；經使用者授權後才提議放 `institution/scripts/`）。
- 回顧點：2026-09-28 看安全與遵守率；2026-10-05 看效益（樣本足夠才判）。
- 工具驗證：`codex exec` 標準呼叫式（含已被標 deprecated 的 `use_legacy_landlock`）於 codex-cli 0.155.1 三輪實測可用（讀檔、resume 均成功），先前「升級後須重測」項目通過；該旗標日後移除時須再測。
- 已知不確定：`~/.codex/config.toml` 現值 effort=low，但兩個最大 Codex session 全為 high（不追溯），故 Desktop 實際 effort 需以 session 內 turn_context 為準。

## 八、交接包（C2 格式）
目標：把兩 harness 的 token 消耗降下來且不削弱共識品質。狀態：協商收斂，Phase 0 完成（腳本＋基線），Phase 1 待使用者授權。AS_OF：2026-09-21 Claude 主 harness 實查。已定：見第三節 C1–C10。changed files：僅 vault 本 decision record 與 handoff 一行；scratchpad 腳本。驗證：selftest 11/11、真實資料比對一致、Codex R3 ACCEPT/[]。live process：P16 tmux 未於本案刷新（沿用 handoff 既有條目，視為 STALE 待實查）。下一步：使用者裁決第五節後，另開新 session 執行 Phase 1（先做 C3 的 dry-run，不直接換入）。授權：Phase 1 未授權。Codex quota（09-21 01:28 讀）：5h 17%／週 19%；Claude：UNKNOWN。
