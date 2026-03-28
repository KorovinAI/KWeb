# Filing Skill — Full Spec

_Version: 1.0_
_Date: 2026-03-18_

---

## Purpose

The filing skill is an **organizational governance layer** that owns the rules for how skills, folders, and subfolders are structured, named, indexed, and archived across the workspace. It does not create skills or subfolders itself — it defines the conventions those tools must follow and enforces consistency.

---

## Scope Boundaries

| This skill OWNS | This skill DOES NOT OWN |
|-----------------|------------------------|
| Naming conventions for skills, folders, and files | Writing SKILL.md content (→ skill-creator) |
| Folder/subfolder structure rules | Creating CLAUDE.md and MEMORY.md in domain folders (→ subfolders skill) |
| Root MEMORY.md index maintenance | Skill evaluation and iteration (→ skill-creator) |
| Domain-level skill inventories | Skill triggering and description optimization (→ skill-creator) |
| Archival process | General behavioral rules (→ root CLAUDE.md) |
| Conflict resolution (skill fits multiple folders) | Version control (→ git skill) |
| Migration protocol when folders reorganize | Writing/deleting memory entries (→ memory-create / memory-delete) |

**Integration model:** skill-creator and subfolders consult this skill's conventions as a dependency. When skill-creator finishes a skill, it consults filing rules for naming and placement. When subfolders creates a folder, filing rules dictate naming, depth, and index updates.

---

## Requirements

### R1 — Naming Conventions

#### R1a — Skill Naming

Skills use a **descriptive hyphenated name** that reflects function, not folder path.

**Format:** `[domain]-[action].skill` or `[domain]-[qualifier]-[action].skill`

**Examples:**

| Skill | Name |
|-------|------|
| Cold outreach emails | `email-cold.skill` |
| Call prep | `call-prep.skill` |
| Lusha contact enrichment | `lusha-enrich.skill` |
| Pipeline review | `pipeline-review.skill` |
| Filing governance (this skill) | `filing.skill` |

**Rules:**
- Lowercase, hyphen-separated. No underscores, no camelCase.
- Max 3 segments. If you need more, the name is too specific — generalize.
- Names are **permanent and portable** — they do not change when a skill moves between folders.
- No folder path abbreviations in the name. The name describes *what* the skill does, not *where* it lives.

#### R1b — Folder and File Naming

**All folders and files use lowercase only.**

- Domain folders: lowercase, single-word or hyphen-separated (`prospecting/`, `mcp/`, `sales-ops/`)
- Skill subfolders: match the skill name without extension (`email-cold/`, `call-prep/`)
- Supporting subfolders: lowercase (`evals/`, `references/`, `assets/`)
- Files: lowercase, hyphen-separated (`tone-guide.md`, `api-notes.md`)
- **Exceptions:** `CLAUDE.md`, `MEMORY.md`, `SKILL.md` — these system files use uppercase by convention and are the only uppercase-named files permitted.
- **Exemptions:** Any directories declared as filing exceptions in the root CLAUDE.md are fully exempt from filing rules.

---

### R2 — Folder Structure

#### R2a — Domain Hierarchy

Domains are **maximum 2 levels deep** from root. Each domain level gets its own `CLAUDE.md` and `MEMORY.md`.

```
workspace/                       ← root (level 0)
├── CLAUDE.md
├── MEMORY.md
├── archive/                     ← single root-level archive
│   └── ...
├── prospecting/                 ← domain (level 1)
│   ├── CLAUDE.md
│   ├── MEMORY.md
│   ├── emails/                  ← sub-domain (level 2) — max depth
│   │   ├── CLAUDE.md
│   │   ├── MEMORY.md
│   │   ├── email-cold/          ← skill folder (not a domain)
│   │   │   ├── SKILL.md
│   │   │   ├── evals/
│   │   │   └── references/
│   │   └── email-followup/
│   │       ├── SKILL.md
│   │       └── references/
│   └── enrichment/              ← sub-domain (level 2)
│       ├── CLAUDE.md
│       ├── MEMORY.md
│       └── lusha-enrich/
│           ├── SKILL.md
│           └── references/
└── mcp/                         ← domain (level 1)
    ├── CLAUDE.md
    ├── MEMORY.md
    └── ...
```

**Domain folder rules:**
- Every domain folder (level 1 and level 2) gets `CLAUDE.md` and `MEMORY.md`.
- Level 2 is the deepest a domain can go. No `prospecting/emails/outbound/` as a domain — if you need that granularity, it's a skill, not a domain.
- `CLAUDE.md` at each level defines behavioral context for that domain. Level 2 inherits from level 1, which inherits from root.
- `MEMORY.md` at each level tracks domain-specific recall and a local skill inventory (see R4b).

