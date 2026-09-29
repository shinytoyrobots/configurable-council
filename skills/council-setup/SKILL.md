---
name: council-setup
description: Configures the council used by the council skill — lists and activates presets, shows the active council, customizes a preset, or builds a new council from scratch. Use when the user says "set up council", "configure council", "change council", "switch council", "new council", "council setup", or names a preset to activate (e.g. "use the elrond council").
license: MIT
compatibility: Companion to the council skill; expects it installed alongside (../council/). Needs file read/write.
metadata:
  author: Robin Cannon
  version: "0.2.0"
---

# Council Setup

Choose, inspect, customize, or create the council the `council` skill convenes.

## Paths

- **Bundled presets** (read-only): `../council/presets/` relative to this skill's directory — the sibling `council` skill ships `default.md` and `elrond.md`. Never edit them.
- **User state**: `${XDG_CONFIG_HOME:-$HOME/.config}/configurable-council/` — resolve it to an absolute path and create it if missing. It holds:
  - `active.md` — the one-line pointer `active-preset: {slug}`
  - `presets/{slug}.md` — user-created or user-edited councils

Available presets are the union of both preset directories; a user preset with the same slug shadows the bundled one. Always resolve a slug in user state first, then in the bundled presets.

## Modes

Choose the mode from the user's request or arguments.

### `list` (default when nothing else is specified)

1. List `*.md` in both preset directories.
2. For each preset, read the `# Council Configuration: …` heading and the description line under it.
3. Read `active.md`. If it is absent, the active council is `default`.
4. Display:

```
Active council: {name} ({slug})

Available presets:
  default     — 5 advisors, general-purpose thinking styles
  elrond      — 11 advisors, Council of Elrond (Tolkien)
```

### `show`

Resolve and load the active preset. Show its name, advisor count, chairman, every advisor with a one-line description, and its tensions.

### `new`

Build a council interactively.

1. **Name and theme** (e.g. "Pirate Crew", "Spartan War Council", "Startup Board"). The theme shapes the characters and the report aesthetic.
2. **Advisor count.** Recommend 5. Require at least 3; say fewer cannot create meaningful tension. Warn above 11, but don't block.
3. **Each advisor:** name, title or role, a 1-2 sentence description (who they are, what shapes their worldview), and a 2-3 sentence thinking style (what they look for, what they ignore, the question they always ask).
4. **Chairman:** either an advisor elevated to chairman (they advise and synthesize) or a separate figure who only synthesizes. Collect a name, a description if separate, and a 1-2 sentence synthesis voice.
5. **Tensions:** propose 2-4 natural pairs (e.g. "X vs Y — optimism vs skepticism") and let the user confirm or adjust.
6. **Theming (optional):** report title (defaults to the council name), session verb (defaults to "convened"), and report aesthetic (defaults to clean and professional).
7. **Write** the council to `presets/{slug}.md` in user state (`{slug}` is the kebab-case name), following [references/config-format.md](references/config-format.md). Then write `active.md` containing `active-preset: {slug}`.
8. **Confirm** with a summary and state that the council is active.

### `{preset-name}` (e.g. `elrond`, `default`)

1. Resolve the slug (user state first, then bundled). If it isn't found, show `list` and ask the user to pick.
2. Summarize the council: its name, advisor count, and advisor names.
3. Ask: "Activate this council? Any changes you want first?"
4. If they want changes, write the edited version to `presets/{preset-name}.md` in user state. It shadows the bundled copy, which is never modified.
5. Write `active.md` containing `active-preset: {preset-name}` and confirm.

## Validation

Check every council you write against the rules in [references/config-format.md](references/config-format.md) before you save it.
