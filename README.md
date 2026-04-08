# BetterDay Project Scope

A living, nested project map for the BetterDay platform build — customer-facing storefront, subscription engine, admin console, and everything else. Edited by Claude Code on both Conner's and Gurleen's machines, viewed in a browser-based dashboard.

**Live dashboard:** https://betterday-foodco.github.io/collab/project-scope/

## What this repo contains

```
.
├── CLAUDE.md                          Protocol auto-loaded by Claude Code sessions in this repo
├── README.md                          This file
├── index.html                         Redirect from repo root to /project-scope/
├── .github/workflows/protocol-check.yml  CI that enforces commit-message + data.json rules on every push
└── project-scope/
    ├── data.json                      The tree of categories, modules, submodules, components
    ├── index.html                     Dashboard renderer (Monday-style nested tree, brand palette, direct GitHub editing)
    └── HOW_TO_USE.md                  Schema + edit protocol for this specific board
```

## How it works

- **data.json** is the source of truth for the scope tree — 499+ leaves across 22 top-level categories (Identity & Auth, Cart, Checkout, Subscriptions, Coupons, Platform & Ops, etc.), each with status, priority, tech notes, dependencies, and free-form notes.
- **index.html** is a single-file dashboard that renders data.json with a collapsible tree, search, priority filter, side-panel detail view, click-to-edit status pills, and a direct-to-GitHub sync path.
- **CLAUDE.md** is auto-loaded by Claude Code whenever a session works inside this repo. It contains the Pre-Edit Protocol — pull first, identify yourself, read HOW_TO_USE.md, state your plan, use the correct commit format. Both Conner's and Gurleen's sessions get it in context automatically.
- **GitHub Action** validates every push: commit messages must start with `project-scope:` and end with `(via Conner)` or `(via Gurleen)`, and data.json must have valid meta + unique IDs.

## How to update the scope

### From Claude

Just ask your Claude Code session:

- *"Mark auth.phone.twilio.send as designed"*
- *"Add a submodule under Rewards for birthday bonus"*
- *"What's P1 and not done?"*
- *"Deep-dive the Checkout category"*

Claude will follow CLAUDE.md protocol automatically (pull, identify, read, edit, commit with the right format, push).

### From the browser

Open the dashboard, click the gear icon, paste a fine-grained GitHub PAT with Contents: Read/Write on this repo, and click Save. After that, every status change you make in the UI commits directly to GitHub. Setup guide lives in `project-scope/HOW_TO_USE.md`.

## Who this is for

Two people: Conner and Gurleen. Both have separate Claude Code sessions that push to this repo. The protocol is designed to prevent the two sessions from stepping on each other — pull before every edit, never rename IDs, append-only notes with dated prefixes, one logical change per commit.

## History

This repo began on 2026-04-06 as a collab board for day-to-day coordination (branches, tasks, messages between Conner and Gurleen). On 2026-04-08 it was repurposed into a dedicated project-scope dashboard as the BetterDay platform build grew beyond what ad-hoc task tracking could support. The old collab board content is preserved in git history and the active Flask → culinary-ops migration work was archived into `project-scope/data.json` as the `flask-migration` category.
