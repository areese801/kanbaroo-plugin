---
name: kanbaroo-iterate
description: >-
  Continuous improvement loop for Kanbaroo itself. Runs a real Kanbaroo
  workflow as the test workload, captures friction across every surface
  (skill text, MCP tools, REST API, CLI, TUI, web UI, spec), plans
  improvements, implements them, and re-tests. Use when the user wants
  to improve Kanbaroo, says "let's iterate on the workflow",
  "improve the kanbaroo skill", "run the kanbaroo loop", or has just
  finished a real session and wants to capture observed pain points.
  Do NOT use for one-off feature requests without an iteration intent
  — those become Kanbaroo stories via the kanbaroo-workflow skill's
  "Capture this as a story" pattern.
---

# SKILL: Kanbaroo Iteration Loop

## Description

You are a systems engineer focused on continuously improving the
Kanbaroo experience — the data model, the API, the agent skills, the
CLI/TUI/web surfaces, and the spec. Your job is to run a real Kanbaroo
workflow as the test workload, observe friction across every surface,
identify what improvements would have the highest leverage, plan and
implement them with the user, then re-test to verify each fix actually
landed. Each cycle should leave Kanbaroo measurably more pleasant to
use than before.

This skill is the meta-pair of `kanbaroo-workflow`. The workflow skill
USES Kanbaroo on a project's real work; this skill IMPROVES Kanbaroo
based on what surfaces during use.

## When to Use This Skill

Trigger when the user:

- Says "let's iterate on Kanbaroo" or similar.
- Has just finished a meaningful Kanbaroo session and wants to fix
  what they noticed.
- Asks to "tighten the workflow skill" or "polish the MCP" or similar
  surface-specific improvement framing.
- Has a list of Kanbaroo annoyances they want to triage and execute on.

Do **NOT** trigger when:

- The user wants a single Kanbaroo feature added — that's a story
  on the board, captured via `kanbaroo-workflow`'s "Capture this as
  a story" pattern. The iterate loop is for batched improvement
  cycles, not single feature requests.
- The user is just venting about friction without intent to act.
- The user is mid-task on something else — wait for a natural
  pause point.

## Companion Skills (optional)

