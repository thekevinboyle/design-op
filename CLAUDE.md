# design-op — Workflow Rules for Claude

You are operating in the **design-op** repo. This file is your operating manual. Read it before doing anything when invoked from this directory.

## What this repo is

A workspace for running a hybrid design-ops workflow. The user (Kevin) drives intent; you orchestrate research and Figma work; Kevin weighs in at decision points.

**Source of truth is Figma, not this repo.** This repo holds workflow rules (this file), an operator guide (`README.md`), and a flat initiative index (`initiatives.md`). Nothing else of substance. Don't store specs, research notes, or decisions here — those go in Figma.

## Where things live

| Thing | Location |
|---|---|
| Design system (tokens, components, governance pages) | **System file** in Figma — URL: `<SYSTEM_FILE_URL>` (set during setup) |
| Per-initiative work (research, spec, designs, build) | **Initiative file** in Figma — one per piece of work |
| Initiative index | `./initiatives.md` |
| Cost-tracking log | `./.perplexity-usage.log` (gitignored) |

## File conventions

**Initiative naming.** kebab-case. The `initiatives.md` row name must match the Figma file name exactly. Examples: `nav-redesign`, `dashboard-empty-states`, `audit-color-tokens-q2-2026`.

**Initiative file pages** (created in this exact order, every time):
1. `Brief` — goal, links, references
2. `Research` — Perplexity output as text frames
3. `Spec` — requirements + acceptance criteria
4. `Designs` — 2–3 directional options
5. `Build` — final selected design
6. `Decisions` — initiative-specific decisions

**System file pages** (must already exist; if not, error and ask Kevin to set up):
1. `Foundations` — token variables
2. `Components` — library
3. `Decisions Log` — system-level decisions, append-only
4. `Audit Log` — dated audit reports
5. `Inbox` — async questions awaiting Kevin
6. `Index` — mirror of `initiatives.md`

## The four loops at a glance

| Trigger | Loop | What it does |
|---|---|---|
| `plan: <description>` | PLAN | Research + draft a spec for a new initiative |
| `design: <initiative-name>` | DESIGN | Explore 2–3 visual directions |
| `build: <initiative-name>` | BUILD | Compose final design from system components |
| `audit: <scope>` | AUDIT | Audit the system; propose fixes |

Detailed protocols below.
