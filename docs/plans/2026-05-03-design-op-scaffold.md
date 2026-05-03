# Design-Op Workflow Scaffold — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Scaffold the `design-op/` repo with workflow rules, install the Perplexity MCP, scaffold the Figma System file, and prove the workflow end-to-end with a smoke test.

**Architecture:** A single `CLAUDE.md` in `design-op/` defines four loop protocols (PLAN / DESIGN / BUILD / AUDIT) that Claude executes when invoked with intent prefixes. Figma is the source of truth — research notes, decisions, and audits live as text frames on dedicated pages. The repo holds workflow rules + an initiative index only. Perplexity MCP provides research; the official Figma MCP provides design read/write.

**Tech Stack:** Markdown (workflow docs), Claude Code MCP system, Perplexity Sonar API via `@perplexity-ai/mcp-server`, official Figma MCP (already installed and authed).

**Reference spec:** `docs/specs/2026-05-03-perplexity-figma-workflow-design.md`

---

## File Structure

| File | Purpose | Created in task |
|---|---|---|
| `.gitignore` | Ignores `.perplexity-usage.log` and OS junk | Task 1 |
| `initiatives.md` | Flat initiative index (name → Figma URL → status) | Task 2 |
| `CLAUDE.md` | Workflow rules — the four loops, weigh-in rules, conventions | Tasks 3–8 |
| `README.md` | Operator guide: setup checklist, daily commands, troubleshooting | Task 9 |

`CLAUDE.md` is split across tasks 3–8 because it's the substantive document and benefits from incremental review. Final file ~300 lines.

---

## Task 1: Add `.gitignore`

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: Verify file does not exist**

Run: `ls -la .gitignore`
Expected: `ls: .gitignore: No such file or directory`

- [ ] **Step 2: Create `.gitignore`**

Write `.gitignore`:

```
# Perplexity API usage tracking — local only
.perplexity-usage.log

# OS junk
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/
*.swp
```

- [ ] **Step 3: Verify content**

Run: `cat .gitignore`
Expected: matches the content above.

- [ ] **Step 4: Commit**

```bash
git add .gitignore
git commit -m "chore: add gitignore for perplexity log and editor junk"
```

---

## Task 2: Create `initiatives.md` skeleton

**Files:**
- Create: `initiatives.md`

- [ ] **Step 1: Verify file does not exist**

Run: `ls -la initiatives.md`
Expected: `ls: initiatives.md: No such file or directory`

- [ ] **Step 2: Create `initiatives.md`**

Write `initiatives.md`:

```markdown
# Initiatives

Flat index of all design-op initiatives. One row per initiative. Claude appends a row when running PLAN; humans edit status.

| Name | Figma URL | Status | Created |
|---|---|---|---|
| _(no initiatives yet — first PLAN loop will add one)_ | | | |

## Status values

- `planning` — PLAN loop in progress; spec being written
- `designing` — DESIGN loop in progress; exploring options
- `building` — BUILD loop in progress; final design under construction
- `done` — Build accepted; merged into product/system
- `archived` — No longer relevant; kept for history
```

- [ ] **Step 3: Verify content**

Run: `cat initiatives.md`
Expected: matches above.

- [ ] **Step 4: Commit**

```bash
git add initiatives.md
git commit -m "feat: add initiatives.md index skeleton"
```

---

## Task 3: Create `CLAUDE.md` — header, project context, file conventions

**Files:**
- Create: `CLAUDE.md`

- [ ] **Step 1: Verify file does not exist**

Run: `ls -la CLAUDE.md`
Expected: `ls: CLAUDE.md: No such file or directory`

- [ ] **Step 2: Create `CLAUDE.md` with the first three sections**

Write `CLAUDE.md`:

````markdown
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
````

- [ ] **Step 3: Verify content**

Run: `cat CLAUDE.md | head -50`
Expected: file starts with `# design-op — Workflow Rules for Claude` and includes the "Where things live" table.

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "feat(claude-md): add header, project context, and file conventions"
```

---

## Task 4: Append `CLAUDE.md` — common preamble (loop-start checklist, weigh-in, write rules, cost tracking)

**Files:**
- Modify: `CLAUDE.md` (append to end)

- [ ] **Step 1: Verify Task 3 sections are present**

Run: `grep -c "## The four loops at a glance" CLAUDE.md`
Expected: `1`

- [ ] **Step 2: Append the common preamble sections**

Use Edit to append after the "## The four loops at a glance" section. Append this content:

````markdown

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
````

- [ ] **Step 3: Verify the appended content is present**

Run: `grep -c "## Loop-start checklist" CLAUDE.md && grep -c "## Tool routing" CLAUDE.md`
Expected: `1` then `1`

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "feat(claude-md): add loop preamble, weigh-in, write rules, cost tracking, tool routing"
```

