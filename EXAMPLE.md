# Example — one way to fill this in

A real version of the pattern, generalized from a personal setup that mixes a
Claude subscription (with an internal reasoning/taste tier) and a separate
high-limit coding-assistant login. Swap in whatever your own fleet looks
like — the values below are illustrative, the *shape* is the point.

## Rankings

| model                          | cost | intelligence | taste |
|---------------------------------|------|--------------|-------|
| high-limit bulk coding model    | 9    | 8            | 5     |
| mid-tier reasoning model        | 5    | 5            | 7     |
| flagship reasoning model        | 4    | 7            | 8     |
| best-taste orchestrator model   | 2    | 9            | 9     |

The orchestrator is the smartest and most tasteful, so it keeps the
decisions. The bulk model is nearly as smart but lower-taste and effectively
unlimited, so it gets the mechanical execution. "Delegate down" means down in
taste, not down in intelligence — usually *sideways* to the bulk model, not
down to a weak one.

## Effort levels

- Orchestrator: highest effort is both the default and the ceiling — going
  higher is token-hungry with worse output, not better.
- Delegated mechanical work: low/medium effort.
- Hard verify/judge passes: high effort.
- Bulk coding model: leave its reasoning effort at default.

## Credits

This pattern was distilled after seeing
[Theo's (@theo) thread](https://x.com/theo/status/2072482460122964067?s=46)
on avoiding rate limits by fixing effort levels and delegating bulk work
sideways, and Matt Shumer's
["How I Prompt Fable"](https://simplemarkdowneditor.com/pub/IbaCrTjLJT?key=uQOQ2NPO3TTUSXyYDjyLf),
which independently arrived at the same shape: goals over steps, house-rule
boundaries, measurable success criteria, builder-never-grades loops, and
building on prior work.
