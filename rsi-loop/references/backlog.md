# BACKLOG.md — format, priorities, regeneration

The backlog is the loop's queue and memory. It must be resumable: any fresh
agent should read it and know exactly what to do next.

## Priorities

- **P0 — reliability & correctness**: it must work and not lose data. Bugs,
  persistence, error handling, flaky-test fixes.
- **P1 — architecture that unblocks scale/quality**: interfaces, seams, test
  harnesses. Build the abstraction now; heavy implementation can come later.
- **P2 — features toward the goal**: user-facing capability.
- **P3 — hardening & polish**: security passes, a11y, docs, CI, refactors that
  pay for themselves.

Within a priority, order by value/effort. The loop always takes the first
unchecked item top-down. Keep items **medium-sized** — self-contained features,
refactors, or bugfixes that are completable in one iteration (typically 2–5
files, one logical change). If an item is too large (an epic), split it into
medium items; if it is too small (a micro-step), batch it with related items.

## Item format

Plain Markdown checkboxes so progress is visible in any viewer:

```markdown
## P0 — reliability & correctness
- [ ] Persist the work queue across restarts (swappable store; local impl now).
  - [ ] follow-up: flush on shutdown.
- [x] Dedup incoming events by id.
```

## Regeneration rule

When **no unchecked item remains anywhere** in the file:

1. Generate a **small batch of 3–5** new medium-sized items — the highest-value
   real gaps toward the project's goal (reliability, performance, security, UX),
   **not** speculative features. Prefer the smallest change that works.
2. Prioritize them P0–P3 and append.
3. Commit the new plan and continue the loop — no human checkpoint needed.

## Starter template

```markdown
# <project> — Backlog (autonomous loop)

Queue for rsi-loop. Goal: <one sentence>.

## Loop rules (every iteration)
1. First unchecked item, P0→P3.
2. Scope check: item must be medium-sized (not micro, not epic); split or batch if needed.
3. Minimal correct change; reuse existing modules; no new deps if a few lines do.
4. Commit only if the full gate (tests + typecheck + build) is green; add a test
   for new logic.
5. Update this file (check off + follow-ups) and docs.
6. Commit per the configured target (default: branch + PR).
7. If the backlog is empty: regenerate a 3–5 item batch and continue.

## P0 — reliability & correctness
- [ ] ...

## P1 — architecture
- [ ] ...

## P2 — features
- [ ] ...

## P3 — hardening
- [ ] ...
```
