---
name: rsi-loop
description: >-
  Run an autonomous development loop on a codebase, driven by a BACKLOG.md:
  advance one medium-sized task per iteration with the repo's own
  tests/typecheck/build as the gate, commit, and (optionally) self-schedule the
  next iteration. Use whenever the user wants a self-running dev or maintenance
  loop: "autonomous loop", "RSI loop", "keep working through my backlog",
  "perpetual development". Each iteration handles a complete, meaningful task —
  not a micro-step — balanced between scope and reliability.
---

# RSI Loop (medium task)

A backlog-driven autonomous loop that advances a project **one medium task at a
time** and can run indefinitely. Each iteration picks up a self-contained item
from the backlog, implements the minimal correct change, runs the repo's full
quality gate, and commits — no micro-decomposition, no multi-agent voting.

## Scope philosophy

This loop is configured for **medium tasks** (not micro-steps, not epics):

| Scope | When to use |
|---|---|
| **Micro** (single function, one file) | Use when the task is trivially checkable; rsi-loop handles these easily but you're paying overhead per item |
| **Medium ← you are here** | A self-contained feature, refactor, or bugfix — touches 2–5 files, one logical change, completable in one iteration. The sweet spot for rsi-loop |
| **Large / epic** | Multi-iteration work that needs decomposition into medium tasks in the backlog |

A good medium task has a clear success criterion, fits in one iteration, but is
substantial enough that completing it feels like real progress.

## Setup (first run)

Resolve the operating contract:

1. If `.rsi-loop.json` exists at the repo root, use it (don't ask again).
2. **Else, if a human is present: ask 2–3 quick questions**, then persist them to
   `.rsi-loop.json` (so later/cloud runs inherit them):
   - *Where should changes land?* → `commit_target`: **branch + PR** (recommended)
     or **main**.
   - *Run forever or one pass?* → `perpetual`.
3. **Else (unattended, e.g. a scheduled cloud run): do NOT block** — use the safe
   defaults below and record them.

Contract fields (defaults shown):

- **commit_target** (`"branch-pr"`): branch + PR for review, or `"main"` behind
  the gate. Default `branch-pr` — unattended unreviewed `main` pushes cause damage.
- **perpetual** (`true`): self-schedule the next iteration when done.
- **max_blast_radius** (`"medium"`): files/lines a single iteration may touch
  before it must stop and ask — allows meaningful medium tasks while keeping a
  wrong turn bounded.

If there is no `BACKLOG.md`, create one (see `references/backlog.md`) by briefly
interviewing the user about the goal, then seed P0–P3 medium-sized items.

## The loop (one iteration)

Do exactly one iteration per invocation, then perpetuate (or stop):

1. **Intake.** Read `BACKLOG.md`. User-reported bugs / open issues addressed to
   the loop take priority; otherwise take the first unchecked item (P0 → P3).

2. **Scope check.** Verify the item is a medium task (not a micro-step, not an
   epic). If it is too large, decompose it into 2–4 medium items in the backlog
   and pick the first. If it is a micro-step, batch it with related items until
   the batch forms a medium task.

3. **Implement** the **minimal correct** change, reusing existing modules. No
   speculative abstractions; no new dependency if a few lines do. Match the
   repo's conventions. Keep it within `max_blast_radius`.

4. **Gate.** Run the **full** quality gate the repo defines (complete test suite
   + typecheck + build). Add a test for genuinely new logic. Commit **only if
   everything is green** — a red gate means fix or revert, never commit red.

5. **Record.** Check off the item in `BACKLOG.md`, add follow-ups, update docs.

6. **Commit** per `commit_target`, with a message naming the item. Don't commit
   session/scheduler state or scratch files.

7. **Regenerate (RSI).** If no unchecked items remain anywhere, generate a small
   batch (3–5) of high-value, non-speculative medium items, commit the plan, and
   **surface it to the user before going deep** — direction is theirs.

8. **Perpetuate.** If `perpetual`, schedule the next iteration (host loop / a
   self-paced wakeup) with a prompt that re-enters this skill; else stop.

## Model

A single capable model (e.g. Sonnet, Opus) handles the entire iteration. Since
each task is medium-sized — not a chain of micro-steps — the model has room to
understand context and produce a coherent change without per-step voting. If the
model repeatedly fails the gate on medium tasks, reduce scope to smaller items.

## Honest boundaries

- **The gate is the only safety net.** This loop has no redundancy/voting, so a
  mistake that the tests don't catch can land. Keep the suite meaningful, and
  prefer `branch-pr` so a human sees changes.
- **Execution, not direction.** It advances the backlog reliably but doesn't
  decide *what* the product should become — it pauses for a human checkpoint when
  it regenerates the backlog, and user requests always win.
- **Medium tasks require judgment.** Unlike micro-steps, a medium task involves
  multiple decisions — the model's ability to make good trade-offs matters more
  here than in a micro-step setup.

## References

- `references/backlog.md` — `BACKLOG.md` format, priorities, and regeneration rule.
