# consensus.md — Claude Code × Codex 共識協商制度

2026-07-10 建立（草稿，待使用者核可後生效）。

## 定位
Claude Code 與 Codex 地位對等，重大決策須協商達成共識後才執行。
Gemini CLI（agy）不參與共識，只接協商完成後派發的雜工（見 `dispatch_to_gemini.md`）。
使用者是最終仲裁：協商不收斂、或雙方對風險評估有分歧時，停下來交使用者裁決，不得擅自執行。

## 什麼事情需要共識（觸發門檻）
需要協商：
- 方法論選擇與變更（模型架構、資料切分、評估指標的變動）
- 實驗設計（新實驗的假設、變因、驗收條件）
- 會寫進論文的結論或主張
- 對 wiki 方法論頁、`.agents/` 制度正本的結構性修改

不需要（直接做，避免協商變成每個小改動的稅）：
- 日常改檔、跑既定實驗、格式整理、錯字修正、既定計畫內的執行步驟

## 協商流程（v2 收斂驅動制，2026-07-11 Claude×Codex 共識，取代固定 3 回合制）

1. **主持方** = 使用者當下對話的 harness；對手方以 headless 方式喚起（多回合沿用同一 session：`codex exec resume --last`）。
2. **第 1 回合盲提案（防錨定）**：主持方只給「問題描述＋背景」，要求對手方獨立提案，不透露主持方傾向。
3. **之後每回合**：主持方維護**單一候選稿**，雙方交換：立場、blocking 問題清單、修改建議、驗收條件。
4. **結構化訊號**：每輪回覆結尾附機器可解析區塊：
   ```
   ---VERDICT---
   STANCE: ACCEPT | REVISE | OBJECT
   BLOCKING: [編號清單，空=[]]
   RESTATE: <一句話複述對方目前最強論點>
   CONFIDENCE: HIGH|MEDIUM|LOW
   ---END---
   ```
5. **收斂的操作型定義**：雙方 `STANCE: ACCEPT` 且 `BLOCKING: []` 且 RESTATE 能準確複述對方最強反對意見與其處理方式。`REVISE`（方向同意但候選稿需改）**不算收斂**；「都可以」等含糊回覆不接受，追問一次。
6. **停止準則**（收斂之外的強制停止，防無限迴圈）：
   - **5 回合檢查點**：仍未收斂 → 主持方向使用者回報進度與分歧點，由使用者決定續打或直接裁決；續行仍受停滯準則約束
   - **10 回合硬上限**
   - **停滯偵測**：連續兩輪 blocking 數無減少、論點開始循環、或對手方工具失敗 → 停止並標記 ESCALATE，整理雙方立場交使用者裁決，**不得冒稱共識**
7. **反方終審（devil's advocate）**：僅對**不可逆決策**（會寫進論文的結論、刪資料、方法論翻案）強制加一輪反方審查；日常共識收斂即結束。一般案件的 RESTATE 已提供基本反方檢查。
8. **防假收斂**：每次同意必附理由、殘餘風險、反證條件；主持人不得替對方宣告滿意。
9. **執行與紀錄**：收斂 → 主持方單獨執行（協商歸協商、執行歸執行）；每案建立 decision record 於 `.agents/decisions/YYYYMMDD_<議題>.md`（記：議題、逐輪候選稿演變、blocking 變化、停止原因、最終決議、少數意見、後續驗證點），並在 `handoff.md` 共識紀錄加一行。
10. 對手方不可用時（token 用盡、服務中斷）：可逆操作可先執行並在 handoff.md 標注「未經共識」；重大不可逆決策一律延後。（沿用 v1）

## 共識紀錄
達成共識後在 `handoff.md` 的「共識紀錄」記一行：日期＋議題＋結論＋雙方立場摘要。

## 喚起指令（2026-07-10 Linux server 驗證）
- Claude → Codex：`codex exec --skip-git-repo-check -s read-only -c tools.web_search=true -m gpt-5.6-sol "<提示詞>"`（模型由使用者 2026-07-11 指定為 gpt-5.6-sol）；多回合沿用同一 session：`codex exec resume --last "<下一輪提示詞>"`
- Codex → Claude：`claude -p "<提示詞>"`（直接執行已驗證可用，claude v2.1.206）。
  注意：Linux server 的核心未開 unprivileged userns，codex 的 bubblewrap 沙箱建不了 namespace，`codex exec` 沙箱內喚起 claude 會失敗；互動模式下的 codex 可經使用者核准跳出沙箱執行。兩個 harness 都安裝在這台 Linux server 上，協商互叫皆在本機完成。
