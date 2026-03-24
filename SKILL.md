---
name: agentic-score
version: 1.1.0
description: |
  Scans your ~/.claude/ setup and scores your agentic level across 6 dimensions.
  Outputs your Dragon Ball character, Agentic Score, radar chart, and top 3 next moves.
  Use when someone asks "how agentic am I?", "what's my agentic level?", "score my setup",
  "run agentic-score", or wants to understand what to improve in their Claude Code system.
  Always run this when someone wants to benchmark their Claude Code setup.
---

# Agentic Score

Scans `~/.claude/` and any active projects to produce a scored card across 6 dimensions.
Assigns a Dragon Ball character and outputs the top 3 highest-leverage next moves.

---

## Step -1 — Version check

Before doing anything else, silently check if a newer version is available:

```bash
LOCAL_VERSION="1.1.0"
REMOTE_VERSION=$(curl -sf --max-time 3 \
  https://raw.githubusercontent.com/Adapt-to-AI/agentic-score/main/SKILL.md \
  | grep "^version:" | head -1 | awk '{print $2}')
```

If `REMOTE_VERSION` is non-empty and differs from `LOCAL_VERSION`, print this banner **before** any other output:

```
┌─────────────────────────────────────────────────────────────────┐
│  ⚠️  Update available: v[LOCAL_VERSION] → v[REMOTE_VERSION]     │
│  Scoring improved — quality over quantity in Skills, Hooks,     │
│  and Autonomy. Run this to update:                              │
│                                                                 │
│  curl -o ~/.claude/skills/agentic-score/SKILL.md \             │
│    https://raw.githubusercontent.com/Adapt-to-AI/              │
│    agentic-score/main/SKILL.md                                  │
└─────────────────────────────────────────────────────────────────┘
```

If curl fails or times out, skip silently — never block the scan over a version check.

---

## Step 0 — Scan (silent, broad, tool-agnostic)

Run all checks silently before producing any output. Search broadly — external users
have different paths, tool names, and conventions. Never assume a specific path.
Score the *outcome*, not the specific tool used.

