# Phase B · Map 抽取執行計劃

> **階段目標**：把 643 集 podcast + 7 份訪談拆成 13 批（B00-B12），每批 spawn 一個 subagent 完整讀完語料，輸出結構化抽取到 `references/research/shards/batch-NN/`。
> **模式**：nuwa「本地語料優先模式」，完全不做網路搜尋。
> **前置依賴**：Phase A 已完成所有 references/sources 文件。Map agent 會需要讀取這些文件當上下文。

## 波次編排（4 波、每波 3-4 批並行）

| 波次 | 批次 | 並行度 | 理由 |
|---:|---|---:|---|
| **Wave 1** | B00 + B01 + B02 + B03 | 4 | 跨訪談 + 2020-2021 疫情 / 科技大漲 / GME 時代 |
| **Wave 2** | B04 + B05 + B06 | 3 | 2021-2022 熊市 + Ukraine + FTX（情緒波動最大，需要品質） |
| **Wave 3** | B07 + B08 + B09 | 3 | 2023-2024 AI 元年 + 崛起（諾亞 / 羅馬 / 挪威） |
| **Wave 4** | B10 + B11 + B12 | 3 | 2024-2026 最近期 + 肖楠 / 烏杉 + 小徐（最重要的最後跑，吸收前面經驗）|

**總體：4 波 × 平均 3.25 批 = 13 批 subagent**

### 為什麼這個順序
1. Wave 1 把 B00（訪談，絕對基礎認識）和早期批次放一起 — 為後面幾波建立「他是誰」的基礎
2. B12 放最後 — 它是「最新最成熟版本」，蒸餾 default 來自這批。先跑完前 12 批、主 agent 對他的模式更熟、再跑 B12 品質最好
3. 每波之間主 agent 閱讀上一波 summary、決定是否調整下一波 prompt（例如：如果 Wave 1 發現某個關鍵字命中率高，Wave 2 prompt 就強化偵測）

## 每批 subagent 的標準任務模板

每批 subagent 都用這個 prompt 結構：

