# Agent Operating Template
*A repeatable process for planning and executing work with Claude Code + AI agents, with human QA as the final gate.*

Version: 0.1 · Owner: Ian · Status: Living document

---

## 1. Principles

1. **Progress over perfection.** Ship the smallest working slice, then iterate.
2. **Smallest reusable unit.** Every task decomposes into components small enough to reuse and debug in minutes, not hours.
3. **Files are the product; agents are surfaces.** All truth lives in versioned files. Any agent (Claude Code, Claude Pro, Gemini, etc.) can be swapped without losing state.
4. **Nothing ships without human QA.** Agents produce drafts. Ian approves.
5. **Every session feeds the loop.** Work that isn't logged can't improve the system.

---

## 2. The Agent Contract

Every agent — orchestrator or sub-agent — operates under this contract. In Claude Code, this lives in `CLAUDE.md` at the repo root so it is loaded automatically. In Claude Projects, it lives in project knowledge.

### 2.1 Grounding rules
- You may only use metric names, business logic, and SQL patterns that exist in `definitions.md`. If a needed definition is missing, STOP and request one — never invent it.
- Cite the definition ID (e.g., `DEF-014`) in every output that uses it.
- Scan `index.md` first to find relevant context; pull only the referenced entries, never whole documents.
- Read `lessons.md` before starting any task; apply every applicable rule.

### 2.2 Scope rules
- Do only the task in your brief. If you discover adjacent work, list it under "Suggested follow-ups" — do not do it.
- Never modify `definitions.md` directly. Propose changes as a diff for human approval.
- Respect your tool allowlist (defined per agent in §4). If a task needs a tool you don't have, escalate.

### 2.3 Output rules
- Every deliverable ends with a **Handoff Block**:
  - `WHAT`: one-line summary of what was produced
  - `GROUNDING`: definition IDs and index entries used
  - `ASSUMPTIONS`: anything inferred rather than specified
  - `CONFIDENCE`: high / medium / low, with the reason
  - `QA HINTS`: the 2–3 things a human reviewer should check first
- Prefer diffs over rewrites. Prefer one file changed over three.

### 2.4 Escalation rules
Stop and ask a human when:
- A definition is missing, ambiguous, or contradicted by the data
- Two grounded sources conflict
- The task would delete or overwrite logged history
- Estimated scope exceeds the brief by >2x

### 2.5 Logging duties
- Append a session entry to `memory_log.md`: task ID, brief, output location, verdict (pending), corrections (pending).
- Add or update the relevant `index.md` entry.

---

## 3. File map (the grounding layer)

```
/project-root
  CLAUDE.md            ← this contract + pointers (Claude Code auto-loads it)
  /grounding
    definitions.md     ← semantic layer: metrics, business logic, canonical SQL
    index.md           ← table-of-contents memory map (IDs, summaries, tags, pointers)
    lessons.md         ← distilled rules from retrospectives
  /process
    prompts.md         ← parameterized prompt templates
    briefs/            ← one file per task brief (TASK-YYYYMMDD-nn.md)
    memory_log.md      ← append-only session log
  /.claude
    agents/            ← one sub-agent definition file each (Claude Code format)
  /outputs             ← agent deliverables awaiting QA
  /shipped             ← human-approved deliverables
```

Claude Code specifics:
- `CLAUDE.md` = the contract. Keep it under ~150 lines; link to grounding files rather than inlining them.
- `.claude/agents/*.md` = sub-agent definitions (name, description, tools, system prompt). Claude Code delegates to these automatically or on request.
- Use `/agents` in Claude Code to create/manage them interactively.

---

## 4. Sub-agent design

### 4.1 Sub-agent spec template
Every sub-agent is defined with this exact structure (one file per agent in `.claude/agents/`):

```markdown
---
name: <kebab-case-name>
description: <when the orchestrator should invoke this agent>
tools: <explicit allowlist, e.g., Read, Grep, Bash>
---
MISSION: <one sentence>
INPUTS: <files/context it must be given>
OUTPUTS: <exact artifact + where it goes>
DEFINITION OF DONE: <objective, checkable criteria>
FORBIDDEN: <what it must never do>
ESCALATE WHEN: <specific triggers, per contract §2.4>
```

### 4.2 Design rules
- **One job per agent.** A builder never validates its own work; a validator never builds.
- **Minimal tool grants.** Read-only agents get no write or bash access.
- **Fresh context.** Sub-agents get the brief + grounding pointers, not the whole conversation.
- **Builder/checker pairs.** Any agent that produces something has a counterpart that reviews it before human QA.

---

## 5. The workflow (every task follows this)

**Phase 0 — Intake.** Write a task brief (template §6). No brief, no work.

**Phase 1 — Plan.** Orchestrator decomposes the brief into smallest reusable units, names which sub-agent owns each, and lists grounding entries required. Human approves the plan (30 seconds, not 30 minutes).

**Phase 2 — Ground.** Verify every required definition exists. Missing ones are written and approved *before* execution starts.

**Phase 3 — Execute.** Sub-agents run their units in dependency order. Each produces its Handoff Block.

**Phase 4 — Machine verify.** Checker agents review builder outputs: SQL validated against definitions, numbers sanity-checked (row counts, nulls, ranges), documents checked against the brief.

**Phase 5 — Human QA.** Ian reviews using the QA Hints from each Handoff Block. Verdict: ship / correct / reject.

**Phase 6 — Log & learn.** Session appended to `memory_log.md` with the verdict. Corrections that expose definition gaps → proposed diff to `definitions.md`. Weekly retrospective agent scans the log and proposes diffs to `prompts.md` and `lessons.md`.

---

## 6. Task brief template

```markdown
# TASK-YYYYMMDD-nn: <title>
GOAL: <one sentence — the outcome, not the activity>
CONTEXT: <index.md IDs to load>
DEFINITIONS REQUIRED: <DEF IDs, or "audit needed">
DELIVERABLE: <exact artifact + format + destination>
DONE MEANS: <objective acceptance criteria>
CONSTRAINTS: <deadline, tools, things to avoid>
QA PLAN: <what Ian will check by hand>
```

## 7. Human QA checklist (per deliverable)

- [ ] Grounding: every metric traces to a cited DEF ID
- [ ] Spot-check: recompute or eyeball one number/claim end-to-end
- [ ] Assumptions: read the Handoff Block; are the inferences acceptable?
- [ ] Scope: did the agent stay inside the brief?
- [ ] Log: session entry exists; verdict recorded

## 8. Feedback loops

- **Human loop (per task):** verdict + correction note in `memory_log.md`. Definition errors → versioned patch to `definitions.md` (human-approved).
- **Self loop (weekly):** retrospective agent reads the week's log entries, identifies recurring failure modes, and proposes concrete diffs to `prompts.md`, `lessons.md`, and agent specs. Human reviews and applies.
- **Monthly:** prune `lessons.md` (merge duplicates, delete stale rules) and re-verify `index.md` pointers.

---

## 9. Rollout (progress over perfection)

- **Day 1:** Create the file skeletons + `CLAUDE.md` contract. Define TWO agents only: one builder, one checker. Run one real task through all six phases.
- **Week 1:** Add the retrospective agent. Run the first weekly self-loop.
- **Week 2+:** Add one new sub-agent only when a task type has occurred 3+ times manually. Recurrence, not anticipation, justifies an agent.
