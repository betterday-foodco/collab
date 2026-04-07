# BetterDay Collab Board

A lightweight project coordination dashboard for the BetterDay engineering team. Tracks branches, tasks, and async messages between Conner and Gurleen.

**Live dashboard:** https://betterday-foodco.github.io/collab/

## What this is

A static HTML page hosted on GitHub Pages that renders the contents of `data.json`. The page is updated by Claude Code instances running in either Conner's or Gurleen's terminal — both sessions follow the same protocol defined in `HOW_TO_USE.md`.

## Files

| File | Purpose | Who edits it |
|---|---|---|
| `index.html` | The dashboard renderer | Rarely (only to fix bugs) |
| `data.json` | All state — branches, tasks, messages | Claude (via either session) |
| `HOW_TO_USE.md` | Protocol that Claude instances follow | Updated when the protocol changes |
| `README.md` | This file | Rarely |

## How to add an update

Just ask Claude:

- *"Update the collab board with the changes from PR #X"*
- *"Add a task for Gurleen to review the par-cart frontend"*
- *"Send Gurleen a message saying the staging deploy is broken"*
- *"Check the collab board for anything I need to do"*

Claude will pull the repo, edit `data.json`, commit, and push.

## How Gurleen can edit it directly

If Claude isn't available, anyone with write access can edit `data.json` directly via the GitHub web UI:

1. Open https://github.com/betterday-foodco/collab/edit/main/data.json
2. Make changes (it's plain JSON)
3. Bump `meta.last_updated` to the current timestamp and set `meta.updated_by` to your name
4. Commit at the bottom of the page

The dashboard updates within ~30 seconds of pushing.
