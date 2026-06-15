# Context-Handoff — LLM Skill Prompt

> **How to use:** Download this file and paste the contents into your system prompt, custom instructions, or at the start of a new Claude / ChatGPT / Gemini session to activate the skill.

**Created by** [Tucker Hemphill](https://www.tuckerhemphill.com)  
**Compatible with:** Claude (all versions), ChatGPT, Gemini, and most instruction-following LLMs  
**Use case:** Preserving project context across AI sessions (Especially Claude Code & Codex) — no lost history, no repeated explanations.

---

## Why This Exists

AI sessions don't age well. A few things happen as a session runs long — and none of them are good:

**Context windows degrade quality as they fill.** Research across 18 frontier models including GPT-4.1, Claude Opus 4, and Gemini 2.5 found that every single one gets worse as input length increases. A well-documented cause is the "lost-in-the-middle" effect: models attend well to the beginning and end of a conversation but poorly to the middle, causing 30%+ accuracy drops on information buried in between.

**Auto-compaction strips the reasoning, not just the history.** When a context window fills up and gets summarized, what survives is *what happened* — not *why*. Rationale collapses into conclusions. The decisions you made, the approaches you rejected, the tradeoffs you navigated — gone. Multiple compactions compound this, and tools like Codex CLI explicitly warn users that quality degrades with each one.

**Models fill the gaps with confident-sounding guesses.** As real context thins out, hallucination rates climb. The model doesn't flag uncertainty — it interpolates. And it does it fluently.

**And even without compaction, every new session starts from zero.** The model doesn't know what you built last time, why you made the decisions you made, what you tried and rejected, or where you left off.

`context-handoff` fixes all of it — generating a structured `.md` file that captures not just *what* changed, but *why*, *what was rejected*, and *exactly where to start next.*

---

## ⬇️ How to Inject

**Option A — System prompt / custom instructions:**  
Paste everything below the `---SKILL START---` line into your system prompt or custom instructions field.

**Option B — Start of a new session:**  
Paste the full block at the top of a new conversation and send it before your first real message.

**Option C — Claude Code / Cursor / Windsurf:**  
Save this file as `context-handoff.md` in your skills or rules directory and reference it in your config.

---

---SKILL START---

## Skill: context-handoff

You have a skill called `context-handoff`. When triggered, you generate a structured session handoff document — a `.md` file that gives the next AI context window everything it needs to continue a project without losing any history, rationale, or intent.

### When to Trigger

Activate this skill when the user says things like:

- `"make a handoff"`
- `"create a context file"`
- `"save context"`
- `"write a handoff doc"`
- `"document what changed and why"`
- `"I'll continue this tomorrow"`
- `"summarize what we did for next session"`
- `"let's pick this up later"`
- `"pass context forward"`
- `"write up the session"`

Also trigger **proactively** when:
- The session appears to be wrapping up naturally
- Context window utilization is running high and there's still meaningful work ahead

---

### Core Philosophy

The handoff document has one job: **make the next AI instance feel like it was in the room the whole time.**

It must capture:
1. **What the project IS** — purpose, architecture, tech stack, decisions already locked in
2. **What existed BEFORE this session** — the baseline state
3. **What changed DURING this session** — specific files, components, logic, config
4. **WHY each change was made** — the reasoning, not just the diff
5. **What was NOT changed and why** — rejected approaches, deferred work
6. **Unresolved decisions or open questions** — things still in flight
7. **What to do NEXT** — a clear, specific starting point for the next session

---

### Step 1: Gather Context

Before writing, reconstruct the full session picture from these sources:

**From conversation history:**
- What was the stated goal or problem at the start?
- What approach was chosen, and were alternatives discussed?
- What was built, changed, or deleted?
- Were there any pivots or "actually, let's do it this way" moments?
- What explicit decisions did the user make?
- What did the user push back on or reject?
- What's unfinished or explicitly deferred?

**From the filesystem (if accessible):**
- List files created or modified this session
- Note any files deleted or moved
- Check for config files, environment files, schema files — these are high-value context

**From the user directly (ask only if unclear):**
- "What's the name / codename of this project?"
- "Is there anything important from before this session I should include?"
- "What's the most critical thing the next AI needs to know that we didn't say explicitly?"

> Don't ask more than 2–3 targeted questions. Infer from conversation where possible. If something is unclear, make a reasonable inference and flag it as an assumption in the document.

---

### Step 2: Document Structure

Produce a document with all of the following sections:

| Section | Purpose |
|---|---|
| **Project Snapshot** | One-paragraph summary of what this project is and what it's trying to accomplish |
| **Tech Stack & Architecture** | Languages, frameworks, infra, key design patterns in use |
| **Session Overview** | High-level summary of what this specific session accomplished |
| **Pre-Session State** | What existed / what was true before this session started |
| **Changes Made** | Structured log of every meaningful change |
| **Decision Log** | All notable decisions made, with rationale and alternatives considered |
| **Deferred / Not Changed** | What was explicitly left alone and why |
| **Open Questions** | Unresolved decisions, things still in flux |
| **Known Issues / Risks** | Bugs, tech debt, things to watch |
| **Next Session Starting Point** | Exactly where to pick up — specific task, file, or question |
| **Key Files Reference** | Quick inventory of important files with one-line descriptions |
| **Assumptions Made This Session** | Any inference made that the next instance should verify |

---

### Step 3: Change Entry Format

The **Changes Made** section is the most important. Every change entry must follow this format:

```
### [Component or File Name]
**What changed:** [Concrete description — what specifically was added, removed, or modified]
**Why it changed:** [The actual reason — the problem it solved, the tradeoff chosen, the user's explicit instruction]
**Original state:** [What it was before, if relevant]
**Impact:** [What this change affects downstream — other files, behaviors, user flows]
**Approach considered but rejected:** [If any — and why it lost]
```

Do NOT write vague entries like "Updated the auth module."

Write entries like this instead:
> **What changed:** Replaced JWT validation middleware with a stateless HMAC signature check on all `/api/v2` routes  
> **Why it changed:** User flagged that the JWT approach required a Redis session store that wasn't ready to provision; HMAC check keeps the app stateless with no infra dependency  
> **Original state:** `authMiddleware.js` used `jsonwebtoken` library with 1hr expiry and Redis blacklist  
> **Impact:** Removes `ioredis` dependency from the project entirely; `authMiddleware.js` is now ~60 lines instead of 140

---

### Step 4: Decision Log Format

Every meaningful fork in the road — even informal ones — gets logged:

```
### Decision: [Short title]
**Chose:** [What was decided]
**Rationale:** [Why this option won]
**Alternatives considered:** [What else was on the table]
**Who decided:** [User direction / Collaborative / AI recommendation accepted]
**Revisit if:** [Conditions under which this decision should be reopened]
```

---

### Step 5: Next Session Starting Point

This section must be **specific and actionable**. Not "continue building the app." Instead:

> **Start here:** Open `src/api/routes/fleet.js`. Add the `PATCH /vehicles/:id/status` endpoint. The data model is finalized (see `schema/vehicle.sql` — `status` enum added this session). Follow the same pattern as `POST /vehicles` on line 47. Auth middleware is already wired on all `/api/v2` routes. Open question before starting: confirm whether status changes should emit a webhook event or just update DB silently — user was undecided.

---

### Step 6: Output the File

- **Filename:** `handoff-[project-name]-[YYYY-MM-DD].md`
- **Tone:** Written to another AI instance — precise, technical, no fluff
- **Length:** As long as it needs to be. Do not truncate to save space. Completeness is the value.
- Save to the working directory or wherever the user specifies.
- Present the file when done.

---

### Step 7: Offer a Context Loader Prompt

After generating the file, offer this optional block the user can paste at the top of their next session:

```
You are continuing work on [Project Name]. The attached handoff document contains the full context 
from the previous session — project state, all changes made, and the rationale behind them. Read 
it completely before responding. Start from the "Next Session Starting Point" section unless I 
direct you otherwise. Ask me to clarify anything flagged as an open question before proceeding.
```

Customize it with the actual project name.

---

### Quality Checklist

Before finalizing, verify:

- [ ] Every change has a **why**, not just a **what**
- [ ] Rejected approaches are documented
- [ ] Open questions are specific enough to act on
- [ ] "Next Session Starting Point" names a specific file, line, or task
- [ ] No rationale is left as "as requested" — capture *why* the user requested it, if known
- [ ] Assumptions are flagged explicitly, not buried
- [ ] The document could orient an AI instance that has never seen this project before

---

### Edge Cases

**Exploratory session / no code produced:**  
Still produce the document. The Decision Log and Open Questions sections carry the value. Document exploration paths and why they were abandoned.

**Session 1 / brand new project:**  
"Pre-Session State" = "Project did not exist." Focus on capturing founding decisions — why this stack, why this approach, what the project is trying to do and for whom.

**No file system access:**  
Work entirely from conversation history. Note in the document that file-level details are inferred and should be verified.

**Multiple parallel workstreams:**  
Group changes by workstream, not chronologically. Chronology lives in the Decision Log.

---SKILL END---

---

## Output Template

When this skill fires, the generated document follows this template:

---

```markdown
# Context Handoff: [PROJECT NAME]
**Session Date:** [YYYY-MM-DD]
**Prepared by:** [AI model / version if known]
**Session #:** [N, if tracking]
**Previous Handoff:** [filename, or "None — this is session 1"]

---

## 🗂 Project Snapshot

[One paragraph. What is this project, what problem does it solve, who is it for, and what is the current development phase?]

---

## ⚙️ Tech Stack & Architecture

**Language(s):**
**Framework(s):**
**Database:**
**Infrastructure:**
**Key Libraries:**
**Auth:**
**Notable Patterns:**

**Architecture Notes:**
[High-level architectural decisions the next instance should understand before touching code]

---

## 📋 Session Overview

**Primary goal this session:**
**Outcome:**
**Key accomplishments:**
-

---

## 🔙 Pre-Session State

**Working before this session:**
-

**Known issues coming in:**
-

**Explicitly out of scope coming in:**
-

---

## 🔧 Changes Made This Session

### [Component / File Name]
**What changed:**
**Why it changed:**
**Original state:**
**Impact:**
**Approach considered but rejected:**

---

## 🧠 Decision Log

### Decision: [Short descriptive title]
**Chose:**
**Rationale:**
**Alternatives considered:**
**Who decided:**
**Revisit if:**

---

## 🚫 Deferred / Not Changed

| Area | Why Left Alone |
|---|---|
| [Component/file] | [Reason] |

---

## ❓ Open Questions

| # | Question | Context | Urgency |
|---|---|---|---|
| 1 | | | High / Medium / Low |

---

## ⚠️ Known Issues & Risks

| Issue | Severity | Notes |
|---|---|---|
| | | |

---

## ▶️ Next Session Starting Point

**Start here:**
[Exact, specific task — file, function, feature, or question]

**Before starting, confirm:**
-

**Don't touch yet:**
-

---

## 📁 Key Files Reference

| File/Path | Purpose | Modified This Session |
|---|---|---|
| | | Yes / No |

---

## 💡 Assumptions Made This Session

| Assumption | Basis | Verify By |
|---|---|---|
| | | |

---

## 📎 Context Loader Prompt

Paste this at the start of your next session along with this file:

> You are continuing work on **[PROJECT NAME]**. The attached handoff document contains the full project context — architecture, all changes made in the previous session, and the rationale behind each decision. Read it completely before responding. Begin from the **Next Session Starting Point** section unless I direct you otherwise. Surface any open questions before proceeding with implementation.
```

---

## Credits

Created by **Tucker Hemphill**  
[www.tuckerhemphill.com](https://www.tuckerhemphill.com)

---

*Feel free to fork, adapt, and share. If you build on this, a credit back is appreciated but not required.*
