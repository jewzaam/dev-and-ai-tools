# {Project Name} — Task List

## How This File Works

This file is the single source of truth for the agentic build loop.

Each time an agent is invoked it:
1. Reads the `→ NEXT:` cursor below to find its task
2. Reads the persona and spec listed in that task entry
3. Executes the task
4. Updates the checkbox for that entry `[ ]` → `[x]`
5. Advances the `→ NEXT:` cursor to the following entry
6. Exits

The judge is the only agent that can move the cursor backwards.
If the judge fails a task it unchecks the relevant entries,
resets the cursor, and writes feedback to a file for the developer to read.

Feedback files live in: `tasks/feedback/`
Persona files live in: `tasks/personas/`
Spec files live in: `tasks/specs/`
Configuration: `orchestrator.yaml`

---

→ NEXT: Task 0.1 — {first_stage}

---

{TASK ENTRIES GO HERE — one section per task}

{The scaffold skill generates entries in one of two patterns:}

{TDD MODE (test_first = true):}
{  TEST → DEV → REVIEW → JUDGE → VERIFY → JUDGE(final)}

{STANDARD MODE (test_first = false):}
{  DEV → REVIEW → JUDGE → TEST → VERIFY → JUDGE(final)}

{EXAMPLE — TDD mode entry (replace with real tasks):}

{
## Task 0.1 — [Title]

- [ ] **TEST** — `persona: tasks/personas/test_writer.md` — attempt 1 of 3
  Read spec: `tasks/specs/task-0.1.md`
  Read architecture: `tasks/ARCHITECTURE_REF.md`
  TDD mode: write tests BEFORE implementation exists.
  Define expected behavior based on spec and architecture interfaces.
  When done:
  - Run `git add -A && git commit -m "test: task 0.1 (TDD)"`
  - Check this box
  - Set cursor to: `→ NEXT: Task 0.1 — DEV`

- [ ] **DEV** — `persona: tasks/personas/developer.md` — attempt 1 of 3
  Read spec: `tasks/specs/task-0.1.md`
  Read existing tests (TDD mode — make them pass)
  If retrying, read feedback and mandated fixes: _(none yet)_
  Implement the task. When done:
  - Commit using commit skill if available, otherwise:
    `git add -A && git commit -m "dev: task 0.1"`
  - Check this box
  - Set cursor to: `→ NEXT: Task 0.1 — REVIEW`

- [ ] **REVIEW** — `persona: tasks/personas/reviewer.md` — attempt 1 of 3
  Read spec: `tasks/specs/task-0.1.md`
  Read the diff: `git diff HEAD~1`
  Run /review skill (or manual review if unavailable).
  Copy review output to: `tasks/feedback/`
  When done:
  - Run `git add -A && git commit -m "review: task 0.1"`
  - Check this box
  - Set cursor to: `→ NEXT: Task 0.1 — JUDGE`

- [ ] **JUDGE** — `persona: tasks/personas/judge.md` — attempt 1 of 3
  Read spec: `tasks/specs/task-0.1.md`
  Read diff: `git diff HEAD~3` (covers test + dev + review commits)
  Read review: `tasks/feedback/Review-task-0.1.md`
  Write verdict to: `tasks/feedback/task-0.1-judge.md`
  Append feedback to: `tasks/FEEDBACK.md`
  Then read the verdict and act on it:

  If verdict is PASS or PASS_WITH_CONCERNS:
  - Check this box
  - Set cursor to: `→ NEXT: Task 0.1 — VERIFY`

  If verdict is FAIL:
  - Do NOT check this box
  - Uncheck DEV and REVIEW above (they must be redone)
  - Increment attempt numbers on DEV, REVIEW, JUDGE by 1
  - If any attempt number would exceed 3: replace checkbox with `[BLOCKED]` and stop
  - Update the feedback reference on DEV: replace `_(none yet)_` with path to verdict file
    and list the mandated_fixes from the verdict
  - Set cursor to: `→ NEXT: Task 0.1 — DEV`

  Feedback for DEV on retry: _(none yet)_

- [ ] **VERIFY** — attempt 1 of 3
  Run: `{verify_command} 2>&1 | tee tasks/feedback/task-0.1-verify.txt`
  If exit code 0 (tests pass):
  - Check this box
  - Set cursor to: `→ NEXT: Task 1.1 — {first_stage}`
  If exit code non-zero (tests fail):
  - Do NOT check this box
  - Uncheck DEV and REVIEW above (they must be redone)
  - Increment attempt numbers on DEV, REVIEW, VERIFY by 1
  - If attempt number would exceed 3: replace checkbox with `[BLOCKED]` and stop
  - Set cursor to: `→ NEXT: Task 0.1 — DEV`

---
}

---

## Final Judge

- [ ] **FINAL JUDGE** — `persona: tasks/personas/judge.md`
  You are doing a final review of the entire project, not a single task.

  Read:
  - `tasks/ARCHITECTURE_REF.md` — what was planned
  - `git log --oneline` — what was actually built
  - All `tasks/feedback/*-judge.md` files — what concerns were raised during the build
  - `tasks/FEEDBACK.md` — accumulated feedback across all iterations

  Write your full verdict to: `tasks/feedback/final-judge.md`

  You are looking for systemic issues that per-task judges may have missed:
  - Patterns that were inconsistent across phases
  - Concerns that appeared in multiple pass_with_concerns verdicts and were never resolved
  - Anything that would make this codebase hard to maintain or extend
  - Review quality trends — did the Reviewer improve or degrade over time?

  Then update `tasks/BUILD_STATUS.md` — overwrite the entire file with one of:

  If PASS:
  APPROVED

  If FAIL:
  FAILED — see tasks/feedback/final-judge.md
