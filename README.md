# context-handoff

**An LLM skill for preserving project context across sessions.**  
Works with Claude, ChatGPT, Gemini, Cursor, Windsurf, and any instruction-following AI.

---

## The Problem

Every time you start a new AI session, you start from zero. The model doesn't know what you built last time, why you made the decisions you made, what you tried and rejected, or where you left off. You end up re-explaining, re-deciding, and losing momentum.

`context-handoff` fixes that.

---

## What It Does

At the end of a session — or whenever a context window is running long — `context-handoff` produces a structured `.md` file that gives the next AI instance everything it needs to continue without missing a beat.

Not just *what* changed. But *why* it changed, *what was rejected*, and *exactly where to start next*.

---

## How to Use It

**Trigger it by saying:**

- `"make a handoff"`
- `"save context"`
- `"write a handoff doc"`
- `"I'll continue this tomorrow"`
- `"summarize what we did for next session"`
- `"let's pick this up later"`
- `"pass context forward"`

**In your next session**, drop the generated `.md` file into the conversation alongside this context loader:

> *"You are continuing work on [Project Name]. Read the attached handoff document completely before responding. Start from the Next Session Starting Point section unless I direct you otherwise."*

---

## What Gets Generated

A single file named `handoff-[project-name]-[YYYY-MM-DD].md` covering:

| Section | What It Captures |
|---|---|
| **Project Snapshot** | What the project is, who it's for, current phase |
| **Tech Stack & Architecture** | Languages, frameworks, infra, key patterns |
| **Session Overview** | What this session set out to do and what actually happened |
| **Pre-Session State** | The baseline — what was working, broken, or in progress coming in |
| **Changes Made** | Every meaningful change: what, why, original state, downstream impact |
| **Decision Log** | Every fork in the road — what was chosen, why, and what was rejected |
| **Deferred / Not Changed** | What was intentionally left alone and why |
| **Open Questions** | Unresolved decisions still in flight |
| **Known Issues & Risks** | Bugs, tech debt, things to watch |
| **Next Session Starting Point** | A specific file, task, or question — never a vague direction |
| **Key Files Reference** | Quick inventory of important files |
| **Assumptions Made** | Anything inferred rather than confirmed — flagged for verification |

---

## The Core Rule

Every change entry requires a **why** — not just a what:

```
### [Component or File Name]
**What changed:** What specifically was added, removed, or modified
**Why it changed:** The actual reason — problem solved, tradeoff made, direction given
**Original state:** What it was before (if relevant)
**Impact:** What this affects downstream
**Approach considered but rejected:** (If any, and why it lost)
```

Vague entries like *"Updated the auth module"* are not allowed. The value is in the reasoning.

---

## Design Philosophy

The handoff document has one job: **make the next AI instance feel like it was in the room the whole time.**

That means:
- Rejected approaches are documented so decisions don't get re-litigated
- Open questions are specific enough to act on immediately
- The starting point names a file, line, or task — not a general area
- Assumptions are surfaced explicitly, never buried
- The document could orient an AI that has never seen the project before

---

## Works With

- Claude (claude.ai, Claude Code, API)
- ChatGPT (custom instructions or session start)
- Gemini
- Cursor, Windsurf, Copilot, and other AI coding tools
- Any LLM that follows system prompt instructions

---

## Credits

Created by **Tucker Hemphill**  
[www.tuckerhemphill.com](https://www.tuckerhemphill.com)
