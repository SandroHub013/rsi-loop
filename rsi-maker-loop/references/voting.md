# Per-step decomposition, voting & verification

This is the engine of the loop. The goal: drive the error rate of each
micro-step toward zero so a long chain of steps doesn't derail.

## Step taxonomy

Classify every micro-step before doing it:

- **Verifiable** — a deterministic checker can say pass/fail: a unit test, the
  type checker, the compiler, a linter, a schema validator, a property/round-trip
  assertion. **Prefer making steps verifiable.** Often you can turn an
  "unverifiable" step into a verifiable one by writing the check first (write the
  test, then make it pass).
- **Unverifiable** — judgment with no ground-truth checker: naming, file
  layout, which existing helper to reuse, an API's shape, "is this the minimal
  change", "is this comment accurate". These get multi-agent voting.

## Verifiable steps — verification *is* the vote

1. Produce a candidate (the change).
2. Run the relevant check (smallest sufficient: the touched test file, then the
   suite; typecheck; build).
3. If it passes → accept. If it fails → fix and retry a small number of times.
4. If it still fails → the step was too big or mis-specified: **re-decompose**
   it into smaller steps, or escalate to the user. Do not paper over a red check.

This is the cheapest and strongest error correction available — use it whenever
a check exists. No K-way ensemble needed.

## Unverifiable steps — multi-agent voting

**Skip the ensemble for obvious, low‑stakes choices** — decide directly. Spinning
up K agents to ratify a trivial decision just burns budget (observed in trials:
3 voters unanimously agreed on an obvious call). Reserve the vote for genuinely
contested or high‑consequence steps (irreversible, security, public API).

When a vote is warranted, use MAKER's scheme (arXiv:2511.09030, "first‑to‑ahead‑by‑k"):

1. Sample **independent** candidates for the *same* step with **different framings**
   (see "Independence" below). First sample at temperature 0, the rest slightly
   higher (e.g. 0.1). Each returns a concrete candidate + one‑line rationale.
2. **Red‑flag and resample** any candidate that is **malformed** (fails a quick
   format/parse check) or **over‑long** (rambling output correlates with errors) —
   don't let it count as a vote.
3. Tally semantically‑equal candidates and accept by **first‑to‑ahead‑by‑k**: the
   first answer that leads by `vote_k` votes (k≈3 is enough for very high
   reliability) wins. Keep sampling until something is ahead by k.
4. If a sensible sampling cap is hit with no winner, spawn a **judge** agent: give
   it the candidates (anonymized, no ordering bias) + the acceptance criteria; it
   picks one with a stated reason.
5. Record the decision and rationale in the commit/PR so it's auditable.

> Why "ahead by k" rather than "k total": it adapts effort to difficulty — easy,
> uncontested steps resolve in ~k samples; only genuinely split steps cost more.
> Cost grows ~`k · steps`, i.e. log‑linear, which is what makes long horizons
> affordable.

### Independence (or the vote is worthless)

Correlated voters that share a blind spot will agree on the *wrong* answer with
full confidence. Force diversity:

- distinct **personas/objectives** per voter, e.g.:
  - *minimalist*: "smallest change that works; reject speculative abstraction";
  - *robustness*: "handle edge cases, errors, and failure modes";
  - *conformance*: "match the existing conventions and patterns in this repo";
- vary **temperature**;
- mix **model families** when the host supports it.

Always include the **minimalist** voter. It is the structural defense against an
infinite loop slowly gold-plating the codebase into bloat.

## Escalation

A step escalates to the user (or a parked issue) when:

- a verifiable step can't be made green after re-decomposition;
- voters are split with no defensible judge decision;
- the step would exceed `max_blast_radius`;
- the step touches something the contract marks as high-consequence (security,
  irreversible, public API) and confidence is low.

Escalating is success, not failure: it's the loop refusing to build on sand.

## Worked example

Backlog item: *"Add delivery receipts (sent/delivered/read) to the chat."*

Decomposition (mixed verifiable/unverifiable):

1. *(unverifiable)* Decide the on-wire representation of receipt state →
   **vote**: minimalist proposes an enum field on the existing message; others
   agree → accept the small option.
2. *(verifiable)* Add the field + migration → write a round-trip test, make it
   pass.
3. *(verifiable)* Sender marks "sent" on ack → unit test on the ack path.
4. *(verifiable)* Recipient emits "delivered"/"read" → test the state transitions.
5. *(unverifiable)* UI affordance (checkmarks) → **vote** (conformance voter
   wins: match the app's existing iconography).
6. *(verifiable)* Full gate: suite + typecheck + build green → commit.

Each step is accepted only when its own check/vote passes, so step 4 never
builds on a broken step 2.
