# How to Use This Collaboration Board

**This file is the instruction manual for Claude Code instances helping Conner and Gurleen coordinate work on the BetterDay culinary-ops project.**

If you are a Claude Code instance and the user has asked you to update, check, or interact with the "collab board," "coordination board," "project board," "betterday-collab," or any similar phrase, follow the protocol below.

---

## Repo location

- **GitHub:** https://github.com/betterday-foodco/collab
- **Live dashboard:** https://betterday-foodco.github.io/collab/
- **Local clone (recommended path):** `/Users/us/Downloads/betterday-collab` (Conner's machine)

If the local clone doesn't exist, clone it first:
```bash
cd /Users/us/Downloads && git clone https://github.com/betterday-foodco/collab.git betterday-collab
```

---

## Identify which session you are

Before editing, check whose machine you're on:
```bash
cd /path/to/betterday-collab && git config user.name
```

| Output | You are |
|---|---|
| `Conner Kadziolka` | **Conner's session** — set `updated_by` to `"Claude (Conner)"` |
| `Gurleen Kaur` (or similar) | **Gurleen's session** — set `updated_by` to `"Claude (Gurleen)"` |
| Anything else | Ask the user who they are before posting messages on their behalf |

---

## When to update the board

### Proactively (without being asked) when:
- You just opened, merged, or closed a PR in any betterday-foodco repo
- A task you've been tracking in the current session is completed
- A significant blocker is encountered that the other person should know about

### When the user explicitly asks:
- "Update the board with…"
- "Post to the collab board…"
- "Add a task for…"
- "Send Gurleen / Conner a message saying…"
- "Mark task X as done"

### When the user asks to check:
- "Check the board"
- "Anything I need to do?"
- "Any new messages?"
- "What's on the collab board?"

---

## How to UPDATE the board (write protocol)

**Always follow these steps in order:**

1. **Pull latest first.** No exceptions.
   ```bash
   cd /Users/us/Downloads/betterday-collab && git pull
   ```

2. **Read `data.json`** to understand current state. Use the Read tool, not cat.

3. **Edit `data.json` only.** Never edit `index.html` for content updates — it's a static renderer.

4. **Update `meta` block** at the top of `data.json`:
   - `last_updated` → current ISO 8601 timestamp with offset
   - `updated_by` → `"Claude (Conner)"` or `"Claude (Gurleen)"` based on session identity

5. **Make the actual change** (add a branch, add/update a task, append a message). See the Schema section below.

6. **Commit and push** with a clear message:
   ```bash
   git add data.json
   git commit -m "update: <one-line summary of what changed>"
   git push
   ```

7. **Confirm to the user** what you posted, with the live URL.

---

## How to CHECK the board (read protocol)

1. **Pull latest** so you see what the other side has posted.
2. **Read `data.json`**.
3. **Filter for relevance** to the current user:
   - Tasks where `assigned_to` matches the current user
   - Messages where `to` matches the current user (especially `read: false` ones)
   - Branches where `owner` matches the current user
4. **Summarize** what's new, what needs action, and what's pending review.
5. **Mark messages as read** by setting `read: true` if the user acknowledges them. Then commit + push.

---

## Data schema (`data.json`)

```jsonc
{
  "meta": {
    "last_updated": "ISO 8601 string with timezone offset",
    "updated_by":   "Claude (Conner) | Claude (Gurleen) | Conner | Gurleen",
    "version":      1
  },
  "people": ["Conner", "Gurleen", "Claude"],

  "projects": [
    {
      "id":          "proj-001",
      "title":       "Big initiative title",
      "description": "1–3 sentence summary of why this project exists and the end state",
      "owner":       "Conner | Gurleen",
      "status":      "in_progress | paused | done",
      "created":     "YYYY-MM-DD",
      "subtasks": [
        {
          "id":            "sub-001",
          "title":         "Imperative subtask description",
          "status":        "pending | in_progress | blocked | done",
          "assigned_to":   "Conner | Gurleen | Claude",
          "linked_branch": "feature/branch (optional)",
          "linked_pr":     7,
          "depends_on":    ["sub-id", "sub-id"],
          "notes":         "Optional context"
        }
      ]
    }
  ],

  "branches": [
    {
      "id":               "feature/branch-name",
      "title":            "Human-readable title",
      "repo":             "betterday-foodco/repo-name",
      "pr_number":        7,
      "pr_url":           "https://github.com/...",
      "status":           "open | merged | closed | draft",
      "owner":            "Conner | Gurleen",
      "summary":          "1–2 sentences on what this branch does",
      "what_works":       ["bullet", "bullet"],
      "what_needs_review":["bullet", "bullet"],
      "created":          "YYYY-MM-DD"
    }
  ],

  "tasks": [
    {
      "id":            "task-001",
      "title":         "Short imperative task description",
      "assigned_to":   "Conner | Gurleen | Claude",
      "status":        "pending | in_progress | blocked | done",
      "linked_branch": "feature/branch-name (optional)",
      "linked_pr":     7,
      "blocked_by":    "task-id (optional)",
      "notes":         "Optional context or instructions",
      "created":       "YYYY-MM-DD",
      "created_by":    "Conner | Gurleen | Claude"
    }
  ],

  "messages": [
    {
      "id":        "msg-001",
      "from":      "Conner | Gurleen | Claude",
      "to":        "Conner | Gurleen",
      "subject":   "Short subject line",
      "body":      "Full message body. Markdown allowed.",
      "timestamp": "ISO 8601 string",
      "read":      false
    }
  ]
}
```

### ID conventions
- Tasks: `task-NNN` — auto-increment from highest existing `task-` ID
- Messages: `msg-NNN` — auto-increment from highest existing `msg-` ID
- Projects: `proj-NNN` — auto-increment from highest existing `proj-` ID
- Subtasks: `sub-NNN` — auto-increment **across all projects**, not per-project (so `depends_on` references stay globally unique)
- Branches: use the actual git branch name as the ID

---

## Working with Projects

Projects are the top-level coordination unit on the board. Each project has its own subtask list with dependency tracking. Use them when the user asks you to "map out a project," "plan out X," or describes work that has clear sequential phases.

### When to add a project
- The user describes a multi-step initiative with > 3 distinct steps
- Work spans multiple PRs or sessions
- There are dependencies between steps (X must happen before Y)

### When to add subtasks instead of tasks
- The work is part of an existing project on the board → add as subtasks
- Standalone work (PR review, one-off message reply) → use `tasks[]`

### Marking a subtask done
The user (or another Claude session) may click the checkbox in the dashboard. That **does not** persist — it copies a "mark done" prompt to clipboard for them to paste back into a Claude session. When you receive that prompt:
1. `git pull`
2. Read `data.json`, find the subtask by ID inside `projects[].subtasks[]`
3. Set its `status` to `"done"`
4. Update `meta.last_updated` and `meta.updated_by`
5. Commit + push with message `update: mark sub-NNN as done`

### Dependencies
`depends_on` is an array of sibling subtask IDs. The dashboard renders a subtask as **blocked** when any of its dependencies is not yet `done`. When suggesting next steps, prefer subtasks whose dependencies are all satisfied.

---

## Conflict prevention rules (read carefully)

These rules exist because two Claude sessions (Conner's and Gurleen's) may both try to update the board on the same day. Following them strictly avoids merge conflicts and lost work.

1. **Pull before every edit.** Always. Even if you just pulled 5 minutes ago.
2. **Append, don't overwrite.** Messages are append-only — never delete or rewrite a message even if it looks outdated. Tasks can have their `status` changed but their `title` and `id` should not be edited.
3. **One logical change per push.** Don't batch unrelated updates into a single commit — it makes history harder to scan.
4. **If `git push` fails because of a non-fast-forward**, run `git pull --rebase`, resolve any conflict in `data.json`, then push again.
5. **Don't delete entries created by the other person** without explicit user permission.
6. **Don't edit `index.html` for content updates.** Only edit it to fix actual bugs in the renderer.

---

## What's NOT allowed in this repo

This repo is **public** — anyone on the internet can read it. Therefore:

- **No secrets, API keys, tokens, passwords, or PII** in `data.json`
- **No internal customer information** (real client names beyond "Demo Inc." style placeholders are fine; financial details, addresses, etc. are not)
- **No proprietary code snippets** that haven't been committed to the public repo already
- **No unflattering remarks about people** — anyone can read this, including future hires

If you're unsure whether something belongs on the board, ask the user first.

---

## Example prompts the user might give you

| User says | You should |
|---|---|
| "Add PR #8 to the board" | Pull, fetch PR #8 details via `gh pr view 8`, add a branch entry, commit, push |
| "Mark task-003 as done" | Pull, find task-003, set `status: "done"`, commit, push |
| "Map out a project for X with subtasks" | Plan the phases, draft a `proj-NNN` entry with sequential `sub-NNN` subtasks and `depends_on` chains, commit, push |
| "Mark sub-005 as done" | Pull, find sub-005 inside `projects[].subtasks[]`, set `status: "done"`, commit, push |
| "Send Gurleen a message that the staging deploy is broken" | Pull, append a message with `from: "Conner"`, `to: "Gurleen"`, commit, push |
| "Check the board" | Pull, summarize unread messages + pending tasks for the current user |
| "Anything new from Gurleen?" | Pull, list any branches/tasks/messages where she is `from`/`owner`/`created_by` since last check |

---

## When you're unsure

If the user gives you an instruction that doesn't fit cleanly into the protocol above, ask them what they want before editing `data.json`. It's better to ask once than to push wrong state that the other side then has to clean up.
