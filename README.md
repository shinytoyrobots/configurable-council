<img src="assets/icon.png" alt="" width="128" align="right">

# Configurable Council

An [Agent Skills](https://agentskills.io) package (and [Claude Code](https://claude.com/claude-code) / [Grok Build](https://docs.x.ai/build/features/skills-plugins-marketplaces) plugin) that runs any question, idea, or decision through a **configurable council of AI advisors**. The advisors independently analyze the question, peer-review each other's answers *anonymously*, and a chairman synthesizes a single, decisive verdict.

It's adapted from [Andrej Karpathy's LLM Council](https://github.com/karpathy/llm-council): where Karpathy dispatches a query to multiple different models and has them critique each other before a chairman model produces the final answer, this runs the same protocol *inside your coding agent* — using parallel sub-agents with distinct thinking lenses instead of distinct models. The composition of the council is fully configurable; the methodology is fixed.

## What you get

Two skills:

| Skill | Claude Code plugin | Standalone / Grok | What it does |
|---|---|---|---|
| `council` | `/council:council` | `/council` | Convenes the active council on a question. Produces a chairman's verdict, a self-contained HTML report, and a full markdown transcript. The skill also triggers on its own when you say "council this", "pressure-test this", and similar phrases. |
| `council-setup` | `/council:council-setup` | `/council-setup` | Lists, activates, shows, customizes, or creates councils. |

Two bundled presets:

- **`default`**: 5 general-purpose advisors (the Contrarian, the First Principles Thinker, the Expansionist, the Outsider, the Executor) with a neutral Chairman. It suits any decision and is used when no council has been chosen.
- **`elrond`**: 11 advisors drawn from Tolkien's Council of Elrond, chaired by Elrond himself. Same rigor, more fun.

You can create any number of custom councils (a pirate crew, a startup board, a Spartan war council, whatever framing helps you think) with `council-setup new`.

## The method

Each council session runs five steps:

1. **Frame** — your raw question is enriched with workspace context (`CLAUDE.md`, referenced files, etc.) and restated as a neutral prompt every advisor receives.
2. **Convene** — all advisors answer in parallel, each leaning fully into their assigned perspective (no hedging, no false balance).
3. **Peer review (anonymized)** — advisors' answers are shuffled into anonymous "Response A/B/C…" and re-reviewed. This is Karpathy's key insight: models are surprisingly good at judging answers they can't attribute. It surfaces blind spots no single advisor caught.
4. **Chairman synthesis** — one agent weighs everything and produces a verdict: where the council agrees, where it clashes, blind spots caught in review, a real recommendation, and the one thing to do first.
5. **Report** — a self-contained HTML report and a full markdown transcript are written to your current directory.

## Install

### Claude Code (plugin)

```
/plugin marketplace add shinytoyrobots/configurable-council
/plugin install council@configurable-council
```

Then:

```
/council:council-setup elrond
/council:council should we rewrite the billing service in Go or stay on Node?
```

### Grok Build (plugin)

Grok Build reads Claude Code marketplaces and plugins without extra configuration. Once the plugin is listed in the [xAI plugin marketplace](https://github.com/xai-org/plugin-marketplace), install it from `/plugins`. Until then, add this repo as a marketplace source in `~/.grok/config.toml`.

### Any Agent Skills–compatible agent (standalone)

Copy or symlink **both** skill folders side by side into your agent's skills directory (e.g. `~/.agents/skills/`, `~/.claude/skills/`, or `~/.grok/skills/`):

```
npx skills add shinytoyrobots/configurable-council
# or
git clone https://github.com/shinytoyrobots/configurable-council
cp -R configurable-council/skills/council configurable-council/skills/council-setup ~/.agents/skills/
```

`council-setup` reads the bundled presets from its sibling `../council/presets/`, so keep the two folders together.

## Configuration & where things are stored

- **Bundled presets** are read-only and live in `skills/council/presets/`.
- **Your state** lives in `${XDG_CONFIG_HOME:-~/.config}/configurable-council/`: `active.md` (the active-council pointer) and `presets/` (councils you create or customize). It sits outside every agent's plugin directory, so it survives plugin updates and is shared across Claude Code, Grok, and any other agent.

When a preset is resolved, your `presets/` folder is checked first, so a customized copy shadows the bundled one without overwriting it.

### Council configuration format

Every council is a single markdown file:

```markdown
# Council Configuration: {Name}

{One-line description.}

## Theming
- **Report title**: {title}
- **Session verb**: {convened | summoned | assembled | …}
- **Report aesthetic**: {a sentence describing the visual style of the HTML report}

## Chairman
### {Chairman Name}
{Description.}
**Synthesis voice**: {How the chairman speaks and synthesizes.}

## Advisors
### {Advisor Name} — {Title}
{Description.}
**Thinking style**: {How they approach problems — what they look for, what they ignore.}

[repeat for each advisor]

## Tensions
- **{Name} vs {Name}** — {the productive disagreement between them}
```

**Rules:** minimum 3 advisors (fewer can't create meaningful tension), maximum recommended 11, a chairman is required, and tensions are optional but recommended.

## Development

```
claude plugin validate --strict .     # Claude Code plugin + marketplace manifests
pip install skills-ref && agentskills validate skills/council && agentskills validate skills/council-setup
```

CI runs both validators on every push. For listing steps, see [SUBMISSION.md](./SUBMISSION.md).

## Credit

The methodology is Andrej Karpathy's [LLM Council](https://github.com/karpathy/llm-council). This project is an independent adaptation and is not affiliated with or endorsed by him.

## License

[MIT](./LICENSE)
