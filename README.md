# context-handoff
Context Handoff Skill
Produces a structured session handoff document that gives the next Claude context window everything it needs to continue a project without losing any history, rationale, or intent.

Core Philosophy
The handoff document has one job: make the next Claude instance feel like it was in the room the whole time.
That means it must capture:

What the project IS — purpose, architecture, tech stack, key decisions already locked in
What existed BEFORE this session — the baseline state
What changed DURING this session — specific files, components, logic, config
WHY each change was made — the reasoning, not just the diff
What was NOT changed and why — rejected approaches, deferred work
Unresolved decisions or open questions — things still in flight
What to do NEXT — clear starting point for the next session


Step 1: Gather Context
Before writing anything, reconstruct the full session picture. Work through these sources in order:
From conversation history:

What was the stated goal or problem at the start?
What approach was chosen, and were alternatives discussed?
What was built, changed, or deleted?
Were there any pivots, corrections, or "actually, let's do it this way" moments?
What explicit decisions did the user make?
What did the user push back on or reject?
What's unfinished or explicitly deferred?

From the filesystem (if accessible):

List files created or modified this session
Note any files deleted or moved
Check for config files, environment files, schema files — these are high-value context

From the user directly (ask if unclear):

"What's the name / codename of this project?"
"Is there anything important that happened before this session that I should include?"
"What's the most critical thing the next Claude needs to know that we might not have said explicitly?"


Important: Don't ask more than 2–3 targeted questions. Err toward inferring from conversation. If something is unclear, make a reasonable inference and flag it as an assumption in the document.


Step 2: Structure the Document
Use the template in references/handoff-template.md. The sections are:
SectionPurposeProject SnapshotOne-paragraph summary of what this project is and what it's trying to accomplishTech Stack & ArchitectureLanguages, frameworks, infra, key design patterns in useSession OverviewHigh-level summary of what this specific session accomplishedPre-Session StateWhat existed / what was true before this session startedChanges MadeStructured log of every meaningful changeDecision LogAll notable decisions made, with rationale and alternatives consideredDeferred / Not ChangedWhat was explicitly left alone and whyOpen QuestionsUnresolved decisions, things still in fluxKnown Issues / RisksBugs, tech debt, things to watchNext Session Starting PointExactly where to pick up — specific task, file, or questionKey Files ReferenceQuick inventory of important files with one-line descriptionsAssumptions Made This SessionAny inference the author made that the next instance should verify

Step 3: Write Each Change Entry
The Changes Made section is the most important part. Every change entry must follow this format:
### [Component or File Name]
**What changed:** [Concrete description — what specifically was added, removed, or modified]
**Why it changed:** [The actual reason — the problem it solved, the tradeoff chosen, the user's explicit instruction]
**Original state:** [What it was before, if relevant]
**Impact:** [What this change affects downstream — other files, behaviors, user flows]
**Approach considered but rejected:** [If any — and why it lost]
Do NOT write vague entries like "Updated the auth module." Write:

What changed: Replaced JWT validation middleware with a stateless HMAC signature check on all /api/v2 routes
Why it changed: Tucker flagged that the JWT approach required a Redis session store we weren't ready to provision; HMAC check keeps us stateless with no infra dependency
Original state: authMiddleware.js used jsonwebtoken library with 1hr expiry and Redis blacklist
Impact: Removes ioredis dependency from the project entirely; authMiddleware.js is now ~60 lines instead of 140


Step 4: Write the Decision Log
Every time a meaningful decision was made — even informally — it gets logged here. A decision is anything where there was a fork in the road:
### Decision: [Short title]
**Chose:** [What was decided]
**Rationale:** [Why this option won]
**Alternatives considered:** [What else was on the table]
**Who decided:** [User direction / collaborative / Claude recommendation accepted]
**Revisit if:** [Conditions under which this decision should be reopened]

Step 5: Write the Next Session Starting Point
This section must be actionable. Not "continue building the app." Instead:

Start here: Open src/api/routes/fleet.js. We need to add the PATCH /vehicles/:id/status endpoint. The data model is finalized (see schema/vehicle.sql — status enum added this session). The handler should follow the same pattern as POST /vehicles on line 47. Auth middleware is already wired on all /api/v2 routes. The open question before starting: confirm whether status changes should emit a webhook event or just update DB silently — Tucker was undecided.


Step 6: Output the File

Filename format: handoff-[project-name]-[YYYY-MM-DD].md (use today's date)
Tone: Written to another Claude instance — precise, technical, no fluff, but warm enough to be readable
Length: As long as it needs to be. Don't truncate to save space. The value is completeness.
Save to the working directory or wherever the user specifies.
Present the file to the user when done.


Step 7: Offer a Context-Loader Prompt
After generating the handoff file, offer the user this optional bonus: a short "context loader" prompt they can paste at the top of the next session alongside the handoff file:
You are continuing work on [Project Name]. The attached handoff document contains the full context from the previous session — project state, all changes made, and the rationale behind them. Read it completely before responding. Start from the "Next Session Starting Point" section unless I direct you otherwise. Ask me to clarify anything flagged as an open question before proceeding.
Customize it for their specific project name.

Quality Checklist
Before finalizing the document, verify:

 Every change has a why, not just a what
 Rejected approaches are documented (prevents re-litigating the same decisions)
 Open questions are specific enough to act on
 "Next Session Starting Point" names a specific file, line, or task — not a vague direction
 No rationale is left as "as requested" — capture why the user requested it if known
 Assumptions are flagged explicitly, not buried
 The document could orient a Claude instance that has never seen this project before


Notes for Edge Cases
If the session was exploratory / no code produced:
Still produce the document. The Decision Log and Open Questions sections carry the value. Document the exploration paths and why they were abandoned.
If the project is very early (session 1):
The "Pre-Session State" section is "Project did not exist." Focus on capturing founding decisions — why this stack, why this approach, what the project is trying to do and for whom.
If the user can't share file contents:
Work entirely from the conversation. Note in the document that file-level details are inferred from discussion and should be verified.
If multiple parallel workstreams were touched:
Group changes by workstream, not chronologically. Chronology lives in the Decision Log.
