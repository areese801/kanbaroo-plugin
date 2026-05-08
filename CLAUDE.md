# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working
**on this repo** — i.e. editing the skills, the plugin manifest, or the
README. For guidance on using the skills *from* a Claude Code session
(once installed), look at each skill's `SKILL.md` directly; those are
the runtime guidance for outer Claude.

## Repo Purpose

This repo ships a single Claude Code plugin: `kanbaroo`. The plugin
bundles three agent skills:

- `kanbaroo-workflow` — teaches outer Claude how to drive a Kanbaroo
  instance via its MCP server (read the board, create / comment /
  transition stories, surface "what's next").
- `kanbaroo-cage-bridge` — augments [`trusty-cage`](https://github.com/areese801/trusty-cage)'s
  `cage-orchestrator` skill flow when both Kanbaroo and trusty-cage are
  active in the same session. Mirrors cage dispatches onto a story.
- `kanbaroo-iterate` — continuous improvement loop for Kanbaroo
  itself. The meta-pair to `kanbaroo-workflow`: workflow USES
  Kanbaroo, iterate IMPROVES it based on what surfaces during use.
  Models on `cage-iterate`'s structure (Steps 1-7 + rules &
  guardrails) but with a Surface Routing table that maps each
  friction type to the right repo + cage-or-direct path.

This repo is **markdown + JSON only** — there is no Python, no build,
no test runner. The plugin is the SKILL.md files plus the manifest.

## Two Adjacent Repos to Keep in Mind

The skills here reference tools / commands that live in two other
repos. When you change something in one of those, you may need a
paired change here.

