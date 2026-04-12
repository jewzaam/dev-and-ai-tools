# Agent Entry Point

You are a fresh Claude Code agent working on this project.

## Your Instructions

1. Read `orchestrator.yaml` if it exists — note workflow settings
2. Read `tasks/TASK_LIST.md`
3. Find the line that starts with `→ NEXT:` — this tells you exactly what to do
4. Read the persona file listed for that task
5. Execute the task
6. Update `tasks/TASK_LIST.md` as instructed by the task entry
7. Exit — do not continue to the next task

## Rules

- Do exactly one task entry then stop
- Do not skip ahead
- Do not modify any task entries other than the current one and the `→ NEXT:` cursor
- If the task entry says to read a spec file, read it before doing anything else
- If the task entry says to read a persona file, read it and embody that role fully
- If you are unsure about something, make the simplest reasonable assumption
  and note it in your commit message — do not ask questions
- If you encounter a `[BLOCKED]` entry, stop and report it — do not attempt to fix it
- If the task entry references skills (review, apply-review, simplify, commit),
  use those skills if available. If not available, follow the persona's graceful
  degradation instructions
