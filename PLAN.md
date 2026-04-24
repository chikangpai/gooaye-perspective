# 股癌蒸餾 · 主計劃書

> **讀這份文件就能接手這個專案**。所有具體細節都在子文件裡，這裡只放「現在到哪」和「去哪找什麼」。
> **版本**：2026-04-23 · Phase A 完成

## 一句話

用 nuwa 的本地語料優先模式蒸餾《股癌 Gooaye》主持人謝孟恭，把 **643 集 podcast + 7 份訪談**提煉成一個可運行的 Claude Skill，讓 AI 能像股癌一樣思考 / 表達 / 做配置判斷。

## 當前狀態（2026-04-23）

- ✅ **Phase A 完成**：目錄 scaffold、batch 索引、批次計劃、生活時間線、經典集數地圖、兩家公司資訊
- 🟡 **Phase B 未啟動**：等用戶 go
- ⬜ **Phase C-E**：依序執行

## 目錄地圖

```
~/.claude/skills/gooaye-perspective/
│
├── PLAN.md                                  ← ⭐ 你現在讀的這份
├── RESUMING-ON-NEW-MACHINE.md               ← 跨機器接續指引
│
├── references/                              ← skill 必要資料
│   ├── extraction-framework.md             （nuwa 原版、三重驗證方法論）
│   ├── skill-template.md                   （nuwa 原版、最終 SKILL.md 模板）
│   └── sources/                             ← 蒸餾必讀的 Phase A 產出
│       ├── gooaye-life-timeline.md         ⭐ bio + 家庭 + 旅行 + 事業時間線
│       ├── gooaye-classic-episodes.md      ⭐ 11 集經典 + 新手補課順序
│       └── gooaye-companies.md             ⭐ 肖楠 vs 烏杉 兩家資本公司
│
├── research-meta/                           ← 蒸餾流程 meta，不是最終產出
│   ├── batch-index.json                    （643 集 → 13 批 JSON）
│   ├── batch-plan.md                       ⭐ 13 批 × 市場 + 人生 + 必讀 EP
│   ├── phase-b-execution-plan.md           ⭐ Phase B 完整執行計劃
│   ├── phase-c-execution-plan.md           ⭐ Phase C reduce 計劃
│   ├── phase-d-execution-plan.md           ⭐ Phase D 提煉 + 組裝計劃
│   └── phase-e-execution-plan.md           ⭐ Phase E 驗證計劃
│
├── scripts/                                 ← 工具
│   ├── merge_research.py                   （nuwa 原版）
│   └── quality_check.py                    （nuwa 原版）
│
├── sources/                                 ← 語料 stub（實際語料在外部，見下方路徑）
│   └── transcripts/
│
└── SKILL.md                                 ← ⚠️ 尚未生成，Phase D 產出
```

### 外部依賴（**不在 skill 目錄內**）

| 類型 | 路徑 | 規模 |
|---|---|---|
| Podcast transcripts | `/Users/brandonpai/Desktop/3-Projects-Gooaye/gooaye-podcast/episodes/` | 643 `.md` / 36 MB |
| Interview transcripts | `/Users/brandonpai/Desktop/3-Projects-Gooaye/gooaye-podcast/interview-transcript/` | 7 `.md` / 384 KB |
| Metadata JSON | `/Users/brandonpai/Desktop/3-Projects-Gooaye/gooaye-podcast/episodes.json` | — |

換機器時這些路徑要重新 remap，見 `RESUMING-ON-NEW-MACHINE.md`。

## 五階段全景

| Phase | 目標 | 輸入 | 輸出 | 耗時估 |
|:---:|---|---|---|---|
| **A** ✅ | 設計批次 + 錨定生活 | 語料 + 用戶整理 | `batch-plan.md` + `sources/*.md` | 已完成 |
| **B** | Map 抽取 | 語料 + Phase A | `shards/batch-NN/` × 13 | ~90 分 |
| **C** | Reduce 合併 | 13 個 shards | `research/01-07.md` canonical 檔案 | ~30 分 |
| **D** | 提煉 + 組裝 | 7 個 canonical | `SKILL.md` | ~30 分 |
| **E** | 驗證 | `SKILL.md` | 通過的 `SKILL.md` 最終版 | ~20 分 |

**預估總耗時**：Phase B-E 全跑約 **2-3 小時**（包含主 agent 判斷間隙）。

## 蒸餾對象關鍵事實速查

- **謝孟恭**，1992-06-06 生，輔大財經法律系
- 老婆 **Liza**（摩爾多瓦生 → 西班牙長大 → 媽媽後搬義大利）
- 大兒子 **諾亞 Noah**（~2021-09）、二兒子 **Killian**（2025-04）
- 兩家投資公司：**肖楠資本 Calocedrus**（2025-08-12 公開、中期）+ **烏杉資本 Lunta**（2025-11-05 公開、Seed）
- 代表梗：**主委 / 秋口幹你娘 / 阿呆谷 / 柳樹理論 / 工具箱**
- 首本書：**《灰階思考》**（2021-04-20）

詳見 `references/sources/gooaye-life-timeline.md` 和 `gooaye-companies.md`。

## 用戶偏好錨點（重要）

1. **Phase B 並行度**：一次 **3-4 批**，分 4 波跑完（**不一次 13 批同時跑**，避免資源打架）
2. **B12 不拆**（73 集）— 因為是最新、最成熟版本、訊息密度最高，值得整批完整處理
3. **本地優先**：**絕對不做網路搜尋**，所有素材來自用戶硬碟
4. **保留衝突**：遇到矛盾不調和、不和稀泥
5. **B00 Interviews 獨立批**：因為是外人提問觸發的長對話，訊號密度比 podcast 還高

## 啟動 Phase B 的觸發

當用戶準備好、在 Claude Code session 裡說任一句：
- 「Phase B go」
- 「啟動 map 抽取」
- 「開始 Wave 1」

主 agent 會：
1. 讀 `phase-b-execution-plan.md`
2. 讀 `batch-plan.md` 對應批次
3. 按 Wave 1 (B00+B01+B02+B03) 同時 spawn 4 個 subagent
4. 等所有完成、review、continue Wave 2...

## 常見問題

**Q: 為什麼 642 而不是 654？**
缺 EP643-EP653（11 集）+ EP162 無日期。`batch-index.json` 的 `total_episodes` = 643。

**Q: 兩家公司都是謝孟恭開的？**
是。**肖楠**（中期陪跑）+ **烏杉**（Seed First Check，跟文策院國家隊合作）。徐震在兩家都是董事（但他是嘖嘖共同創辦人，跟「過世聽眾小徐」是不同人）。

**Q: Skill 完成後放哪？**
`~/.claude/skills/gooaye-perspective/SKILL.md`（本目錄根目錄）。Claude Code 會自動偵測、任何 session 都能觸發。

**Q: 能分享給朋友嗎？**
整個 `~/.claude/skills/gooaye-perspective/` 目錄是 self-contained，可以打包 / git push。但注意：
- 外部語料（`/Users/brandonpai/Desktop/3-Projects-Gooaye/`）不在其中，朋友收到的 skill 是**已經蒸餾好的成品**，不需要原始語料
- 若朋友想自己蒸餾則需要自備語料
