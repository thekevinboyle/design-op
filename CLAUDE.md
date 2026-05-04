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

---

## PLAN loop

**Trigger:** `plan: <one-sentence description>` or `plan: <description> (initiative: <kebab-name>)`

If no initiative name is given, propose one in kebab-case based on the description; confirm with Kevin in terminal in one line; continue with the chosen name.

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
