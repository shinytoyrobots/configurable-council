# Council report and transcript spec

## HTML report — `council-report-{timestamp}.html`

A single self-contained HTML file with inline CSS and no external requests. Style it according to the council's **Report aesthetic**. If none is given, use a white background, subtle borders, a readable sans-serif font, and soft accent colors.

Contents, in order:
1. **Report title** (from the council's Theming), with the question beneath it.
2. **The chairman's verdict**, displayed prominently. Most readers read only this.
3. **An agreement/disagreement visual**: advisors grouped by position, each with their name and a one-line stance. Keep it clean and scannable.
4. **One collapsible section per advisor** holding their full response, collapsed by default (`<details>`), each with a subtle color accent or icon.
5. **A collapsible section of peer-review highlights.**
6. **A footer** giving the timestamp, the council name, the session verb (e.g. "Convened 2026-09-29 14:02"), and the question.

## Transcript — `council-transcript-{timestamp}.md`

Include:
- The original question, word for word
- The framed question
- Every advisor response, labeled by advisor name and title
- Every peer review, followed by the anonymization mapping (Response A → advisor name, …)
- The chairman's full synthesis

The transcript is the lasting artifact. If the user reruns the council after making changes, the earlier transcript shows how the thinking evolved.
