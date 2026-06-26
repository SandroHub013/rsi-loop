---
name: rsi-loop
description: >-
  Run a lean autonomous development loop on a codebase, driven by a BACKLOG.md:
  advance one item per iteration with the repo's own tests/typecheck/build as the
  gate, commit, and (optionally) self-schedule the next iteration — with NO
  multi-agent voting, to keep token cost low. Use whenever the user wants a simple
  self-running dev or maintenance loop: "autonomous loop", "RSI loop", "keep
  working through my backlog", "perpetual development on a budget", or a single
  capable model steadily shipping backlog items. For long-horizon work that must
  not drift or accumulate errors across many dependent steps, for weak/cheap
  models, or for high-consequence uncheckable decisions, prefer the sibling
  rsi-maker-loop (which adds MAKER per-step error correction at higher cost).
---

# RSI Loop (lean)

A backlog-driven autonomous loop that advances a project one item at a time and
can run indefinitely. It is the **economical default**: a single capable model
does each iteration, and the repo's own quality gate is the safety net. No
decomposition into voting ensembles — for that, use `rsi-maker-loop`.

## When to use this vs rsi-maker-loop

- **rsi-loop (this one):** clear, checkable work where a single strong model is
  reliable; you want low token cost. The gate (tests/typecheck/build) catches
  mistakes at the end of the iteration.
- **rsi-maker-loop:** long-horizon chains where errors compound, weak/cheap
  models, or unverifiable high-consequence decisions — it adds per-step error
  correction (decomposition + first-to-ahead-by-k voting), at higher cost.

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
- **max_blast_radius** (small): how much one iteration may change before it must
  stop and ask — keeps a wrong turn cheap.

If there is no `BACKLOG.md`, create one (see `references/backlog.md`) by briefly
interviewing the user about the goal, then seed P0–P3 items.

## The loop (one iteration)

Do exactly one iteration per invocation, then perpetuate (or stop):

1. **Intake.** Read `BACKLOG.md`. User-reported bugs / open issues addressed to
   the loop take priority; otherwise take the first unchecked item (P0 → P3).
2. **Implement** the **minimal correct** change, reusing existing modules. No
   speculative abstractions; no new dependency if a few lines do. Match the
   repo's conventions. Keep it within `max_blast_radius`.
3. **Gate.** Run the **full** quality gate the repo defines (complete test suite
   + typecheck + build). Add a test for genuinely new logic. Commit **only if
   everything is green** — a red gate means fix or revert, never commit red.
4. **Record.** Check off the item in `BACKLOG.md`, add follow-ups, update docs.
5. **Commit** per `commit_target`, with a message naming the item. Don't commit
   session/scheduler state or scratch files.
6. **Regenerate (RSI).** If no unchecked items remain anywhere, generate a small
   batch (3–5) of high-value, non-speculative items, commit the plan, and
   **surface it to the user before going deep** — direction is theirs.
7. **Perpetuate.** If `perpetual`, schedule the next iteration (host loop / a
   self-paced wakeup) with a prompt that re-enters this skill; else stop.

## Model

Because there is **no per-step error correction beyond the gate**, the single
model doing the work should be **capable**: default to a strong model (e.g. Opus,
or Sonnet for a cheaper but still strong option). On easy, clear tasks a strong
single shot is both reliable and cheaper than running an ensemble — that is the
whole reason this lean loop exists. If you find the model repeatedly failing the
gate on hard steps, that is the signal to switch to `rsi-maker-loop`.

## Honest boundaries

- **The gate is the only safety net.** This loop has no redundancy/voting, so a
  mistake that the tests don't catch can land. Keep the suite meaningful, and
  prefer `branch-pr` so a human sees changes.
- **Execution, not direction.** It advances the backlog reliably but doesn't
  decide *what* the product should become — it pauses for a human checkpoint when
  it regenerates the backlog, and user requests always win.
- **Long horizons drift.** Over very many dependent steps, errors can accumulate
  (the problem MAKER addresses). For that regime, use `rsi-maker-loop`.

## References

- `references/backlog.md` — `BACKLOG.md` format, priorities, and regeneration rule.
