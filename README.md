# Agentic Orchestrator

A Claude Code plugin for multi-agent software development. Write requirements, let AI agents build it through a quality-gated orchestration loop.

## Prerequisites

- [Podman](https://podman.io/)
- [Claude Code](https://claude.ai/code) with authentication
- [review skill](https://github.com/jewzaam/claude-skill-review) — required by the Reviewer persona

**Optional:**
- [xenon](https://pypi.org/project/xenon/) — cyclomatic complexity checking (Judge falls back to lines-of-code checks if not installed)

## Setup

```bash
# Auth directory for sandbox tokens
mkdir -p ~/.claude-sandbox/auth

# Copy environment template
cp .env.example .env

# Authenticate (first time only — opens browser)
tools/run-claude-sandbox.sh --host-network
```

## Workflow

### 1. Write Requirements

Create `docs/requirements.md` describing what to build.

### 2. Generate Architecture and Tasks

```bash
tools/run-claude-sandbox.sh --task "Read .claude/skills/project-bootstrap/SKILL.md and bootstrap from docs/requirements.md"
```

Produces `docs/ARCHITECTURE.md` and `docs/TASKS.md`. Review and correct before proceeding.

### 3. Scaffold the Agentic Workflow

```bash
tools/run-claude-sandbox.sh --task "Read .claude/skills/agentic-scaffold/SKILL.md and scaffold from docs/TASKS.md and docs/ARCHITECTURE.md"
```

Creates task specs, personas, `orchestrator.yaml`, and `loop.sh`.

### 4. Configure (optional)

Edit `orchestrator.yaml` to adjust:
- `workflow.test_first` — TDD mode (default: true) or standard mode
- `models.*` — which Claude model per stage
- `quality.*` — complexity checking, lines limit
- `standards_fallback` — path to personal coding standards

### 5. Run the Build Loop

```bash
./loop.sh
```

The loop runs each task through these stages:

**TDD mode** (default):
```
TEST → DEV → REVIEW → JUDGE → VERIFY
```

**Standard mode** (`test_first: false`):
```
DEV → REVIEW → JUDGE → TEST → VERIFY
```

Stages:
- **TEST** — Writes tests against architecture interfaces (TDD) or against implementation (standard)
- **DEV** — Implements the spec; in TDD mode, makes tests pass
- **REVIEW** — Runs the `/review` skill to produce findings (C/I/S severity)
- **JUDGE** — Scores on 6 dimensions, triages review findings, mandates fixes
- **VERIFY** — Runs the test suite

The Judge produces a `mandated_fixes` list. If the verdict is FAIL, the Developer must address every mandated fix before the next attempt. After 3 failed attempts, the task is marked `[BLOCKED]` and the loop stops for human intervention.

The loop exits when `tasks/BUILD_STATUS.md` shows `APPROVED` or `FAILED`.

### 6. Review and Push

All commits (dev, test, review, judge verdicts) are in git history.

## Judge Scoring

6 dimensions, each rated 1-5:

| Dimension | Blocking if |
|-----------|-------------|
| Spec Compliance | < 4 |
| Implementation Quality | < 3 |
| Test Quality | < 3 |
| Code Reuse & Consistency | < 3 |
| Slop Detection | < 3 |
| Complexity | < 3 |

The Judge also reviews the Reviewer's output quality and collects feedback to `tasks/FEEDBACK.md` for persona refinement.

## Sandbox

The sandbox (`tools/run-claude-sandbox.sh`) runs Claude Code in a Podman container:

- **Filesystem**: Only the worktree is mounted
- **Network**: No network by default (`--host-network` for host access, `--isolated` for compose services only)
- **Security**: `--cap-drop ALL`, `--no-new-privileges`, `--read-only`, `--pids-limit 256`
- **Resources**: 4GB memory, 2 CPUs (configurable via `.env`)

```bash
# Interactive session
tools/run-claude-sandbox.sh

# One-shot task
tools/run-claude-sandbox.sh --task "implement pagination"

# With host network access
tools/run-claude-sandbox.sh --host-network --task "test API integration"

# Shell access for debugging
tools/run-claude-sandbox.sh --shell
```

## Project Structure

```
.claude-plugin/
  plugin.json              # Plugin manifest

.claude/
  personas/                # Agent role definitions
    developer.md           # Implementation, standards fallback, TDD mode
    test_writer.md         # Adversarial testing, TDD mode
    reviewer.md            # Runs /review skill, produces findings
    judge.md               # 6-dimension scoring, mandated fixes, feedback
  skills/
    project-bootstrap/     # Requirements → Architecture + Tasks
    agentic-scaffold/      # Architecture + Tasks → executable workflow
    agentic-loop/          # Run the orchestration loop
  templates/               # Copied per-project by agentic-scaffold
    orchestrator.yaml      # Workflow configuration
    loop.sh, TASK_LIST.md, RUN.md, BUILD_STATUS.md

containers/
  claude-sandbox/
    Containerfile          # Hardened container (Node 22 + Python 3 + uv)

tools/
  run-claude-sandbox.sh    # Podman sandbox wrapper
  validate-loop-prereqs.sh # Prereq validation script
```

## Configuration

Environment variables (set in `.env`):

| Variable | Default | Description |
|----------|---------|-------------|
| `CLAUDE_SANDBOX_MEMORY` | `4g` | Memory limit |
| `CLAUDE_SANDBOX_CPUS` | `2` | CPU limit |
| `CLAUDE_SANDBOX_IMAGE` | `localhost/claude-sandbox:latest` | Container image |
| `CLAUDE_SANDBOX_AUTH_DIR` | `~/.claude-sandbox/auth` | Auth token directory |
| `PODMAN_PROJECT` | (basename of worktree) | Project identifier |

## License

Apache 2.0
