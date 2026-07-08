---
name: orchestrate-work
description: Decide which model or agent should handle a piece of work based on cost, intelligence, and taste tradeoffs, require a pass/fail bar before delegating, and gate every commit or push against standing invariants. Use when starting non-trivial multi-step work, deciding whether to delegate a task to a subagent or cheaper model, setting up a review or grading loop, or defining pre-commit quality gates for agent-driven work.
---

# Orchestrate work

Decide *who* does the work before doing it. Route tasks to the cheapest capable
model, require a pass/fail bar before any delegation, and grade with a
different model than the one that built it.

## Quick start

1. List the models/agents you actually have access to.
2. Rank each on three axes: **cost** (availability/rate-limit headroom, not
   just dollars), **intelligence** (how hard a problem it can handle
   unsupervised), **taste** (UI/UX, code quality, API design, prose).
3. Route by the table in [Routing](#routing).
4. Before delegating anything, write the pass/fail test. No test, no
   delegation.
5. Grade with a different model than the one that built it.
6. Before any commit/push, run [The gate](#the-gate).

## Rank your fleet

Fill in your own models — this table is the shape, not the values. Higher is
better on every axis, including cost: 9 means cheap/available, 2 means
scarce/expensive.

| model                          | cost | intelligence | taste |
|---------------------------------|------|--------------|-------|
| cheapest, highest-limit model   | 9    | ?            | ?     |
| mid-tier model                  | 5    | ?            | ?     |
| flagship reasoning model        | ?    | 9            | ?     |
| best-taste orchestrator model   | lowest | highest    | highest |

The exact numbers don't matter. What matters is naming, up front, which model
is cheap-and-good-enough versus scarce-and-precious, so routing is consistent
instead of a per-task judgment call. See [EXAMPLE.md](EXAMPLE.md) for a real
filled-in version.

## Routing

- **Decisions, planning, taste-critical synthesis** → your best-taste,
  best-judgment model. Don't delegate the thinking itself.
- **Bulk / mechanical work** (clear-spec implementation, migrations,
  refactors) → your cheapest capable model. Spend it freely — that's the
  point of ranking cost.
- **User-facing work** (UI, copy, API design) → never your cheapest model.
  Taste-critical output needs a taste-ranked model.
- **Reviews** → a *different* model than the one that wrote the code.
  Same-model review inherits the same blind spots.
- **Token-hungry grunt work** (browser driving, log crunching, heavy
  analysis) → offload it, and return only the conclusion — keep the raw dump
  out of the main context.
- **Escalate sideways before up.** When a cheap model's output falls short,
  try another cheap/mid model before reaching for your most expensive one.

## The bar

Adjectives are not acceptance criteria. Every delegated task needs a concrete
pass/fail test, declared *before* the work starts.

- Write the test first: "a stranger can't tell it from the reference," "all
  call sites migrated and the type-checker is clean," "p95 under 200ms."
- Can't measure it? The first delegation is inventing the measuring stick —
  not the build.
- **The builder never grades its own work.** Grading goes to a different
  model, in a fresh context, pointed at the real output, mandated to prove it
  FAILS.

## Loops

A loop is only valid against a pre-declared bar — no bar, no loop, it just
circles.

Grade against the bar → name the single biggest gap → close it → re-grade
with an independent grader. Stop when an independent grader genuinely can't
find a remaining gap — not when the builder says "I think it's done."

## The gate

Before any commit or push, run a review pass — a different model than
whoever wrote the change — whose only job is to check the diff against your
standing invariants and the declared bar. It returns PASS or a list of
violations. Violations block the push until fixed or explicitly waived. See
[REFERENCE.md](REFERENCE.md) for a starter set of invariants to adapt.

## More

- [REFERENCE.md](REFERENCE.md) — starter set of standing invariants (house
  rules) to adapt to your own risk tolerance.
- [EXAMPLE.md](EXAMPLE.md) — a real filled-in version of this pattern, with
  credits.
