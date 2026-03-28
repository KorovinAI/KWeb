# governance/ — Workspace Governance

This folder contains skills that govern how the workspace is organized, structured, and maintained. These are meta-skills — they define conventions that other skills and workflows follow.

---

## Scope

- Filing conventions (naming, placement, indexing, archival, migration) — auto-triggers on file/folder changes
- Git version control (init, submodules, tagging, health checks, session wrap)
- Governance audit (`/gov` command)

## What Belongs Here

Skills that define *rules and structure* for the workspace. Not skills that do actual work (those go in domain folders).

## Commands

- **`/wrap`** — Session wrap. Commits and pushes all changes across submodules and root. Lives in `git-manager/commands/wrap.md`.
- **`/gov`** — On-demand governance audit. Validates naming, structure, indexes, compliance. Lives in `commands/gov.md`.

## MEMORY SYSTEM

This folder contains a file called MEMORY.md. It is your external memory for this workspace — use it to bridge the gap between sessions.

**At the start of every session:** Read MEMORY.md before responding. Use what you find to inform your work — don't announce it, just be informed by it.

**Memory is user-triggered only.** Do not automatically write to MEMORY.md. Only add entries when the user explicitly asks.

**All memories are persistent.** Entries stay until the user explicitly asks to remove or change them.

**Flag contradictions.** If the user asks you to remember something that conflicts with an existing memory, flag it and ask how to reconcile.
