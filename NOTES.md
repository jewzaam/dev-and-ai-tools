# Setup (on the host, one time):

```bash
mkdir -p ~/source/pixel-canvas/docs
cp ~/source/dev-and-ai-tools/pixel-canvas.md ~/source/pixel-canvas/docs/requirements.md
cd ~/source/pixel-canvas
git init && git add -A && git commit -m "init: add requirements"
echo "PODMAN_PROJECT=pixel-canvas" > .env
```

# Run the skills (start a Claude Code session with the plugin loaded):

```bash
cd ~/source/pixel-canvas
claude --plugin-dir ~/source/dev-and-ai-tools
```

# Then inside that session:

## generate ARCHITECTURE.md + TASKS.md

```claude
/project-bootstrap docs/requirements.md
```

Review those docs, edit if needed.  Re-run skill to refine.

## generate task infrastructure

```claude
/agentic-scaffold docs/TASKS.md docs/ARCHITECTURE.md
```

## validate & run loop.sh, which spawns

```claude
/agentic-loop
```

validates prereqs, runs loop.sh, which spawns sandbox containers per stage


