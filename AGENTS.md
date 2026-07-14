# Orchestrate work

Decide *who* does the work before doing it. Route tasks to the cheapest capable
model or agent, require a pass/fail bar before any delegation, and grade with
a different model than the one that built it.

## Quick start

1. List the models/agents you actually have access to.
2. Rank each on three axes: **cost** (availability/rate-limit headroom, not
   just dollars), **intelligence** (how hard a problem it can handle
   unsupervised), **taste** (UI/UX, code quality, API design, prose).
3. Classify the task by type — decision, bulk, taste, review, analysis, or
   loop — then route by the table below.
4. Before delegating anything, write the pass/fail test. No test, no
   delegation.
5. Send the delegate a full kickoff package, not just a prompt.
6. Grade with a different model than the one that built it.
7. Before any commit/push, run the gate check.

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
instead of a per-task judgment call.

## Routing

Classify first — pick the dominant type, then route:

| type | what it covers | route |
|------|----------------|-------|
| **decision** | planning, architecture calls, synthesis, final judgments | your best-taste, best-judgment model — don't delegate the thinking itself |
| **bulk** | clear-spec implementation, migrations, mechanical refactors, data crunching | your cheapest capable model — spend it freely, that's the point of ranking cost |
| **taste** | UI, copy, API design — anything user-facing | a taste-ranked model, never your cheapest |
| **review** | grading a plan, diff, or implementation | a *different* model than the one that built it — same-model review inherits the same blind spots |
| **analysis** | token-hungry grunt work: log crunching, heavy codebase reads, browser driving | offload it; return only the conclusion, keep the raw dump out of the main context |
| **loop** | long-running iterate-until-bar work | orchestrator sets the bar and grades; a cheap model builds each iteration |

- **Bulk with a dependency chain** (e.g. schema change → codegen →
  consumers): serialize *only* across the genuine dependency seam; fan out
  the independent file groups on each side in parallel (isolated workspaces
  when they'd touch overlapping files). Don't default to one giant
  sequential pass.
- **Escalate sideways before up.** When a cheap model's output falls short,
  try another cheap/mid model before reaching for your most expensive one.
- **Verification counts as analysis.** When computer use — clicking through
  the app, eyeballing a render — is the easiest way to complete *or verify*
  work, offload that too. Don't drive a browser on your scarcest model just
  because the task is "checking" rather than "building".

## Routing to an out-of-family model

Your orchestrator's `model` parameter may only accept models from one family
(e.g. Claude-only, OpenAI-only). If your cheapest/bulk model lives outside
that family and is reachable only through its own CLI or API, don't fight
the parameter — wrap it:

- **Spawn a thin, low-effort in-family wrapper agent** whose only job is to
  write a self-contained prompt, invoke the out-of-family model via its
  CLI/API, and return the result. Give the wrapper a schema if you need
  structured output back.
- **Label these agents distinctly** (e.g. a `bulk-model:` prefix) — your
  orchestration UI will show the wrapper's in-family model, so the label is
  the only sign of who actually did the work.
- **Watch tool timeouts.** A long external run can outlast your shell tool's
  default timeout; pass an explicit timeout, or run it in the background and
  poll for a report file.
- **Isolate parallel writers.** Parallel wrapper agents that edit files need
  isolated workspaces (e.g. git worktrees) so their edits don't collide in a
  shared checkout.
- **Budget separately.** Token/spend tracking built into your orchestrator
  usually only counts in-family tokens — external work is invisible to it,
  so track its cost (or lack of one) on its own.

## Kickoff package

Every delegation ships with, up front:

- **The bar** — the pass/fail test.
- **The floor** — the best prior artifact or trace to match and beat, so
  you're not re-explaining what good looks like.
- **Budgets, not permission-per-use** — spend limits declared once, so the
  delegate never stops to ask.
- **Where credentials live** — named at kickoff, same reason.
- **Return conditions** — come back only if truly blocked or facing a
  decision only the owner can make. Everything else: make the call and keep
  moving.
- **Scope honesty** — the one other sanctioned stop: if the ask is too much
  work at once, stop and state that clearly. Propose a split or a smaller
  first slice instead of silently attempting the pile.

## The bar

Adjectives are not acceptance criteria. Every delegated task needs a concrete
pass/fail test, declared *before* the work starts.

- Write the test first: "a stranger can't tell it from the reference," "all
  call sites migrated and the type-checker is clean," "p95 under 200ms."
- Can't measure it? The first delegation is inventing the measuring stick —
  not the build.
- The builder never grades its own work. Grading goes to a different model,
  in a fresh context, pointed at the real output, mandated to prove it FAILS.

## Loops

A loop is only valid against a pre-declared bar — no bar, no loop, it just
circles.

Grade against the bar → name the single biggest gap → close it → re-grade
with an independent grader. Stop when an independent grader genuinely can't
find a remaining gap — not when the builder says "I think it's done."

## Standing invariants (house rules)

Hold no matter how the goal is reached — when a rule here conflicts with
speed or convenience, the rule wins.

1. No delegation without a bar. Adjectives ("high quality", "clean") are not
   bars.
2. The builder never grades its own work.
3. No hard-coded special cases in agent behavior — describe the desired
   behavior and let the model reason.
4. Taste-critical output never ships from your cheapest or lowest-taste
   model.
5. Set a floor model you never go below for anything user-facing or
   safety-relevant.
6. Escalate sideways before up.
7. Report reality — failing tests, skipped steps, and unverified claims get
   stated plainly, no rounding up to "done".
8. Nothing sensitive leaves the machine.

## The gate

Before any commit or push, run a review pass — a different model than
whoever wrote the change — whose only job is to check the diff against the
invariants above and the declared bar. It returns PASS or a list of
violations. Violations block the push until fixed or explicitly waived.

---

Distilled from [Theo's (@theo) thread](https://x.com/theo/status/2072482460122964067?s=46)
and Matt Shumer's
["How I Prompt Fable"](https://simplemarkdowneditor.com/pub/IbaCrTjLJT?key=uQOQ2NPO3TTUSXyYDjyLf).
Full version with a worked example: https://github.com/udhaya21/orchestrate-work
