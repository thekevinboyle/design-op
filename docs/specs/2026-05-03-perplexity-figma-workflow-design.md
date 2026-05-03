# Perplexity + Figma Hybrid Design-Ops Workflow

**Date:** 2026-05-03
**Status:** Draft for review
**Owner:** Kevin Boyle

## Goal

A repeatable workflow where Claude Code, in the `design-op/` directory, uses the Perplexity MCP (research) and the official Figma MCP (read/write designs) together to **plan, design, build, and maintain** a design system and the product work that consumes it. The user (Kevin) drives and weighs in at decision points; Claude executes between decisions.

## Constraints (decided in brainstorming)

- **Hybrid execution.** Claude orchestrates; Kevin weighs in at meaningful decisions. Not fully autonomous, not fully manual.
- **Scope.** Both the design system itself *and* product work that consumes it. Full lifecycle: plan / design / build / maintain.
- **Source of truth.** Figma. Research notes, decisions, specs, and audit trails live in Figma — not in markdown.
- **Repo footprint.** Minimal. `design-op/` holds workflow rules and an initiative index only — not content.

## Approach

Two Figma file shapes plus a tiny repo. Four named loops (`plan` / `design` / `build` / `audit`) that Claude runs end-to-end on intent, with structured checkpoints for human input.

## File architecture

### A. The System file (one, ongoing)

Single Figma file; six pages:

| Page | Purpose |
|---|---|
| `Foundations` | Color, type, spacing, motion tokens (Figma variables) |
| `Components` | The component library |
| `Decisions Log` | Append-only text frames: `YYYY-MM-DD · <decision> · <why>` |
| `Audit Log` | Dated audit reports written by Claude |
| `Inbox` | Pending questions Claude needs Kevin to answer (async channel) |
| `Index` | Mirror of `initiatives.md` for in-Figma navigation |

### B. Initiative files (one per piece of work)

Created on demand for each feature, redesign, or system improvement. Six pages:

| Page | Purpose |
|---|---|
| `Brief` | Goal, links to System file, references |
| `Research` | Perplexity output: pattern survey, competitive teardown, accessibility notes, sources |
| `Spec` | Agreed requirements + acceptance criteria |
| `Designs` | Exploration: 2–3 options Claude generated |
| `Build` | Final selected design composed from system components |
| `Decisions` | Initiative-specific decisions (system-level decisions cross-post to System file) |

### C. Repo (`design-op/`)

| File | Purpose |
|---|---|
| `CLAUDE.md` | Workflow rules: the four loops, weigh-in mechanism, file conventions, tool usage |
| `README.md` | Operator guide for Kevin: how to start each loop, what to expect |
| `initiatives.md` | Flat index — `name → Figma URL → status`. The only persistent text outside Figma. Phone book, not content. |
| `.perplexity-usage.log` | Gitignored. Token-count log per Perplexity call for spend monitoring. |
| `docs/specs/` | This document and any future design specs. |

## The four loops

Each loop = one Claude Code session in `design-op/`. Trigger by intent prefix.

### PLAN loop — `plan: <description>`

1. Create Initiative file (`mcp__figma__create_new_file`); add row to `initiatives.md`.
2. Call `perplexity_research` once for pattern survey + `perplexity_search` 2–3× for competitive references.
3. Write findings as text frames on `Research` page (chunked under 20 KB write cap).
4. Draft `Spec` (problem, scope, acceptance criteria, open questions).
5. **Weigh-in:** open questions surface in terminal *and* drop to System file `Inbox`. Wait for Kevin. Update Spec. Append to `Decisions`.

### DESIGN loop — `design: <initiative-name>`

1. Read System file: `search_design_system`, `get_variable_defs` — only use existing tokens/components.
2. Optional `perplexity_search` for visual references if Spec requires.
3. `generate_figma_design` produces 2–3 directional options on `Designs` page.
4. **Weigh-in:** terminal Q&A — pick a direction, request iteration, or kill one.
5. Append to `Decisions`.

### BUILD loop — `build: <initiative-name>`

1. Take chosen direction.
2. `use_figma` to compose final design with proper components, variants, auto layout, token references — on `Build` page.
3. Flag any one-offs (no matching component) — log as system-improvement candidates to System file `Inbox`.
4. **Weigh-in:** Kevin reviews in Figma; requests fixes via terminal.

### MAINTAIN loop — `audit: <scope>`

1. Read System file metadata, components, variables.
2. `perplexity_ask` for current standards questions where useful (cheap, fast).
3. Cross-reference `Decisions Log` — flag drift (component diverges from last decision).
4. Write dated audit report to `Audit Log` page: findings + proposed fixes.
5. **Weigh-in:** Kevin approves fixes per item; Claude applies via `use_figma`. Each applied fix → `Decisions Log` entry.

**Rule:** loops are deliberately shallow. One session, one Initiative file (mostly), one batch of decisions. No multi-day stateful workflows. Resuming = re-reading the file.