```
你負責 Batch [ID] 的 Map 抽取任務。

===== 背景（必讀） =====
- 這是蒸餾《股癌 Gooaye》主持人謝孟恭的 skill 專案
- 完整計劃：~/.claude/skills/gooaye-perspective/research-meta/batch-plan.md（讀 B[ID] 段落）
- 生活時間線：~/.claude/skills/gooaye-perspective/references/sources/gooaye-life-timeline.md
- 經典集數：~/.claude/skills/gooaye-perspective/references/sources/gooaye-classic-episodes.md
- 兩家公司：~/.claude/skills/gooaye-perspective/references/sources/gooaye-companies.md
- 抽取方法論：~/.claude/skills/gooaye-perspective/references/extraction-framework.md

===== 你的素材範圍 =====
[For B00:] /Users/chikangpai/code/gooaye-perspective/corpus/interview-transcript/ 下 7 份 md
[For B01-B12:] /Users/chikangpai/code/gooaye-perspective/corpus/episodes/EP{lo}_*.md 到 EP{hi}_*.md
[清單直接從 batch-index.json 讀 batches[ID].episodes]

===== 輸出位置 =====
寫入：~/.claude/skills/gooaye-perspective/references/research/shards/batch-{ID:02}/

===== 必輸出 5 個檔案 =====

1. `claims.md` — 本批反覆出現的論點 / 框架 / 立場
   格式：
   ## 論點 N: [一句話標題]
   - **EP 出現**: EP123, EP145, EP201（列出所有 EP）
   - **頻率**: 出現 X 次 / 跨 Y 個主題
   - **原句舉例**: > "[逐字原句]"（EP123）
   - **相關術語 / 類比**: [若有]
   - **首次出現 or 後來演變**: [是否立場有變化]

2. `expressions.md` — 表達 DNA 原句（verbatim，不總結！）
   四個 section:
   ### 高頻用詞 / 口頭禪
   [EP 原句 + 集數，至少 20 條]
   ### 髒話 / 罵人方式 / 情緒爆點
   [「幹你娘」「秋口」「他媽的」等出現的原文片段]
   ### 類比 / 比喻 / 自創術語
   [「柳樹理論」「阿呆谷」「工具箱」... 的原句]
   ### 幽默模式（自嘲 / 荒誕 / 挖苦 / 冷笑話）
   [原句示例]

3. `decisions.md` — 實際進出場、立場表態、事後認錯
   格式：
   ## 決策 N: [事件 / 股票 / 立場]
   - **EP**: EP123 (日期)
   - **背景**: [市場 / 個人狀況]
   - **他的判斷**: [原句]
   - **行動 / 建議**: [有無實際進出場？]
   - **事後結果 / 修正**: [EP156 是否有回過頭來認錯 / 確認]

4. `life-events.md` — 個人生活事件
   格式：
   ## 事件 N: [標題]
   - **EP**: EP123 (日期)
   - **類別**: 婚姻 / 育兒 / 旅行 / 家族 / 健康 / 朋友 / 職涯
   - **原文摘錄**: > "..."
   - **情緒標記**: [高興 / 焦慮 / 憤怒 / 悲傷 / 中性]
   - **錨點對照**: [對照 gooaye-life-timeline.md 哪一條]

5. `timeline.md` — 此批期間的時序立場演變
   一條時間線，每個重要節點記：EP 號 / 日期 / 市場背景 / 他的反應 / 個人事件

===== 必讀「Classic 深度」集數 =====
本批含經典集（見 gooaye-classic-episodes.md）：
[列出屬於本批的經典集 EP 號]

對每個經典集**額外寫** `classics/EPxxx-deep.md`（300-500 字），內容：
- 本集核心論點 1-3 條（帶原句）
- 使用的術語 / 類比
- 本集的情緒狀態（冷靜 / 爆氣 / 自嘲）
- 梗集的話：梗的完整脈絡
- 該集對蒸餾最有價值的 1 個 moment

===== 偵測清單（遇到必記錄 verbatim） =====

**生活事件關鍵字**（完整清單見 batch-plan.md「Map agent 持續偵測清單」）:
挪威 / 北歐 / Norway / 瑞典 / 義大利 / Rome / 羅馬 / 西班牙 / Tavernes / Valencia /
岳家 / 丈母娘 / 娘家 / 歐洲 / Liza / 諾亞 / Noah / Killian / Choco / 兒子 / 懷孕 /
威航 / 機師 / 飛行員 / 旅館 / 灰階思考 /
肖楠 / Calocedrus / 烏杉 / 烏衫 / Lunta / 常青基金 / Evergreen / First Check /
文策院 / TAICCA / 嘖嘖 / 徐震 / 好翊 / GameWorks / 芒果派對 /
紅眼露比 / Night Rampage / 杯狗 / 本氣遊戲 /
小徐（過世聽眾；與董事徐震區分）

**自創術語（疑似）關鍵字**:
柳樹理論 / 阿呆谷 / 工具箱 / 走火入魔 / 灰階 / 主委 / 破壞式創新低

===== 執行品質要求 =====

1. **原句不總結**：expressions.md 嚴格要求 verbatim 引用。summary 可以後面加，但不能取代原句
2. **EP 一定要帶**：所有記錄必須有 `(EPxxx)` 標記，方便 Phase C 交叉驗證
3. **矛盾保留**：如果本批內他前後立場變化，**保留兩個版本**並註明，不要和稀泥
4. **不編造**：沒讀到的不要寫。寧可 claims.md 只有 5 條紮實的、也不要 20 條灌水的
5. **本地優先**：只能從指定 EP 檔案 + 訪談檔 + references 文件來。**禁止 WebSearch 或其他外部查詢**
6. **讀全文**：不能只掃標題、首尾、或關鍵字跳讀。每集全文讀過一次（可以快讀）

===== 完成標記 =====

最後輸出一個 `_SUMMARY.md` 到該 shard 目錄，內容：
- 實際讀了 X / Y 集（應該 = Y）
- claims 條數、expressions 條數、decisions 條數、life-events 條數
- 3 個最值得主 agent 關注的發現
- 任何失敗 / 跳過的集數（稀有，但要報）
```

