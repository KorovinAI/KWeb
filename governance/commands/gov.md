---
name: gov
description: >
  On-demand governance audit for the KWeb workspace. Validates naming conventions,
  folder structure, index integrity, and filing compliance across all files and folders.
  Reports violations and fixes them.
  TRIGGER: /gov, "run governance", "check governance", "audit workspace",
  "check filing", "is everything filed correctly".
  This command does NOT auto-trigger. It only runs when explicitly invoked.
  It does NOT commit or push — use /wrap for that.
---

# /gov — Governance Audit

Runs a full governance check across the workspace. Validates that everything is named, placed, indexed, and structured correctly per the filing skill rules. Fixes violations it finds.

---

## What This Command Does

1. **Scan** the workspace for governance violations
2. **Report** what's wrong (or confirm everything is clean)
3. **Fix** violations with your confirmation
4. Does NOT commit or push — run `/wrap` after if needed

---

## Audit Checklist

Run through each check in order. For each violation found, report it clearly and propose the fix.

### 1. Naming Conventions (R1)

Scan all folders and files for naming violations:
- Folders must be lowercase, hyphen-separated
- Skill names must be `[domain]-[action]` format, max 3 segments
- Files must be lowercase, hyphen-separated
- Only exceptions: `CLAUDE.md`, `MEMORY.md`, `SKILL.md`, `README.md`
- Check exemptions declared in root CLAUDE.md (currently: `output-files/`, `websites/`)

```bash
# Find non-lowercase folders (excluding exempted directories and .git)
find . -type d -not -path './.git*' -not -path './websites*' -not -path './output-files*' -not -path './.claude*' | grep '[A-Z]' || echo "OK"

# Find non-lowercase files (excluding system files)
find . -type f -not -path './.git*' -not -path './websites*' -not -path './output-files*' -not -path './.claude*' -not -name 'CLAUDE.md' -not -name 'MEMORY.md' -not -name 'SKILL.md' -not -name 'README.md' -not -name '.DS_Store' | grep '[A-Z]' || echo "OK"
```

### 2. Folder Structure (R2)

- Every domain folder (level 1 and 2) must have `CLAUDE.md` and `MEMORY.md`
- No domain folders deeper than 2 levels from root
- Skill folders must only contain `SKILL.md` and `.skill` at root — supporting material in subfolders (`evals/`, `references/`, `assets/`, `scripts/`)
- No `CLAUDE.md` or `MEMORY.md` inside skill folders

### 3. Index Integrity (R4)

**Root MEMORY.md Subfolder Index (R4a):**
- Read root `MEMORY.md`
- Compare the Subfolder Memory Index against actual domain folders on disk
- Flag any folders that exist but aren't indexed
- Flag any indexed folders that no longer exist

**Domain-Level Skill Inventories (R4b):**
- For each domain folder, read its `MEMORY.md`
- Compare the `## Skills` table against actual skill folders on disk
- Flag any skills that exist but aren't listed
- Flag any listed skills that no longer exist

### 4. Frontmatter Compliance (R7)

- Every `SKILL.md` must have YAML frontmatter with `name` and `description`
- The `description` must include trigger phrases

```bash
# Find SKILL.md files and check for frontmatter
find . -name 'SKILL.md' -not -path './.git*' -not -path './websites*'
```

For each, read the first few lines and verify the `---` delimited frontmatter block exists with both fields.

### 5. Output Files Compliance (R8)

- Check if any generated deliverables (`.xlsx`, `.csv`, `.pdf`, `.docx`, `.pptx`) exist outside `output-files/`
- Check workspace root for stray files that don't belong

### 6. Skill Folder Hygiene (R2b)

- Check each skill folder for files that shouldn't be at root level (anything other than `SKILL.md` and `.skill`)
- Check for stray `CLAUDE.md` or `MEMORY.md` files inside skill folders

---

## Output Format

**If everything is clean:**
> Governance audit complete. No violations found.

**If violations are found:**
> **Governance audit — [N] issues found:**
>
> 1. **[Rule]:** [Description of violation]
>    **Fix:** [What needs to happen]
>
> 2. **[Rule]:** [Description of violation]
>    **Fix:** [What needs to happen]
>
> Fix all issues now?

Wait for confirmation, then fix everything. After fixing, re-run the checks to confirm clean.

---

## When to Use

- After creating new files or folders — verify they're correctly placed
- After a big refactor — make sure nothing was left behind
- Periodically — workspace health check
- Before `/wrap` if you want to be thorough

This command does NOT auto-run. It only fires when you type `/gov` or explicitly ask for a governance check.