#### R2b — Skill Folder Structure

Every skill gets its own subfolder within its parent domain folder.

**Rules:**
- The skill subfolder name matches the skill name (without `.skill` extension).
- The only files permitted at the skill folder root are `SKILL.md` and the packaged `.skill` installer file. No other `.md` files. No `CLAUDE.md`. No `MEMORY.md`.
- The `.skill` installer file (a zip archive used for installation) is always stored in its own skill folder — not in the parent domain folder or root.
- All supporting material goes into named subfolders:

| Subfolder | Contents |
|-----------|----------|
| `evals/` | Eval JSON, grading results, benchmark data |
| `references/` | Reference docs, style guides, examples |
| `assets/` | Templates, images, fonts |
| `scripts/` | Executable helper code |

- New subfolders can be created as needed (e.g., `drafts/`, `templates/`). The principle: **keep the skill root clean — SKILL.md and .skill installer only.**

---

### R3 — Folder Placement

When a new skill is created and its domain doesn't match an existing folder:

1. **Check existing folders.** Does this skill clearly belong under an existing domain folder?
2. **If yes** → place it there.
3. **If no clear fit** → ask the user: "This doesn't fit neatly under [existing folders]. Should I create a new folder for [proposed domain], or do you want to file it somewhere specific?"
4. **If it fits two folders equally** → ask the user for a decision. Present both options with a one-line rationale for each. Do not guess.

**Never create a skill at the root level.** Skills always live inside a domain folder.

---

### R4 — Indexing

#### R4a — Root Memory Index

The root `MEMORY.md` file contains a **Subfolder Memory Index** table. This table is the single source of truth for what domain folders exist and what they contain.

**Update triggers — the index MUST be updated when:**
- A new domain folder is created
- A domain folder is renamed, moved, or deleted
- A domain folder's purpose materially changes

**Index format:**

```markdown
## Subfolder Memory Index

| Subfolder | MEMORY.md | CLAUDE.md | What's stored there |
|-----------|-----------|-----------|-------------------|
| `mcp/` | Yes | Yes | Lusha tool status, workarounds, account info |
| `prospecting/` | Yes | Yes | Lead research, enrichment, outreach skills |
| `prospecting/emails/` | Yes | Yes | Email skills — cold, follow-up, nurture |
```

**Format rules:**
- One row per domain folder (level 1 and level 2).
- `Subfolder` column uses backtick-wrapped relative path from root.
- `MEMORY.md` and `CLAUDE.md` columns: "Yes", "Yes (empty)", or "No".
- `What's stored there` column: one-line summary, max ~15 words.
- Skill-level subfolders are NOT indexed here. Those are tracked in domain-level MEMORY.md (see R4b).
- Alphabetical order by subfolder path.

#### R4b — Domain-Level Skill Inventory

Each domain folder's `MEMORY.md` includes a `## Skills` section listing active skills in that domain.

**Format:**

```markdown
## Skills

| Skill | Folder | Status | Added |
|-------|--------|--------|-------|
| `email-cold` | `email-cold/` | Active | 2026-03-18 |
| `email-followup` | `email-followup/` | Active | 2026-03-19 |
```

**Rules:**
- Updated whenever a skill is added, archived, or moved in/out of the domain.
- Status is either "Active" or "Archived (→ archive/)".
- This is the local index — the root index tracks domains, domain MEMORY.md tracks skills within that domain.

---

### R5 — Migration Protocol

When folders are reorganized (renamed, merged, restructured):

1. **Skill names do not change.** Names are function-based, not path-based. Moving `email-cold/` from `prospecting/` to `outreach/` does not rename the skill.
2. **Update the root MEMORY.md index** to reflect new folder locations.
3. **Update domain-level skill inventories** — remove from old domain's MEMORY.md, add to new domain's MEMORY.md.
4. **Update any CLAUDE.md references** in affected folders that pointed to the old structure.
5. **Verify skill functionality** — ensure SKILL.md paths to references, scripts, and assets still resolve correctly. Fix any broken relative paths.
6. **Log the migration** in the destination folder's MEMORY.md: what moved, when, from where.

---

### R6 — Archival

There is **one archive folder at root level**: `archive/`.

When a skill is deprecated or replaced:

