# Changelog

## 0.2.0 — 2026-09-29

### Changed
- Migrated `commands/` to [Agent Skills](https://agentskills.io) format: `skills/council/SKILL.md` and `skills/council-setup/SKILL.md`. Commands are now `/council:council` and `/council:council-setup` (previously `/council:run` and `/council:setup`).
- Bundled presets moved into the skill, at `skills/council/presets/{default,elrond}.md`.
- User state moved from `${CLAUDE_PLUGIN_DATA}` to `${XDG_CONFIG_HOME:-~/.config}/configurable-council/`, so it works in Grok Build and in standalone skill installs and is shared across agents. The pointer is now a single line: `active-preset: {slug}`.
- If no council is active, the `default` preset is used rather than stopping.
- Report and transcript details moved to `skills/council/references/report-spec.md`, so they load only when needed.

### Added
- Marketplace description and full plugin entry metadata.
- CI that runs `claude plugin validate --strict` and the Agent Skills reference validator.

## 0.1.0

- Initial release.
