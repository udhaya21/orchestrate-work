# orchestrate-work

A [Claude Code](https://claude.com/claude-code) skill for deciding *who* does
a piece of work before doing it — route by cost/intelligence/taste, require a
pass/fail bar before delegating, grade with a different model than the one
that built it, and gate every commit against standing invariants.

## Install

Clone into your skills directory:

```bash
git clone https://github.com/udhaya21/orchestrate-work ~/.claude/skills/orchestrate-work
```

Claude Code picks up `SKILL.md` automatically. Fill in your own model
rankings in its table — see [EXAMPLE.md](EXAMPLE.md) for a worked version.

## What's in here

- `SKILL.md` — the pattern itself: rank your fleet, route by the table,
  require a bar, grade independently, gate before shipping.
- `REFERENCE.md` — a starter set of standing invariants to adapt.
- `EXAMPLE.md` — a real filled-in example, with credits.

## Credits

Distilled from [Theo's (@theo)](https://x.com/theo/status/2072482460122964067?s=46)
rate-limit thread and Matt Shumer's
["How I Prompt Fable"](https://simplemarkdowneditor.com/pub/IbaCrTjLJT?key=uQOQ2NPO3TTUSXyYDjyLf).

## License

MIT — see [LICENSE](LICENSE).
