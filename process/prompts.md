# Prompt Templates

## P-001: Kick off a task (paste into Claude Code)
> Read CLAUDE.md, then process/briefs/TASK-<id>.md. Plan the smallest reusable units, name which sub-agent owns each, list required DEF IDs, and wait for my approval before executing.

## P-002: Weekly retrospective
> Read the last 7 days of process/memory_log.md. Identify recurring failure modes and root causes. Propose specific diffs to grounding/lessons.md, process/prompts.md, or the agent specs in .claude/agents/. Output diffs only — do not apply them.

## P-003: New definition draft
> Draft a definitions.md entry for <metric> using the DEF template. Mark every field you inferred rather than confirmed. Output as a diff for my approval.
