# Phase C · Reduce 合併執行計劃

> **階段目標**：把 13 批的 shards（62+ 檔）合併成 nuwa 標準的 6 個 canonical research files + 新增的 `07-life-events.md`。用**跨批次頻率統計**識別真信念。
> **前置**：Phase B 已完成，`shards/batch-00-interviews/` 到 `shards/batch-12/` 都有輸出。

## 輸入

```
~/.claude/skills/gooaye-perspective/references/research/shards/
├── batch-00-interviews/
│   ├── claims.md, expressions.md, decisions.md, life-events.md, timeline.md
│   └── _SUMMARY.md
├── batch-01/ to batch-12/
│   ├── claims.md, ..., timeline.md
│   ├── classics/EPxxx-deep.md (選擇性)
│   └── _SUMMARY.md
```

## 輸出

nuwa 標準 6 檔 + 股癌專用 1 檔，全在 `references/research/`：

| 檔案 | 內容來源 |
|---|---|
| `01-writings.md` | 13 批 `claims.md` 去重合併；頻率 ≥3 批 = 核心論點（放最前）|
| `02-conversations.md` | 13 批 `expressions.md`（對話 / 即興場景部分）+ 經典集深度摘要 |
| `03-expression-dna.md` | 13 批 `expressions.md`（高頻詞 / 髒話 / 類比 / 幽默部分）|
| `04-external-views.md` | **僅 B00 Interviews** 中外人提問觸發的自我揭露 + 誠實標註「無批評類外部語料」|
| `05-decisions.md` | 13 批 `decisions.md` 按時序合併 + 立場變化追蹤 |
| `06-timeline.md` | 13 批 `timeline.md` + `life-events.md` 按日期合併，產出完整時序 |
| `07-life-events.md` 🆕 | 13 批 `life-events.md` 專用合併（股癌 podcast 個人成分極高，獨立管理）|

## Reduce 核心邏輯

### 1. 跨批次頻率統計（識別真信念）

對每條 claim：
- 收集所有提到的 EP 集合
- 計算跨批次 coverage = 出現在幾個不同 batch
- **判定**：
  - coverage ≥ 3 批且 ≥ 5 集 → **核心論點** (core belief)
  - coverage 2 批或 3-4 集 → **穩定論點** (stable claim)  
  - coverage 1 批或 ≤ 2 集 → **情境性言論**（仍記錄，但歸類為「特定情境觀點」）

### 2. 矛盾保留

跨批次發現立場衝突時：
- **不合併、不和稀泥**
- 在 `01-writings.md` 新增「內在張力」section，明列衝突組
- 格式：
  ```
  ### 張力 N: [主題]
  - **早期立場** (B03 EP125): > "..."
  - **近期立場** (B11 EP548): > "..."
  - **可能的轉變原因**: [市場 / 生活 / 思考演化]
  ```
- 這些張力在 Phase D 提煉心智模型時會變成「智識深度」的指標

### 3. 生活事件時序化

合併 13 批 `life-events.md` 時：
- 按日期 / EP 排序
- 對照 `gooaye-life-timeline.md` 的錨點逐一驗證：
  - 已錨定 → 確認集數並補充原文
  - 未錨定 → 加入新條目
  - 衝突 → 標記衝突、兩邊都保留
- 輸出到 `07-life-events.md` 並**反向更新** `references/sources/gooaye-life-timeline.md`

## 工具支援

利用 nuwa 已提供的 `scripts/merge_research.py`（已在 skills/gooaye-perspective/scripts/）：
- 原設計是掃 6 檔、統計來源數 / 一手/二手占比
- 本專案需**改寫 / 擴充**：
  - 加入跨批次頻率統計函數
  - 加入「EP 引用完整性」check（每條 claim 是否都有 EP 錨）
- 建議新增 `scripts/reduce_shards.py`（Phase C 專用）：
  ```python
  # 偽碼
  def reduce():
    shards = scan('references/research/shards/')
    for category in ['claims','expressions','decisions','life-events','timeline']:
      merged = merge_across_shards(shards, category)
      dedup_with_frequency(merged)
      write_to_canonical_file(merged)
    validate_ep_references()
    write_conflict_report()
  ```

## 主 agent vs subagent 分工

Phase C 建議**主 agent 手動執行**，不 spawn subagent：
- 合併需要大量判斷（去重邊界、矛盾認定、頻率閾值）
- 主 agent 有 Phase B 全過程 context，最懂哪些值得保留
- 若語料量過大，可以分 3 趟：
  - 趟 1：`claims.md` + `03-expression-dna.md`
  - 趟 2：`05-decisions.md` + `06-timeline.md` + `07-life-events.md`
  - 趟 3：`02-conversations.md` + `04-external-views.md`

## Phase C 完成標準

- [ ] 7 個 canonical 檔案都寫好（`01-*.md` 到 `07-*.md`）
- [ ] `04-external-views.md` 明確標註「本專案純本地語料，無批評類外部意見」
- [ ] 至少識別出 **≥ 3 個核心論點**（coverage ≥ 3 批）
- [ ] 至少識別出 **≥ 2 對內在張力**（跨時期立場衝突）
- [ ] `gooaye-life-timeline.md` 已反向更新（加入 Phase B 新發現的錨點）
- [ ] 產出 `phase-c-done.md` 摘要：合併統計、top claims、top conflicts

## Phase C → Phase D 交棒

Phase D 會以 `references/research/01-*.md` 到 `07-*.md` 為唯一輸入，提煉心智模型、決策啟發式、表達 DNA。因此 Phase C 品質決定 Phase D 上限。

**主 agent 在 Phase C 結束時必須 review**：
- 抽查每個 canonical 檔案 3-5 條隨機記錄
- 確認 EP 引用 valid（grep corpus 能找到）
- 確認沒有憑訓練語料編造的段落
- 若有疑慮 → 回 Phase B 補跑對應批次