1. **Ask for explicit approval** before archiving. State what will be archived and what the effect is. Do not proceed without a clear "yes."
2. **Move** the skill's entire subfolder into `archive/` at root level (e.g., `archive/email-cold/`).
3. **Remove the skill from active use.** Archived skills must not trigger, must not appear in active skill lists, and must not be referenced by any CLAUDE.md as available. The skill is effectively dead.
4. **Update the root MEMORY.md index** if the domain folder's description changes as a result.
5. **Update the domain-level skill inventory** — mark status as "Archived (→ archive/)" or remove the row.
6. **Add a note** in the domain folder's MEMORY.md: `[skill-name] archived on [date] — [reason]`.

**Archive grows without limit.** No cleanup policy, no expiration. Archived skills are retained indefinitely for reference. They can be restored if needed, but restoration requires explicit user approval and re-registration.

---

### R7 — Skill Trigger Definition (Frontmatter)

Every skill has a YAML **frontmatter** block at the top of its `SKILL.md` file. This is the metadata that tells Claude *when* to invoke the skill and *what* it does.

**What frontmatter is:** A YAML block enclosed in `---` delimiters at the very top of the file. It's not displayed as content — it's parsed as configuration. Claude reads the `name` and `description` fields from every skill's frontmatter to decide which skill to activate for a given user request.

**Structure:**

```yaml
---
name: email-cold
description: >
  Draft cold outreach emails for sales prospects.
  Trigger whenever the user asks to write a cold email,
  first-touch message, or initial outreach to a new prospect.
  Also trigger when the user says "draft outreach to [person]",
  "cold email [company]", or "write intro email."
---
```

**The `description` field is the primary trigger mechanism.** Claude scans all available skill descriptions to decide which skill matches the user's request. A poorly written description means the skill won't fire when it should, or will fire when it shouldn't.

**Rules for writing trigger descriptions:**
- Lead with a one-sentence summary of what the skill does.
- Follow with explicit trigger phrases — the actual words a user might say.
- Include edge cases: "Also trigger when..." for non-obvious use cases.
- Be slightly "pushy" — err on the side of triggering. It's better to trigger and be dismissed than to miss a valid use case.
- Include negative triggers if there's a risk of confusion: "Do NOT trigger for [X] — use [other-skill] instead."

**How to test triggers:** After writing the description, mentally run 5-10 realistic user prompts against it. Would this description cause Claude to pick this skill? Would any of them accidentally trigger a different skill? Adjust until the coverage feels right. For rigorous testing, use skill-creator's description optimization workflow.

**The filing skill's own frontmatter:**

```yaml
---
name: filing
description: >
  Organizational governance for skills, folders, and workspace structure.
  Enforces naming conventions, folder placement, indexing, archival,
  and migration rules. Consulted as a dependency by skill-creator and
  subfolders skills — rarely invoked directly. Trigger when the user
  asks about filing conventions, folder organization, skill naming,
  archival, or workspace structure. Also trigger when creating,
  moving, renaming, or archiving skills or folders.
---
```

---

## Questions to Ask When Filing

When this skill is consulted for a filing decision, use these as a checklist:

1. **What domain does this skill serve?** (maps to folder)
2. **Does a domain folder already exist?** (placement)
3. **Is a sub-domain level needed, or does it fit at level 1?** (hierarchy depth)
4. **What should the skill be named?** (naming convention)
5. **Does it need supporting material subfolders?** (evals, references, etc.)
6. **Does the root index need updating?** (always check)
7. **Does the domain-level skill inventory need updating?** (always yes)

---

## Out of Scope

- Writing skill content or SKILL.md → skill-creator
- Creating CLAUDE.md / MEMORY.md for new domain folders → subfolders skill
- General workspace behavioral rules → root CLAUDE.md
- Writing/deleting memory entries → memory-create / memory-delete skills
- Version control → git skill (built separately)
- The `memory/` directory → productivity plugin (exempt, declared in root CLAUDE.md)

---

# Conflict Analysis & Change Specs

## Change Spec 1: skill-creator — MUST CHANGE

**Source:** Anthropic-provided skill. You did not build this. Changes require modifying the installed skill or wrapping it with a custom layer.

**Current behavior:** skill-creator handles the full lifecycle — intent capture, skill writing, testing, iteration, packaging. It makes no reference to filing conventions, naming rules, or folder placement. It will create CLAUDE.md and MEMORY.md inside skill folders if it sees fit. It places output wherever seems logical with no index awareness.