```bash
# ── SKILLS ──────────────────────────────────────────────────────────────────
SKILL_COUNT=$(ls ~/.claude/skills/ 2>/dev/null | wc -l | tr -d ' ')
FRONTMATTER_COUNT=$(find ~/.claude/skills/ -name "SKILL.md" | xargs grep -l "^---" 2>/dev/null | wc -l | tr -d ' ')
MISSING_FM=$((SKILL_COUNT - FRONTMATTER_COUNT))
GATE_COUNT=$(grep -rl "gate\|⛔\|GATE\|self.review\|design.preview" ~/.claude/skills/ 2>/dev/null | wc -l | tr -d ' ')

# ── HOOKS ───────────────────────────────────────────────────────────────────
HOOK_COUNT=$(cat ~/.claude/settings.json 2>/dev/null | python3 -c \
  "import sys,json; d=json.load(sys.stdin); hooks=d.get('hooks',{}); \
  print(sum(len(v) if isinstance(v,list) else 1 for v in hooks.values()))" \
  2>/dev/null || echo "0")

# ── CLAUDE.MD ───────────────────────────────────────────────────────────────
CLAUDE_LINES=$(wc -l < ~/.claude/CLAUDE.md 2>/dev/null || echo "0")
RULE_COUNT=$(grep -c "RULE\|ENFORCEMENT\|GATE" ~/.claude/CLAUDE.md 2>/dev/null || echo "0")

# ── MEMORY & KNOWLEDGE ──────────────────────────────────────────────────────
# Search broadly — failures/ may be at different paths
FAILURES_COUNT=$(find ~/.claude -type d -name "failures" 2>/dev/null | \
  xargs -I{} ls {} 2>/dev/null | wc -l | tr -d ' ')
# Lessons: any markdown with 5+ ### headers anywhere in ~/.claude/
LESSONS_COUNT=$(find ~/.claude -name "*.md" 2>/dev/null | \
  xargs grep -l "^###" 2>/dev/null | \
  xargs grep -c "^###" 2>/dev/null | \
  awk -F: '$2>=5 {sum+=$2} END{print sum+0}')
# Knowledge base: any persistent knowledge folder
KNOWLEDGE_EXISTS=$(find ~/.claude -type d \( -name "knowledge" -o -name "memory" -o -name "learnings" \) 2>/dev/null | head -1 | grep -q "." && echo "YES" || echo "NO")

# ── PROJECTS ────────────────────────────────────────────────────────────────
# Search common project locations — not just ~/projects/
PROJECT_CLAUDE_COUNT=$(find ~ -name "CLAUDE.md" \
  -not -path "~/.claude/*" \
  -not -path "*/node_modules/*" \
  -maxdepth 6 2>/dev/null | wc -l | tr -d ' ')
PRODUCT_STATE_COUNT=$(find ~ -name "PRODUCT_STATE.md" -maxdepth 6 2>/dev/null | wc -l | tr -d ' ')
# Mission Control: search by name, not path
MISSION_CONTROL=$(find ~ -name "MISSION_CONTROL.md" -maxdepth 5 2>/dev/null | head -1 | grep -q "." && echo "YES" || echo "NO")
STATUS_SKILL=$(find ~/.claude/skills -name "SKILL.md" 2>/dev/null | \
  xargs grep -l "status\|mission.control\|product.state" 2>/dev/null | head -1 | grep -q "." && echo "YES" || echo "NO")

# ── SELF-IMPROVING WIRES ────────────────────────────────────────────────────
# Score the pattern, not the specific skill name
# Wire 1: any skill that updates CLAUDE.md
WIRE1=$(find ~/.claude/skills -name "SKILL.md" 2>/dev/null | \
  xargs grep -l "CLAUDE\.md\|claude\.md" 2>/dev/null | head -1 | grep -q "." && echo "YES" || echo "NO")
# Wire 2: any skill that handles test failures automatically
WIRE2=$(find ~/.claude/skills -name "SKILL.md" 2>/dev/null | \
  xargs grep -l "test.*fail\|fail.*fix\|fix.bug\|auto.*fix" 2>/dev/null | head -1 | grep -q "." && echo "YES" || echo "NO")
# Wire 3: any persistent knowledge folder with files
WIRE3=$(find ~/.claude -type d \( -name "knowledge" -o -name "memory" \) 2>/dev/null | \
  xargs -I{} find {} -name "*.md" 2>/dev/null | head -1 | grep -q "." && echo "YES" || echo "NO")
# Wire 4: any skill that learns from feedback/outreach
WIRE4=$(find ~/.claude/skills -name "SKILL.md" 2>/dev/null | \
  xargs grep -l "learning\|feedback\|outreach.*learn\|style.rule" 2>/dev/null | head -1 | grep -q "." && echo "YES" || echo "NO")
WIRE_COUNT=$(echo "$WIRE1 $WIRE2 $WIRE3 $WIRE4" | tr ' ' '\n' | grep -c "YES" || echo "0")

# ── CONTROL / INFRASTRUCTURE ────────────────────────────────────────────────
# Tool-agnostic: score the outcome (Mac-off capability), not CCBot specifically
# Check any Telegram bridge
TELEGRAM_BRIDGE=$(find ~ -name ".env" -maxdepth 5 2>/dev/null | \
  xargs grep -l "TELEGRAM\|BOT_TOKEN" 2>/dev/null | head -1 | grep -q "." && echo "YES" || echo "NO")
# Check any running Claude server (systemd, tmux, screen)
CLAUDE_SERVER=$(systemctl list-units --all 2>/dev/null | grep -i "claude\|ccbot\|ccmux" | grep -q "." && echo "YES" || \
  tmux ls 2>/dev/null | grep -qi "claude\|ccbot\|project" && echo "YES" || echo "NO")
# Check for claude-mem or similar memory tool
CLAUDE_MEM=$(pip show claude-mem 2>/dev/null | grep -q "Name" && echo "YES" || \
  find ~ -name "claude-mem" -o -name ".claude-mem" 2>/dev/null | head -1 | grep -q "." && echo "YES" || echo "NO")
# Count Telegram topics (any bridge config)
TELEGRAM_TOPICS=$(find ~ -name ".env" -maxdepth 5 2>/dev/null | \
  xargs grep -h "TOPIC\|topic\|PROJECT" 2>/dev/null | grep -v "^#" | wc -l | tr -d ' ')
# Remote Control
REMOTE_CONTROL=$(grep -i "remote.control\|remoteControl" ~/.claude/settings.json 2>/dev/null | grep -q "." && echo "YES" || echo "NO")

# ── AUTONOMY ────────────────────────────────────────────────────────────────
CRON_COUNT=$(crontab -l 2>/dev/null | grep -v "^#" | grep -c "." || echo "0")
# Any Claude skill running on schedule
SCHEDULED_COUNT=$(crontab -l 2>/dev/null | grep -i "claude\|skill\|ccbot" | grep -c "." || echo "0")
AGENT_SDK=$(find ~ -name "package.json" -maxdepth 6 2>/dev/null | \
  xargs grep -l "agent-sdk\|claude-code" 2>/dev/null | head -1 | grep -q "." && echo "YES" || echo "NO")
# Trigger.dev or any webhook-based automation
AUTOMATION_TOOL=$(find ~ -name "*.json" -o -name "*.ts" -o -name "*.js" 2>/dev/null | \
  xargs grep -l "trigger.dev\|webhook\|n8n\|zapier" 2>/dev/null | head -1 | grep -q "." && echo "YES" || echo "NO")

# ── MCP SERVERS ─────────────────────────────────────────────────────────────
MCP_COUNT=$(cat ~/.claude/settings.json 2>/dev/null | \
  python3 -c "import sys,json; d=json.load(sys.stdin); print(len(d.get('mcpServers',{})))" \
  2>/dev/null || echo "0")
```

