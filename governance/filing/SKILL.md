---
name: filing
description: >
  Organizational governance for skills, folders, and workspace structure.
  Enforces naming conventions, folder placement, indexing, archival, and migration rules.
  Consulted as a dependency by skill-creator and subfolders skills — rarely invoked directly
  by the user. Trigger when the user asks about filing conventions, folder organization,
  skill naming, archival, workspace structure, or "where should this go." Also trigger
  whenever creating, moving, renaming, or archiving skills or folders. Trigger even if the
  user doesn't say "filing" — any question about workspace organization or skill placement
  should use this skill.
---

# Filing Skill

You are a governance layer for the workspace. You own the rules for how skills, folders, and files are named, placed, indexed, and archived. You do not create skills or write SKILL.md content — you define the conventions that other tools (skill-creator, subfolders) must follow.

For the full spec with change specs and conflict analysis, read `references/filing-skill-spec.md`.

---

## When This Skill Is Consulted

This skill fires in two modes:

**Direct invocation** — the user asks about filing, naming, placement, or structure.

**Dependency invocation** — another skill (skill-creator, subfolders) has finished its work and needs filing conventions applied. In this mode, validate the output against the rules below and fix any violations.

---

## Filing Checklist

Run through these questions for every filing decision:

1. **What domain does this skill serve?** → maps to a folder
2. **Does a domain folder already exist?** → placement
3. **Is a sub-domain level needed, or does it fit at level 1?** → hierarchy depth
4. **What should the skill be named?** → naming convention
5. **Does it need supporting material subfolders?** → evals, references, etc.
6. **Does the root MEMORY.md index need updating?** → always check
7. **Does the domain-level skill inventory need updating?** → always yes for skill changes

---

## R1 — Naming Conventions

### Skills

Format: `[domain]-[action]` or `[domain]-[qualifier]-[action]`

Rules:
- Lowercase, hyphen-separated. No underscores, no camelCase.
- Max 3 segments. More than 3 means the name is too specific — generalize.
- Names are permanent and portable. They do not change when a skill moves between folders.
- Names describe *what* the skill does, not *where* it lives. No folder path abbreviations.

Examples: `email-cold`, `call-prep`, `lusha-enrich`, `pipeline-review`, `filing`

### Folders and Files

All lowercase. No exceptions except system files.

- Domain folders: lowercase, single-word or hyphen-separated (`prospecting/`, `mcp/`, `sales-ops/`)
- Skill subfolders: match the skill name (`email-cold/`, `call-prep/`)
- Supporting subfolders: lowercase (`evals/`, `references/`, `assets/`)
- Files: lowercase, hyphen-separated (`tone-guide.md`, `api-notes.md`)
- **Only exceptions:** `CLAUDE.md`, `MEMORY.md`, `SKILL.md` — uppercase by system convention.

### Exemptions

Any directories declared as filing exceptions in the root CLAUDE.md are fully exempt from filing rules. Do not rename them, index them, or apply any filing rules to them.

---

## R2 — Folder Structure

### Domain Hierarchy

Domains go **maximum 2 levels deep** from root. Each domain level gets `CLAUDE.md` and `MEMORY.md`.

```
workspace/                       ← root (level 0)
├── CLAUDE.md
├── MEMORY.md
├── archive/                     ← single root-level archive
├── prospecting/                 ← domain (level 1)
│   ├── CLAUDE.md
│   ├── MEMORY.md
│   ├── emails/                  ← sub-domain (level 2) — max depth
│   │   ├── CLAUDE.md
│   │   ├── MEMORY.md
│   │   └── email-cold/          ← skill folder (not a domain)
│   │       └── SKILL.md
│   └── enrichment/              ← sub-domain (level 2)
│       ├── CLAUDE.md
│       ├── MEMORY.md
│       └── lusha-enrich/
│           └── SKILL.md
└── governance/                  ← domain (level 1)
    ├── CLAUDE.md
    ├── MEMORY.md
    └── filing/
        └── SKILL.md
```

