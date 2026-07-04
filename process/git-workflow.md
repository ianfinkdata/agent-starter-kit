# Multi-Device Workflow

## The model
The repo is the memory. Every machine (and every cloud session) is a
disposable surface over the same files — the same principle as the original
five-file design, now with git as the sync layer.

    Local machine A ──push──▶ GitHub ◀──PR── Claude Code (cloud)
                               │
    Local machine B ◀──pull────┘

## One-time setup per machine
1. Clone the repo.
2. Create ~/.my.cnf per process/mysql-setup.md (local machines with DB only).
3. Done — CLAUDE.md, agents, and grounding all travel with the clone.

## Per-session ritual (enforced by the contract)
- Start: git pull
- End:   git add -A && git commit -m "TASK-<id>: <what>" && git push

## Why conflicts won't bite
- memory_log.md and lessons.md use git's union merge (.gitattributes), so
  two machines appending in parallel merge cleanly instead of conflicting.
- definitions.md changes go through PRs from anywhere, so the registry can
  never diverge silently.

## Cloud sessions (Claude Code on the web / GitHub)
- Always branch: task/<task-id>. Never push to main.
- The PR is the QA gate: validator findings and Handoff Blocks go in the PR
  description; your review + merge is the human verdict.
- No DB access in the cloud: SQL is grounded on grounding/schema.md and
  marked unexecuted; you run it in a local session before shipping.
- Refresh schema.md locally whenever the database schema changes.