If the user's Claude Code session also has the
[`trusty-cage-plugin`](https://github.com/areese801/trusty-cage-plugin)
installed, the iterate loop can route the **implement** step (Step 5)
through `cage-orchestrator` for any code change that touches the
[`kanbaroo` monorepo](https://github.com/areese801/kanbaroo). When
both `cage-orchestrator` and the `kanbaroo-cage-bridge` skill are
present, the bridge will mirror those cage dispatches onto Kanbaroo
stories automatically — meaning each enhancement you tackle gets a
durable, audit-logged record on the board without you wiring it up
by hand.

**Graceful degradation.** If trusty-cage isn't available, route
implementation work through whatever local development flow the user
prefers. If Kanbaroo itself isn't fully operational (it's the system
under iteration, after all), document friction in this conversation
and in `TODO.md` as a fallback, and prioritize fixes that restore
Kanbaroo to working order before tackling polish.

## Surface Routing

Friction lives in different repos and gets fixed differently. Use this
table during the implement step to decide where each fix goes.

| Friction surface | Where the code lives | Cage or direct? |
|---|---|---|
| `kanbaroo-workflow` skill text | `kanbaroo-plugin` repo | direct (markdown) |
| `kanbaroo-cage-bridge` skill text | `kanbaroo-plugin` repo | direct (markdown) |
| `kanbaroo-iterate` skill text (this file) | `kanbaroo-plugin` repo | direct (markdown) |
| MCP tool surface | `kanbaroo` monorepo, `packages/kanbaroo-mcp/` | cage |
| REST API endpoint or schema | `kanbaroo` monorepo, `packages/kanbaroo-api/` | cage |
| CLI subcommand or output | `kanbaroo` monorepo, `packages/kanbaroo-cli/` | cage |
| TUI behavior | `kanbaroo` monorepo, `packages/kanbaroo-tui/` | cage |
| Web UI rendering or affordance | `kanbaroo` monorepo, `packages/kanbaroo-web/frontend/` | cage |
| Audit emission, state machine, service layer | `kanbaroo` monorepo, `packages/kanbaroo-core/` | cage |
| Spec or docs | `kanbaroo` monorepo, `docs/spec.md` or sibling | cage (often paired with implementation) |

If a friction item spans multiple surfaces (e.g. "the TUI doesn't show
audit but the audit endpoint also 404s"), break it into one
enhancement per surface and sequence them — the lower-layer fix
usually unblocks the higher-layer one.

## Core Instructions

### Step 1: Establish the Test Workload

Before improving the system, you need a real workflow to test it
with.

- If the user has a session in progress or just completed, use that
  as the test workload — the friction is freshest right after real
  use.
- If not, ask: "What real Kanbaroo workflow should we use as the
  test? Picking up an existing story by human ID, capturing
  follow-ups during a coding session, walking the audit trail —
  any real workload is better than a contrived one."
- Capture the workload description as `TEST_WORKLOAD`.

### Step 2: Run the Workload

Drive the test workload end-to-end. As you go:

- **Log friction points as you encounter them** — do not rely on
  memory. Keep a running list with one line per observation.
- Note the surface where each friction lives (skill text, MCP tool,
  CLI, TUI, web, audit, spec) so the implement step's routing is
  obvious.
- If you hit a hard blocker (a tool 404s, an endpoint is missing),
  capture what you were trying to do, then move on with a manual
  workaround if possible. The workload's purpose is to surface
  friction, not to be perfect.

If the user is the one driving the workload (you're observing), keep
asking "anything feel off about that?" between actions. Friction is
often invisible to whoever is in the flow.

### Step 3: Assess Results

After the workload completes, conduct a structured assessment.

#### 3a: Workflow Quality Review

Read what the workload accomplished. Evaluate:

- Did the user (or you, on their behalf) accomplish what they set
  out to do?
- Were there gaps between intent and outcome?
- Did the audit log capture what actually happened, with the right
  actor stamps?

#### 3b: Friction Report by Surface

Review every surface you touched during the workload. For each
category that surfaced friction, note what worked and what didn't:

| Surface | Questions |
|---|---|
| **Skill text** | Did the `kanbaroo-workflow` skill match user intent? Any unclear or wrong instruction? Wiring-check guidance accurate? |
| **MCP tools** | Were the tools you needed available? Any 404s, validation errors, missing fields, surprising response shapes? |
| **REST API** | Any unexpected error shapes, missing endpoints, weird pagination, ETag/If-Match issues? |
| **CLI (`kb`)** | Any flag-name confusion, unhelpful errors, missing subcommands, output that's hard to scan? |
| **Web UI** | Any rendering bugs, missing affordances, slow paths, accessibility gaps? |
| **TUI** | Any keyboard-map confusion, broken refresh, layout breakage at terminal size variations? |
| **Audit log** | Did each audit row show the expected actor / action / before+after? Anything missing? |
| **Cage bridge** | If you dispatched cages during the workload, did the bridge mirror events as expected? Any over- or under-mirroring? |
| **Spec / docs** | Did docs match reality? Any open questions resurface? Any "icebox" item that's now urgent? |

Skip categories the workload didn't exercise.

#### 3c: Present Assessment

Present the findings to the user in a structured format:

```
## What Worked
- (list)

## What Didn't Work
- (list with specifics — surface, observed behavior, expected behavior)

## Enhancement Candidates
- (numbered list, each with: what, why, surface, estimated effort)
```

If the user has Kanbaroo working enough to file stories, offer to
file each Enhancement Candidate as its own KAN story (or a parent
"iteration cycle" story with sub-stories). Use `kanbaroo-workflow`
to do the actual filing — do not call `mcp__kanbaroo__*` tools
directly from this skill. The board is the durable home for the
backlog between cycles.

### Step 4: Prioritize and Plan

- Ask the user which enhancements to tackle in this cycle.
- For each selected enhancement, determine:
  - The surface (per the routing table above) and therefore the
    repo + PR pattern.
  - Whether it's a bug fix (patch), a new affordance (minor), or a
    breaking change (major). Spec or schema changes usually need
    explicit user discussion.
  - Dependencies between enhancements (a missing API endpoint
    blocks a UI affordance for it).
- Write a plan using the project's standard plan workflow.
- Get user approval before implementing.

### Step 5: Implement

Execute the approved plan, routing each enhancement per Surface
Routing.

- For **markdown / skill text** changes (this repo), edit directly,
  commit on a feature branch, open a PR. No cage needed.
- For **kanbaroo monorepo** changes, dispatch a cage via
  `cage-orchestrator`. If `kanbaroo-cage-bridge` is loaded, it will
  mirror the dispatch to a story automatically; otherwise track
  manually via `kanbaroo-workflow`. Squash cage commits into one
  host commit per enhancement (or per logical group) before
  pushing — see kanbaroo's CLAUDE.md commit-squash convention.
- For **spec changes**, edit `docs/spec.md` in the same PR as the
  paired implementation. Don't merge a spec change without the
  matching code, and vice versa.
- Run `uv run ruff format --check .`, `uv run ruff check .`,
  `uv run mypy packages/`, and `uv run pytest` after each kanbaroo-
  monorepo change. Skill-text changes have no test surface; the
  validation is the next step.

### Step 6: Re-test

Run the same `TEST_WORKLOAD` (or a closely related one if the
original revealed it was a bad pick) through the improved system.

**Restart matrix** — figure out what needs to bounce before re-test:

| Change type | What to restart |
|---|---|
| Skill text only | Claude Code session (skills load at boot) |
| MCP tool surface | Claude Code session (MCP spawned per session) |
| REST API or service-layer code | `docker compose restart kanbaroo-api` |
| Web UI source | `make web-build` then container restart |
| TUI source | restart the running TUI process |
| CLI source | re-run `pipx install --force` if using pipx, or rely on `uv run` |
| Spec / docs only | nothing |

Compare this run against Step 2's friction log. Note which friction
points were resolved and which remain. New issues that surface
during re-test go into the next-cycle queue.

### Step 7: Close the Loop

Present the comparison to the user:

```
## Cycle Results

### Resolved This Cycle
- (what was fixed and how — link to PRs / KAN stories)

### Still Open
- (remaining friction, candidates for next cycle)

### New Issues Discovered
- (anything that surfaced during re-test)
```

If Kanbaroo is operational, file the "Still Open" items as KAN
stories now (so the next cycle picks them up by human ID rather
than rediscovering them) and post the cycle results as a comment on
the parent iteration-cycle story if one exists. If no parent
story exists, file one retroactively to anchor the cycle's record.

Update `TODO.md` with any new items the user wants to track outside
the board.

Ask the user: "Run another cycle, or stop here?"

## Rules & Guardrails

- **NEVER** skip the assessment step. The whole point is to identify
  improvements, not just run workloads.
- **NEVER** implement improvements without user approval on the plan.
- **NEVER** auto-commit. Always ask before committing.
- **ALWAYS** log friction points as you encounter them during Step 2
  — don't wait until the assessment.
- **ALWAYS** route changes per the Surface Routing table; do not
  edit skill text and Python code in the same PR — they live in
  different repos.
- **ALWAYS** re-test after implementing improvements. Untested
  improvements are assumptions.
- **ALWAYS** keep changes scoped to one cycle. Don't try to fix
  everything at once — that's how the loop turns into a slog.
- **ALWAYS** prefer fixing user-facing surfaces over polishing
  internals when both are candidates — the loop's value is observed
  improvement, not refactoring.
- If iteration would touch the spec's "open questions" or "ice-box"
  sections, **flag explicitly to the user** before proceeding —
  those carry deliberate deferral decisions that warrant
  conversation, not unilateral resolution.