**What breaks without changes:**
- Skills get created with arbitrary names that don't follow R1a.
- Skills may land in the wrong folder or at root level, violating R3.
- Skill folders may contain CLAUDE.md/MEMORY.md files, violating R2b.
- The root index and domain skill inventories won't be updated, breaking R4a and R4b.
- Old skill versions get overwritten silently, with no archival prompt per R6.

**Required changes:**

1. **Post-creation filing step.** After a skill is drafted and approved, skill-creator must consult filing conventions before saving:
   - Apply the naming convention from R1a.
   - Determine folder placement per R3 (ask if ambiguous).
   - Create the skill subfolder with the correct structure per R2b.
   - Ensure SKILL.md is the only file at the skill folder root; supporting material goes in subfolders.

2. **Naming enforcement.** When the user provides a skill name, validate it against R1a. If it doesn't conform, suggest a corrected version.

3. **No skill-level CLAUDE.md or MEMORY.md.** skill-creator must not create these files inside skill folders. Behavioral context lives at the domain level only.

4. **Index triggers.** After creating a skill:
   - If a new domain folder was created → update root MEMORY.md index (R4a).
   - Always update the domain-level skill inventory (R4b).

5. **Archival awareness.** When updating/replacing a skill, check if the old version should be archived per R6. Ask the user.

**What stays the same:** Intent capture, SKILL.md writing, eval/test loops, description optimization, packaging.

**Recommended implementation:** Since skill-creator is an Anthropic skill you can't easily fork, the cleanest approach is to make the filing skill a mandatory post-step. When skill-creator finishes, the filing skill is consulted to validate naming, enforce placement, clean up any extra files, and update indexes. This avoids modifying skill-creator itself.

---

## Change Spec 2: subfolders skill (cowork-os) — MUST CHANGE

**Source:** cowork-os plugin. You did not build this. Changes require modifying the installed plugin or overriding behavior via CLAUDE.md instructions.

**Current behavior:** Creates folders with CLAUDE.md and MEMORY.md through an interview process. No index updates, no naming enforcement, no depth limits, no concept of "domain vs. skill folder."

**What breaks without changes:**
- Folders get created with Title Case or mixed case names, violating R1b.
- The root MEMORY.md index is never updated, breaking R4a.
- Domain folders can nest infinitely, violating R2a's 2-level cap.
- New MEMORY.md files don't include a `## Skills` section, breaking R4b.
- No distinction between domain folders and skill folders — could create CLAUDE.md/MEMORY.md inside a skill folder.

**Required changes:**

1. **Mandatory index update.** After creating any new domain folder, update the root MEMORY.md Subfolder Memory Index per R4a. Non-negotiable.

2. **Lowercase naming.** All folder names must be lowercase per R1b. Enforce regardless of user input. "Prospecting" becomes `prospecting/`.

3. **Domain depth enforcement.** Must not create domain folders deeper than 2 levels from root. If a user requests a third level, push back: suggest flattening or treating it as a skill folder.

4. **Skill inventory initialization.** When creating a new domain folder, include an empty `## Skills` section in the new MEMORY.md per R4b format.

5. **Domain vs. skill folder awareness.** Add a question to the interview: "Is this folder for organizing skills, or for a project/workspace?" If it's a skill folder (housing a single SKILL.md), do NOT create CLAUDE.md/MEMORY.md — that violates R2b.

**What stays the same:** Interview flow, CLAUDE.md/MEMORY.md content creation, identity overrides, memory isolation.

**Recommended implementation:** Add filing rules to root CLAUDE.md as operating instructions that override subfolders skill defaults. The subfolders skill reads CLAUDE.md at session start, so behavioral overrides there will be respected without modifying the plugin itself.

---

## Change Spec 3: memory-create skill (cowork-os) — SHOULD CHANGE (guardrail)

**Source:** cowork-os plugin. You did not build this.

**Current behavior:** Writes user-triggered entries to MEMORY.md in the current folder. Locates or creates MEMORY.md, checks for contradictions, writes the entry, confirms. No awareness of structured sections.

