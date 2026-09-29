---
name: council
description: Runs a question, idea, or decision through a configurable council of AI advisors who analyze it independently, peer-review each other anonymously, and have a chairman synthesize a final verdict (Karpathy's LLM Council method). Use when the user says "council this", "run the council", "convene the council", "summon the council", "pressure-test this", "stress-test this", "debate this", or "war room this", or presents a genuine decision with stakes and multiple options ("should I X or Y", "which option", "is this the right move", "I'm torn between") and wants it examined from several angles. Not for simple yes/no questions, factual lookups, or a casual "should I" without a real tradeoff.
license: MIT
compatibility: Designed for agents that can spawn parallel sub-agents (Claude Code, Grok Build). Needs file read/write for configuration, reports, and transcripts.
metadata:
  author: Robin Cannon
  version: "0.2.0"
---

# Council

Run a question through a configurable council of AI advisors using Karpathy's LLM Council methodology. The council's composition comes from configuration; the methodology is fixed.

Karpathy dispatches a query to multiple models, has them peer-review each other anonymously, then a chairman produces the final answer. This skill does the same thing with sub-agents that use different thinking lenses instead of different models.

## Load the active council

Paths:
- **Bundled presets** (read-only): `presets/` in this skill's directory — `default.md`, `elrond.md`.
- **User state**: `${XDG_CONFIG_HOME:-$HOME/.config}/configurable-council/` — resolve it to an absolute path first. It holds `active.md` (the active-council pointer) and `presets/{slug}.md` (user-created or user-edited councils).

Resolution:
1. Read `active.md` from the user state directory and take the `active-preset` value (e.g. `active-preset: elrond`).
2. If `active.md` is missing or empty, use `default` and tell the user once: "Using the default council. Run the `council-setup` skill to choose or build another."
3. Load `{slug}.md` from the user state `presets/` directory first; fall back to the bundled `presets/` directory. If neither exists, list what is available and ask the user to pick.

Parse from the preset:
- **Council name** — the `# Council Configuration: {Name}` heading
- **Theming** — report title, session verb, report aesthetic
- **Chairman** — name, description, synthesis voice
- **Advisors** — name and title (from `### Name — Title`; title may be absent), description, thinking style
- **Tensions** — optional pairs that guide the chairman's synthesis
- **Advisor count** — the number of `### ` headings under `## Advisors`
- **Peer reviewer count** — `min(advisor_count, 7)`

## Step 1: Frame the question

**A. Gather context.** The question is often the tip of the iceberg. Spend no more than about 30 seconds finding the 2-3 files that would make advice specific rather than generic:
- `CLAUDE.md`, `AGENTS.md`, or similar project instruction files
- Any `memory/` folder
- Files the user referenced or attached
- Recent `council-transcript-*.md` files in the working directory (avoid re-counciling the same ground)

**B. Frame it.** Rewrite the user's question plus that context as one clear, neutral prompt every advisor receives, covering:
1. The core decision or question
2. Key context from the user's message
3. Key context from workspace files (stage, audience, constraints, past results, relevant numbers)
4. What is at stake

Do not add an opinion or steer. If the question is too vague, ask exactly one clarifying question, then proceed. Keep the framed question for the transcript.

## Step 2: Convene the council (parallel)

Spawn every advisor as a sub-agent at the same time. If sub-agents are unavailable, write each advisor's response in turn, finishing one before reading the next. Each response is 150-300 words.

```
You are {advisor.name}, {advisor.title}, on the {council.name}.

Your perspective: {advisor.description}
Your thinking style: {advisor.thinking_style}

A matter has been brought before the council:
---
{framed_question}
---

Speak from your perspective. Be direct and specific. Do not hedge or try to be balanced — the other members of the council will cover the angles you do not. Lean fully into your assigned perspective.

Keep your response between 150-300 words. No preamble. Go straight into your counsel.
```

If an advisor has no title, omit `, {advisor.title}`.

## Step 3: Anonymous peer review (parallel)

This step is what makes the council more than "ask N times."

Label the responses Response A, B, C… with a **random** advisor-to-letter mapping so there is no positional bias. Record the mapping for the transcript. Spawn `min(advisor_count, 7)` reviewers, each seeing all anonymized responses:

```
You are reviewing the outputs of a council of advisors. {advisor_count} advisors independently answered this question:
---
{framed_question}
---

Here are their anonymized responses:

**Response A:**
{response}

**Response B:**
{response}

[...continue for all responses...]

Answer these three questions. Be specific. Reference responses by letter.

1. Which response is the strongest? Why?
2. Which response has the biggest blind spot? What is it missing?
3. What did ALL responses miss that the council should consider?

Keep your review under 250 words. Be direct.
```

## Step 4: Chairman synthesis

One agent receives the framed question, all advisor responses (de-anonymized), all peer reviews, and the anonymization map, so the chairman can match letter-based feedback to each advisor. The chairman may disagree with the majority if the reasoning supports it, and uses the council's Tensions to find productive disagreement.

```
You are {chairman.name}, presiding over the {council.name}.

{chairman.description}

Your job is to synthesize the counsel of {advisor_count} advisors and their peer reviews into a final verdict.

The matter brought before the council:
---
{framed_question}
---

COUNSEL OF THE COUNCIL:

{for each advisor:}
**{advisor.name} ({advisor.title}):**
{response}

PEER REVIEWS (these refer to responses by letter):
{all peer reviews}

ANONYMIZATION MAP:
{Response A → advisor.name, Response B → advisor.name, …}

TENSIONS TO EXAMINE:
{tensions, or "none listed"}

{chairman.synthesis_voice}

Produce the council verdict using this exact structure:

## Where the Council Agrees
[Points multiple advisors converged on independently. These are high-confidence signals.]

## Where the Council Clashes
[Genuine disagreements. Present both sides. Explain why reasonable advisors disagree.]

## Blind Spots the Council Caught
[Things that only emerged through peer review. Things individual advisors missed that others flagged.]

## The Recommendation
[A clear, direct recommendation. Not "it depends." A real answer with reasoning.]

## The One Thing to Do First
[A single concrete next step. Not a list. One thing.]
```

## Step 5: Write the report and transcript

Show the chairman's verdict to the user, then write two files to the current working directory, using one shared timestamp (`YYYYMMDD-HHMMSS`):

```
council-report-{timestamp}.html    # visual report for scanning
council-transcript-{timestamp}.md  # full record for reference
```

Read [references/report-spec.md](references/report-spec.md) for what each file must contain. Open the HTML report when done (`open` on macOS, `xdg-open` on Linux) if the environment allows it.