If someone requests a third domain level (e.g., `prospecting/emails/outbound/`), push back. Either flatten it or treat the third level as a skill, not a domain.

`CLAUDE.md` at each level defines behavioral context. Level 2 inherits from level 1, which inherits from root.

`MEMORY.md` at each level tracks domain-specific recall and a local skill inventory (see R4b below).

### Skill Folder Structure

Every skill gets its own subfolder within its parent domain.

The only files permitted at the skill folder root are `SKILL.md` and the packaged `.skill` installer file. Nothing else. No `CLAUDE.md`. No `MEMORY.md`. These live at the domain level only.

Supporting material goes in named subfolders:

| Subfolder | Contents |
|-----------|----------|
| `evals/` | Eval JSON, grading results, benchmark data |
| `references/` | Reference docs, style guides, examples |
| `assets/` | Templates, images, fonts |
| `scripts/` | Executable helper code |

The `.skill` installer file (a zip archive used for installation) is kept in the skill folder root alongside `SKILL.md`. When packaging a skill, always save the `.skill` file inside its own skill folder — not in the parent domain folder or root.

**Reinstall rule:** The installed version of a skill (in `.skills/skills/`) is read-only and does not auto-sync with the workspace copy. After any modification to a SKILL.md, you must re-package the `.skill` file and prompt the user to reinstall. Never skip this step — an uninstalled update is a silent regression.

The principle: keep the skill root clean — `SKILL.md` and `.skill` installer only. Everything else goes in subfolders.

### Plugin File Placement

`.plugin` installer files (zip archives used for plugin installation) are placed at the **domain root** of the highest-order folder that contains the skills the plugin orchestrates. Not inside a skill subfolder, not at workspace root.

Example: `governance.plugin` lives in `governance/` because it orchestrates `filing/` and `git-manager/`, both of which are children of `governance/`.

The same reinstall rule applies: after modifying any component of a plugin, repackage the `.plugin` file and prompt the user to reinstall.

---

## R3 — Folder Placement

When placing a new skill:

1. Check existing domain folders. Does this skill clearly belong under one?
2. If yes → place it there.
3. If no clear fit → ask the user: "This doesn't fit neatly under [existing folders]. Should I create a new folder for [proposed domain], or do you want to file it somewhere specific?"
4. If it fits two folders equally → ask the user. Present both options with a one-line rationale each. Do not guess.

**Never create a skill at root level.** Skills always live inside a domain folder.

---

## R4 — Indexing

### R4a — Root Memory Index

The root `MEMORY.md` contains a Subfolder Memory Index. This is the single source of truth for domain folders.

Update it when:
- A new domain folder is created
- A domain folder is renamed, moved, or deleted
- A domain folder's purpose materially changes

Format:

```markdown
## Subfolder Memory Index

| Subfolder | MEMORY.md | CLAUDE.md | What's stored there |
|-----------|-----------|-----------|-------------------|
| `governance/` | Yes | Yes | Filing, git, workspace governance skills |
| `mcp/` | Yes | No | Lusha tool status, workarounds, account info |
| `prospecting/` | Yes | Yes | Lead research, enrichment, outreach skills |
```

Rules:
- One row per domain folder (level 1 and level 2).
- Backtick-wrapped relative paths.
- MEMORY.md and CLAUDE.md columns: "Yes", "Yes (empty)", or "No".
- What's stored there: one-line summary, max ~15 words.
- Skill-level folders are NOT indexed here — those go in domain MEMORY.md.
- Alphabetical order.

### R4b — Domain-Level Skill Inventory

Each domain's `MEMORY.md` includes a `## Skills` section.

Format:

```markdown
## Skills

| Skill | Folder | Status | Added |
|-------|--------|--------|-------|
| `email-cold` | `email-cold/` | Active | 2026-03-18 |
```

Rules:
- Updated when a skill is added, archived, or moved.
- Status: "Active" or "Archived (→ archive/)".
- This is the local index. Root tracks domains; domain MEMORY.md tracks skills.

---

## R5 — Migration

When folders are reorganized:

