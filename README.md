# context-handoff

**Skill for Claude Code sessions**  
Generates a structured handoff document at the end of a session so the next Claude context window can pick up exactly where you left off — with full history, rationale, and a clear starting point.

---

## What It Does

When a session ends (or a context window is running long), `context-handoff` writes a `.md` file that captures everything a fresh Claude instance needs to continue the work without losing anything. Not just *what* changed — but *why*, *what was rejected*, and *where to start next*.

---

## When It Triggers

The skill activates when you say things like:

- `"make a handoff"`
- `"create a context file"`
- `"save context"`
- `"write a handoff doc"`
- `"I'll continue this tomorrow"`
- `"summarize what we did for next session"`
- `"let's pick this up later"`
- `"pass context forward"`

It also triggers **proactively** when the session appears to be wrapping up, or when context window utilization is approaching a threshold that would risk losing history.

---

## Output

A single `.md` file named:

```
handoff-[project-name]-[YYYY-MM-DD].md
```

### Sections in the handoff document:

| Section | What It Captures |
|---|---|
| **Project Snapshot** | One-paragraph summary of what the project is and what it's trying to accomplish |
| **Tech Stack & Architecture** | Languages, frameworks, infra, key design patterns |
| **Session Overview** | High-level summary of what this specific session accomplished |
| **Pre-Session State** | What existed / what was true before this session started |
| **Changes Made** | Structured log of every meaningful change — with what, why, original state, and downstream impact |
| **Decision Log** | All notable decisions made, with rationale and alternatives considered |
| **Deferred / Not Changed** | What was intentionally left alone and why |
| **Open Questions** | Unresolved decisions and things still in flux |
| **Known Issues / Risks** | Bugs, tech debt, things to watch |
| **Next Session Starting Point** | Specific file, task, or question — not a vague direction |
| **Key Files Reference** | Quick inventory of important files with one-line descriptions |
| **Assumptions Made This Session** | Inferences the author made that the next instance should verify |

---

## The Change Entry Format

Every change is logged in this structure — the *why* is mandatory:

```
### [Component or File Name]
**What changed:** Concrete description of what was added, removed, or modified
**Why it changed:** The actual reason — problem solved, tradeoff chosen, user direction
**Original state:** What it was before (if relevant)
**Impact:** What this change affects downstream
**Approach considered but rejected:** (If any, and why it lost)
```

---

## Bonus: Context Loader Prompt

After generating the handoff file, the skill offers an optional **context loader prompt** — a short block you paste at the top of your next session alongside the `.md` file:

```
You are continuing work on [Project Name]. The attached handoff document contains the full context 
from the previous session — project state, all changes made, and the rationale behind them. Read 
it completely before responding. Start from the "Next Session Starting Point" section unless I 
direct you otherwise. Ask me to clarify anything flagged as an open question before proceeding.
```

---

## Design Philosophy

The handoff document has one job: **make the next Claude instance feel like it was in the room the whole time.**

That means:
- Every change has a **why**, not just a **what**
- Rejected approaches are documented (prevents re-litigating the same decisions)
- Open questions are specific enough to act on
- "Next Session Starting Point" names a specific file, line, or task
- No rationale is left as "as requested" — capture *why* the user requested it, if known
- Assumptions are flagged explicitly, not buried
- The document could orient a Claude instance that has never seen the project before

---

## Edge Case Behavior

**Exploratory session / no code produced:**  
Still produces the document. The Decision Log and Open Questions sections carry the value.

**Session 1 / brand new project:**  
"Pre-Session State" = "Project did not exist." Focus captures founding decisions — why this stack, why this approach, what the project is trying to do and for whom.

**No file system access:**  
Works entirely from conversation history. Notes in the document that file-level details are inferred and should be verified.

**Multiple parallel workstreams:**  
Groups changes by workstream, not chronologically. Chronology lives in the Decision Log.

---

## Location

```
/mnt/skills/user/context-handoff/SKILL.md
```

---

## Credits

Created by **Tucker Hemphill**  
[www.tuckerhemphill.com](https://www.tuckerhemphill.com)
