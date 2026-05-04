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

---

## Loop-start checklist (every loop begins with this)

Before doing the work of any loop, in order:

1. **Read recent System file `Decisions Log`** (last 30 days). Know what's been decided so you don't re-litigate.
2. **Read System file `Inbox`.** If anything pending might affect this loop, surface it to Kevin in one sentence: "Inbox has 2 pending items related to this — want to address them first?"
3. **Confirm intent.** State in one sentence what you're about to do, then proceed without waiting for confirmation. Example: "Running PLAN for `dashboard-empty-states` — researching empty-state patterns then drafting a spec."

## Weigh-in mechanism — three channels

Use the channel that matches urgency.

### Sync (terminal Q&A) — DEFAULT

For decisions you need *right now* to proceed. Plain prompt in the terminal. Wait for Kevin's answer. Continue.

### Async (Inbox page in System file)

For things you noticed but don't need answered immediately. Write one text frame on the `Inbox` page. Format:

```
[YYYY-MM-DD] Initiative: <name>
QUESTION: <one-paragraph question>
SUGGESTED: <your best guess and why>
```

### Decisions Log (durable record — never asks)

After every weigh-in (sync or async resolution), append to the `Decisions` page of the relevant file. Format:

```
[YYYY-MM-DD · <initiative-name>] <decision>. Why: <reason>.
```

System-level decisions (anything that changes the System file) ALSO get cross-posted to System file `Decisions Log`.

**HARD RULE: Never modify the Decisions Log retroactively.** If a decision is reversed, append a new entry citing the prior one ("Reverses 2026-04-12 entry on icon rotation; new evidence: ..."). The log is the audit trail.

## Figma write rules

The Figma MCP `use_figma` write call has a **20 KB output cap (beta)**.

- **One text frame per finding.** Never combine multiple research findings, multiple decisions, or multiple components into one frame.
- **On 20 KB error:** split the content in half and retry. If the half still fails, split again. If a single semantic unit (one finding, one decision) cannot fit even when isolated, summarize it and link to a longer text frame elsewhere on the page.
- **Announce writes:** before writing to a Figma page, post one terminal line: `Writing to <PageName> page now (~Ns)`. This lets Kevin avoid editing the same page concurrently.

## Cost tracking

Every Perplexity MCP call gets logged to `./.perplexity-usage.log`. Format (one line per call):

```
2026-05-03T14:22:15Z perplexity_research prompt_chars=412 response_chars=4821 est_cost_usd=0.18 initiative=dashboard-empty-states
```

Cost estimation: use the model's published per-token rate × estimated tokens (chars / 4). Round to nearest cent.

**At loop start**, run `tail -50 .perplexity-usage.log | awk` (or read with the Read tool) to compute the last 7 days' total. If > $2.00, surface to Kevin: "Perplexity spend last 7 days: $X.XX — heads up." If > $5.00, also suggest downgrading default research calls from `perplexity_research` to `perplexity_ask`.

## Tool routing — quick reference

| Need | Tool |
|---|---|
| Deep multi-step research (1×/loop max) | `perplexity_research` |
| Single web question, fast | `perplexity_ask` |
| Web search with ranked results | `perplexity_search` |
| Logical analysis without web search | `perplexity_reason` |
| Read Figma file metadata (pages, frames) | `mcp__figma__get_metadata` |
| Read design context for a node | `mcp__figma__get_design_context` |
| Read System file tokens | `mcp__figma__get_variable_defs` |
| Find components by name in System | `mcp__figma__search_design_system` |
| Create a new Figma file | `mcp__figma__create_new_file` |
| Generate 2–3 design options from a description | `mcp__figma__generate_figma_design` |
| Write text frames, pages, components | `mcp__figma__use_figma` |
| Take screenshot of a node | `mcp__figma__get_screenshot` |
