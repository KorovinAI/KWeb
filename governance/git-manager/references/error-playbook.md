# Error & Push Failure Playbook

Reference for git-manager error handling. When a push fails or git throws an error,
find it here, explain it plainly to the user, give the fix.

---

## General Rule

When any git command fails:
1. Read the error output
2. Identify the error type from this playbook
3. Explain in one sentence what went wrong (plain English, no jargon)
4. Give the exact fix command(s)
5. Confirm: the commit is safe locally, nothing is lost
6. Wait — do not retry automatically

---

## Authentication Errors

### `Authentication failed` / `remote: Invalid username or password`
**What happened:** GitHub rejected the credentials. PAT may be expired, revoked, or not stored correctly.
**Fix:**
```bash
# Clear cached credential
git credential reject <<EOF
protocol=https
host=github.com
EOF

# Re-enter credentials (will prompt)
git push origin main
```
Or regenerate PAT at `https://github.com/settings/tokens` and re-run setup.

### `Permission denied (publickey)`
**What happened:** SSH key not found or not added to GitHub.
**Fix:** Switch to HTTPS or add SSH key to GitHub account settings.
```bash
git remote set-url origin <configured-remote-url>
git push origin main
```

---

## Remote / Conflict Errors

### `rejected — non-fast-forward`
**What happened:** Remote has commits that aren't in the local repo (e.g. you edited GitHub directly, or pushed from another machine).
**Fix — safe rebase:**
```bash
git pull --rebase origin main
git push origin main
```
⚠️ If rebase shows conflicts, stop and surface them to the user before proceeding.

### `fatal: remote origin already exists`
**What happened:** Running `git remote add origin` when a remote is already set.
**Fix:**
```bash
git remote set-url origin <configured-remote-url>
```

### `fatal: 'origin' does not appear to be a git repository`
**What happened:** Remote URL is wrong or the GitHub repo doesn't exist.
**Fix:** Verify the repo exists at the configured remote URL, then:
```bash
git remote -v                    # check current remote
git remote set-url origin <configured-remote-url>
git ls-remote origin HEAD        # test connection
```

### `fatal: repository not found`
**What happened:** GitHub repo doesn't exist yet, or PAT doesn't have access to it.
**Fix:** Create the repo at `github.com/new`, private, no README. Then push.

---

## Merge Conflicts

### Conflict markers in file after `git pull --rebase`
**What happened:** Same lines were changed both locally and remotely. Git can't auto-merge.
**What to do:**
1. Run `git diff --name-only --diff-filter=U` to list conflicted files
2. Show the user the conflicting sections from each file
3. Ask the user to decide: keep local, keep remote, or merge manually
4. After resolution: `git add <file>` then `git rebase --continue`
5. Then push

Never resolve a conflict silently. Always show the user what conflicted and get a decision.

---

## Large File Errors

### `remote: error: File X exceeds GitHub's file size limit`
**What happened:** A file over 100MB was accidentally committed.
**Fix:**
```bash
git rm --cached <large-file>
echo "<large-file>" >> .gitignore
git commit --amend --no-edit
git push origin main
```
⚠️ `--amend` modifies the last commit — trigger the risky ops gate before running this.

---

## Stash Errors

### Uncommitted changes blocking a pull
**What happened:** Can't pull because local changes would be overwritten.
**Fix:**
```bash
git stash                  # temporarily shelve changes
git pull origin main       # get remote changes
git stash pop              # reapply shelved changes
```
If stash pop has conflicts → treat as merge conflict above.

---

## Repo State Errors

### `fatal: not a git repository`
**What happened:** Running git commands outside the repo directory, or `.git/` was deleted.
**Fix:** Navigate to the workspace root and verify `.git/` exists. If missing, run Module 1 (init) again.

### `HEAD detached at <hash>`
**What happened:** Checked out a specific commit rather than a branch. Changes made here won't be saved to any branch.
**Fix:**
```bash
git checkout main
```
If changes were made in detached state and you want to keep them:
```bash
git checkout -b temp-recovery
git checkout main
git merge temp-recovery
git branch -d temp-recovery
```
⚠️ Trigger risky ops gate before merging. Explain what's happening to the user first.

---

## Submodule Errors

### `fatal: no submodule mapping found in .gitmodules for path 'websites/X'`
**What happened:** A directory exists in `websites/` that looks like a submodule but isn't registered in `.gitmodules`.
**Fix:** Either add it properly as a submodule or remove the directory if it shouldn't be tracked:
```bash
# To add it properly:
git submodule add <remote-url> websites/<client-name>

# To remove the orphaned directory:
git rm --cached websites/<client-name>
```

### Submodule HEAD detached
**What happened:** Submodules check out a specific commit by default, leaving HEAD detached. Any new commits made in this state won't be on a branch.
**Fix:**
```bash
cd websites/<client-name>
git checkout main
cd "<workspace-root>"
```

### `error: cannot push ... submodule has unpushed changes`
**What happened:** Root repo references a submodule commit that hasn't been pushed to the submodule's remote yet.
**Fix:** Push the submodule first, then push root:
```bash
cd websites/<client-name>
git push origin main
cd "<workspace-root>"
git push origin main
```

### Submodule not initialised after clone
**What happened:** Cloning the root repo doesn't automatically populate submodule directories.
**Fix:**
```bash
git submodule update --init --recursive
```

### Submodule shows as modified but nothing changed inside
**What happened:** The submodule's checked-out commit differs from what root expects. Often happens after pulling root without updating submodules.
**Fix:**
```bash
git submodule update
```
This checks out the commit that root expects. If you intentionally advanced the submodule, commit the new reference at root instead.

---

## Queued / Pending Commits

If a push fails, the commit is safe locally. To verify:
```bash
git log --oneline origin/main..HEAD    # commits not yet on remote
```
Output shows any commits waiting to be pushed. Nothing is lost.

When the user is ready to retry: `git push origin main`.

**For submodules:** check each independently:
```bash
git submodule foreach 'git log --oneline origin/main..HEAD 2>/dev/null'
```