---

## Task 5: Append `CLAUDE.md` — PLAN loop protocol

**Files:**
- Modify: `CLAUDE.md` (append)

- [ ] **Step 1: Verify Task 4 sections are present**

Run: `grep -c "## Tool routing" CLAUDE.md`
Expected: `1`

- [ ] **Step 2: Append PLAN loop section**

Append this content to `CLAUDE.md`:

````markdown

---

## PLAN loop

**Trigger:** `plan: <one-sentence description>` or `plan: <description> (initiative: <kebab-name>)`

If no initiative name is given, propose one in kebab-case based on the description; confirm with Kevin in terminal in one sentence; continue with the chosen name.

**Steps:**

1. **Loop-start checklist** (see above section).

2. **Create the Initiative file.**
   - Call `mcp__figma__create_new_file` with name = initiative-name (kebab-case).
   - Get the new file's URL from the response. Hold it.
   - Use `mcp__figma__use_figma` to create the six pages in order: `Brief`, `Research`, `Spec`, `Designs`, `Build`, `Decisions`.

3. **Update `initiatives.md`.**
   - Append a row: `| <name> | <figma-url> | planning | <YYYY-MM-DD> |`.
   - Replace the placeholder row if it's still the only row.

4. **Update System file `Index` page.**
   - Append a text frame mirroring the `initiatives.md` row.

