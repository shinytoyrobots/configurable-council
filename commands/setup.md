---
description: "Configure your LLM Council — choose a preset, customize members, or build from scratch. Use when: 'set up council', 'configure council', 'change council', 'switch council', 'new council', 'council setup'."
argument-hint: "<list|show|new|preset-name>"
---

# Council Setup

Configure which council of advisors will be used when the user runs `/council:run`. The active selection is stored as a pointer file `council-active.md`. Presets are stored as `council-preset-*.md` files.

This plugin keeps two locations separate:
- **Shipped presets** (read-only) live in `${CLAUDE_PLUGIN_ROOT}/presets/` and are distributed with the plugin (e.g. `council-preset-default.md`, `council-preset-elrond.md`).
- **Runtime state** (writable, persists across plugin updates) lives in `${CLAUDE_PLUGIN_DATA}/`. The active pointer (`council-active.md`) and any user-created or user-edited presets are written here — never into `${CLAUDE_PLUGIN_ROOT}`, which is replaced on update.

When resolving a preset by filename, always check `${CLAUDE_PLUGIN_DATA}/` first, then fall back to `${CLAUDE_PLUGIN_ROOT}/presets/`.

---

## Read Context

Read these at startup:
- `${CLAUDE_PLUGIN_DATA}/council-active.md` (current active council, if it exists; runtime state, may be absent on first run)
- Shipped presets: Glob `${CLAUDE_PLUGIN_ROOT}/presets/council-preset-*.md`
- User presets: Glob `${CLAUDE_PLUGIN_DATA}/council-preset-*.md`

The full set of available presets is the union of both Globs (a user preset with the same filename as a shipped one shadows it).

---

## Modes

Parse `$ARGUMENTS` to determine mode:

### `list` (or no arguments)

Show available presets and the currently active council.

1. Glob both `${CLAUDE_PLUGIN_ROOT}/presets/council-preset-*.md` and `${CLAUDE_PLUGIN_DATA}/council-preset-*.md`
2. For each preset, read the first line (the `# Council Configuration: ...` heading) and the description line beneath it
3. Read `${CLAUDE_PLUGIN_DATA}/council-active.md` and show its name (or "none configured" if absent)
4. Display as a clean list:

```
Active council: [name from council-active.md, or "none — run /council:setup <preset>"]

Available presets:
  default     — 5 advisors, general-purpose thinking styles
  elrond      — 11 advisors, Council of Elrond (Tolkien)
```

### `show`

Display the current active council configuration in detail.

1. Read `${CLAUDE_PLUGIN_DATA}/council-active.md`, resolve its `preset-file`, and load that preset (DATA first, then ROOT/presets)
2. Show: name, advisor count, chairman name, all advisor names with their one-line descriptions, and listed tensions

### `new`

Interactive creation of a new council from scratch.

1. **Ask for a name and theme.** Example: "Pirate Crew", "Spartan War Council", "Silicon Valley Board", "The Avengers". The theme shapes the character descriptions and report aesthetic.

2. **Ask how many advisors.** Recommend 5 as the sweet spot. Warn if under 3 ("the council needs at least 3 members for meaningful tension"). Allow up to 11. More than 11 is not recommended — say so, but don't hard-block.

3. **For each advisor, collect:**
   - Name (e.g., "Captain Ahab", "Spock", "The CFO")
   - Title or role (e.g., "The Obsessive", "The Logician", "The Numbers Person")
   - Description: 1-2 sentences grounding the character. Who are they? What shapes their worldview?
   - Thinking style: 2-3 sentences describing how they approach problems. What do they look for? What do they ignore? What question do they always ask?

4. **Ask who the chairman is.** Can be one of the advisors elevated to chairman (they still advise AND synthesize), or a separate figure who only synthesizes. Collect:
   - Name and description (if separate from advisors)
   - Synthesis voice: 1-2 sentences on how the chairman speaks and what they prioritize in synthesis

5. **Identify tensions.** Based on the advisors defined, propose 2-4 natural tensions (e.g., "X vs Y — optimism vs skepticism"). Ask the user to confirm or adjust.

6. **Set report theming (optional).** Ask if they want custom report styling. If yes, collect:
   - Report title (defaults to the council name)
   - Session verb (e.g., "convened", "summoned", "assembled" — defaults to "convened")
   - Report aesthetic (a sentence describing the visual feel, e.g., "Dark mode with neon accents, cyberpunk feel" or "Clean minimalist, lots of whitespace")
   - If they skip, use the default clean professional style

7. **Write the configuration.** Save the full council configuration as a preset in the runtime state directory: `${CLAUDE_PLUGIN_DATA}/council-preset-{slug}.md` where `{slug}` is a kebab-case version of the council name. Then write the 2-line pointer file `${CLAUDE_PLUGIN_DATA}/council-active.md`:
   ```
   active-preset: {slug}
   preset-file: council-preset-{slug}.md
   ```

8. **Confirm.** Show a summary of the council and confirm it's active.

### `<preset-name>` (e.g., `elrond`, `default`)

Activate a preset.

1. Resolve `council-preset-{preset-name}.md`: check `${CLAUDE_PLUGIN_DATA}/` first, then `${CLAUDE_PLUGIN_ROOT}/presets/`
2. If not found in either location, show available presets and ask the user to pick one
3. Show a summary: name, advisor count, advisor names
4. Ask: "Activate this council? Any changes you want first?"
5. If they want changes, write the edited preset content to `${CLAUDE_PLUGIN_DATA}/council-preset-{preset-name}.md` (never edit the shipped copy in `${CLAUDE_PLUGIN_ROOT}` — write the modified version into the DATA dir, where it shadows the shipped one)
6. Write the 2-line pointer file `${CLAUDE_PLUGIN_DATA}/council-active.md`:
   ```
   active-preset: {preset-name}
   preset-file: council-preset-{preset-name}.md
   ```
7. Confirm activation

---

## Council Configuration Format

All council configs — presets and active — use this structure:

```markdown
# Council Configuration: {Name}

{One-line description of the council.}

## Theming

- **Report title**: {title}
- **Session verb**: {verb}
- **Report aesthetic**: {1-2 sentences describing the visual style}

## Chairman

### {Chairman Name}

{1-2 sentence description of the chairman.}

**Synthesis voice**: {1-2 sentences on how the chairman speaks and synthesizes.}

## Advisors

### {Advisor Name} — {Title}

{1-2 sentence description.}

**Thinking style**: {2-3 sentences on how they approach problems.}

[repeat for each advisor]

## Tensions

- **{Name} vs {Name}** — {tension description}
[repeat for each tension pair]
```

---

## Validation Rules

- Minimum 3 advisors (hard requirement — fewer than 3 cannot create meaningful tension)
- Maximum recommended 11 (warn but don't block above this)
- Chairman is required (every council needs a synthesizer)
- Each advisor must have a name, description, and thinking style
- Tensions are optional but recommended — suggest them if the user doesn't provide any
