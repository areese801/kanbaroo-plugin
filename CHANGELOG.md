# Changelog

All notable changes to `kanbaroo-plugin` are recorded here. This
project follows [Semantic Versioning](https://semver.org/) and
[Keep a Changelog](https://keepachangelog.com/).

---

## [0.4.0] - 2026-05-08

### Added

- **`kanbaroo-iterate` skill.** The meta-pair to `kanbaroo-workflow`:
  workflow USES Kanbaroo on real work, iterate IMPROVES Kanbaroo
  based on what surfaces during use. Modeled on trusty-cage's
  `cage-iterate` (Steps 1–7 + rules & guardrails), with two
  Kanbaroo-specific additions:
  - **Surface Routing table** maps each friction type (skill text,
    MCP tools, REST API, CLI, TUI, web UI, audit, spec) to the right
    repo and a cage-or-direct decision.
  - **Restart matrix** in Step 6 documents what to bounce per fix
    type before re-test (Claude Code session, `docker compose
    restart`, web rebuild, etc.).
- README `Skills shipped` section + the activation-table row for
  `kanbaroo-iterate`.
- `CLAUDE.md` repo-purpose paragraph + layout tree updated to list
  three skills.
- `CLAUDE.md` Versioning + Release Process sections strengthened
  with the "always bump on skill changes (the cache keys on
  version)" rule and a paired CHANGELOG-in-same-PR convention,
  matching `trusty-cage-plugin`'s established pattern.

## [0.3.0] - 2026-05-06

### Added

- Initial extract from the `kanbaroo` monorepo at v0.3.0. Two
  skills shipped:
  - **`kanbaroo-workflow`** — teaches outer Claude how to drive a
    Kanbaroo instance via its MCP server.
  - **`kanbaroo-cage-bridge`** — augments trusty-cage's
    `cage-orchestrator` flow when Kanbaroo MCP is also active in
    the session.
- Marketplace + plugin-subdir layout (`.claude-plugin/marketplace.json`
  at repo root, plugin content under `plugins/kanbaroo/`) so
  `/plugin marketplace add areese801/kanbaroo-plugin` and
  `/plugin install kanbaroo@kanbaroo-plugin` work.
- `CLAUDE.md` aimed at developers working ON the plugin (distinct
  from the SKILL.md files which guide outer Claude when USING the
  plugin).