| Adjacent repo | What lives there | When it affects this repo |
|---|---|---|
| [`areese801/kanbaroo`](https://github.com/areese801/kanbaroo) | The Kanbaroo Python packages, including `kanbaroo-mcp` (the MCP server the workflow skill talks to) | If MCP tool names, signatures, or surface change, the SKILL.md text needs to track those changes. |
| [`areese801/trusty-cage-plugin`](https://github.com/areese801/trusty-cage-plugin) | The trusty-cage skills (`cage-orchestrator`, `cage-iterate`) | If the cage-orchestrator workflow steps (numbered "Step 1", "Step 7", etc.) shift, the cage-bridge skill's hook-point references need updating. |

The Kanbaroo monorepo also references this plugin from
`docs/deployment-dogfood.md` and from the `kb project init`
post-install checklist. If you change the install instructions in the
README here, mirror that change in the Kanbaroo monorepo.

## Repo Layout

This repo is a **Claude Code marketplace** that ships one plugin
(`kanbaroo`). The marketplace manifest lives at the repo root; the
plugin itself lives in a subdir under `plugins/`. Mirrors the layout
of [`trusty-cage-plugin`](https://github.com/areese801/trusty-cage-plugin).

```
.claude-plugin/
  marketplace.json     # marketplace manifest (name, owner, plugins[])
plugins/
  kanbaroo/
    .claude-plugin/
      plugin.json      # plugin manifest (name, version, repo, license, keywords)
    skills/
      kanbaroo-workflow/
        SKILL.md       # runtime guidance for outer Claude
      kanbaroo-cage-bridge/
        SKILL.md       # runtime guidance for outer Claude when both
                       # Kanbaroo and trusty-cage are active
      kanbaroo-iterate/
        SKILL.md       # runtime guidance for the iteration loop —
                       # improving Kanbaroo itself based on observed
                       # friction
LICENSE
README.md
CLAUDE.md              # this file
```

**Two manifests, two purposes:**

- `.claude-plugin/marketplace.json` advertises the marketplace + an
  index of the plugins it contains. `/plugin marketplace add` reads
  this when registering the marketplace.
- `plugins/kanbaroo/.claude-plugin/plugin.json` is the plugin's own
  manifest (version, license, keywords). `/plugin install` reads
  this when installing the plugin.

If you add a new plugin to this repo (unlikely but possible), add a
new entry under `marketplace.json`'s `plugins` array AND create a
corresponding `plugins/<new-plugin-name>/` directory with its own
`.claude-plugin/plugin.json`.

## SKILL.md Conventions

A SKILL.md is a markdown file with a YAML frontmatter block at the top:

```markdown
---
name: skill-id
description: |
  One-paragraph description that Claude Code matches against user
  intent. Be specific about WHEN this skill should fire and WHEN
  it should NOT fire.
---

# Human-Readable Skill Title

## Purpose
...

## When to Use This Skill
...

## When NOT to Use This Skill
...
```

The `description` field is load-bearing for skill discovery: Claude
Code's matcher uses it to decide which skills to load into a session
when. Edits to that field can dramatically change activation
behavior. Test by opening a fresh Claude Code session and probing
with the kinds of phrases the skill should and should not match.

## Style and Spelling

- Bias toward American English: `color`, `behavior`, `recognize`,
  `optimize`, `flavored`, `modeled`, `honors`. Mirrors the
  conventions in [Kanbaroo's CLAUDE.md](https://github.com/areese801/kanbaroo/blob/main/CLAUDE.md)
  and the global `~/.claude/CLAUDE.md`.
- Don't change quoted text from external sources (user messages,
  third-party docs, library API names) just to enforce dialect.
- Stdlib API names take precedence over dialect:
  `asyncio.CancelledError`, `Future.cancelled()` etc. use British
  spellings because the Python stdlib does. Don't Americanize prose
  next to those names — match the surrounding API.
- Avoid em-dashes in user-facing strings (project convention from
  Kanbaroo).
- Hard-wrap paragraphs at ~72 characters — keeps diffs reviewable.
- Don't include emojis unless the user explicitly asks.

## Versioning

This plugin follows [Semantic Versioning](https://semver.org/).

- **MAJOR**: a skill is removed, or its activation contract changes
  in a way that breaks existing workflows.
- **MINOR**: a new skill is added; an existing skill grows new
  capabilities; SKILL.md guidance is materially expanded.
- **PATCH**: skill body fixes, README updates, manifest tweaks.

Plugin version usually matches the Kanbaroo release whose MCP tool
surface it targets, but they are not enforced to be identical. When
they diverge, document the supported Kanbaroo range in the README.

## Release Process

1. Bump `version` in `.claude-plugin/plugin.json`.
2. Update README + skill bodies as appropriate.
3. Commit on a feature branch, open a PR, get the diff reviewed.
4. After merge, tag: `git tag v<version> && git push --tags`.
5. No PyPI step — Claude Code's marketplace mechanism reads the
   tagged commit directly via the GitHub source.

## Testing a Skill Change

There is no automated test runner for skill content. Validate by:

1. Local dev install (clone this repo + symlink into Claude Code's
   plugin cache as documented in the README's "Local development
   install" section).
2. Restart Claude Code in a project where the skill's preconditions
   are met.
3. Probe with phrases that should match the skill's description and
   confirm activation. Probe with phrases that should NOT match and
   confirm the skill stays quiet.
4. For `kanbaroo-cage-bridge`, validate the no-op invariant: open
   a session in a project that does NOT have Kanbaroo MCP wired,
   dispatch a cage via cage-orchestrator, confirm the bridge does
   not interfere.

## What This Repo Does NOT Do

- **No Python code.** This is skills + manifest only. If you find
  yourself wanting to ship a Python helper, it likely belongs in
  the [Kanbaroo monorepo](https://github.com/areese801/kanbaroo)
  under one of the existing packages (probably `kanbaroo-cli` for
  user-facing helpers).
- **No tests directory.** SKILL.md is prose; there's nothing to
  unit-test. The "test" is the runtime activation behavior, which
  has to be checked interactively.
- **No CI.** GitHub Actions might be added later (markdown lint,
  manifest schema validation, link check) but there's no current
  pipeline.

## Things to Watch For

- **Skill description drift.** If you tweak `kanbaroo-workflow`'s
  description, the kinds of user requests it activates on can
  change in surprising ways. After any description edit, do a
  before-and-after fresh-session probe.
- **Cross-repo wording drift.** Several of these documents reference
  Kanbaroo concepts (workspace key, story human ID, actor_id
  convention, etc.). When the Kanbaroo project renames or refactors
  one of those concepts, the skill text here needs the same edit.
  There is no compile-time check; the failure mode is silent
  staleness.
- **Tool-surface drift.** The "Available MCP Tools" section of
  `kanbaroo-workflow/SKILL.md` lists tool names + intent. If a
  new MCP tool ships in `kanbaroo-mcp` and the skill doesn't know
  about it, outer Claude won't reach for it. Worse: if a tool is
  removed, the skill telling outer Claude to call it produces 404s.

## Quick Commands

```bash
# Validate both manifests
jq -e . .claude-plugin/marketplace.json
jq -e . plugins/kanbaroo/.claude-plugin/plugin.json

# See which skills the plugin currently ships
ls -d plugins/kanbaroo/skills/*/

# Tag a release after merge
git tag v<version> && git push --tags
```

## Local Dev Install

To iterate on skill content without going through the marketplace:

```bash
# Add this repo as a marketplace (one-time)
/plugin marketplace add areese801/kanbaroo-plugin

# Install the plugin (one-time)
/plugin install kanbaroo@kanbaroo-plugin
```

The `/plugin` slash commands run inside any Claude Code session.
After install, edits to SKILL.md files in this repo do NOT
auto-propagate — you'll need to either pull the marketplace
again or symlink the plugin subdir into the cache for live
iteration:

```bash
ln -s "$(pwd)/plugins/kanbaroo" \
      ~/.claude/plugins/cache/kanbaroo-plugin/plugins/kanbaroo
```

Restart Claude Code after either approach.