## 每批的個別化參數

**B00 Interviews**：

- 素材：`interview-transcript/` 7 檔
- Episode range: N/A — 用訪談檔名引用
- Classics：無（整批都是 "他者視角 + 長對話" 雙重黃金素材）
- Output: `references/research/shards/batch-00-interviews/`

**B01 EP1-55**，classics = EP20
**B02 EP56-105**，classics = EP57, EP100 (老婆/馬雲/多元配置)
**B03 EP106-155**，classics = EP102, EP117
**B04 EP156-205**，classics = EP169, EP180
**B05 EP206-258**，classics = EP260
**B06 EP259-310**，classics = EP275
**B07 EP311-360**，classics = EP327, EP339
**B08 EP361-414**，classics = (無硬錨，但留意 EP397 育兒 / EP414 年底回顧)
**B09 EP415-467**，classics = **EP431-435 挪威系列（必讀）**
**B10 EP468-518**，classics = **EP494 / EP496 羅馬 / EP513 / EP515 二胎焦慮**
**B11 EP519-570**，classics = **EP525 / EP548 / EP549 / EP550 / EP555**
**B12 EP571-654**，classics = **EP578, EP589, EP607, EP608, EP618, EP629-633, EP634, EP635, EP638, EP645, EP648, EP654**（這批幾乎每集都是）

## 並行失敗處理

- **單批超時**（Map agent 跑 > 15 分鐘無有效輸出）：主 agent 不等，先收其他批。之後重啟該批，給更窄的 prompt（例如只先寫 expressions.md）
- **Agent 結果衝突**：**保留**，交給 Phase C reduce 時處理
- **某批發現 critical gap**（例如完全沒記錄到預期的生活事件）：主 agent 決定是否補跑
- **Batch 品質評估**：每波結束後主 agent 抽讀 `_SUMMARY.md` + 2 個隨機 shard 檔案，品質不過標的重跑

## Phase B 完成標準

- [ ] 13 個 shard 目錄都存在：`shards/batch-00-interviews/`, `shards/batch-01/` ... `shards/batch-12/`
- [ ] 每個目錄至少含 5 個標準檔 + `_SUMMARY.md`
- [ ] 經典集的 `classics/EPxxx-deep.md` 都產出（約 30+ 檔）
- [ ] 主 agent 產出 Phase B 總摘要：`research-meta/phase-b-done.md`，包含：
  - 13 批各自的 key findings
  - 全局發現的最 surprising 5 件事
  - 對 Phase C reduce 的建議（哪些論點值得跨批次追蹤）

## 預估耗時

- 每個 subagent：大約 5-10 分鐘（讀 50-75 集 markdown + 寫 ~2000 行輸出）
- 4 波序列執行、每波 ~10 分鐘 = **~40 分鐘總**
- 若主 agent 機器負載高、加上品質 review 間隙，預估 **~60-90 分鐘** 跑完

## 給新機器 operator 的 quick start

如果是換機器接續，執行順序：
1. 確認 `~/.claude/skills/gooaye-perspective/` 完整（見 RESUMING-ON-NEW-MACHINE.md）
2. 確認 corpus 路徑：`/Users/chikangpai/code/gooaye-perspective/corpus/` 下 `episodes/` 和 `interview-transcript/` 都在
3. 跑 `python3 ~/.claude/skills/gooaye-perspective/scripts/validate_phase_a.py`（待建 — 新機器接續時順便寫）
4. 開 Claude Code session，說：「繼續 gooaye-perspective 蒸餾，從 Phase B Wave 1 開始」
5. 主 agent 會 read PLAN.md → read phase-b-execution-plan.md → spawn subagents
