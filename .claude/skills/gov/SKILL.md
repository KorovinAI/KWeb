---
name: gov
description: >
  On-demand governance audit for the KWeb workspace. Validates naming conventions,
  folder structure, index integrity, and filing compliance across all files and folders.
  Reports violations and fixes them. Does not commit or push.
---

# /gov — Governance Audit

Read the full audit spec from `governance/commands/gov.md` and the filing rules from `governance/filing/SKILL.md`, then execute the full audit.

## Quick Reference

1. Check naming conventions (R1) — all folders and files lowercase, hyphen-separated
2. Check folder structure (R2) — domain folders need CLAUDE.md and MEMORY.md, max 2 levels deep
3. Check index integrity (R4) — root MEMORY.md subfolder index and domain skill inventories must match disk
4. Check frontmatter compliance (R7) — every SKILL.md needs name and description
5. Check output files compliance (R8) — no deliverables outside output-files/
6. Check skill folder hygiene (R2b) — only SKILL.md and .skill at skill folder root

Report violations clearly. Propose fixes. Wait for confirmation before fixing.

This does NOT commit or push — tell the user to run `/wrap` after if changes were made.
