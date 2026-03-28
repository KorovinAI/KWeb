---
name: wrap
description: >
  End-of-session commit and push. Reviews all changes across client website
  submodules and the KWeb root repo, presents a summary, and on confirmation
  commits and pushes everything to GitHub.
  TRIGGER: /wrap, "I'm done for today", "wrap up", "session wrap",
  "commit and push everything", "push everything".
  Do NOT trigger on casual "done" mid-conversation.
---

# /wrap — Session Wrap

Commits and pushes all changes across the workspace. Run this when you're done working.

---

## Flow

### Step 1 — Scan for changes

Check all levels for uncommitted work:

**Client submodules:**
```bash
git submodule foreach --quiet 'changes=$(git status --short); if [ -n "$changes" ]; then echo "=== $name ==="; echo "$changes"; fi'
```

**Root repo:**
```bash
git status --short
```

If everything is clean at both levels:
> "Everything is clean. Nothing to commit or push."

Stop.

### Step 2 — Present change summary

Show the user a single, clear summary of everything that changed:

> **Changes this session:**
>
> **[client-name]** (submodule)
> - modified: index.html, styles.css
> - new: about.html
>
> **Root (KWeb)**
> - modified: governance/git-manager/SKILL.md, MEMORY.md
> - submodule ref: websites/[client-name]
>
> **Proposed commits:**
> 1. `[client-name]: [summary from diff]`
> 2. `session-wrap: [date] — [summary of root changes]`
>
> **Confirm to commit and push all?**

Wait for a single YES/confirmation before proceeding. If the user declines or wants changes, adjust.

### Step 3 — Commit (inside-out)

**3a. Client submodules first.**

For each submodule with changes:
```bash
cd websites/<client-name>
git add -A
git commit -m "<client-name>: <summary from diff>"
cd "<workspace-root>"
```

**3b. Root repo.**

Root will now show submodule reference updates plus any direct file changes:
```bash
git add -A
git commit -m "session-wrap: <date> — <summary of all root changes>"
```

Use Claude's built-in commit workflow for generating the commit message. The message should follow the format from `references/commit-playbook.md` — read the diff, write what changed, include the why.

### Step 4 — Pre-commit safety scan

Before each commit, scan staged files for:
- Secret patterns (`sk-`, `ghp_`, `Bearer `, `password=`, `token=`, `api_key`, `.env`, `.pem`, `.key`)
- Files over 5MB
- Accidentally staged ignored files

If any issue found: block, report, fix before retrying.

### Step 5 — Push (submodules first)

**5a. Push each submodule that was committed:**
```bash
cd websites/<client-name>
git push origin main
cd "<workspace-root>"
```

**5b. Push root:**
```bash
git push origin main
```

Order matters — submodules must be pushed before root, or root's submodule reference will point to a commit that doesn't exist on the remote yet.

### Step 6 — Confirm

Report the result:

> **Session wrapped:**
> - **[client-name]:** [N] files committed and pushed
> - **Root (KWeb):** [N] files committed and pushed
> - All repos synced to GitHub.

If any push failed, report which one failed and why (see `references/error-playbook.md`). The local commits are safe regardless.

---

## Edge Cases

**No submodule changes, only root changes:** Skip submodule steps. Commit and push root only.

**No root changes, only submodule changes:** Commit and push submodules. Then check if root has a dirty submodule reference — if so, commit and push the ref update at root too.

**Multiple submodules changed:** Commit each independently in step 3a, then do one root commit that captures all submodule ref updates plus any root file changes.

**Push fails:** Report the error. The commit is safe locally. See `references/error-playbook.md` for diagnosis. Do not retry automatically — report and wait.

**User wants to exclude something:** If the user says "don't commit [X]", respect it. Stage selectively instead of `git add -A`.