Print debug block before scoring:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SCAN RESULTS

  Skills:      [SKILL_COUNT] total · [FRONTMATTER_COUNT] wired · [MISSING_FM] missing frontmatter
  Hooks:       [HOOK_COUNT]
  Rules:       [RULE_COUNT] in CLAUDE.md ([CLAUDE_LINES] lines)
  Lessons:     [LESSONS_COUNT]
  Failures:    [FAILURES_COUNT] files
  Knowledge:   [KNOWLEDGE_EXISTS]
  Projects:    [PROJECT_CLAUDE_COUNT] with CLAUDE.md · [PRODUCT_STATE_COUNT] with PRODUCT_STATE.md
  Mission Ctrl:[MISSION_CONTROL]
  Wires:       [WIRE_COUNT]/4 (W1:[WIRE1] W2:[WIRE2] W3:[WIRE3] W4:[WIRE4])
  Telegram:    [TELEGRAM_BRIDGE] bridge · [TELEGRAM_TOPICS] topics
  Server:      [CLAUDE_SERVER]
  claude-mem:  [CLAUDE_MEM]
  Cron jobs:   [CRON_COUNT] · Claude scheduled: [SCHEDULED_COUNT]
  Agent SDK:   [AGENT_SDK]
  MCP servers: [MCP_COUNT]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Key principle for external users:** If something looks like it should exist but wasn't
detected, note it in the debug block but do NOT silently score 0. Instead note:
"[item] not detected — may exist under different path or name. Score may be conservative."



```bash
# Skills
SKILL_COUNT=$(ls ~/.claude/skills/ 2>/dev/null | wc -l | tr -d ' ')
FRONTMATTER_COUNT=$(find ~/.claude/skills/ -name "SKILL.md" | xargs grep -l "^---" 2>/dev/null | wc -l | tr -d ' ')
MISSING_FM=$((SKILL_COUNT - FRONTMATTER_COUNT))

# Hooks
HOOK_COUNT=$(cat ~/.claude/settings.json 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); hooks=d.get('hooks',{}); print(sum(len(v) if isinstance(v,list) else 1 for v in hooks.values()))" 2>/dev/null || echo "0")

# CLAUDE.md
CLAUDE_LINES=$(wc -l < ~/.claude/CLAUDE.md 2>/dev/null || echo "0")
RULE_COUNT=$(grep -c "RULE\|ENFORCEMENT\|GATE" ~/.claude/CLAUDE.md 2>/dev/null || echo "0")

# Memory & knowledge
FAILURES_COUNT=$(ls ~/.claude/knowledge/failures/ 2>/dev/null | wc -l | tr -d ' ')
LESSONS_COUNT=$(grep -c "^###" ~/.claude/knowledge/*/lessons.md 2>/dev/null || grep -rc "^###" ~/.claude/tasks/lessons.md 2>/dev/null | awk -F: '{sum+=$2} END{print sum}' || echo "0")

# Projects — check multiple locations
PROJECT_COUNT_LOCAL=$(find ~/projects/ -name "CLAUDE.md" -maxdepth 3 2>/dev/null | wc -l | tr -d ' ')
PROJECT_COUNT_HOME=$(find ~ -name "CLAUDE.md" -not -path "~/.claude/*" -maxdepth 5 2>/dev/null | wc -l | tr -d ' ')
MISSION_CONTROL=$(ls ~/.claude/MISSION_CONTROL.md 2>/dev/null && echo "FOUND" || echo "NOT FOUND at ~/.claude/MISSION_CONTROL.md")
PRODUCT_STATE_COUNT=$(find ~ -name "PRODUCT_STATE.md" -maxdepth 6 2>/dev/null | wc -l | tr -d ' ')

# Self-improving wires — check multiple signals
WIRE1=$(grep -l "CLAUDE.md\|claude\.md" ~/.claude/skills/session-insights/SKILL.md 2>/dev/null && echo "FOUND" || echo "NOT FOUND — grep 'CLAUDE.md' in session-insights/SKILL.md returned empty")
WIRE2=$(grep -rl "fix.bug\|fix_bug" ~/.claude/skills/ 2>/dev/null | head -1 | grep -q "." && echo "FOUND" || echo "NOT FOUND — no fix-bug reference in deploy-and-verify or similar")
WIRE3=$(ls ~/.claude/knowledge/ 2>/dev/null && echo "FOUND (knowledge/ exists)" || echo "NOT FOUND")
WIRE4=$(grep -rl "outreach\|style.rules\|style_rules" ~/.claude/knowledge/ 2>/dev/null | head -1 | grep -q "." && echo "FOUND" || echo "NOT FOUND")

# Control / infrastructure
CCBOT=$(ls ~/.ccbot/.env 2>/dev/null && echo "FOUND at ~/.ccbot/.env" || echo "NOT FOUND at ~/.ccbot/.env")
SYSTEMD=$(find /etc/systemd /home -name "ccbot.service" 2>/dev/null | head -1 | grep -q "." && echo "FOUND" || echo "NOT FOUND")
TELEGRAM_TOKEN=$(cat ~/.ccbot/.env 2>/dev/null | grep -i "BOT_TOKEN" | grep -q "." && echo "SET" || echo "NOT SET")
TELEGRAM_TOPICS=$(cat ~/.ccbot/.env 2>/dev/null | grep -i "TOPIC\|topic" | wc -l | tr -d ' ')
CLAUDE_MEM=$(pip show claude-mem 2>/dev/null | grep -q "Name" && echo "INSTALLED" || which claude-mem 2>/dev/null || echo "NOT FOUND")

# Autonomy
CRON_COUNT=$(crontab -l 2>/dev/null | grep -v "^#" | grep -c "." || echo "0")
SCHEDULED_COUNT=$(find ~ -name "*.service" -path "*/systemd/*" 2>/dev/null | wc -l | tr -d ' ')
AGENT_SDK=$(find ~ -name "*.json" -o -name "*.js" 2>/dev/null | xargs grep -l "agent-sdk\|AgentOptions\|ClaudeAgent" 2>/dev/null | head -1 | grep -q "." && echo "FOUND" || echo "NOT FOUND")
```

