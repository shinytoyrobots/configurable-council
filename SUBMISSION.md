# Submission checklist

Run both validators before submitting (see README → Development). Push to GitHub first. Both listings point at the public repo.

## Anthropic: Claude plugin directory

1. `claude plugin validate --strict .` must pass.
2. Submit at <https://claude.ai/directory/manage> (requires a paid claude.ai plan). Check the pre-submission checklist at <https://claude.com/docs/plugins/pre-submission-checklist>.
3. Repo: `https://github.com/shinytoyrobots/configurable-council`, plugin `council`.

Note: `anthropics/skills` is a reference and examples repo, not a submission queue.

## xAI: Grok Build plugin marketplace

1. Get the commit to pin (full 40-character SHA): `git ls-remote https://github.com/shinytoyrobots/configurable-council.git HEAD`
2. Fork `xai-org/plugin-marketplace` and append this entry to `plugins` in `.grok-plugin/marketplace.json`:

   ```json
   {
     "name": "council",
     "description": "Run any decision through a configurable council of AI advisors who analyze independently, peer-review anonymously, and synthesize a chairman's verdict.",
     "category": "productivity",
     "source": {
       "source": "url",
       "url": "https://github.com/shinytoyrobots/configurable-council.git",
       "sha": "<40-char SHA from step 1>"
     },
     "homepage": "https://github.com/shinytoyrobots/configurable-council",
     "keywords": ["council", "llm-council", "decision-making", "peer-review"],
     "version": "0.2.0",
     "author": { "name": "Robin Cannon" }
   }
   ```

3. `python3 scripts/generate-plugin-index.py && python3 scripts/validate-catalog.py`
4. Open a PR. Code-owner review is required.
5. For each release, bump `sha` (and `version`) in that entry.

## Cross-agent registries

Standalone installs work through `npx skills add shinytoyrobots/configurable-council` and by copying `skills/*` into `~/.agents/skills/`. Directories such as agentskills.io, LobeHub, and mdskills.ai index public GitHub skill repos, and some also accept manual submission.