## Weigh-in mechanism

Three channels at different urgencies:

### 1. Terminal Q&A (default — synchronous)

For decisions Claude needs to proceed *now*. Plain prompt in terminal; Kevin answers; Claude continues.

### 2. Inbox page in System file (async)

For things Claude noticed but doesn't need answered immediately. Each Inbox entry is one text frame:

```
[2026-05-03] Initiative: nav-redesign
QUESTION: We're using a one-off icon for "settings" because the system icon is rotated 12°.
Should I (a) fix the system icon, (b) add a variant, or (c) leave the one-off?
SUGGESTED: (a) — the rotation looks like a leftover from an old visual style.
```

Next session, Claude reads `Inbox` first and surfaces anything pending.

### 3. Decisions Log (durable record — Claude never asks)

After every weigh-in, Claude appends an entry. Format:

```
[2026-05-03 · nav-redesign] Use system icon, fix rotation. Why: leftover from v1; consistency > preserving artifact.
```

When starting any loop, Claude reads recent Decisions Log entries so settled questions don't get re-litigated.

**Hard rule:** Claude never modifies the Decisions Log retroactively. If a decision is reversed, append a new entry citing the old one. The log is the audit trail.

## Setup

### One-time install (Kevin, ~10 min)

1. Get a Perplexity API key at `console.perplexity.ai`. Pro subscription includes ~$5/mo API credit.
2. Add Perplexity MCP to Claude Code:
   ```
   claude mcp add perplexity --env PERPLEXITY_API_KEY="<key>" -- npx -y @perplexity-ai/mcp-server
   ```
3. Confirm Figma MCP is authed (it is). Confirm Full seat on the Figma team (Dev seats can't write).

### One-time `design-op/` scaffold (Claude, in next session)

- Write `CLAUDE.md` with the four-loop protocol and tool routing rules.
- Write `README.md` operator guide.
- Create empty `initiatives.md`.
- Add `.gitignore` line for `.perplexity-usage.log`.

### One-time Figma scaffold (Kevin, ~5 min)

- Create the System file with the six pages listed above.
- Paste the System file URL into `design-op/CLAUDE.md` so Claude knows where SoT lives.
- Initiative files are created on demand by Claude.

### Day-to-day operation

```
cd ~/Documents/web/design-op
claude
> plan: redesign the empty states for the dashboard
```

Claude reads `CLAUDE.md`, recognizes `plan:` → runs PLAN loop. Same pattern for `design:` / `build:` / `audit:`.

Other commands:
- `inbox` — read pending Inbox items
- `status <initiative-name>` — read latest state of an Initiative file
- `continue <initiative-name> <loop>` — re-enter a loop on an existing initiative

## Failure modes & limits

| # | Failure | Mitigation |
|---|---|---|
| 1 | `perplexity_research` is slow (30–60s) | Max one `_research` call per loop. Use `_search` for follow-ups. Claude announces slow calls in terminal. |
| 2 | Figma `use_figma` 20 KB write cap (beta) | Chunk writes — one text frame per finding. On failure, split and retry. |
| 3 | No comments API in official Figma MCP | Use text frames + Inbox page. Add community MCP (`thirdstrandstudio/mcp-figma`) only if Inbox model proves clunky after real use. |
| 4 | Published library write permissions undocumented | Keep System file as Full-seat-editable (not locked-down published library) while iterating. Revisit if org needs lock-down — likely with a "staging system file" pattern. |
| 5 | Perplexity costs creep | Log every call to `.perplexity-usage.log` with token counts. Weekly sanity check. If high, downgrade default from `_research` to `_ask` in `CLAUDE.md`. |
| 6 | Stale state when Kevin edits Figma between sessions | Claude always re-reads Initiative file (`get_metadata` + relevant pages) at loop start. Never trusts cached context. |
| 7 | Two-people-one-canvas conflicts | Claude announces "writing to <page> now" in terminal. Kevin backs off ~10s per chunk. Rare in practice — most weigh-ins return control to Kevin. |

## Out of scope (intentional)

- Multi-collaborator design review (this is a Kevin+Claude workflow, not a team workflow).
- Code Connect / design-to-code (different loop; could add later).
- Cross-team design system propagation (out of scope until multiple teams).
- Automated visual regression of Figma frames (Figma plugins exist; not via MCP today).

## Open questions for the implementation plan

- Exact prose for `CLAUDE.md` workflow rules (tool routing per loop, write-chunking rules, loop-start checklist).
- Exact `README.md` operator guide content.
- Whether `design-op/` becomes its own git repo or stays as an untracked subdirectory of `web/` (currently the parent is on an O&G scraper branch — committing here would mix concerns).
- Whether to scaffold the Figma System file via `create_new_file` + `use_figma` from a one-shot Claude session, or have Kevin create it manually (likely manual — file ownership matters).