Print the debug output in this format before scoring:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SCAN DEBUG — what was found

  SKILLS
  ✓ Total skills:        [SKILL_COUNT]
  ✓ With frontmatter:    [FRONTMATTER_COUNT]
  ✗ Missing frontmatter: [MISSING_FM] ← these cannot auto-trigger

  HOOKS
  ✓ Hook count:          [HOOK_COUNT]

  CLAUDE.MD
  ✓ Lines:               [CLAUDE_LINES]
  ✓ Rules found:         [RULE_COUNT]

  KNOWLEDGE
  ✓ failures/ files:     [FAILURES_COUNT]
  ✓ Lessons:             [LESSONS_COUNT]

  PROJECTS
  ✓ CLAUDE.md files:     [PROJECT_COUNT_LOCAL] (~/projects/) / [PROJECT_COUNT_HOME] (home)
  ✓ PRODUCT_STATE.md:    [PRODUCT_STATE_COUNT]
  - Mission Control:     [MISSION_CONTROL]

  SELF-IMPROVING WIRES
  - Wire 1 (→ CLAUDE.md): [WIRE1]
  - Wire 2 (→ fix-bug):   [WIRE2]
  - Wire 3 (knowledge):   [WIRE3]
  - Wire 4 (outreach):    [WIRE4]

  CONTROL
  - CCBot:               [CCBOT]
  - systemd service:     [SYSTEMD]
  - Telegram token:      [TELEGRAM_TOKEN]
  - Telegram topics:     [TELEGRAM_TOPICS]
  - claude-mem:          [CLAUDE_MEM]

  AUTONOMY
  - Cron jobs:           [CRON_COUNT]
  - Scheduled services:  [SCHEDULED_COUNT]
  - Agent SDK:           [AGENT_SDK]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Important scoring notes from debug:**
- If Mission Control NOT FOUND: check actual filename and path, do not penalise if file exists under different name
- If wires NOT FOUND: ask user "I couldn't detect Wire 1 — does session-insights SKILL.md reference updating CLAUDE.md?" and adjust score based on answer
- If projects < 3 but user has droplet: note "Projects on droplet not visible to local scan — ask user for count"
- If Telegram topics = 0 but CCBot found: ask "How many Telegram topics do you have?" before scoring Control

Never silently score 0 on something that might exist but wasn't detectable.
Ask before penalising.

---

## Step 1 — Scan the system

Run all checks silently before producing any output. Collect raw counts and booleans.

```bash
# Skills
ls ~/.claude/skills/ 2>/dev/null | wc -l
find ~/.claude/skills/ -name "SKILL.md" | xargs grep -l "^---" 2>/dev/null | wc -l

# Hooks
cat ~/.claude/settings.json 2>/dev/null | grep -c "hook\|Hook"

# CLAUDE.md rules
wc -l ~/.claude/CLAUDE.md 2>/dev/null
grep -c "RULE\|ENFORCEMENT\|GATE" ~/.claude/CLAUDE.md 2>/dev/null

# Memory & knowledge
ls ~/.claude/knowledge/ 2>/dev/null
ls ~/.claude/knowledge/failures/ 2>/dev/null | wc -l
cat ~/.claude/knowledge/*/lessons.md 2>/dev/null | grep -c "^###"

# Projects
find ~ -name "CLAUDE.md" -not -path "~/.claude/*" 2>/dev/null | wc -l
find ~ -name "PRODUCT_STATE.md" 2>/dev/null | wc -l
ls ~/.claude/MISSION_CONTROL.md 2>/dev/null

# Session insights / self-improving wires
grep -r "CLAUDE.md" ~/.claude/skills/session-insights/SKILL.md 2>/dev/null
grep -r "fix-bug\|fix_bug" ~/.claude/skills/deploy-and-verify/SKILL.md 2>/dev/null
ls ~/.claude/knowledge/failures/ 2>/dev/null
grep -r "claude-mem\|claude_mem" ~/.claude/skills/session-insights/SKILL.md 2>/dev/null

# Control / infrastructure
ls ~/.ccbot/.env 2>/dev/null
cat ~/.ccbot/.env 2>/dev/null | grep -i "telegram\|bot_token"
grep -r "remote.control\|remote-control" ~/.claude/settings.json 2>/dev/null

# Autonomy / scheduled tasks
crontab -l 2>/dev/null
find ~ -name "*.service" 2>/dev/null | head -5
grep -r "scheduled\|cron\|loop" ~/.claude/skills/ 2>/dev/null | grep -i "daily\|weekly\|sunday" | wc -l
```

