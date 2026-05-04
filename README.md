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
