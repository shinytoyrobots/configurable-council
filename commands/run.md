---
description: "Run any question, idea, or decision through your configured council of AI advisors who independently analyze it, peer-review each other anonymously, and synthesize a final verdict. Based on Karpathy's LLM Council methodology. MANDATORY TRIGGERS: 'council this', 'run the council', 'summon the council', 'convene the council', 'pressure-test this', 'stress-test this', 'debate this', 'war room this'. STRONG TRIGGERS (use when combined with a real decision or tradeoff): 'should I X or Y', 'which option', 'what would you do', 'is this the right move', 'validate this', 'get multiple perspectives', 'I can't decide', 'I'm torn between'. Do NOT trigger on simple yes/no questions, factual lookups, or casual 'should I' without a meaningful tradeoff. DO trigger when the user presents a genuine decision with stakes, multiple options, and context that suggests they want it pressure-tested from multiple angles."
---

# Council

Run a question through a configurable council of AI advisors using Karpathy's LLM Council methodology. The council composition is read from configuration; the methodology is fixed.

This is adapted from Andrej Karpathy's LLM Council. He dispatches queries to multiple models, has them peer-review each other anonymously, then a chairman produces the final answer. We do the same thing inside Claude using sub-agents with different thinking lenses instead of different models.

---

## Read Context

At the start of every session, resolve the active council configuration:

1. Read the active-council pointer `council-active.md` from the runtime state directory `${CLAUDE_PLUGIN_DATA}` (this file is written by `/council:setup` and persists across plugin updates).
2. If the file does not exist or is empty, tell the user: "No council is configured. Run `/council:setup` to choose or create a council." Then stop.
3. Parse the `preset-file` value from the YAML pointer (e.g. `preset-file: council-preset-default.md`).
4. Load the referenced preset by resolving it in this order (first match wins):
   - `${CLAUDE_PLUGIN_DATA}/{preset-file}` — a user-created or user-edited preset
   - `${CLAUDE_PLUGIN_ROOT}/presets/{preset-file}` — a preset shipped with the plugin

Parse from the loaded preset:
- **Council name** — from the `# Council Configuration: {Name}` heading
- **Theming** — report title, session verb, report aesthetic
- **Chairman** — name, description, synthesis voice
- **Advisors** — each advisor's name, title (from the `### Name — Title` heading), description, and thinking style
- **Tensions** — the listed tension pairs (optional, used to guide chairman synthesis)
- **Advisor count** — derived from the number of `### ` headings under `## Advisors`
- **Peer reviewer count** — `min(advisor_count, 7)`

---

## How a Council Session Works

### Step 1: Frame the Question (with context enrichment)

When the user triggers the council, do two things before framing:

**A. Scan the workspace for context.** The user's question is often just the tip of the iceberg. Before framing, quickly scan for and read any relevant context files:
- `CLAUDE.md` or `claude.md` in the project root or workspace
- Any `memory/` folder
- Any files the user explicitly referenced or attached
- Recent council transcripts in the current folder (to avoid re-counciling the same ground)
- Any other context files relevant to the specific question

Use `Glob` and quick `Read` calls to find these. Don't spend more than 30 seconds. You're looking for the 2-3 files that would give advisors context for specific, grounded advice instead of generic takes.

**B. Frame the question.** Take the user's raw question AND the enriched context and reframe it as a clear, neutral prompt that all advisors will receive. The framed question should include:
1. The core decision or question
2. Key context from the user's message
3. Key context from workspace files (business stage, audience, constraints, past results, relevant numbers)
4. What's at stake (why this decision matters)

Don't add your own opinion. Don't steer it. But DO make sure each advisor has enough context to give a specific, grounded answer rather than generic advice.

If the question is too vague, ask one clarifying question. Just one. Then proceed.

Save the framed question for the transcript.

### Step 2: Convene the Council (N sub-agents in parallel)

Spawn all advisors simultaneously as sub-agents. Each gets their identity and the framed question.

Each advisor should produce a response of 150-300 words. Long enough to be substantive, short enough to be scannable.

**Sub-agent prompt template:**
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

For advisors that have a title (parsed from `### Name — Title`), use the title. If no title, just use the name.

### Step 3: Peer Review (anonymized, min(N, 7) sub-agents in parallel)

This is the step that makes the council more than just "ask N times." It's the core of Karpathy's insight.

Collect all advisor responses. Anonymize them as Response A, B, C, etc. (randomize which advisor maps to which letter so there's no positional bias).

Spawn `min(advisor_count, 7)` peer reviewers. Each sees all anonymized responses and answers three questions:
1. Which response is the strongest and why? (pick one)
2. Which response has the biggest blind spot and what is it?
3. What did ALL responses miss that the council should consider?

**Reviewer prompt template:**
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

### Step 4: Chairman Synthesis

One agent receives everything: the original question, all advisor responses (now de-anonymized), and all peer reviews.

The chairman's job is to produce the final council verdict using this structure:

1. **Where the Council Agrees** — points multiple advisors converged on independently. High-confidence signals.
2. **Where the Council Clashes** — genuine disagreements. Don't smooth these over. Present both sides and explain why reasonable advisors disagree. Use the Tensions from the config to guide where to look for productive disagreement.
3. **Blind Spots the Council Caught** — things that only emerged through peer review. Things individual advisors missed that others flagged.
4. **The Recommendation** — a clear, actionable recommendation. Not "it depends." Not "consider both sides." A real answer. The chairman can disagree with the majority if the reasoning supports it.
5. **The One Thing to Do First** — a single concrete next step. Not a list. One thing.

**Chairman prompt template:**
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

PEER REVIEWS:
{all peer reviews}

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

### Step 5: Generate the Council Report

After the chairman synthesis is complete, generate a visual HTML report.

**File:** `council-report-{timestamp}.html` (written to the current working directory)

The report should be a single self-contained HTML file with inline CSS. The styling should follow the **Report aesthetic** from the council config. If no aesthetic is specified, use the default: white background, subtle borders, readable sans-serif font, soft accent colors.

The report should contain:
1. **The question** at the top, beneath the **Report title** from config
2. **The chairman's verdict** prominently displayed (this is what most people will read)
3. **An agreement/disagreement visual** — a simple visual showing which advisors aligned and which diverged. Group by position with advisor names and their one-line stance. Keep it clean and scannable.
4. **Collapsible sections** for each advisor's full response (collapsed by default). Use a subtle icon or color accent per advisor.
5. **Collapsible section** for the peer review highlights
6. **A footer** showing the timestamp and what was counciled

Open the HTML file after generating it so the user can see it immediately.

### Step 6: Save the Full Transcript

Save the complete transcript as `council-transcript-{timestamp}.md` (in the current working directory). This includes:
- The original question
- The framed question
- All advisor responses (labeled with advisor names)
- All peer reviews (with anonymization mapping revealed)
- The chairman's full synthesis

This transcript is the artifact. If the user wants to run the council again on the same question after making changes, having the previous transcript lets them see how the thinking evolved.

---

## Output Format

Every council session produces two files:
```
council-report-{timestamp}.html    # visual report for scanning
council-transcript-{timestamp}.md  # full transcript for reference
```