Also check: MCP servers in settings.json, active project count, gates per skill (look for ⛔ or "gate" in SKILL.md files).

---

## Step 2 — Score each dimension

Score 0–10 in half-steps. Not a strict ladder — each check adds weight.

### 1. Skills & Memory
*How smart is the system?*

Quality over quantity. A tight system with 10 surgical skills beats 100 copy-paste ones.

| Check | Points |
|---|---|
| CLAUDE.md exists with rules (not just boilerplate) | +1.0 |
| 10+ skills | +1.0 |
| 30+ skills with YAML frontmatter | +1.5 |
| 75+ skills | +0.5 |
| 5+ skills with gates (⛔ or "gate" or "self-review") | +1.0 |
| Human review checklist inside any skill | +0.5 |
| Multi-skill chain defined (skill A → B → C in CLAUDE.md or skill) | +0.5 |
| Framework or architecture doc exists (AGENTS.md, PLAYBOOK.md, etc.) | +0.5 |
| tasks/lessons.md with 10+ lessons | +0.5 |
| tasks/lessons.md with 30+ lessons | +1.0 |
| ~/.claude/knowledge/ directory exists | +0.5 |
| architecture-decisions.md exists | +0.5 |
| failures/ directory exists with files | +1.0 |
| ISC rule in CLAUDE.md | +0.5 |

### 2. Hooks & Gates
*How enforced are the rules?*

Coverage matters, not count. 3 hooks covering all event types beats 7 redundant ones.

| Check | Points |
|---|---|
| 1-2 hooks in settings.json | +1.5 |
| 4+ hooks | +1.5 |
| All 4 event types covered (Start/End/PrePush/PostEdit) OR 7+ hooks | +1.5 |
| SessionStart + SessionEnd both covered | +1.0 |
| PrePush blocker hook exists | +1.0 |
| PostEdit tsc/lint hook exists | +1.0 |
| Self-review gate in main build skill | +1.0 |
| Design Preview Enforcement in CLAUDE.md | +0.5 |
| Rule verification greps in SessionStart | +1.0 |

### 3. Multi-project & Context
*How much context survives?*

| Check | Points |
|---|---|
| 2+ projects with own CLAUDE.md | +1.5 |
| 5+ projects | +1.5 |
| MISSION_CONTROL.md exists | +1.5 |
| /status skill exists and auto-runs | +1.5 |
| PRODUCT_STATE.md per project | +1.0 |
| product-catalog.json exists | +1.0 |
| /drift-check skill exists | +1.0 |
| Cross-project scanning in any skill | +1.0 |

### 4. Control
*How freely can you direct it?*

Score the **outcome** — can the user direct Claude from anywhere with Mac off?
Not the specific tool (CCBot, n8n, custom webhook — all equivalent).

| Check | Points |
|---|---|
| Claude Code installed and in use | +1.0 |
| Remote Control configured | +1.0 |
| claude-mem or equivalent memory tool installed | +1.0 |
| Any Telegram bridge configured (.env with BOT_TOKEN) | +2.0 |
| Server running any Claude bridge (systemd/tmux confirmed) | +2.0 |
| Mac-off capability verified (bridge + server both live) | +3.0 |
| 2+ project sessions remotely accessible | +2.0 |
| 3+ project sessions (full project coverage) | +1.5 |
| Voice input wired (Wispr Flow or similar) | +1.0 |

**Why Mac-off gets +3.0:** This is a category change. Remote Control needs Mac open.
A running server with Telegram bridge means Claude works while you sleep, from any
device, for any project. Tool doesn't matter — CCBot, n8n, custom script all score equally.

**CCBot multiplier:** if Telegram bridge + server + 3+ topics all confirmed →
multiply Control dimension score × 1.2

### 5. Self-improving
*Does it get smarter alone?*

Score the **pattern**, not the specific skill name. External users won't have
`session-insights` or `deploy-and-verify` — but may have equivalent wires.

| Check | Points |
|---|---|
| Any skill that captures session learnings exists | +1.5 |
| Wire 1 detected: any skill references updating CLAUDE.md | +1.5 |
| Wire 2 detected: any skill handles test failures → auto-fix | +1.5 |
| Wire 3 detected: knowledge/ memory/ or learnings/ folder with files | +1.5 |
| Wire 4 detected: any skill learns from feedback or outreach | +1.0 |
| failures/ or equivalent directory with 3+ files | +1.0 |
| Memory tool (claude-mem or equivalent) wired in | +1.0 |
| autoresearch planned or running | +1.0 |

### 6. Autonomy
*Does it work without being asked?*

One cron job is a start, not a system. Score scales with breadth and coverage.

