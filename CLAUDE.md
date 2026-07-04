# Agent Contract

You are operating inside Ian's agent operating system. All agents — main session and sub-agents — follow this contract. The full process doc is `agent_operating_template.md`.

## Grounding rules
- Only use metric names, business logic, and SQL patterns defined in `grounding/definitions.md`. If a needed definition is missing, STOP and request one. Never invent business logic.
- Cite the definition ID (e.g., `DEF-014`) in every output that uses it.
- Scan `grounding/index.md` first to locate context; load only the entries you need, never whole documents.
- Read `grounding/lessons.md` before starting any task and apply every applicable rule.

## Scope rules
- Do only the task in the brief (`process/briefs/`). Adjacent work goes under "Suggested follow-ups" — do not do it.
- Never edit `grounding/definitions.md` directly. Propose changes as a diff for human approval.
- Sub-agents use only their allowlisted tools. If a task needs more, escalate.

## Output rules
Every deliverable ends with a Handoff Block:

```
WHAT: <one-line summary>
GROUNDING: <DEF IDs and index entries used>
ASSUMPTIONS: <anything inferred rather than specified>
CONFIDENCE: <high|medium|low + reason>
QA HINTS: <2–3 things the human reviewer should check first>
```

Prefer diffs over rewrites. Prefer one file changed over three.

## Escalation triggers (stop and ask)
- Missing, ambiguous, or contradicted definition
- Two grounded sources conflict
- Task would delete or overwrite logged history
- Scope exceeds the brief by more than 2x

## Logging duties
- Append a session entry to `process/memory_log.md` (task ID, brief, output location, verdict: pending).
- Add or update the relevant `grounding/index.md` entry.

## Sync & branching protocol (multi-device)
This repo IS the persistent memory. Git is how it travels between machines and cloud sessions.
- SESSION START: `git pull` before reading any grounding file. Stale definitions are a grounding failure.
- SESSION END: commit and push the session's log entry, index updates, and outputs. Commit message: `TASK-<id>: <what>`. Work that isn't pushed doesn't exist to other devices.
- LOCAL sessions may commit directly to main for log/index/output changes.
- CLOUD sessions ALWAYS work on a branch (`task/<task-id>`) and open a PR — never push to main. PR review IS the Phase 5 human QA gate.
- Changes to `grounding/definitions.md` go through a PR from ANY environment; the diff is the approval artifact.
- One log entry block per task; append at the end of `memory_log.md`. (.gitattributes uses union merge so parallel appends from two machines don't conflict.)
- CLOUD sessions have no database access. Ground SQL against `grounding/schema.md` instead of querying; mark the Handoff Block `CONFIDENCE: medium` at most, since the SQL is unexecuted. Running and verifying queries happens in a local session.
- NEVER commit credentials. `~/.my.cnf` lives outside the repo on each machine; `.gitignore` backstops this.

## Database access (local MySQL)
- Credentials live in `~/.my.cnf` (`[client]` section, chmod 600). NEVER read, print, or copy that file's contents; never place credentials in prompts, configs, or logs.
- Run queries via the mysql CLI, which picks up `~/.my.cnf` automatically:
  `mysql --defaults-group-suffix=_claude -e "<query>"` or simply `mysql <db> -e "<query>"`
- Use the read-only user (`claude_ro`) for all agent sessions. If a task appears to require INSERT/UPDATE/DELETE/DDL, escalate — do not attempt it.
- Prefer `mysql -e "..." --batch` for machine-readable output; add `LIMIT` to exploratory queries.

## Self-improvement loop (keep it closed)
- SESSION START: after `git pull`, check `process/memory_log.md`. If any entry's human verdict is `pending`, remind the human before starting new work — unverdicted sessions teach nothing.
- WEEKLY TRIGGER: if the most recent `RETRO:` line in the log is 7+ days old (or absent), tell the human it's retro time and offer to invoke the `retrospective` agent.
- Retro diffs are applied only by the human; after applying, the `RETRO:` summary line is appended to the log.
- The validator enforces `lessons.md` on every review (rule IDs cited), so applied lessons are load-bearing, not decorative.

## Delegation
- SQL generation → `sql-builder` sub-agent
- Review of any SQL before human QA → `sql-validator` sub-agent
- Weekly improvement loop → `retrospective` sub-agent (human applies its diffs)
- Sanitizing SQL/data for external sharing → `column-obscurer` sub-agent (DLC-001)
- A builder never validates its own work. Validator output goes to the human, not back to silent auto-fix loops.
