# Project Scope Dashboard — Protocol

**Sub-board under `betterday-foodco/collab`. A Claude-editable, browser-viewable nested project map for the entire BetterDay platform build.**

> ⚠️ **You must read `CLAUDE.md` at the repo root before editing anything in this folder.** That file is auto-loaded by Claude Code and contains the Pre-Edit Protocol (pull first, identify yourself, commit format, conflict resolution) that applies to every file in this repo. This document is the schema-specific layer on top of that protocol.

If you are a Claude Code instance and the user has asked you to update, check, or interact with the "project scope," "scope board," "project map," "scope dashboard," or any similar phrase, follow this protocol.

This file sits alongside the existing collab-board `HOW_TO_USE.md`. The two boards share a repo but have different purposes:

| Board | Purpose | Data file |
|---|---|---|
| **Collab board** (root) | Day-to-day coordination: branches, tasks, messages, subtasks | `data.json` |
| **Project scope** (this folder) | Long-term platform map: every module, submodule, and leaf component | `project-scope/data.json` |

---

## Locations

- **GitHub:** https://github.com/betterday-foodco/collab/tree/main/project-scope
- **Live dashboard:** https://betterday-foodco.github.io/collab/project-scope/
- **Local clone:** `/Users/us/Downloads/betterday-collab` (Conner's machine)

---

## Identify which session you are

Same rule as the root collab board:

```bash
cd /Users/us/Downloads/betterday-collab && git config user.name
```

| Output | You are |
|---|---|
| `Conner Kadziolka` | **Conner's session** — mark commits `(via Conner)`, set `updated_by: "Claude (Conner)"` |
| `Gurleen Kaur` (or similar) | **Gurleen's session** — mark commits `(via Gurleen)`, set `updated_by: "Claude (Gurleen)"` |
| Anything else | Ask the user who they are before editing |

---

## When to update the scope board

### Proactively (without being asked) when:
- A component is completed — flip its status to `done`
- Work begins on a component — flip to `in-progress`
- A blocker is identified — flip to `blocked` and add a note
- A new module/submodule/component is identified in conversation — add it with `to-build`

### When the user explicitly asks:
- "Update the scope board…"
- "Mark X as done"
- "Add a new component under Y called Z"
- "What's blocked in the auth module?"
- "Apply this [PROJECT-SCOPE UPDATE] block…"

---

## How to UPDATE (write protocol)

1. **Pull latest first. Always.**
   ```bash
   cd /Users/us/Downloads/betterday-collab && git pull
   ```

2. **Read `project-scope/data.json`** with the Read tool.

3. **Edit `project-scope/data.json` only.** Do not touch `project-scope/index.html` unless fixing a renderer bug. Do not touch the root `data.json` (that's the other board).

4. **Update the `meta` block** at the top of `project-scope/data.json`:
   - `last_updated` → current ISO 8601 with timezone offset
   - `updated_by` → `"Claude (Conner)"` or `"Claude (Gurleen)"`

5. **Make the change** following the schema below.

6. **Commit and push**, using this message format:
   ```
   project-scope: <one-line summary> (via Conner|Gurleen)
   ```
   Examples:
   - `project-scope: mark auth.phone.twilio.send as designed (via Conner)`
   - `project-scope: add Express Checkout submodule under Checkout (via Gurleen)`
   - `project-scope: add tech notes to cart.pricing.engine.compute (via Conner)`

7. **Confirm to the user** what changed, with the live URL.

---

## How to CHECK (read protocol)

1. **Pull latest.**
2. **Read `project-scope/data.json`.**
3. **Answer the user's question** by traversing the category → module → submodule → component tree.
4. Useful filters:
   - "What's P1 and not done?" → leaves where `priority === 1` and `status !== "done"`
   - "What is `<category>` blocked on?" → leaves where `status === "blocked"` under that category
   - "What's next?" → leaves where all `dependencies` are `done` and status is `to-build` or `designed`

---

## Level 2 — Direct browser editing (live)

The dashboard now supports **direct GitHub commits from the browser**. A user clicks a status pill, the change is queued in localStorage, and when they hit "Sync" the dashboard:

1. Fetches the latest `project-scope/data.json` from GitHub via the Contents API (gets the current `sha`)
2. Applies pending changes on top of the fetched version (not the stale local copy)
3. Updates `meta.last_updated` and `meta.updated_by` to `"Claude (<identity>)"`
4. Builds a commit message in the format `project-scope: <summary> (via <identity>)`
5. PUTs the new content back to GitHub with the `sha`
6. On 409 conflict, auto-pulls latest and prompts the user to retry

This means **clicks from the browser follow the same commit protocol as Claude edits**. The GitHub Action (`.github/workflows/protocol-check.yml`) enforces the commit message format server-side — any commit missing the `(via Conner|Gurleen)` suffix is rejected.

Users set up direct editing by clicking the ⚙ settings icon in the dashboard header and pasting a fine-grained GitHub PAT. The token is scoped to this repo only, stored in the browser's localStorage, and never leaves the device except to hit the GitHub API directly.

If no token is set, the dashboard falls back to the **copy-for-Claude modal** below — the same user-paste-into-Claude flow that existed before Level 2.

---

## Applying `[PROJECT-SCOPE UPDATE]` blocks from the dashboard

The browser dashboard has a "Sync pending changes" button that generates a block formatted like this:

```
[PROJECT-SCOPE UPDATE]
Apply the following changes to betterday-collab/project-scope/data.json and push.
Use the existing HOW_TO_USE protocol: pull first, update meta.last_updated and meta.updated_by,
commit with message starting 'project-scope:' and including '(via Conner)'.

• auth.phone.twilio.send — "Send OTP endpoint"
    status: "in-progress"
    started_at: "2026-04-08T10:30:00.000Z"
• cart.pricing.engine.compute — "Compute cart totals"
    status: "done"
    completed_at: "2026-04-08T11:15:00.000Z"
```

When the user pastes this into a Claude session:

1. **Pull latest.**
2. **Read `project-scope/data.json`.**
3. For each bullet, find the node by `id` (depth-first search through `categories[].modules[].submodules[].components[]`) and apply the listed field changes.
4. **Update `meta.last_updated` and `meta.updated_by`.**
5. **Commit and push** with a message summarizing all the changes:
   ```
   project-scope: sync N changes from dashboard (via Conner)
   ```
6. **Report back** which nodes changed, which didn't (if a node wasn't found), and a one-line summary of the impact.

---

## Data schema (`project-scope/data.json`)

### Top-level shape
```jsonc
{
  "meta": {
    "title":        "BetterDay Platform — Project Scope",
    "subtitle":     "…",
    "last_updated": "ISO 8601 with offset",
    "updated_by":   "Claude (Conner) | Claude (Gurleen)",
    "version":      1,
    "session":      1
  },
  "legend": {
    "priorities": { "1": {label, color, description}, ... },
    "statuses":   { "done": {...}, "in-progress": {...}, ... }
  },
  "categories": [Category, ...],
  "journeys":   []   // Populated in session 3
}
```

### Category → Module → Submodule → Component hierarchy

```jsonc
{
  "id": "auth",                       // snake-case top-level id
  "name": "Identity & Auth",
  "priority": 1,                      // 1..5 (aspirational; leaves drive the real rollup)
  "description": "Plain English summary",
  "tech_notes": "Stack/architecture notes for the whole category",
  "modules": [
    {
      "id": "auth.foundation",        // dot.path.id
      "name": "Auth Foundation",
      "priority": 1,
      "description": "…",
      "submodules": [
        {
          "id": "auth.foundation.schema",
          "name": "Prisma schema — auth tables",
          "priority": 1,
          "components": [              // leaves live here
            {
              "id":          "auth.foundation.schema.identity",
              "name":        "AuthIdentity model",
              "status":      "to-build",
              "priority":    1,
              "description": "One-line to paragraph",
              "tech": {
                "language":   "Prisma",
                "framework":  "NestJS",
                "database":   "Postgres (Neon)",
                "vendor":     "Twilio Verify",
                "tables":     ["AuthIdentity", "Session"],
                "endpoints":  ["POST /api/auth/phone/start"],
                "crons":      [],
                "notes":      "Freeform extra context"
              },
              "dependencies": ["other.leaf.id", "other.leaf.id"],
              "tags":         ["security", "foundation"],
              "notes":        "Freeform note area — editable from the dashboard",
              "started_at":   "ISO when work began (optional)",
              "completed_at": "ISO when finished (optional)"
            }
          ]
        }
      ]
    }
  ]
}
```

### ID conventions
- **Dot-path:** `auth.foundation.schema.identity` — category, module, submodule, component joined by dots
- **All lowercase.** Hyphens allowed inside a segment.
- **Stable.** Never rename an existing ID once created. Rename the `name` field instead. IDs are referenced from `dependencies` arrays and the dashboard's pending queue.

### Status vocabulary
Exactly these six values:
- `done` — built, tested, live
- `in-progress` — actively being worked on now
- `designed` — schema / demo / prior design exists, ready to implement
- `to-build` — identified, in scope, not started
- `blocked` — can't proceed until a dependency or decision resolves
- `skipped` — consciously dropped; kept visible for context

### Priority values
- `1` — Foundation. Must exist first.
- `2` — Core. Required for a functional platform.
- `3` — Launch polish. Required for a credible launch.
- `4` — Post-launch soon. Wanted within a month.
- `5` — Future. Out of scope for 2026.

### Dependency rules
- `dependencies` is an array of leaf IDs (or node IDs) that must be `done` before this leaf can move to `in-progress`.
- The dashboard currently shows dependencies as plain text, not visually enforced. Future sessions will add blocking state.

---

## Safety nets

These conventions prevent the two Claude sessions from stepping on each other:

1. **Pull before every edit.** Even if you just pulled five minutes ago.
2. **Never rename an existing `id`.** Rename the `name` field instead.
3. **Never delete a node created by the other person** without explicit user permission. Use `status: "skipped"` if scope is dropped.
4. **Append to `notes` fields with a dated prefix** when adding to an existing note:
   ```
   2026-04-08 (Conner): Decided on argon2id over bcrypt.
   2026-04-09 (Gurleen): Also rotate refresh tokens on password change.
   ```
5. **Commit messages always start with `project-scope:`** and end with `(via Conner)` or `(via Gurleen)` so the git log reads naturally and filters cleanly:
   ```bash
   git log --oneline --grep="project-scope:"
   ```
6. **One logical change per commit.** Don't batch a dozen unrelated status flips.
7. **On merge conflict in `data.json`:**
   - Run `git pull --rebase`
   - If both sides edited the same node, preserve both sets of changes (combine fields; concat notes with dated prefixes)
   - If either side added a new node, keep both
   - Push the merged result
8. **Never commit pending changes from your own browser dashboard without applying them via data.json first.** The browser localStorage is a scratchpad, not the source of truth.

---

## Adding new nodes — quick reference

### Add a new category
New top-level area of the platform. Add to `categories[]`:
```jsonc
{
  "id": "loyalty",
  "name": "Loyalty Stack",
  "priority": 3,
  "description": "…",
  "tech_notes": "…",
  "modules": []
}
```

### Add a new module to a category
Under `categories[].modules[]`.

### Add a new submodule to a module
Under `modules[].submodules[]`.

### Add a new component (leaf)
Under `submodules[].components[]`. Must include at least `id`, `name`, `status`, `priority`, `description`.

### Move a node
If the user asks to move a node to a different parent:
1. Copy the node (preserve its `id` — dot path is decorative, not structural)
2. Delete from old location
3. Insert at new location
4. Note the move in the commit message

---

## Example prompts the user might give

| User says | You should |
|---|---|
| "Mark Twilio Verify as in progress" | Pull, find `auth.phone.twilio.send` (and `.verify`), set status, push |
| "What's blocked?" | Pull, walk all leaves, filter `status === "blocked"`, summarize with notes |
| "What should I build next?" | Pull, find P1/P2 leaves with `status in (to-build, designed)` whose dependencies are all `done` |
| "Add Apple Pay to Express Checkout" | (Already there — confirm + point them to `checkout.express.apple`) |
| "Add a submodule under Rewards for tier system" | Pull, add `rewards.tiers` submodule with a few starter components, push |
| "Sync my pending changes" | Wait for user to paste the `[PROJECT-SCOPE UPDATE]` block, apply it |
| "What's in the Cart category?" | Pull, traverse `cart`, summarize by module and status |

---

## What's NOT allowed

- No secrets, API keys, tokens, passwords, PII in `data.json`
- No internal customer data
- No unflattering remarks about people — this repo is public
- No editing of `index.html` for content updates — it is a static renderer; use `data.json`

If you're unsure whether something belongs on the scope board or on the collab board, ask the user. Rule of thumb:

- **Collab board:** "this week's work," tasks, branches, messages, short-lived coordination
- **Scope board:** "the platform we're building," long-lived architectural map