| Check | Points |
|---|---|
| Any cron job running | +1.0 |
| 2+ Claude-specific scheduled tasks | +1.5 |
| 3+ scheduled tasks (system runs itself) | +1.5 |
| /daily-metrics or equivalent running on schedule | +1.0 |
| Desktop Scheduled Tasks confirmed | +1.0 |
| /product-ux-review or /session-insights scheduled | +1.0 |
| Trigger.dev in any project | +1.0 |
| Agent SDK configured or planned | +1.0 |
| Post-deploy monitoring (/loop) | +1.0 |

---

## Step 3 — Calculate Agentic Score

```
Agentic Score =
  (skill count × 2)
+ (hook count × 15)
+ (self-improving wires firing × 50)
+ (lesson count × 3)
+ (MCP servers × 10)
+ (active projects × 20)
+ (average gates per skill × 5)
+ (server running × 100)
+ (failures/ files × 10, max 50)
+ (scheduled tasks confirmed × 75)
+ (Agent SDK live × 150)
+ (CCBot + Telegram topics × 75 each)   ← Mac-off capability
+ (3+ Telegram topics bonus × 100)      ← parallel project control

CCBot multiplier: if CCBot + systemd + 3+ topics all confirmed →
  multiply Control dimension score × 1.2
  (reflects category change, not just incremental improvement)
```

---

## Step 4 — Assign character

Overall level = average of 6 dimensions.

| Score | Level | Character | Description |
|---|---|---|---|
| 0–500 | 1.0–2.5 | **Krillin** | Tries hard, limited power, always needs help |
| 500–1000 | 2.5–4.0 | **Yamcha** | Some skills, good intentions, not quite there |
| 1000–1500 | 4.0–5.5 | **Tien** | Disciplined, methodical, solid but manually operated |
| 1500–2000 | 5.5–6.5 | **Piccolo** | Smart, strategic, proper systems, trains the agent |
| 2000–2500 | 6.5–7.5 | **Vegeta** | Powerful, controlled, operates independently |
| 2500–3000 | 7.5–8.5 | **Goku** | Balanced mastery, everything connected, keeps growing |
| 3000–3500 | 8.5–9.0 | **Super Saiyan** | Transformed. System runs without effort. |
| 3500+ | 9.0–10.0 | **Ultra Instinct** | System acts without conscious input. You just approve. |

---

## Step 5 — Output the card

Use box-drawing characters. No markdown headers inside the card. Clean, screenshot-worthy.

```
┌──────────────────────────────────────────────┐
│  AGENTIC SCORE                               │
│  scanned: ~/.claude/  •  [date]              │
├──────────────────────────────────────────────┤
│                                              │
│  LEVEL  [X.X] / 10   ⬡ [CHARACTER]          │
│  "[one-line character description]"          │
│                                              │
│  Skills & Memory    [bar] [score]            │
│  Hooks & Gates      [bar] [score]            │
│  Multi-project      [bar] [score]            │
│  Control            [bar] [score]            │
│  Self-improving     [bar] [score]            │
│  Autonomy           [bar] [score]            │
│                                              │
│  AGENTIC SCORE  [number]                     │
│                                              │
├──────────────────────────────────────────────┤
│  YOUR STRONGEST: [dimension]                 │
│  YOUR GAP: [dimension]                       │
│                                              │
│  NEXT FORM: [next character]                 │
│  "[what unlocks the next form]"              │
├──────────────────────────────────────────────┤
│  TOP 3 MOVES                                 │
│  1. [specific action — 1 sentence]           │
│  2. [specific action — 1 sentence]           │
│  3. [specific action — 1 sentence]           │
└──────────────────────────────────────────────┘
```

Progress bar format (20 chars wide):
- 10.0 = `████████████████████`
- 7.5  = `███████████████░░░░░`
- 5.0  = `██████████░░░░░░░░░░`
- 2.5  = `█████░░░░░░░░░░░░░░░`

---

## Example outputs

### Example 1 — Krillin (beginner)

```
┌──────────────────────────────────────────────┐
│  AGENTIC SCORE                               │
│  scanned: ~/.claude/  •  Jan 2026            │
├──────────────────────────────────────────────┤
│                                              │
│  LEVEL  2.0 / 10   ⬡ KRILLIN                │
│  "Trying hard. The potential is there."      │
│                                              │
│  Skills & Memory    ████░░░░░░░░░░░░░░░░  2.0│
│  Hooks & Gates      ██░░░░░░░░░░░░░░░░░░  1.0│
│  Multi-project      ███░░░░░░░░░░░░░░░░░  1.5│
│  Control            ████░░░░░░░░░░░░░░░░  2.0│
│  Self-improving     █░░░░░░░░░░░░░░░░░░░  0.5│
│  Autonomy           █░░░░░░░░░░░░░░░░░░░  0.5│
│                                              │
│  AGENTIC SCORE  124                          │
│                                              │
├──────────────────────────────────────────────┤
│  YOUR STRONGEST: Control                     │
│  YOUR GAP: Self-improving, Autonomy          │
│                                              │
│  NEXT FORM: YAMCHA                           │
│  "Build 10 skills. Write your first rules."  │
├──────────────────────────────────────────────┤
│  TOP 3 MOVES                                 │
│  1. Write a CLAUDE.md with 5 real rules now  │
│  2. Build your first reusable SKILL.md       │
│  3. Add 1 hook (tsc after every edit)        │
└──────────────────────────────────────────────┘
```

