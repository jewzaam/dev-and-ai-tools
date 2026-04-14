---
name: agentic-loop
description: Run the agentic orchestration loop. Use after agentic-scaffold has generated task infrastructure. Executes loop.sh to run tasks through the DEV/TEST/REVIEW/JUDGE cycle until approved or blocked.
version: 1.0.0
allowed-tools: [Bash, Read, Edit]
---

# Skill: Agentic Loop

## Purpose

Execute the agentic build loop for a scaffolded project. This is the
"run" step after project-bootstrap and agentic-scaffold have prepared
the task infrastructure.

## Process

### Step 1 — Validate Prerequisites

Run the validation script using Bash:
```bash
${CLAUDE_PLUGIN_ROOT}/tools/validate-loop-prereqs.sh
```

If the script exits non-zero, stop and tell the user to run agentic-scaffold first.

If the script exits zero, it prints the current state: task specs, personas,
cursor position, and build status. Use this to confirm readiness.

### Step 2 — Show Configuration

If `orchestrator.yaml` exists, display:
- Workflow mode (TDD or standard)
- Model routing
- Quality gate settings

### Step 3 — Run the Loop

```bash
./loop.sh
```

This runs until one of:
- `tasks/BUILD_STATUS.md` is `APPROVED`
- `tasks/BUILD_STATUS.md` starts with `FAILED`
- `[BLOCKED]` appears in `tasks/TASK_LIST.md`

### Step 4 — Report Results

After the loop exits, report:
1. Final build status
2. Summary of `tasks/FEEDBACK.md` (if it exists)
3. Any `[BLOCKED]` entries
4. The final judge verdict (if build completed)
