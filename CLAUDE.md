# CLAUDE.md — Required Pre-Edit Protocol for `betterday-collab`

**If you are a Claude Code instance and you are about to read, edit, commit, or push ANY file in this repository, you MUST complete the protocol checklist below before making any changes. This file is auto-loaded by Claude Code when the working directory is inside this repo, so there is no excuse for skipping it.**

This repo contains two boards:

| Board | Data file | Protocol document |
|---|---|---|
| **Collab board** (day-to-day coordination) | `data.json` | `HOW_TO_USE.md` |
| **Project scope** (platform-wide module map) | `project-scope/data.json` | `project-scope/HOW_TO_USE.md` |

Both follow the same core rules. The specific schemas differ.

---

## 🛑 Pre-edit checklist — complete every single time

Before making any edit to any file in this repo, you must:

1. **Pull first, always.**
   ```bash
   cd /Users/us/Downloads/betterday-collab && git pull
   ```
   No exceptions. Even if you pulled 30 seconds ago. Even if you think nothing has changed. Pull anyway. This is the single most important rule — without it, two Claude sessions will overwrite each other.

2. **Identify yourself.**
   ```bash
   cd /Users/us/Downloads/betterday-collab && git config user.name
   ```
   - `Conner Kadziolka` → you are **Conner's session**. Use `"Claude (Conner)"` in `updated_by` fields and `(via Conner)` in commit messages.
   - Anything else → you are **Gurleen's session**. Use `"Claude (Gurleen)"` / `(via Gurleen)`.
   - If you cannot determine the identity, stop and ask the user.

3. **Read the right protocol document.**
   - Editing files outside `project-scope/` → read `HOW_TO_USE.md` in the repo root.
   - Editing files inside `project-scope/` → read `project-scope/HOW_TO_USE.md`.
   - Editing both → read both.

4. **Read the file you are about to edit.** Use the `Read` tool, not `cat`. Understand the current state before you change it.

5. **State your plan.** Before you make any changes, tell the user (in plain English) exactly:
   - Which file(s) you will edit
   - Which field(s), node(s), or record(s) you will change
   - Which `meta` fields you will update
   - What your commit message will be
   Get implicit or explicit acknowledgement, then proceed.

---

## ✍️ Edit rules

These apply to any data file edit (root `data.json` OR `project-scope/data.json`):

1. **Update the `meta` block** of the file you edit:
   - `meta.last_updated` → current ISO 8601 timestamp with timezone offset (e.g. `2026-04-08T14:30:00-06:00`)
   - `meta.updated_by` → `"Claude (Conner)"` or `"Claude (Gurleen)"`

2. **Never rename an existing `id`.** IDs are referenced by dependencies and pending queues. Rename the `name` field instead.

3. **Never delete a node created by the other person** without the user explicitly telling you to. Use `status: "skipped"` to drop scope while keeping the record visible.

4. **Append to notes with a dated prefix** when adding to existing text:
   ```
   2026-04-08 (Conner): Settled on argon2id after reviewing OWASP.
   2026-04-09 (Gurleen): Also need to rotate refresh tokens on password change.
   ```

5. **One logical change per commit.** Don't batch unrelated edits. If the user asks for multiple unrelated changes, make multiple commits.

6. **Messages are append-only.** Never delete or rewrite a message that was already committed, even if it looks outdated.

---

## 📝 Commit rules

1. **Commit message format** — these are enforced by a GitHub Action on push:

   Root `data.json` edits:
   ```
   update: <one-line summary>
   ```
   or, for project/subtask changes:
   ```
   <verb>: <one-line summary>
   ```

   Project-scope edits (anything in `project-scope/`):
   ```
   project-scope: <one-line summary> (via Conner|Gurleen)
   ```
   Examples:
   ```
   project-scope: mark auth.phone.twilio.send as designed (via Conner)
   project-scope: add Express Checkout submodule under Checkout (via Gurleen)
   project-scope: sync 3 changes from dashboard (via Conner)
   ```

2. **HEREDOC for multi-line commit messages.** Always pass multi-line commit messages via HEREDOC to preserve formatting:
   ```bash
   git commit -m "$(cat <<'EOF'
   project-scope: add coupon engine submodules (via Conner)

   - Add coupons.engine.rules with 8 restriction components
   - Add coupons.engine.redemption with live-verify logic
   - Mark all as to-build, P2
   EOF
   )"
   ```

3. **Never use `--no-verify`, `--no-gpg-sign`, or `--amend`.** Create a new commit instead of amending. If a hook rejects your commit, fix the issue and make a new commit.

4. **Always push after committing** unless the user explicitly asks you to stage work for later.

---

## 🔀 Conflict resolution

If `git push` is rejected because of a non-fast-forward (someone else committed first):

1. Run `git pull --rebase`.
2. If `data.json` or `project-scope/data.json` has a merge conflict:
   - Preserve **both** sets of changes
   - For array fields (like `categories`, `modules`, `subtasks`), keep both new entries
   - For scalar fields edited by both sides, prefer the other person's edit and append your edit as a note
   - For `meta.last_updated`, use the later timestamp
3. Commit the merge resolution with a message like `merge: resolve conflict in data.json`.
4. Push again.
5. Tell the user what you merged.

**Never `git reset --hard` or `git push --force`** to resolve conflicts. You will destroy the other person's work.

---

## 🚫 Absolute don'ts

- **Don't edit `index.html`** for content changes — it is a static renderer; update `data.json` instead.
- **Don't commit secrets** — no API keys, tokens, passwords, or PII. The repo is public.
- **Don't skip the pull.** Ever.
- **Don't rename ids.** Ever.
- **Don't delete the other person's work without explicit permission.**
- **Don't batch unrelated changes into one commit.**

---

## 🪝 Applying `[PROJECT-SCOPE UPDATE]` or `[COLLAB UPDATE]` blocks from the dashboard

When the user pastes a block like:
```
[PROJECT-SCOPE UPDATE]
Apply the following changes to betterday-collab/project-scope/data.json and push.
...
```

Treat it as a formal instruction to follow this protocol end-to-end:
1. Pull
2. Read the relevant HOW_TO_USE.md
3. Read the target data.json
4. Apply the changes field-by-field from the block
5. Update meta
6. Commit with the required format (`project-scope: ... (via <name>)`)
7. Push
8. Report back which nodes changed and the live URL

**Do not skip steps. Do not improvise. The block is explicit — follow it exactly.**

---

## ✅ Acknowledgement

By reading this file, you are acknowledging that every edit you make in this repo follows the protocol above. If you find yourself about to skip a step, stop and re-read this file. The rules exist because two Claude sessions share this repo, and they will step on each other if you don't follow them.