---

### Example 2 — Piccolo (intermediate)

```
┌──────────────────────────────────────────────┐
│  AGENTIC SCORE                               │
│  scanned: ~/.claude/  •  Feb 2026            │
├──────────────────────────────────────────────┤
│                                              │
│  LEVEL  5.5 / 10   ⬡ PICCOLO                │
│  "Strategic. The system works. You work it." │
│                                              │
│  Skills & Memory    ██████████████░░░░░░  7.0│
│  Hooks & Gates      ████████░░░░░░░░░░░░  4.0│
│  Multi-project      ████████████░░░░░░░░  6.0│
│  Control            ██████░░░░░░░░░░░░░░  3.0│
│  Self-improving     ████████░░░░░░░░░░░░  4.0│
│  Autonomy           ████░░░░░░░░░░░░░░░░  2.0│
│                                              │
│  AGENTIC SCORE  1,240                        │
│                                              │
├──────────────────────────────────────────────┤
│  YOUR STRONGEST: Skills & Memory             │
│  YOUR GAP: Autonomy, Control                 │
│                                              │
│  NEXT FORM: VEGETA                           │
│  "Get off your laptop. Control from phone."  │
├──────────────────────────────────────────────┤
│  TOP 3 MOVES                                 │
│  1. Set up Remote Control (/config → enable) │
│  2. Add 3 more hooks (PrePush, SessionEnd)   │
│  3. Wire /session-insights → CLAUDE.md       │
└──────────────────────────────────────────────┘
```

---

### Example 3 — Super Saiyan (advanced, based on real map)

```
┌──────────────────────────────────────────────┐
│  AGENTIC SCORE                               │
│  scanned: ~/.claude/  •  Mar 2026            │
├──────────────────────────────────────────────┤
│                                              │
│  LEVEL  8.8 / 10   ⬡ SUPER SAIYAN           │
│  "Transformed. The system carries the load." │
│                                              │
│  Skills & Memory    ███████████████████░  9.5│
│  Hooks & Gates      ███████████████░░░░░  7.5│
│  Multi-project      ████████████████░░░░  8.0│
│  Control            ███████████████████░  9.5│
│  Self-improving     █████████████████░░░  8.5│
│  Autonomy           █████████████░░░░░░░  6.5│
│                                              │
│  AGENTIC SCORE  3,247                        │
│                                              │
├──────────────────────────────────────────────┤
│  YOUR STRONGEST: Skills & Control            │
│  YOUR GAP: Autonomy                          │
│                                              │
│  NEXT FORM: ULTRA INSTINCT                   │
│  "Make the system act. Stop being the trigger"│
├──────────────────────────────────────────────┤
│  TOP 3 MOVES                                 │
│  1. Verify Desktop Scheduled Tasks running   │
│  2. Wire autoresearch on /outreach overnight │
│  3. Deploy Agent SDK on server               │
└──────────────────────────────────────────────┘
```

---

## Notes

- Never guess — only score what you can verify by reading actual files
- If ~/.claude/ doesn't exist, score everything 0 and explain how to start
- The top 3 moves must be specific and actionable — not "improve your skills"
- **TOP 3 MOVES RULE: Only suggest moves whose underlying check FAILED in the scan.** Never suggest setting up something the user already has. Cross-reference your scan results before writing each move.
- Character description should feel like a compliment, not a roast
- Always show the next form and what unlocks it — the gap is the point

---

## Step 6 — Generate HTML report

After outputting the terminal card, generate a full HTML report and open it automatically.

Write the file to `~/.claude/reports/agentic-score-[YYYY-MM-DD].html` using the template below.
Then open it:
- Mac: `open ~/.claude/reports/agentic-score-[date].html`
- Linux: `xdg-open ~/.claude/reports/agentic-score-[date].html`

### What to inject into the DATA object:

Replace the DATA block in the template with the actual scanned values:

```javascript
const DATA = {
  character: '[krillin|yamcha|tien|piccolo|vegeta|goku|supersaiyan|ultrainstinct]',
  characterName: '[CHARACTER NAME IN CAPS]',
  tagline: '"[one-line character description]"',
  level: [X.X],
  agenticScore: [number],
  scores: [skills, hooks, multi, control, selfimproving, autonomy],  // each 0-10
  strongest: '[dimension name]',
  strongestSub: '[one sentence]',
  gap: '[dimension name]',
  gapSub: '[one sentence]',
  nextForm: '[next character name]',
  nextSub: '[what unlocks the next form]',
  moves: [
    { text: '[move 1]', cmd: '[terminal command or action]' },
    { text: '[move 2]', cmd: '[terminal command or action]' },
    { text: '[move 3]', cmd: '[terminal command or action]' }
  ]
};
```

