# 換機器接續 · Quick Start

> 當你在另一台機器繼續這個專案時的 checklist。預估 setup 時間 **10-15 分鐘**。

## 1. 前置檢查

在新機器的 terminal 跑：

```bash
# Claude Code 已裝
which claude                              # 應該有輸出

# Python 3 可用
python3 --version                          # 應 ≥ 3.8

# git 可用
git --version
```

## 2. 確認 skill 目錄同步

這份專案的所有**設計與計劃**都在 `~/.claude/skills/gooaye-perspective/`，包括：
- 4 個 phase execution plans
- batch-plan.md、batch-index.json
- 3 個 sources 文件（life-timeline / classic-episodes / companies）
- extraction-framework + skill-template（來自 nuwa）

### 同步方式（擇一）

#### 方式 A：rsync（建議）

從原機器拉：

```bash
# 在新機器
mkdir -p ~/.claude/skills
rsync -avz -e ssh \
  原機器IP:~/.claude/skills/gooaye-perspective/ \
  ~/.claude/skills/gooaye-perspective/
```

#### 方式 B：git（若已推到自己 repo）

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone <your-repo>/gooaye-perspective.git
```

#### 方式 C：手動 zip + scp

```bash
# 原機器
cd ~/.claude/skills
tar -czf gooaye-perspective.tgz gooaye-perspective/
scp gooaye-perspective.tgz 新機器:~/

# 新機器
mkdir -p ~/.claude/skills
cd ~/.claude/skills
tar -xzf ~/gooaye-perspective.tgz
```

### 驗證 skill 目錄

```bash
ls ~/.claude/skills/gooaye-perspective/
# 應看到：PLAN.md  RESUMING-ON-NEW-MACHINE.md  references/  research-meta/  scripts/  sources/

ls ~/.claude/skills/gooaye-perspective/references/sources/
# 應看到：gooaye-life-timeline.md  gooaye-classic-episodes.md  gooaye-companies.md

ls ~/.claude/skills/gooaye-perspective/research-meta/
# 應看到 batch-index.json + batch-plan.md + phase-b/c/d/e-execution-plan.md
```

## 3. 補上語料（關鍵）

Skill repo 是 **public**，所以原始語料（`corpus/`）已加入 `.gitignore`、不會隨 `git clone` 過來。必須另外從 private corpus repo 複製進去。

### 目標結構

```
<skill-root>/corpus/
├── episodes/               （662 `EPxxx_*.md`）
├── interview-transcript/   （7 `.md`）
└── episodes.json
```

### 從 private repo 複製

```bash
# 先把 private corpus clone 下來（任意位置都可）
git clone git@github.com:chikangpai/gooaye-podcast.git /tmp/gooaye-podcast

# 複製到 skill 的 corpus/
SKILL=~/code/gooaye-perspective            # 或 $(readlink ~/.claude/skills/gooaye-perspective)
mkdir -p "$SKILL/corpus"
cp -R /tmp/gooaye-podcast/episodes            "$SKILL/corpus/"
cp -R /tmp/gooaye-podcast/interview-transcript "$SKILL/corpus/"
cp    /tmp/gooaye-podcast/episodes.json        "$SKILL/corpus/"

# 驗證
ls "$SKILL/corpus/episodes" | wc -l            # 應 ≥ 662
ls "$SKILL/corpus/interview-transcript" | wc -l # 應 7+
```

### ⚠️ Skill 放在不同路徑時

所有計劃檔內 hardcoded 的是 `/Users/chikangpai/code/gooaye-perspective/corpus/...`。若新機器 skill 放在別處，全域替換：

```bash
cd <new-skill-root>
grep -rl '/Users/chikangpai/code/gooaye-perspective/corpus' . | \
  xargs sed -i '' 's|/Users/chikangpai/code/gooaye-perspective|/NEW/PATH|g'
```

（macOS: `sed -i ''`；Linux: `sed -i`）

## 4. 驗證 Phase A 產出完整性

```bash
cd ~/.claude/skills/gooaye-perspective

# 所有計劃檔存在
for f in PLAN.md RESUMING-ON-NEW-MACHINE.md \
         references/sources/gooaye-life-timeline.md \
         references/sources/gooaye-classic-episodes.md \
         references/sources/gooaye-companies.md \
         research-meta/batch-index.json \
         research-meta/batch-plan.md \
         research-meta/phase-b-execution-plan.md \
         research-meta/phase-c-execution-plan.md \
         research-meta/phase-d-execution-plan.md \
         research-meta/phase-e-execution-plan.md; do
  [ -f "$f" ] && echo "✓ $f" || echo "✗ MISSING: $f"
done
```

## 5. 恢復 session

在新機器開 Claude Code：

```bash
cd /Users/chikangpai/code/nuwa-skill    # 或你原本的工作目錄
claude
```

進到 session 後，丟一句給我：

> 「繼續 gooaye-perspective 蒸餾。讀 `~/.claude/skills/gooaye-perspective/PLAN.md` 確認狀態、告訴我下一步做什麼」

我會：
1. 讀 PLAN.md 確認目前 phase
2. 讀 `research-meta/` 對應 phase 計劃
3. 問你是否啟動 / 調整，或直接 spawn subagents

## 6. 已完成階段的 artifact（若有）

接續時若已完成某些 phase，對應產出應該也同步過來：

| 已完成 Phase | 驗證檔案 |
|---|---|
| Phase B | `references/research/shards/batch-{00,01,...,12}/` 都存在 |
| Phase C | `references/research/01-writings.md` ~ `07-life-events.md` 存在 |
| Phase D | `SKILL.md` 根目錄存在 |
| Phase E | `research-meta/phase-e-done.md` 存在 |

驗證：

```bash
cd ~/.claude/skills/gooaye-perspective
ls references/research/ 2>/dev/null
ls SKILL.md 2>/dev/null
ls research-meta/phase-*-done.md 2>/dev/null
```

跟目前 PLAN.md 的「當前狀態」比對。

## 7. 遇到問題？

| 問題 | 解法 |
|---|---|
| 找不到 corpus | 確認 `<skill-root>/corpus/` 有東西；沒有的話從 private repo `chikangpai/gooaye-podcast` 複製進去 |
| Phase B subagent spawn 失敗 | 檢查 Claude Code permissions；或逐批手動跑 |
| 某集抓不到 | 可能在缺檔範圍（EP643-653 已知 gap），標記略過 |
| quality_check.py 報錯 | 見 nuwa 原始 repo `~/.claude/skills/nuwa-skill/` 的 scripts 使用說明 |
| SKILL.md 已存在但你想重跑 | `mv SKILL.md SKILL.md.bak`，重啟 Phase D |

## 8. 原機器快照建議（離開前）

**在原機器**（brandonpai 主 mac）做快照，方便以後回溯：

```bash
cd ~/.claude/skills
tar -czf ~/gooaye-perspective-snapshot-$(date +%Y%m%d).tgz gooaye-perspective/
# 丟 Dropbox / Drive / Git
```

這樣即便不同步也永遠有備份。
