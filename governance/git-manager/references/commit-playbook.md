# Commit Playbook

Reference for git-manager commit timing, message generation, and batching rules.

---

## When to Commit

Commits are always user-triggered — via `/wrap` (session wrap) or the built-in `/commit`. Never auto-commit.

One commit per completed logical unit of work. The table below defines what constitutes a unit of work.

| Situation | What gets committed | Where |
|-----------|---------------------|-------|
| Skill update finished | SKILL.md + .skill + any associated MEMORY.md changes | Root repo |
| New domain folder created | New folder's CLAUDE.md + MEMORY.md + root MEMORY.md index update | Root repo |
| Client website updated | Changed files in submodule; submodule ref update at root | Client submodule, then root |
| Session closing (`/wrap`) | Full sweep — everything uncommitted | Both levels |
| Ad-hoc commit (`/commit`) | Whatever is currently modified | Detect level |

**Do NOT commit:**
- Mid-workflow while actively editing
- On every individual file save
- At root level before committing inside a modified submodule (commit inside-out)
- Without user confirmation

---

## MEMORY.md Batching Rule

MEMORY.md changes bundle with the task that caused them.

**Example:**
- You update the filing SKILL.md → one commit: `filing: generalise naming conventions`
- During the same task, MEMORY.md was also updated → that change is IN the same commit, not separate
- Session closes and `/wrap` captures any remaining MEMORY.md changes in the session-wrap commit

---

## Client Website Commit Order

When client website files have changed:

1. **Commit inside the submodule first.** `cd` into `websites/<client-name>/`, stage, commit.
2. **Then commit the submodule reference update at root level.** Back at root, `git add websites/<client-name>` stages the new submodule pointer. Commit at root.
3. **Push submodule before root.** If you push root first, the submodule reference points to a commit that doesn't exist on the remote yet.

**Commit message conventions:**
- Inside the submodule: `<client-name>: <what changed>` — e.g., `acme-corp: rebuild contact page with new form`
- At root level (submodule ref update): `websites/<client-name>: update submodule ref — <summary>` — e.g., `websites/acme-corp: update submodule ref — new contact page`

If multiple submodules changed in the same session, commit each submodule independently, then do one root-level commit that updates all submodule refs together.

---

## Commit Message Format

`<scope>: <what changed and why>`

**Scope selection:**
| What changed | Scope |
|---|---|
| Single skill | Skill name: `filing` |
| Domain folder created | Domain name: `governance` |
| MEMORY.md file(s) only | `<folder>/memory`: `governance/memory` |
| Root CLAUDE.md | `root` |
| Client site (inside submodule) | Client name: `acme-corp` |
| Client site (root submodule ref) | `websites/<client-name>` |
| Session sweep | `session-wrap` |
| Multiple domains | `workspace` |

**Message body:** Read the diff. Write what actually changed. Include the *why* when context makes it visible. Never write "updated file" or "changes".

**Length:** One line, under 72 characters. No period at end.

**Examples:**
```
filing: generalise naming conventions, remove hardcoded workspace references
governance: add git-manager skill — SKILL.md, commands, references
governance/memory: update skills inventory — git-manager v2.0
acme-corp: rebuild homepage with new hero section and responsive nav
websites/acme-corp: update submodule ref — homepage rebuild
session-wrap: 2026-03-28 — governance updates + acme-corp homepage
workspace: filing + git-manager updates, memory index synced
```

**Bad examples (never use):**
```
update
changes
fixed stuff
update SKILL.md
misc changes 2026-03-19
```

---

## Version Tagging

Tags are created for significant skill milestones — not every commit.

**Tag format:** `<skill-name>-v<major>.<minor>`
Examples: `filing-v1.0`, `git-manager-v2.0`

**Version bump decision tree:**
```
Did the skill's core behaviour, workflow, or output format change?
  YES → Major bump (v1.x → v2.0)
  NO  → Did it add new logic, fix edge cases, or improve quality?
          YES → Minor bump (v1.0 → v1.1)
          NO  → No tag needed, just commit
```

**When uncertain:** Propose minor. Explain the reasoning. Let the user decide.

**Process:**
1. `git tag -l "<skill-name>-*"` — check existing tags
2. Propose: tag name + one-line annotation
3. Wait for the user's confirmation
4. `git tag -a <tag> -m "<description>"`
5. `git push origin <tag>`

Tags are never deleted without explicit approval and the risky ops gate.

---

## Session Wrap Flow (`/wrap`)

See `commands/wrap.md` for the full flow. Summary:

1. Scan submodules and root for uncommitted changes
2. Present a single change summary to the user
3. On confirmation: commit submodules first (inside-out), then root
4. Push submodules first, then root
5. Confirm everything is synced