The template file lives at:
`~/.claude/skills/agentic-score/report-template.html`

Copy it, inject the DATA block, save to reports/, open it.

**Injection method (Python — use string position, NOT regex):**
```python
with open(template_path, 'r') as f:
    template = f.read()
start = template.find('const DATA={')
end = template.find('};', start) + 2
result = template[:start] + new_data_block + template[end:]
with open(out_path, 'w') as f:
    f.write(result)
```
Do NOT use `re.sub` with the comment pattern — the `──` Unicode dashes may not match.


---

## Step 7 — Fun Finds

After outputting the terminal card, scan for 2-3 specific surprising observations. These are the most shareable part. Go specific — generic observations are not fun.

Look for:
- Commit timing patterns (midnight commits, weekend streaks, 6 consecutive days)
- Unusual ratios (e.g. 100 skills but 0 scheduled tasks — "you built a race car with no ignition")
- Hook count vs skill count gap
- failures/ directory age vs lesson count
- Number of projects vs number with PRODUCT_STATE.md
- Any skill with a particularly creative name
- The gap between strongest and weakest dimension

Format:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

THINGS WE NOTICED

  → [specific, funny, true observation with real numbers]
  → [specific, funny, true observation with real numbers]
  → [specific, funny, true observation with real numbers]
```

---

## Step 8 — Agentic ID

Generate a unique ID from the scan. Format: `[CHAR]-[SCORE]-[HASH]`

- CHAR: SSJ / UI / GKU / VGT / PIC / TEN / YMC / KRL
- SCORE: the Agentic Score number
- HASH: first 2 chars of (score × 31 + level × 100) in base36, uppercase

Example: `SSJ-3247-A4`

Print at bottom of terminal card:
```
  ID: SSJ-3247-A4   (proof-of-work — share this with your card)
```

---

## Step 9 — Community pitch

Print after the terminal card. Character-specific opening line, then the pitch.

Character lines:
- Krillin: "You ran this. That means the instinct is already there."
- Yamcha: "You are building. Come compare notes with people further along."
- Tien: "Your discipline is rare. The community will accelerate you."
- Piccolo: "Your system is real. You deserve a room of people who get it."
- Vegeta: "You operate independently. Imagine what you can do with peers at the same level."
- Goku: "You have balanced mastery. The community needs more people like you."
- Super Saiyan: "You built something most people do not believe is possible."
- Ultra Instinct: "You are at the frontier. Come help define what comes next."

Then print:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  "[CHARACTER LINE]"

  Adapt to AI is a WhatsApp-based global community where
  top founders and C-levels from high-growth companies
  share learnings and help each other on their agentic
  journey, using Claude Code (although we make exceptions
  for builders on Cursor/OpenClaw).

  Built by Giuseppe Belpiede (exited his last company to
  Revelo, +$50M raised) and Raffaello Starace (raised VC
  from SoftBank, +$100M ARR company in LATAM).

  The community includes founders who are currently in YC
  or alumni, that have raised from top VC funds
  (e.g. SoftBank, Tiger Global).

  IF level >= 8.0 (qualifies for private group):

    Your level qualifies you for the private group.
    Direct access to top builders. Everyone has a repo.
    No lurkers.

    ╔══════════════════════════════════════════════════╗
    ║  Private group → https://chat.whatsapp.com/     ║
    ║  JyJng7w0OxiK1MJN3jqdyh?mode=gi_t              ║
    ╚══════════════════════════════════════════════════╝

  IF level < 8.0 (open community):

    Join the open community — free content, learnings, and
    discussions. We share content from the private community
    to the public ones. Reach Level 8.0 to qualify.
    You are [X] levels away.

    ╔══════════════════════════════════════════════════╗
    ║  Open community → https://chat.whatsapp.com/    ║
    ║  Hwnh1V0YiOi5ORgDauhwyq                        ║
    ╚══════════════════════════════════════════════════╝

  Someone else should run this? Send them /agentic-score

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Step 10 — Save Agentic ID to disk

After generating the ID, save it to `~/.agentic-score-id`:

```bash
cat > ~/.agentic-score-id << EOF
AGENTIC ID: [ID]
Character:  [CHARACTER NAME]
Level:      [X.X] / 10.0
Score:      [SCORE]
Qualifies:  [Private group (Level 8+) | Open community]
Scanned:    [DATE]
Generated by /agentic-score — adaptto.ai
EOF
```

Tell the user:
```
  ID saved to ~/.agentic-score-id
  Share this as proof-of-work when joining the community.
  Run cat ~/.agentic-score-id to retrieve it anytime.
```

## Community link logic

- Level >= 8.0 → private group link: https://chat.whatsapp.com/JyJng7w0OxiK1MJN3jqdyh?mode=gi_t
- Level < 8.0  → open community link: https://chat.whatsapp.com/Hwnh1V0YiOi5ORgDauhwyq

Always show BOTH links — private group link only if they qualify, open link always.
For non-qualifiers: show the gap ("You are 1.2 levels from private access").