**What breaks without changes:**
- User memory entries could be written into the `## Skills` table or `## Subfolder Memory Index` instead of under `## Memory`, corrupting filing records.
- If memory-create creates a new MEMORY.md, it won't include a `## Skills` section — but that's fine, since the filing skill handles that via subfolders. The risk is that memory-create could create a MEMORY.md in a skill folder (where one shouldn't exist per R2b).
- Contradiction checking could flag filing records as conflicts with user memories.

**Realistic risk assessment:** LOW. In practice, memory-create writes under `## Memory` and is unlikely to touch structured sections. The risk is edge-case: a user says "remember this" in a context where the filing sections are right there. The guardrail is defensive, not urgent.

**Required changes:**

1. **Section-awareness.** Never write user entries into `## Skills` or `## Subfolder Memory Index`. User entries go under `## Memory` only.

2. **Don't treat filing sections as contradictions.** A user memory about a skill (e.g., "email-cold is broken") is not contradicted by the Skills table showing it as Active.

3. **Don't create MEMORY.md in skill folders.** If a user triggers "remember this" while working in a skill folder, write to the parent domain's MEMORY.md instead.

**Recommended implementation:** Add a note to root CLAUDE.md under the Memory System section: "When writing to MEMORY.md, always write under `## Memory`. Never modify `## Skills` or `## Subfolder Memory Index` sections — those are managed by the filing system." This instructs the memory-create skill without modifying it.

---

## Change Spec 4: memory-delete skill (cowork-os) — SHOULD CHANGE (guardrail)

**Source:** cowork-os plugin. You did not build this.

**Current behavior:** Removes user-triggered entries from MEMORY.md. Supports targeted deletion and full clears.

**What breaks without changes:**
- "Clear everything" wipes the entire MEMORY.md, destroying the `## Skills` inventory and `## Subfolder Memory Index`. These are filing records that should survive a memory clear.
- Targeted deletion could match text in a filing section and delete a Skills table row.

**Realistic risk assessment:** MEDIUM on "clear everything" (destructive and hard to recover from). LOW on targeted deletion (unlikely match, but possible).

**Required changes:**

1. **Protect structured sections during full clear.** "Clear everything" must only clear entries under `## Memory`. Preserve `## Skills` and `## Subfolder Memory Index` intact. If the user explicitly asks to clear those too, flag: "The Skills inventory and folder index are managed by the filing system. Clearing them would break skill tracking. Are you sure?"

2. **Targeted deletion respects boundaries.** Only search under `## Memory` for matches. If a match appears in a filing section, explain: "That's in the skill inventory, not your memory notes. Want me to update the filing record instead?"

**Recommended implementation:** Same as memory-create — add protective instructions to root CLAUDE.md under the Memory System section. The memory-delete skill reads CLAUDE.md and will respect the guardrails.

---

## Change Spec 5: memory-management (productivity plugin) — NO CHANGE, EXEMPT

**Source:** Productivity plugin. You did not build this.

**Current behavior:** Two-tier memory system using `CLAUDE.md` for working memory and a `memory/` directory for the full knowledge base (`glossary.md`, `people/`, `projects/`). Completely different convention from the filing system.

**What breaks without changes:** Nothing. These systems are isolated. The `memory/` directory is exempt from filing rules (declared in root CLAUDE.md under Filing Exceptions).

**Required changes:** None.

**Watch item:** If the productivity plugin ever starts creating folders outside `memory/` that look like domain folders, or writes to MEMORY.md sections managed by the filing system, re-evaluate. Currently there's no overlap.

---

## Change Spec 6: task-management (productivity plugin) — NO CHANGE, NO CONFLICT

**Source:** Productivity plugin. Uses `TASKS.md`.

**Required changes:** None. No overlap with filing conventions.

---

# Cross-Reference Summary

| Skill | Owner | Conflict | Action | Approach |
|-------|-------|----------|--------|----------|
| **skill-creator** | Anthropic | MUST CHANGE | Add post-creation filing step, naming enforcement, index updates | Filing skill as mandatory post-step; don't modify skill-creator directly |
| **subfolders** | cowork-os | MUST CHANGE | Index updates, lowercase naming, depth cap, skill inventory init, domain/skill distinction | Override behavior via root CLAUDE.md instructions |
| **memory-create** | cowork-os | SHOULD CHANGE | Section-awareness guardrail | Add protective note to root CLAUDE.md Memory System section |
| **memory-delete** | cowork-os | SHOULD CHANGE | Protect filing sections from clear/delete | Add protective note to root CLAUDE.md Memory System section |
| **memory-management** | Productivity plugin | NO CHANGE | Exempt — different system | Exception declared in root CLAUDE.md |
| **task-management** | Productivity plugin | NO CHANGE | No overlap | None needed |

---

# Open Items

1. **Productivity plugin `memory/` directory** — Currently exempt and isolated. If the productivity plugin is updated and starts creating structures that overlap with filing conventions, re-evaluate the exemption. For now, the root CLAUDE.md exception is sufficient.
