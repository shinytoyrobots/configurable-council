# Council configuration format

Every council, bundled or user-created, is one markdown file with this structure:

```markdown
# Council Configuration: {Name}

{One-line description of the council.}

## Theming

- **Report title**: {title}
- **Session verb**: {convened | summoned | assembled | …}
- **Report aesthetic**: {1-2 sentences describing the visual style of the HTML report}

## Chairman

### {Chairman Name}

{1-2 sentence description of the chairman.}

**Synthesis voice**: {1-2 sentences on how the chairman speaks and synthesizes.}

## Advisors

### {Advisor Name} — {Title}   (the " — {Title}" suffix is optional)

{1-2 sentence description.}

**Thinking style**: {2-3 sentences on how they approach problems.}

[repeat for each advisor]

## Tensions

- **{Name} vs {Name}** — {the productive disagreement between them}
[repeat for each tension pair]
```

## Validation rules

- At least 3 advisors (required: fewer cannot create meaningful tension).
- At most 11 advisors recommended (warn above that, but don't block).
- A chairman is required.
- Every advisor needs a name, a description, and a thinking style. A title is optional.
- Tensions are optional but recommended. If the user gives none, suggest some.
