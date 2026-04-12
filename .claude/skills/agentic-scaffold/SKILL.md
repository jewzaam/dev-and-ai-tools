---
name: agentic-scaffold
description: Convert ARCHITECTURE.md + TASKS.md into executable agentic workflow. Use after project-bootstrap produces approved docs. Creates task specs, TASK_LIST.md, personas, and loop.sh.
version: 1.0.0
argument-hint: <tasks-file> <architecture-file>
allowed-tools: [Read, Write, Edit, Bash, Glob]
---

# Skill: Agentic Scaffold

## Purpose
Transform a task document into a complete agentic build loop — spec files,
TASK_LIST.md, personas, and execution scripts — ready to run with loop.sh.

## When to Use This Skill
After the project-bootstrap skill has produced docs/ARCHITECTURE.md and
docs/TASKS.md, and those documents have been reviewed and approved.

## Input
- `tasks_file` (string, required) — Path to the task document (e.g., `docs/TASKS.md`)
- `architecture_file` (string, required) — Path to the architecture document (e.g., `docs/ARCHITECTURE.md`)

## Process

Read {tasks_file} and {architecture_file} fully before producing any output.

---

### Step 1 — Create Folder Structure

Create these directories if they don't exist:
```
tasks/
tasks/specs/
tasks/feedback/
tasks/personas/
```

---

### Step 2 — Spec Files

For every task in {tasks_file}, create `tasks/specs/task-{id}.md`.

Each spec file must contain exactly:

```markdown
# Task {id} — {title}

## Phase
{phase number}

## Description
{full task description verbatim from {tasks_file}}

## Acceptance Criteria
{acceptance criteria verbatim from {tasks_file}}

## Verify Scope
{backend | frontend | both}
```

Rules:
- Copy descriptions and acceptance criteria verbatim — do not summarise
- Verify scope rules:
  - Backend-only tasks (no UI changes): backend
  - Frontend-only tasks (no API changes): frontend
  - Tasks touching both API and UI: both
  - Deployment tasks: backend
  - PWA / polish tasks: frontend
  - When unclear: both

---

### Step 3 — TASK_LIST.md

Copy the template from `.claude/templates/TASK_LIST.md`.

Replace the example task entries with real entries from {tasks_file}.
The entry format depends on the workflow mode in `orchestrator.yaml`.

**Read `orchestrator.yaml`** (copied in Step 5) to determine workflow mode.
Default: `test_first = true`.

**TDD mode** (`test_first = true`) — each task gets entries in this order:
TEST, DEV, REVIEW, JUDGE, VERIFY

**Standard mode** (`test_first = false`) — each task gets entries in this order:
DEV, REVIEW, JUDGE, TEST, VERIFY

Use the entry patterns from `.claude/templates/TASK_LIST.md` for the selected mode.
Substitute task IDs, titles, verify commands, and cursor targets for each task.

The key differences from the old format:
- REVIEW stage added between DEV and JUDGE
- In TDD mode, TEST runs before DEV
- JUDGE now reads `tasks/feedback/Review-task-{id}.md` and appends to `tasks/FEEDBACK.md`
- JUDGE uses `mandated_fixes` in its verdict to specify what DEV must fix on retry

VERIFY command by scope:
- `backend`: `cd backend && python -m pytest -v`
- `frontend`: `cd frontend && npm test -- --watchAll=false`
- `both`: `cd backend && python -m pytest -v 2>&1 | tee tasks/feedback/task-{id}-verify.txt && cd ../frontend && npm test -- --watchAll=false`

Note: for `both`, adjust the tee command so output from both test runs
goes to the same verify file.

Cursor rules:
- Each task's VERIFY PASS cursor points to the next task's first stage entry
  (TEST in TDD mode, DEV in standard mode)
- The very last task's VERIFY PASS cursor points to: `→ NEXT: FINAL JUDGE`
- The initial `→ NEXT:` cursor at the top of the file points to the first task's
  first stage (TEST in TDD mode, DEV in standard mode)

The FINAL JUDGE entry is already in the template — do not modify it.
Insert all task entries between the header and the FINAL JUDGE section.

---

### Step 4 — Persona Files

Copy the four persona files verbatim from templates:
- `.claude/personas/developer.md` → `tasks/personas/developer.md`
- `.claude/personas/test_writer.md` → `tasks/personas/test_writer.md`
- `.claude/personas/reviewer.md` → `tasks/personas/reviewer.md`
- `.claude/personas/judge.md` → `tasks/personas/judge.md`

Do not modify persona file content.

---

### Step 5 — Execution Files

Copy these files verbatim:
- `.claude/templates/RUN.md` → `tasks/RUN.md`
- `.claude/templates/BUILD_STATUS.md` → `tasks/BUILD_STATUS.md`
- `.claude/templates/loop.sh` → `loop.sh`
- `.claude/templates/orchestrator.yaml` → `orchestrator.yaml`

Make loop.sh executable:
```bash
chmod +x loop.sh
```

---

### Step 6 — ARCHITECTURE_REF.md

Produce `tasks/ARCHITECTURE_REF.md` — a condensed version of
{architecture_file} for use as agent context.

Rules:
- Maximum 200 lines
- Include: tech stack table, file/directory layout, key patterns and
  conventions, naming conventions, existing helpers to reuse,
  orchestrator.yaml settings that affect agent behavior
- Exclude: rationale prose, out of scope sections, lengthy descriptions
- Every agent invocation will read this file — keep it dense and useful,
  not comprehensive

---

### Step 7 — Validation

1. Count tasks in {tasks_file}
2. Count spec files in tasks/specs/
3. Count task sections in tasks/TASK_LIST.md (excluding FINAL JUDGE)
4. Confirm all three counts match
5. Confirm the last task JUDGE PASS cursor points to FINAL JUDGE
6. Confirm tasks/personas/ has developer.md, test_writer.md, reviewer.md, judge.md
7. Confirm loop.sh is executable
8. Confirm orchestrator.yaml exists
9. Print output summary

## Output Summary Format

```
=== Agentic Scaffold Complete ===

Specs:          tasks/specs/          ({n} files)
Task list:      tasks/TASK_LIST.md    ({n} tasks)
Personas:       tasks/personas/       (developer, test_writer, reviewer, judge)
Config:         orchestrator.yaml
Arch ref:       tasks/ARCHITECTURE_REF.md
Execution:      tasks/RUN.md, tasks/BUILD_STATUS.md, loop.sh

To run one step at a time:
  tools/run-claude-sandbox.sh --task-file tasks/RUN.md

To run automatically until complete or blocked:
  ./loop.sh
```

### Step 8 — Transient Artifact Gitignore

Write `tasks/.gitignore`:
```
*
```

This prevents task artifacts from being committed to the project repository.
The agentic workflow files are transient — they exist only during the build.

---

## Quality Rules

- All task IDs must be consistent across TASKS.md, specs/, and TASK_LIST.md
- The TASK_LIST.md cursor chain must be unbroken from first task to FINAL JUDGE
- No task entry may reference a spec file that doesn't exist
- Persona files must include reviewer.md in addition to developer, test_writer, judge
- Persona files must not be modified from the templates
- loop.sh, RUN.md, and orchestrator.yaml must not be modified from the templates
