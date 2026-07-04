<div align="center">

# rsi-loop

**Autonomous, backlog-driven development loop for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).**

One skill, one idea: turn a `BACKLOG.md` into shipped, verified changes — one
medium-sized task at a time — and run indefinitely.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
![Claude Code skill](https://img.shields.io/badge/Claude%20Code-skills-8A63FF)

</div>

---

## What it is

`rsi-loop` is a lean autonomous loop that advances a codebase one **medium task
per iteration**. Each iteration picks the highest-priority unchecked item from
`BACKLOG.md`, implements the minimal correct change, runs the repo's full
quality gate (tests + typecheck + build), and commits. It can self-schedule to
run perpetually, regenerating the backlog when empty.

**Medium tasks are the sweet spot:** self-contained features, refactors, or
bugfixes — typically 2–5 files, one logical change, completable in a single
iteration. Large enough to feel like real progress, small enough that a single
model handles them reliably without micro-step voting.

## Workflow

![rsi-loop workflow](docs/rsi-loop.png)

## Why

Autonomous coding loops fail over long horizons because errors **accumulate**.
`rsi-loop` keeps it simple: a capable model advances the backlog, the repo's
own gate is the safety net. No multi-agent voting, no micro-step decomposition
— just steady, gated progress on meaningful tasks.

## Install

Copy the skill folder so that `SKILL.md` sits at `…/skills/rsi-loop/SKILL.md`:

```bash
git clone https://github.com/SandroHub013/rsi-maker-loop /tmp/rsi-loops
cp -r /tmp/rsi-loops/rsi-loop  ~/.claude/skills/rsi-loop
```

Then ask in natural language — "start the RSI loop on this repo", "run an
autonomous loop", "keep working through my backlog".

## Configure

Optional per-repo config (`.rsi-loop.json`):

```json
{ "commit_target": "branch-pr", "perpetual": true, "max_blast_radius": "medium" }
```

`commit_target`: `"branch-pr"` (open PRs — recommended) or `"main"`.

## Honest boundaries

- **The gate is the only safety net.** This loop has no redundancy/voting, so a
  mistake that the tests don't catch can land. Keep the suite meaningful, and
  prefer `branch-pr` so a human sees changes.
- **Execution, not direction.** It advances the backlog reliably but doesn't
  decide *what* the product should become — it pauses for a human checkpoint when
  regenerating the backlog; user bugs/requests always take priority.
- **Medium tasks need good judgment.** Unlike micro-steps, each iteration makes
  multiple decisions — the model's ability to make sound trade-offs matters.

## Layout

```
.
├─ rsi-loop/            # the loop
│  ├─ SKILL.md
│  └─ references/backlog.md
├─ README.md
└─ LICENSE
```

## License

[MIT](LICENSE).
