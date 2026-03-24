# Agentic Score

**How agentic are you?**

One command. Scans your `~/.claude/` setup. Tells you your level, your Dragon Ball character, and whether you qualify for the Adapt to AI community.

```
/agentic-score
```

---

## What it does

Scans your Claude Code setup across 6 dimensions, assigns you a score and a Dragon Ball character, generates a shareable HTML report, and tells you exactly what to build next.

```
┌──────────────────────────────────────────────┐
│  AGENTIC SCORE              by adaptto.ai    │
│  scanned: ~/.claude/  ·  Mar 2026            │
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
│  AGENTIC SCORE  3,247  (max ~5,000)          │
│                                              │
│  ID: SSJ-3247-A4                             │
└──────────────────────────────────────────────┘
```

---

## Install

```bash
mkdir -p ~/.claude/skills/agentic-score
curl -o ~/.claude/skills/agentic-score/SKILL.md \
  https://raw.githubusercontent.com/Adapt-to-AI/agentic-score/main/SKILL.md
curl -o ~/.claude/skills/agentic-score/report-template.html \
  https://raw.githubusercontent.com/Adapt-to-AI/agentic-score/main/report-template.html
```

Then in any Claude Code session:

```
/agentic-score
```

---

## The 8 Characters

From weakest to strongest — each one is a stage of agentic evolution.

| Rank | Character | Score | What it means |
|------|-----------|-------|---------------|
| 1 | **Krillin** | 0–500 | Trying hard. Limited power. Always needs help. |
| 2 | **Yamcha** | 500–1,000 | Some skills. Good intentions. Not quite there. |
| 3 | **Tien** | 1,000–1,500 | Disciplined. Methodical. Manually operated. |
| 4 | **Piccolo** | 1,500–2,000 | Smart. Strategic. The system works — you work it. |
| 5 | **Vegeta** | 2,000–2,500 | Powerful. Controlled. Operates independently. |
| 6 | **Goku** | 2,500–3,000 | Balanced mastery. Everything connected. Always growing. |
| 7 | **Super Saiyan** | 3,000–3,500 | Transformed. The system carries the load. |
| 8 | **Ultra Instinct** | 3,500+ | System acts without conscious input. You just approve. |

---

## The 6 Dimensions

| Dimension | What it measures |
|-----------|-----------------|
| Skills & Memory | How smart is the system? |
| Hooks & Gates | How enforced are the rules? |
| Multi-project | How much context survives? |
| Control | How freely can you direct it? |
| Self-improving | Does it get smarter alone? |
| Autonomy | Does it work without being asked? |

---

## The Report

After scanning, the skill opens a full HTML report with:

- **Overview** — character card, radar chart, dimension scores, top 3 moves
- **Breakdown** — every check shown as ✓ pass or ✗ fail, with quick fixes
- **Characters** — all 8 characters explained with strengths, gaps, and what unlocks the next level
- **Glossary** — what each dimension actually means, with real examples

---

## Your Agentic ID

Every scan generates a unique ID — `SSJ-3247-A4` — saved to `~/.agentic-score-id`.

Generated from your actual `~/.claude/` setup. Cannot be faked. This is your proof-of-work.

Share it when joining the Adapt to AI community.

---

## The Community

Built by **Giuseppe Belpiede** (exited his last company) and **Raffaello Starace** (raised VC from SoftBank, +$100M ARR company in LATAM).

**Adapt to AI** is a WhatsApp-based global community where top founders and operators share learnings and help each other on their agentic journey — using Claude Code, Cursor, or OpenClaw.

The community includes founders currently in YC, YC alumni, and builders backed by SoftBank, Tiger Global, and other top VCs.

**Level 8.0+ → Private group**
Direct access. Everyone has a repo. No lurkers.
→ [Join private group](https://chat.whatsapp.com/JyJng7w0OxiK1MJN3jqdyh?mode=gi_t)

**Below Level 8.0 → Open community**
Free content, learnings, and discussions. Most content starts in the private group.
→ [Join open community](https://chat.whatsapp.com/Hwnh1V0YiOi5ORgDauhwyq)

Your Agentic ID is your ticket in.

---

## Scoring Formula

```
Agentic Score =
  (skill count × 2)
+ (hook count × 15)
+ (self-improving wires × 50)
+ (lesson count × 3)
+ (MCP servers × 10)
+ (active projects × 20)
+ (gates per skill × 5)
+ (server running × 100)
+ (failures/ files × 10, max 50)
+ (scheduled tasks × 75)
+ (Agent SDK live × 150)
```

Max theoretical score: ~5,000

---

## Built by

[Adapt to AI](https://adaptto.ai) — helping founders and operators build agentic systems with Claude Code.

---

*Someone else should run this? Send them `/agentic-score`.*