5. **Write the `Brief` page.**
   - One text frame: goal (the user's description, polished), link back to System file URL, today's date.

6. **Research pass.**
   - Call `perplexity_research` ONCE with a focused prompt: "Pattern survey for <topic>. Return: established patterns, common pitfalls, accessibility considerations, 3–5 reference products with brief notes." Announce in terminal: "Running deep research (~45–60s)..."
   - Call `perplexity_search` 1–3× for specific competitive references mentioned in the spec or surfaced by the research call.
   - **Log each call** to `.perplexity-usage.log` per the Cost tracking section.

7. **Write findings to `Research` page.**
   - One text frame per distinct finding. Suggested layout (top to bottom):
     - Frame 1: "Patterns" — bulleted list of established patterns with one-line descriptions.
     - Frame 2: "Pitfalls" — bulleted list.
     - Frame 3: "Accessibility" — bulleted list.
     - Frames 4–N: one per reference product, including link and 1-paragraph teardown note.
     - Frame N+1: "Sources" — list of all URLs cited, deduplicated.
   - Respect the 20 KB write cap; chunk further if needed.

8. **Draft the `Spec` page.**
   - Sections as separate text frames: `Problem`, `Goals (numbered)`, `Non-goals`, `Acceptance criteria (checklist)`, `Open questions`.
   - `Acceptance criteria` should be specific and testable: "Empty state appears within 200ms of confirming an empty result set" not "fast".

9. **Weigh-in (sync).**
   - In the terminal, list the `Open questions` from the Spec. Wait for Kevin to answer.
   - Update the `Spec` page (modify the `Open questions` frame to mark resolved ones; update other frames if answers change them).
   - Append one entry per resolved question to the `Decisions` page on the Initiative file.

10. **Surface anything else as Inbox items** (System file `Inbox` page) per Weigh-in mechanism. Examples: discovered missing system component, naming convention questions, token gaps.

11. **Update `initiatives.md`** status from `planning` to `designing` (signals ready for the DESIGN loop). Also update the corresponding row on the System file `Index` page.

12. **Done.** Final terminal output: "PLAN complete for `<name>`. Spec at <Initiative file URL> · Spec page. Open questions: 0 unresolved. Run `design: <name>` when ready to explore."
````

- [ ] **Step 3: Verify**

Run: `grep -c "## PLAN loop" CLAUDE.md`
Expected: `1`

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "feat(claude-md): add PLAN loop protocol"
```

---

## Task 6: Append `CLAUDE.md` — DESIGN loop protocol

**Files:**
- Modify: `CLAUDE.md` (append)

- [ ] **Step 1: Verify Task 5 is present**

Run: `grep -c "## PLAN loop" CLAUDE.md`
Expected: `1`

- [ ] **Step 2: Append DESIGN loop section**

Append:

````markdown

---

## DESIGN loop

**Trigger:** `design: <initiative-name>`

**Preconditions:** Initiative file exists with `Spec` page populated. If not, error and tell Kevin to run PLAN first.

**Steps:**

1. **Loop-start checklist** (see above).

2. **Read the Spec.**
   - `mcp__figma__get_metadata` on the Initiative file to get page node IDs.
   - `mcp__figma__get_design_context` on the `Spec` page node. Hold the spec contents.

3. **Read the System file constraints.**
   - `mcp__figma__get_variable_defs` on System file (cached for the session).
   - `mcp__figma__search_design_system` for components likely relevant to the spec topic (e.g., for a button-related spec, search "button").
   - You will only generate designs that use existing tokens and components. Note any apparent gaps for the BUILD loop or as Inbox items.

4. **Optional reference search.**
   - If the Spec mentions or implies needing visual inspiration, call `perplexity_search` 1–2× for visual references. Log to `.perplexity-usage.log`.
   - Skip if Spec is sufficient on its own.

5. **Generate 2–3 directional options.**
   - Call `mcp__figma__generate_figma_design` 2–3 times with distinct directional prompts. Each prompt MUST cite the spec's acceptance criteria and reference specific System file tokens by name.
   - Place each option as a separate frame on the `Designs` page, labeled `Option A`, `Option B`, `Option C`.

6. **Write a short evaluation frame.**
   - One text frame on the `Designs` page above the options: 2–3 lines per option summarizing tradeoffs. Format:
     ```
     Option A: <tradeoff line>
     Option B: <tradeoff line>
     Option C: <tradeoff line>
     ```

7. **Weigh-in (sync).**
   - In terminal: "Three options on Designs page. A: <one-line>. B: <one-line>. C: <one-line>. Pick one to take into BUILD, request iteration on a specific option, or kill an option."
   - Apply Kevin's choice. If iteration requested, regenerate the targeted option only and re-prompt.

8. **Append to `Decisions` page.**
   - `[YYYY-MM-DD · <name>] Selected Option <X> for build. Why: <Kevin's reason or your inferred reason>.`

9. **Update `initiatives.md`** status from `designing` to `building`. Also update the System file `Index` page row.

10. **Done.** Terminal: "DESIGN complete for `<name>`. Selected: Option <X>. Run `build: <name>` when ready."
````

- [ ] **Step 3: Verify**

Run: `grep -c "## DESIGN loop" CLAUDE.md`
Expected: `1`

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "feat(claude-md): add DESIGN loop protocol"
```

---

## Task 7: Append `CLAUDE.md` — BUILD loop protocol

**Files:**
- Modify: `CLAUDE.md` (append)

- [ ] **Step 1: Verify Task 6 is present**

Run: `grep -c "## DESIGN loop" CLAUDE.md`
Expected: `1`

- [ ] **Step 2: Append BUILD loop section**

Append:

````markdown

---

## BUILD loop

**Trigger:** `build: <initiative-name>`

**Preconditions:** `Designs` page has a selected option recorded in the most recent `Decisions` entry for this initiative. If not, error.

**Steps:**

1. **Loop-start checklist** (see above).

2. **Read selected design.**
   - `mcp__figma__get_metadata` on Initiative file. Find the selected Option frame on `Designs` page.
   - `mcp__figma__get_design_context` on that Option frame. Capture structure.

3. **Read System file inventory.**
   - `mcp__figma__search_design_system` for each component used in the design.
   - `mcp__figma__get_variable_defs` for tokens (use cached if already pulled this session).

4. **Compose final design on `Build` page.**
   - Use `mcp__figma__use_figma` to build the final layout, swapping any ad-hoc shapes from the Option frame for actual System components and token references.
   - Use auto layout. Use variants where appropriate.
   - Group into one top-level frame named `<initiative-name> · final`.

5. **Identify and log one-offs.**
   - For each piece that could not be expressed using existing components/tokens, write one Inbox item to System file `Inbox`. Format:
     ```
     [YYYY-MM-DD] Initiative: <name>
     ONE-OFF: <description of the ad-hoc piece, screenshot link, why no system match>
     SUGGESTED: <add new component | add variant | add token | leave one-off>
     ```

6. **Weigh-in (sync).**
   - Terminal: "BUILD frame on `Build` page. <N> one-offs flagged to Inbox. Open the Build page in Figma and let me know if anything needs revision."
   - Wait for Kevin's review. Apply revisions iteratively (each revision = one targeted `use_figma` call to the affected sub-frame).

7. **Append to `Decisions` page** (Initiative file).
   - `[YYYY-MM-DD · <name>] BUILD accepted. Components used: <list>. Tokens used: <list>. One-offs: <count, see Inbox>.`

8. **Update `initiatives.md`** status to `done`.

9. **Update System file `Index` page** to mirror.

10. **Done.** Terminal: "BUILD complete for `<name>`. Final on Build page. Status updated to `done`."
````

- [ ] **Step 3: Verify**

Run: `grep -c "## BUILD loop" CLAUDE.md`
Expected: `1`

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "feat(claude-md): add BUILD loop protocol"
```

---

## Task 8: Append `CLAUDE.md` — AUDIT loop protocol

**Files:**
- Modify: `CLAUDE.md` (append)

- [ ] **Step 1: Verify Task 7 is present**

Run: `grep -c "## BUILD loop" CLAUDE.md`
Expected: `1`

- [ ] **Step 2: Append AUDIT loop section**

Append:

````markdown

---

## AUDIT loop

**Trigger:** `audit: <scope>` where scope is one of:
- `system` — full audit of System file
- `tokens` — audit `Foundations` page only
- `components` — audit `Components` page only
- `<component-name>` — audit a single named component

**Steps:**

1. **Loop-start checklist** (see above).

2. **Read the scope.**
   - `mcp__figma__get_metadata` on System file.
   - For `system`: read all of `Foundations` and `Components`.
   - For `tokens`: `mcp__figma__get_variable_defs`.
   - For `components`: `mcp__figma__search_design_system` with empty/wildcard query, then `get_design_context` on each.
   - For a specific component: `search_design_system` for the name, then `get_design_context`.

3. **Read full `Decisions Log`** (System file). Use this to detect drift — components or tokens that changed without a corresponding decision.

4. **Optional standards check.**
   - For specific questions ("is naming convention X still recommended in 2026?"), call `perplexity_ask` (cheap, fast). Skip if no concrete question.
   - Log per Cost tracking.

5. **Compose audit report.**
   - Write to System file `Audit Log` page. Top-level frame title: `Audit · <scope> · <YYYY-MM-DD>`.
   - Inside the top frame, child frames per finding. Each finding format:
     ```
     FINDING: <one-sentence summary>
     EVIDENCE: <what you observed: component name, token value, etc.>
     SEVERITY: low | medium | high
     PROPOSED FIX: <specific change>
     ```
   - Cap: max 20 findings per audit run. If more, prioritize high-severity and note "N additional low/medium findings deferred to next audit."

6. **Weigh-in (sync).**
   - Terminal: "Audit complete: <N> findings (H: <h>, M: <m>, L: <l>). Want to walk through them now? Reply with finding numbers to apply, `all` to apply all, or `none` to defer."
   - For each approved fix:
     - Apply via `mcp__figma__use_figma`.
     - Append entry to System file `Decisions Log`: `[YYYY-MM-DD · audit:<scope>] <fix description>. Why: audit finding <N>.`
   - Deferred findings stay on the `Audit Log` page for next session.

7. **Done.** Terminal: "AUDIT complete for `<scope>`. <N> fixes applied, <M> deferred."
````

- [ ] **Step 3: Verify all four loops are present**

Run: `grep -c "## .* loop" CLAUDE.md`
Expected: `4` (PLAN, DESIGN, BUILD, AUDIT)

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "feat(claude-md): add AUDIT loop protocol"
```

---

## Task 9: Write `README.md` operator guide

**Files:**
- Create: `README.md`

- [ ] **Step 1: Verify file does not exist**

Run: `ls README.md 2>&1`
Expected: `ls: README.md: No such file or directory`

- [ ] **Step 2: Create `README.md`**

Write `README.md`:

````markdown
# design-op

A workspace for running design system + product design work with Claude Code as a hybrid collaborator. You drive intent, Claude executes research and Figma work, you weigh in at decision points.

**Source of truth is Figma.** This repo holds workflow rules (`CLAUDE.md`), this guide, and an initiative index (`initiatives.md`).

---

## One-time setup

You only do this once. After this, day-to-day usage is one command.

### 1. Get a Perplexity API key

1. Go to [console.perplexity.ai](https://console.perplexity.ai).
2. Sign in (use your Pro account if you have one — gives ~$5/mo API credit).
3. Create an API key. Copy it.

### 2. Install the Perplexity MCP

In your terminal:

```bash
claude mcp add perplexity --env PERPLEXITY_API_KEY="<paste-key-here>" -- npx -y @perplexity-ai/mcp-server
```

Verify:

```bash
claude mcp list | grep perplexity
```

You should see `perplexity` listed.

### 3. Confirm the Figma MCP is set up

The official Figma MCP is already installed (you can see it in `claude mcp list`). Two requirements to **write** to Figma:

- Your Figma seat is **Full** (not Dev). Check at figma.com → top-right avatar → Settings.
- The Figma MCP is authed with the Figma account that has Full seat access.

### 4. Create the System file in Figma

In Figma:

1. Create a new Figma Design file. Name it `Design System` (or whatever you like — but you'll paste the URL into `CLAUDE.md` next).
2. Add these six pages (in this order):
   - `Foundations` — your tokens (color, type, spacing, motion variables)
   - `Components` — your component library
   - `Decisions Log` — leave empty; Claude appends
   - `Audit Log` — leave empty; Claude appends
   - `Inbox` — leave empty; Claude appends
   - `Index` — leave empty; Claude mirrors `initiatives.md` here

3. Copy the file URL.

### 5. Paste the System file URL into `CLAUDE.md`

In `CLAUDE.md`, find the line `URL: <SYSTEM_FILE_URL>` and replace `<SYSTEM_FILE_URL>` with your actual System file URL.

Commit:

```bash
git add CLAUDE.md
git commit -m "config: set System file URL"
git push
```

Setup done.

---

## Day-to-day usage

Open a terminal in this directory:

```bash
cd ~/Documents/web/design-op
claude
```

Then type one of these intents:

| Command | What happens |
|---|---|
| `plan: <description>` | Claude researches the topic, drafts a spec in a new Figma Initiative file, asks you the open questions |
| `design: <initiative-name>` | Claude generates 2–3 visual directions on the Designs page; you pick one |
| `build: <initiative-name>` | Claude composes the final design from system components on the Build page |
| `audit: <scope>` | Claude audits the System file; you approve fixes |

Other useful intents:

| Command | What happens |
|---|---|
| `inbox` | Claude reads the System file Inbox page and surfaces pending items |
| `status <initiative-name>` | Claude reads the latest state of an Initiative file and summarizes |
| `continue <initiative-name> <loop>` | Re-enter a loop on an existing initiative (e.g., `continue nav-redesign design`) |

### Example session

```
> plan: redesign the empty states for the dashboard

[Claude proposes initiative name "dashboard-empty-states", confirms,
 creates Figma file, runs research (~60s), writes findings, drafts spec]

Open questions:
1. Should empty states include illustrations or stay type-only?
2. CTA priority: primary or secondary?

> 1: type-only is fine. 2: secondary — we don't want to over-promote actions when there's nothing there yet.

[Claude updates spec, logs decisions, finishes PLAN]

PLAN complete for `dashboard-empty-states`. Run `design: dashboard-empty-states` when ready.
```

---

## How decisions get made

Three channels (Claude picks the right one for each situation):

1. **Sync (terminal)** — Claude asks you a direct question; you answer; it proceeds. Use this for anything blocking the loop.
2. **Async (Inbox page in System file)** — Claude noticed something but doesn't need an answer right now. Drops a question on the Inbox page. Next session, Claude reads the Inbox first and surfaces pending items.
3. **Decisions Log** — Every weigh-in (sync or async) gets logged to a Decisions page. Append-only. This is what makes the workflow survive across sessions — Claude reads recent entries before any loop so it doesn't re-litigate settled questions.

You don't need to manage any of this manually. Claude does it.

---

## Cost monitoring

Perplexity calls are pay-as-you-go (your Pro subscription gives ~$5/mo credit). Claude logs every call to `.perplexity-usage.log` (gitignored).

Sanity check spend any time:

```bash
tail -50 .perplexity-usage.log
```

Or get this week's total:

```bash
awk -v cutoff="$(date -v-7d -u +%Y-%m-%dT%H:%M:%SZ)" \
  '$1 >= cutoff { sum += $5 } END { print "Last 7 days: $" sum }' \
  .perplexity-usage.log
```

Claude also surfaces this at loop start if spend is climbing.

---

## Troubleshooting

**"Figma MCP says I need a Full seat to write."** You're on a Dev seat. Fix in Figma → Settings → Plans.

**"Perplexity MCP not found."** Re-run the install command from setup step 2. Verify with `claude mcp list`.

**"Claude is writing to Figma but I don't see anything."** Refresh the Figma file in your browser. The MCP writes through the API, not your live editor session.

**"`use_figma` failed with 20 KB error."** Claude should auto-split and retry. If it fails twice, ask Claude to break the content into smaller chunks manually. This is a beta limit and may go away.

**"Two of us editing the same Figma page is causing weird state."** Claude announces "Writing to <page> now" before each write. Wait the ~10s announced before re-editing.

---

## What's NOT in scope

- Multi-collaborator review workflows (this is a you+Claude setup)
- Code Connect / design-to-code generation (different problem)
- Cross-team design system propagation
- Automated visual regression of Figma frames

If you need any of these, propose them as a new spec — don't bolt them on to this workflow.
````

- [ ] **Step 3: Verify content**

Run: `grep -c "## One-time setup" README.md && grep -c "## Day-to-day usage" README.md`
Expected: `1` then `1`

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs: add operator README with setup, daily usage, and troubleshooting"
```

---

## Task 10: [MANUAL — Kevin runs] Install Perplexity MCP and verify

This task requires Kevin's hands. Claude can guide but cannot complete.

**Prerequisites:**
- Active Perplexity Pro subscription (or willingness to pay-as-you-go)
- Terminal access in a directory where `claude` is available

- [ ] **Step 1: Open the Perplexity API console**

Kevin: open [console.perplexity.ai](https://console.perplexity.ai). Sign in. Create a new API key. Copy it to clipboard.

- [ ] **Step 2: Install the MCP**

Kevin runs in terminal (paste the key in place of `<KEY>`):

```bash
claude mcp add perplexity --env PERPLEXITY_API_KEY="<KEY>" -- npx -y @perplexity-ai/mcp-server
```

Expected output: a confirmation that the MCP was added.

- [ ] **Step 3: Verify install**

Run: `claude mcp list`
Expected: line containing `perplexity` and the npm package name.

- [ ] **Step 4: Smoke-test the MCP**

In a new Claude Code session in any directory, prompt:

```
Use the perplexity_ask tool to answer: "What is the current major version of React, as of May 2026?"
```

Expected: Claude calls `perplexity_ask` (you'll see the tool call), gets a response with the answer, returns it. The fact that the call succeeds at all is the smoke-test goal — answer correctness is secondary.

- [ ] **Step 5: Confirm cost is being tracked at Perplexity's end**

Kevin: refresh the Perplexity console. The recent call should show in usage. This confirms billing is wired correctly.

- [ ] **Step 6: Mark this task done**

Tell Claude: "Perplexity MCP installed and verified."

(No commit — this is environment setup, not a code change.)

---

## Task 11: [MANUAL — Kevin] Create Figma System file and update CLAUDE.md

This task requires Kevin to create the Figma file. Claude can write the URL replacement once Kevin provides it.

- [ ] **Step 1: Create the System file**

Kevin in Figma:
1. New Figma Design file. Name: `Design System` (or your preferred name).
2. Create six pages (in this order):
   - `Foundations`
   - `Components`
   - `Decisions Log`
   - `Audit Log`
   - `Inbox`
   - `Index`

If you have an existing design system you want to use as the System file, you can use that — just confirm the six governance pages exist (add any that are missing).

- [ ] **Step 2: Copy the file URL**

Kevin: copy the Figma URL of the file (browser address bar — should look like `https://www.figma.com/design/<fileKey>/Design-System`).

- [ ] **Step 3: Update CLAUDE.md**

Kevin or Claude: replace `<SYSTEM_FILE_URL>` in `CLAUDE.md` with the actual URL.

If Claude is doing it, use Edit:

```
old_string: URL: `<SYSTEM_FILE_URL>`
new_string: URL: `<actual URL Kevin provided>`
```

- [ ] **Step 4: Verify**

Run: `grep "<SYSTEM_FILE_URL>" CLAUDE.md`
Expected: no matches (the placeholder has been replaced).

- [ ] **Step 5: Commit**

```bash
git add CLAUDE.md
git commit -m "config: set System file URL to actual Figma file"
git push
```

---

## Task 12: End-to-end smoke test — run PLAN loop on a throwaway initiative

This is the real test of the entire workflow. If this passes, the system is operational.

**Prerequisites:** Tasks 10 and 11 complete.

- [ ] **Step 1: Start a fresh Claude Code session in `design-op/`**

```bash
cd ~/Documents/web/design-op
claude
```

- [ ] **Step 2: Run the smoke-test PLAN intent**

In the Claude session, prompt:

```
plan: smoke test — explore patterns for keyboard shortcut affordances
```

Claude should:
1. Read CLAUDE.md and recognize the `plan:` prefix.
2. Propose initiative name (something like `keyboard-shortcut-affordances`); confirm in one line.
3. Run loop-start checklist (read recent Decisions Log + Inbox; both empty on first run, that's fine).
4. Create Figma Initiative file with six pages.
5. Update `initiatives.md` and System file `Index` page.
6. Write `Brief` page.
7. Run `perplexity_research` once + `perplexity_search` 1–3 times. Announce slow call.
8. Write findings to `Research` page as multiple text frames.
9. Draft `Spec` page with frames.
10. Surface open questions in terminal.

- [ ] **Step 3: Answer the open questions**

Kevin: respond to each question in the terminal. Anything reasonable — this is just a smoke test. Claude should update Spec, log decisions to `Decisions` page.

- [ ] **Step 4: Verify Figma artifacts**

Open the Initiative file URL Claude reported. Confirm:

- [ ] All six pages exist (`Brief`, `Research`, `Spec`, `Designs`, `Build`, `Decisions`)
- [ ] `Brief` page has goal + System file link
- [ ] `Research` page has multiple text frames (patterns, pitfalls, accessibility, references, sources)
- [ ] `Spec` page has frames for `Problem`, `Goals`, `Non-goals`, `Acceptance criteria`, `Open questions`
- [ ] `Open questions` frame shows resolved status for the answers you gave
- [ ] `Decisions` page has at least one entry per question you answered

Open the System file. Confirm:
- [ ] `Index` page has a row for the new initiative
- [ ] `Inbox` page may or may not have items (depends on what Claude noticed; either is OK for smoke test)

- [ ] **Step 5: Verify repo artifacts**

Run: `cat initiatives.md`
Expected: a row for the smoke-test initiative with status `designing` (PLAN updates it from `planning` → `designing` on completion), today's date, real Figma URL.

Run: `cat .perplexity-usage.log`
Expected: at least 2 lines (one `perplexity_research` + at least one `perplexity_search`).

- [ ] **Step 6: Verify cost tracking**

Run:
```bash
awk '{ sum += $5 } END { print "Total est cost: $" sum }' .perplexity-usage.log
```
Expected: a small dollar amount (likely $0.10–$0.30 for one PLAN loop).

- [ ] **Step 7: Archive the smoke-test initiative**

Update its row in `initiatives.md` status to `archived`. Optionally delete the Figma file if you don't want it cluttering your drive.

```bash
git add initiatives.md
git commit -m "test: smoke-test PLAN loop verified, archive test initiative"
git push
```

- [ ] **Step 8: Mark the workflow operational**

If all of the above passed, the workflow is operational. You can now run real `plan:` / `design:` / `build:` / `audit:` loops.

If any step failed, investigate before relying on the workflow. Common failure modes:
- Perplexity tool not called → check MCP install (Task 10 step 3)
- Figma write failed → check Full seat status (Task 10) and System file URL is correct (Task 11)
- 20 KB error not handled → check that CLAUDE.md write rules section exists (Task 4)
- Decisions Log not appended → check that Initiative file `Decisions` page was created (Task 5 step 2)

---

## Done

When Task 12 passes, the design-op workflow is live. Use it day-to-day per the README.
