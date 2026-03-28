# KWeb — Website Building Workspace

This is Ilia's website building shop. KWeb is where client websites are built, managed, and deployed via GitHub Pages. Each client project lives as a repo inside the `websites/` folder.

---

## MEMORY SYSTEM

This folder contains a file called MEMORY.md. It is your external memory for this workspace — use it to bridge the gap between sessions.

**At the start of every session:** Read MEMORY.md before responding. Use what you find to inform your work — don't announce it, just be informed by it.

**Memory is user-triggered only.** Do not automatically write to MEMORY.md. Only add entries when the user explicitly asks — using phrases like "remember this," "don't forget," "make a note," "log this," "save this," or "create session notes." When triggered, write the information to MEMORY.md immediately and confirm you've done it.

**All memories are persistent.** Entries stay in MEMORY.md until the user explicitly asks to remove or change them. Do not auto-delete or expire entries.

**Flag contradictions.** If the user asks you to remember something that conflicts with an existing memory, don't silently overwrite it. Flag the conflict and ask how to reconcile it.

---

## Operating Rules

**Push back on sub-optimal requests.** If a request seems like the wrong approach, a waste of effort, or could be done better — say so before executing. Challenge the thinking, propose alternatives, explain why. Don't be a yes-man. The goal is the best outcome, not the fastest compliance.

**Filing governance on every file/folder change.** After any file or folder is created, moved, or renamed, consult the **filing** skill to validate naming conventions, folder placement, and structure. Update the root MEMORY.md index and domain-level skill inventories as needed. This auto-triggers — do not skip it.

**No auto-commit or auto-push.** Git operations require explicit user action. Use `/wrap` to commit and push everything at session end, or the built-in `/commit` for ad-hoc commits. Never commit or push without user confirmation.

**`/wrap`** — Session wrap. Reviews all changes across submodules and root, presents a summary, commits inside-out, pushes to GitHub. Run when done working.

**`/gov`** — On-demand governance audit. Validates naming, structure, indexes, and compliance. Does not commit or push. Run anytime to check workspace health.

---

## Workspace Structure

- **`websites/`** — Client website repos. Each client gets their own independent git repo here, connected to the root as a **git submodule**. Each repo has its own GitHub remote (typically `KorovinAI/<client-name>`) and is deployed via GitHub Pages.
- **`governance/`** — Skills that govern how the workspace is organized, structured, and maintained. Meta-skills — they define conventions that other skills and workflows follow.
- **`output-files/`** — Centralized dump for all generated deliverables (spreadsheets, PDFs, docs, decks, CSVs). Flat directory, self-documenting filenames (include client name + date).

## Git Architecture

KWeb uses a two-level git structure:
- **Root repo** (`KorovinAI/KWeb`, public) — tracks governance skills, workspace config, and submodule references.
- **Client repos** (each in `websites/<client-name>/`) — independent repos connected as git submodules. Each has its own commit history, remote, and GitHub Pages deployment.

Changes inside a client site are committed in the submodule first, then the submodule reference is updated at root level. Always push submodules before pushing root. See the **git-manager** skill for full workflow details.

---

## Output Files

All generated deliverables (`.xlsx`, `.csv`, `.pdf`, `.docx`, `.pptx`, `.html`) go to `output-files/` at the workspace root. No exceptions — skills must not dump outputs into their own folders, domain folders, or the workspace root. Flat directory, self-documenting filenames (include client name + date).

---

## Filing Exceptions

**`output-files/` directory** — Centralized dump for all generated deliverables. Exempt from domain folder rules — no CLAUDE.md, no MEMORY.md, not indexed in Subfolder Memory Index. See Output Files section above.

**`websites/` directory** — Contains client repos, each with their own git history. Exempt from domain folder rules — each repo manages its own structure independently. Not indexed in Subfolder Memory Index.

---

## Subfolder Creation Protocol

When the user asks to create a new subfolder, use the **subfolders** skill. It handles the full interview, CLAUDE.md and MEMORY.md creation, identity overrides, and memory isolation.
