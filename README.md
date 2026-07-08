# orchestrate-work

A [Claude Code](https://claude.com/claude-code) skill for deciding *who* does
a piece of work before doing it — route by cost/intelligence/taste, require a
pass/fail bar before delegating, grade with a different model than the one
that built it, and gate every commit against standing invariants.

## Install

This repo is a self-hosted marketplace containing one plugin. From inside
Claude Code:

```
/plugin marketplace add udhaya21/orchestrate-work
/plugin install orchestrate-work@orchestrate-work
```

Once installed, fill in your own model rankings in `SKILL.md`'s table — see
`EXAMPLE.md` for a worked version.

## What's in here

- `.claude-plugin/plugin.json` — the plugin manifest.
- `.claude-plugin/marketplace.json` — self-hosts this repo as its own
  marketplace listing.
- `skills/orchestrate-work/SKILL.md` — the pattern itself: rank your fleet,
  route by the table, require a bar, grade independently, gate before
  shipping.
- `skills/orchestrate-work/REFERENCE.md` — a starter set of standing
  invariants to adapt.
- `skills/orchestrate-work/EXAMPLE.md` — a real filled-in example, with
  credits.

## Credits

Distilled from [Theo's (@theo)](https://x.com/theo/status/2072482460122964067?s=46)
rate-limit thread and Matt Shumer's
["How I Prompt Fable"](https://simplemarkdowneditor.com/pub/IbaCrTjLJT?key=uQOQ2NPO3TTUSXyYDjyLf).

## License

MIT — see [LICENSE](LICENSE).