1. **Skill names do not change.** Names are function-based, not path-based.
2. Update the root MEMORY.md index.
3. Update domain-level skill inventories — remove from old, add to new.
4. Update CLAUDE.md references that pointed to old structure.
5. Verify SKILL.md paths to references/scripts/assets still resolve. Fix broken relative paths.
6. Log the migration in the destination folder's MEMORY.md.

---

## R6 — Archival

There is **one archive folder at root level**: `archive/`.

Process:

1. **Ask for explicit approval.** State what will be archived and the effect. Do not proceed without a clear "yes."
2. Move the skill's entire subfolder to `archive/` (e.g., `archive/email-cold/`).
3. Remove from active use. Archived skills must not trigger, must not appear in any CLAUDE.md as available.
4. Update root MEMORY.md index if needed.
5. Update domain skill inventory — mark "Archived (→ archive/)" or remove.
6. Add archival note in domain MEMORY.md: `[skill-name] archived on [date] — [reason]`.

Archive grows without limit. No cleanup, no expiration. Restoration requires explicit approval.

---

## R7 — Frontmatter

Every skill needs YAML frontmatter at the top of SKILL.md with `name` and `description`.

The `description` is the primary trigger mechanism. Rules for writing it:
- Lead with a one-sentence summary.
- Follow with explicit trigger phrases.
- Include edge cases: "Also trigger when..."
- Be slightly pushy — better to trigger and be dismissed than miss a valid case.
- Include negative triggers if confusion risk exists: "Do NOT trigger for X — use Y instead."

---

## Integration With Other Skills

This skill defines conventions. It does not modify other skills' code. Integration works through root CLAUDE.md instructions that other skills read at session start.

**skill-creator** — After creating a skill, consult filing rules for naming (R1), placement (R3), folder structure (R2b), and index updates (R4). Do not create CLAUDE.md or MEMORY.md inside skill folders.

**subfolders** — After creating a domain folder, update root index (R4a), enforce lowercase naming (R1b), enforce 2-level depth cap (R2a), and initialize an empty Skills section in MEMORY.md (R4b).

**memory-create** — Write user entries under `## Memory` only. Never into `## Skills` or `## Subfolder Memory Index`. Don't treat filing sections as contradictions.

**memory-delete** — On "clear everything," preserve `## Skills` and `## Subfolder Memory Index`. Only clear entries under `## Memory`. If user explicitly asks to clear filing sections, flag it as destructive.

For full conflict analysis and implementation approaches, read `references/filing-skill-spec.md`.

---

## R8 — Output Files

All generated deliverables — spreadsheets (`.xlsx`, `.csv`), PDFs (`.pdf`), Word documents (`.docx`), presentations (`.pptx`), HTML reports (`.html`) — go to `output-files/` at the workspace root.

Rules:
- Output files are **not** skill artifacts. They do not live inside skill folders or domain folders.
- `output-files/` is a flat directory. No subdirectories by type or domain.
- Filenames must be self-documenting. Include company name and date where applicable (e.g., `Aruma_Contacts_2026-03-18.xlsx`).
- `output-files/` is **exempt** from domain folder rules: no `CLAUDE.md`, no `MEMORY.md`, not indexed in the Subfolder Memory Index.
- Skills that generate output files must save them to `output-files/`, not to their own folder, their parent domain, or the workspace root.

---

## Quick Reference

| What | Convention |
|------|-----------|
| Skill name | `domain-action`, lowercase, hyphenated, max 3 segments |
| Folder name | lowercase, hyphenated |
| Domain depth | Max 2 levels from root |
| Domain files | CLAUDE.md + MEMORY.md at every domain level |
| Skill folder | SKILL.md + .skill installer at root, supporting material in subfolders |
| Archive | Single `archive/` at root, explicit approval required |
| Index (root) | Subfolder Memory Index in root MEMORY.md |
| Index (domain) | Skills table in domain MEMORY.md |
| Output files | All deliverables → `output-files/`, flat, self-documenting names |
| Reinstall | After any SKILL.md edit, re-package .skill and prompt user to reinstall |
| Exempt | `memory/`, `output-files/` — not governed as domains |
