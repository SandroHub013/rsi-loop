<div align="center">

# rsi-loop

**Fully agentic, backlog-driven development loop for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).**

One skill, one idea: turn a `BACKLOG.md` into shipped, verified changes — one
medium-sized task at a time — with zero human intervention.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
![Claude Code skill](https://img.shields.io/badge/Claude%20Code-skills-8A63FF)

</div>

---

## What it is

`rsi-loop` is a fully agentic autonomous loop that advances a codebase one
**medium task per iteration**. Each iteration picks the highest-priority
unchecked item from `BACKLOG.md`, implements the minimal correct change, runs
the repo's full quality gate (tests + typecheck + build), and commits. It
self-schedules perpetually, regenerating the backlog when empty — no questions,
no checkpoints, no human in the loop.

**Medium tasks are the sweet spot:** self-contained features, refactors, or
bugfixes — typically 2–5 files, one logical change, completable in a single
iteration. Large enough to feel like real progress, small enough that a single
model handles them reliably.

## Workflow

![rsi-loop workflow](docs/rsi-loop.png)

## How it runs

1. **Start** — no config prompts: uses safe defaults (`branch-pr`, `perpetual`,
   `max_blast_radius: medium`). Infers project goal from repo structure.
2. **Loop** — pick first unchecked backlog item → implement → gate → commit.
3. **Regenerate** — when backlog is empty, generate 3–5 new items aligned to the
   project goal and continue immediately — no human checkpoint.
4. **Perpetuate** — schedule the next iteration and repeat indefinitely.

## Install

```bash
git clone https://github.com/SandroHub013/rsi-maker-loop /tmp/rsi-loops
cp -r /tmp/rsi-loops/rsi-loop  ~/.claude/skills/rsi-loop
```

Then: "start the RSI loop on this repo".

## Configure

Optional per-repo config (`.rsi-loop.json`):

```json
{ "commit_target": "branch-pr", "perpetual": true, "max_blast_radius": "medium" }
```

If absent, these defaults are used silently.

## Honest boundaries

- **The gate is the only safety net.** No redundancy, no voting. A mistake the
  tests don't catch can land. Keep the suite meaningful.
- **Direction is self-determined.** The loop decides what to build from the
  project's own signals — no human checkpoint at regeneration.

## Layout

```
.
├─ rsi-loop/
│  ├─ SKILL.md
│  └─ references/backlog.md
├─ README.md
└─ LICENSE
```

## License

[MIT](LICENSE).
