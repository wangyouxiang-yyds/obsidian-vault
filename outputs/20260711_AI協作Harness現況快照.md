---
date: 2026-07-11
type: pipeline
tags: [harness, ai-collaboration, institution, agent-dispatch, multi-agent, snapshot, claude-code, antigravity, codex, oil-detection]
---

# AI 協作 Harness 現況快照（2026-07-11）

> 本篇為狀態記錄型筆記，記錄 2026-07-11 當下的 harness 架構、任務目標、現有 agent、AI 模型角色分工快照，供日後回顧比對。內容會隨制度演進而過時，若需查最新狀態請優先讀 `.agents/handoff.md`（工作階段交接檔）與 `.agents/institution/` 正本。

## 1. Harness 架構：雙 harness + 共用正本（institution）

- **規則正本**：`/home/alanyh/.agents/institution/`（2026-07-05 建立），是 Claude Code 與 Antigravity 兩個 harness 的唯一規則來源。
  - 常載三份：`00_diagnosis.md`（易錯修法）、`10_model_dispatch.md`（模型調度守則）、`20_judgment_rubrics.md`（判斷 rubric）
  - `30_maintenance.md`（維護協議）、`90_letter_to_future.md`（給未來 session 的信）
  - `research/oil_spill_project_status.md`：研究現況的單一事實來源（所有實驗數字、已否定路線、已知 bug、路徑）
  - `templates/`：search / implement / refactor / research / review 五份交辦範本
- **薄索引**：兩個專案的 CLAUDE.md 與 agy 的 `/root/.gemini/GEMINI.md` 只做路由，不複寫規則。
- **知識層**：Obsidian vault（`/home/alanyh/obsidian-vault`）承載 wiki、文獻 pipeline、handoff；寫入唯一通道是 obsidian-vault-manager subagent。
- **第三方夥伴**：codex（GPT 系）為對等夥伴，透過 `codex exec` 呼叫，有 consensus.md 協商制度與 handoff.md 交接——目前只接 vault，尚未接 institution。

## 2. 任務目標（研究現況）

- **研究主題**：8-band 多光譜衛星影像油汙偵測（semantic segmentation）。
- **路線**：2026-06-22 放棄非監督（F1≈0.008），轉監督式。分支 A = 0422 sliding-window repo（論文主軸）；分支 B = OIL_PROJECT_MutiBand_GT_expand（GT-centric patch：bbox 中心 ±128、2,897 patches、3-fold stratified CV）。
- **歷史關鍵數字**（3-fold pooled oil IoU 平均）：baseline（FocalLoss+DeepLabV3）0.332；tversky（α0.3/β0.7）0.3915，唯一通過雙 gate（Wilcoxon p=9.5e-14）；cw31 均值 0.394 但 Wilcoxon 不顯著且 tiny 場景顯著變差，判定劣於 tversky。
- **目前正在跑**（2026-07-11 06:28 UTC 啟動，約 2 天）：串行兩組 3-fold 實驗——`baseline_v3plus`（架構換成 smp DeepLabV3Plus，isolate 架構效果）→ `tversky_v3plus`（最佳 loss 疊上新架構）。資料凍結、seed 42，其餘全鎖死。
- **本日完成**：F1–F10 前處理審查（與 codex 共識）第一批工程修補——F4 fail-fast、F5 VRT 幾何契約、F7 blake2b 穩定 hash、F8 N_FOLDS 自動偵測、F9 val drop_last=False、F3 preflight 驗證器（355 場景 ALL PASS）；歷史模型 bitwise 迴歸驗證通過。
- **驗收標準**：雙 gate（pooled_oil_iou 超 baseline 1 std（>0.362）＋ 355 場景 per-scene Wilcoxon p<0.05）＋ 51 大場景 P/R 分解。

## 3. 現有 Agent（Claude 端 subagents，全域 `/root/.claude/agents/`）

| Agent | 角色 |
|---|---|
| obsidian-vault-manager | vault 寫入唯一通道（筆記/daily note/文獻/handoff） |
| antigravity-consultant | 橋接 agy：即時網路搜尋、跨家第二意見、低風險體力活初篩 |
| code-reviewer | 三模式 code review，明確要求才觸發 |
| 內建：Explore / Plan / general-purpose / claude-code-guide / statusline-setup | 唯讀搜索 / 規劃 / 通用 / Claude Code 問答 / statusline 設定 |

## 4. AI 模型角色分工

- **Claude Code 主線**：決策、審核、整合、寫結論（「指揮官不下場」）。本 session 由 Claude Fable 5 以背景 job 執行；日常預設 Sonnet 5，難題升級 Opus 4.8，雜活降級 Haiku 4.5。
- **agy（Antigravity / Gemini）**：即時 Google 搜尋、文獻查證、跨模型第二意見。建議日常值 Gemini 3.5 Flash (High)，升級 Gemini 3.1 Pro；也可選跨家模型（Claude Sonnet/Opus 4.6、GPT-OSS 120B）。
- **codex（GPT 系）**：對等協商夥伴，用於方法論共識（本次 F1–F10 前處理審查即 Claude×Codex 共識產物）。
- **調度原則**：交辦必含三要素（目標動機/驗收條件/回報格式）；驗證不自驗（fresh-context read-back / 實跑測試 / 高風險加第二意見）；升降級路徑（便宜模型錯 1 次升一級、中階連錯 2 次帶失敗軌跡升級、最多兩輪後問人）。

## 相關筆記

- [[GT_expand_pipeline]] — 分支 B（GT_expand）pipeline 正文：雙 gate 評估協議、tversky/cw31 實驗線細節
- [[VRT_pipeline_02_模型訓練]] — 訓練 pipeline 工程細節，含 best.pt / EMA 相關修正記錄
- [[20260702_波段選擇消融實驗規劃]] — baseline pooled oil IoU 0.332 數字出處與消融實驗規劃
- 制度正本（非 wiki 頁面，位於 vault 根目錄，依既有慣例以路徑而非 wikilink 引用）：`.agents/handoff.md`（工作階段交接檔）、`.agents/consensus.md`（Claude×Codex 協商制度）
