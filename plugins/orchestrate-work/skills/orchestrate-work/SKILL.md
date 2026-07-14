---
name: orchestrate-work
description: Classify a piece of work by type (decision, bulk, taste, review, analysis, loop) and route it to the right model or agent based on cost, intelligence, and taste tradeoffs, require a pass/fail bar before delegating, and gate every commit or push against standing invariants. Use at the kickoff of any non-trivial multi-step work, when deciding whether to delegate a task to a subagent or cheaper model, setting up a review or grading loop, or defining pre-commit quality gates for agent-driven work.
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
3. Classify the task by type — decision, bulk, taste, review, analysis, or
   loop — then route by the table in [Routing](#routing).
4. Before delegating anything, write the pass/fail test. No test, no
   delegation.
5. Send the delegate a full [kickoff package](#kickoff-package), not just a
   prompt.
6. Grade with a different model than the one that built it.
7. Before any commit/push, run [The gate](#the-gate).

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

Classify first — pick the dominant type, then route:

| type | what it covers | route |
|------|----------------|-------|
| **decision** | planning, architecture calls, synthesis, final judgments | your best-taste, best-judgment model — don't delegate the thinking itself |
| **bulk** | clear-spec implementation, migrations, mechanical refactors, data crunching | your cheapest capable model — spend it freely, that's the point of ranking cost |
| **taste** | UI, copy, API design — anything user-facing | a taste-ranked model, never your cheapest |
| **review** | grading a plan, diff, or implementation | a *different* model than the one that built it — same-model review inherits the same blind spots |
| **analysis** | token-hungry grunt work: log crunching, heavy codebase reads, browser driving | offload it; return only the conclusion, keep the raw dump out of the main context |
| **loop** | long-running iterate-until-bar work | orchestrator sets the bar and grades; a cheap model builds each iteration (see [Loops](#loops)) |

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

## Task granularity

Chunk work into separate tasks by concern, even when each piece is small. A
request listing several independent asks is several tasks, not one bundle to
build and ship together — "it's small" is a reason it's cheap to keep
separate, not a reason to merge it into something else.

- **Test: "would reverting this alone make sense?"** If yes, it's its own
  task/commit, regardless of size. Three trivial fixes are three
  build-grade-commit cycles, not one.
- **Bundling breaks bisectability and dilutes review.** One diff holding
  several unrelated changes can't be reverted piecemeal, and a review pass
  split across several concerns catches less on each than focused passes
  would.
- Reviewing/grading several small pieces together in one pass is fine — the
  *commits* still split by concern before anything ships.
- Applies inside a single delegation too: once a decision is locked, don't
  silently build every resulting piece as one lump. Split the build the same
  way you'd split the commits.

## Agent teams: share context, don't re-read

Deploying several agents against the same codebase without sharing what's
already been found means each one re-reads and re-searches the same ground
independently — tokens spent on rediscovery, not new work.

- **Scout once, fan out from the summary.** Have one agent read the relevant
  files first and hand the digest — paths, line numbers, excerpts — into
  every later agent's prompt. Agents should arrive with what they need, not
  re-derive it from scratch.
- **Reuse a shared session/context where your tooling supports it**, instead
  of spinning a fresh agent that re-reads everything from zero when it needs
  the same grounding you already have.
- **Pipeline over parallel-from-scratch** when work has a natural handoff:
  downstream agents should receive the *result* of the previous stage, not
  restart the search themselves.
- If your orchestrator's agents don't share context by default, thread the
  grounding through explicitly — a scout-stage result passed into every
  prompt — rather than repeating "go read X again" in each one.

## Bound every long-running task

Any delegated agent should be scoped to finish soon. An unattended one can
run away silently — this happened for real: a delegated task ran over 8
hours unattended, which is not acceptable. A check that never stops anything
is how that happens even with monitoring in place — the fix has to name the
kill, not just the look.

- **Scope for a short expected runtime up front.** The smaller and more
  concrete the task (see Task granularity above), the sooner it finishes and
  the less likely it ever needs the rules below.
- **Short calls: use your tool's own timeout.** Most shell/agent tools cap
  explicit timeouts at some ceiling (often around 10 minutes) — use it; that
  covers short calls outright.
- **Anything that could run longer: background it, then schedule a kill —
  not just a scheduled check.** Launch it in the background and set a
  check-in for the expected-duration deadline. If it's still running at that
  check-in, that's not "still working," it's stuck — a sandbox limitation, a
  retry loop, waiting on input it'll never get.
- **When it's stuck: stop it and start a fresh one — don't keep waiting on
  the same run.** Kill the stalled task, note briefly why it stalled so the
  retry avoids the same trap, and launch a new attempt rather than resuming
  or babysitting the original. A restarted task with a tighter scope usually
  finishes; the same stuck run rarely recovers on its own.
- **Long is fine; unattended is not.** A big migration can legitimately run
  for hours, but only as an intentional, monitored choice with a real
  stop-and-replace condition attached — never the default consequence of not
  setting a bound.

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

- **The bar** — the pass/fail test from [The bar](#the-bar).
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
