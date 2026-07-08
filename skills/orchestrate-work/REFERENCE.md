# Reference — standing invariants

A starter set of gate rules to adapt to your own fleet and risk tolerance.
These are meant to hold no matter how the goal is reached — when a rule here
conflicts with speed or convenience, the rule wins.

1. **No delegation without a bar.** Every piece of delegated work carries a
   concrete pass/fail test declared before the build. Adjectives ("high
   quality", "clean", "polished") are not bars.
2. **The builder never grades its own work.** Grading goes to a different
   model, in a fresh context, pointed at the real output (pixels, running
   app, diff), mandated to prove FAIL.
3. **No hard-coded special cases in agent behavior.** Describe the desired
   behavior in the system prompt and let the model reason — don't bolt on
   regex filters or if-chains for individual cases.
4. **Taste-critical output never ships from your cheapest or lowest-taste
   model.** UI, copy, and API design come from a taste-ranked model.
5. **Set a floor model you never go below**, regardless of cost pressure, for
   anything user-facing or safety-relevant.
6. **Escalate sideways before up.** Don't reach for your most expensive or
   scarce model when a cheaper one would clear the bar.
7. **Report reality.** Failing tests, skipped steps, and unverified claims
   get stated plainly — no rounding up to "done".
8. **Nothing sensitive leaves the machine.** External services get progress
   notes and screenshots — never secrets, credentials, or proprietary
   source.

## The gate, in detail

Before any commit or push: run a rule-check pass — a different model than
whoever wrote the change, at your highest reasoning effort — whose only job
is to check the diff against the rules above and the declared bar. It
returns PASS or a list of violations. Violations block the push until fixed,
or until explicitly waived.
