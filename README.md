# orchestrate-work

A skill for deciding *who* does a piece of work before doing it — route by
cost/intelligence/taste, require a pass/fail bar before delegating, grade
with a different model than the one that built it, and gate every commit
against standing invariants. Installable as a plugin in [Claude Code](https://claude.com/claude-code) and
Codex CLI, as a standalone skill via `npx skills add`, and readable as plain
instructions by anything that supports `AGENTS.md`.

## Install

**Claude Code:**

```
/plugin marketplace add udhaya21/orchestrate-work
/plugin install orchestrate-work@orchestrate-work
```

**Codex CLI:**

```
codex plugin marketplace add udhaya21/orchestrate-work
codex plugin add orchestrate-work@orchestrate-work
```

**Any agent, via the [`skills` CLI](https://skills.sh):**

```
npx skills add udhaya21/orchestrate-work
```

Installs the skill straight into whichever agents you pick (Claude Code,
Codex, Cursor, and others) — no plugin machinery needed.

**Cursor, Aider, Windsurf, Gemini CLI, GitHub Copilot, or anything else that
reads [`AGENTS.md`](AGENTS.md):** copy or symlink `AGENTS.md` into your
project root, or paste its contents into your tool's instructions file.

Once installed, fill in your own model rankings in the table — see
[`plugins/orchestrate-work/skills/orchestrate-work/EXAMPLE.md`](plugins/orchestrate-work/skills/orchestrate-work/EXAMPLE.md)
for a worked version.

## What's in here

- `AGENTS.md` — the pattern as plain, tool-agnostic instructions.
- `.claude-plugin/marketplace.json` — self-hosts this repo as a Claude Code
  marketplace.
- `.agents/plugins/marketplace.json` — self-hosts this repo as a Codex CLI
  marketplace.
- `plugins/orchestrate-work/` — the shared plugin directory both marketplaces
  point at:
  - `.claude-plugin/plugin.json` / `.codex-plugin/plugin.json` — per-tool
    plugin manifests, same underlying skill.
  - `skills/orchestrate-work/SKILL.md` — the pattern itself: rank your
    fleet, route by the table, require a bar, grade independently, gate
    before shipping.
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
