# KWeb — Website Building Workspace

This is Ilia's website building shop. KWeb is where client websites are built, managed, and deployed. Each client project lives as a repo inside the `websites/` folder.

**Web build standard:** The default stack, hosting, database, architecture, SEO/AEO rules, capability modules, and two-tier delivery model for every client website are defined in [`web-stack.md`](./web-stack.md) at the workspace root. **Read it before starting or modifying any site in `websites/`.** It is the single source of truth for how Korovin AI builds websites — keep it updated (with rationale) when the stack evolves.

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

## Local Testing & Deploy Discipline

Hard-won lesson: Phase 3 of the agency site burned 20 production deploys debugging a routing issue that was reproducible — and fixable — locally. The build/test/push loop must stay local until the work is genuinely done. These rules apply to every client site in `websites/`.

**Test locally before pushing — always.**
- Build the production output locally first: `npm run build` (or stack equivalent).
- Preview the production build locally. The dev server (`npm run dev`) is for editing, not verification — it does not match production behaviour. Production-equivalent preview by stack:
  - **Next.js with `output: 'export'`** (static): `npx serve out`
  - **Next.js with server runtime**: `npm run start`
  - **Astro**: `npm run preview`
  - Add more here as new stacks join the workspace.
- If a route works against `localhost`, it'll work on the deployment platform 95%+ of the time. The remaining 5% is platform-specific bugs — diagnose those with discipline (below), not by push-and-check.

**Push only at session end, not during.**
- Iterative push-and-check cycles burn deployment quotas fast. Netlify's free tier allows ~300 build minutes / 20 deploys per month — one over-zealous debugging session can blow the entire budget.
- Use `/wrap` to commit and push everything when work is done. Mid-session pushes should be rare and intentional.
- Exception: a one-line fix you're 100% sure of (typo, env var) can push directly. Most debug cycles should NOT trigger a push.

**Diagnostic discipline.**
- When a deployed page misbehaves, reproduce it locally first by running the same production build (`npm run build`) and inspecting the output (e.g. `out/`, `.next/server/`, `dist/`).
- If a diagnostic genuinely needs to run on the deployment infrastructure (rare), expose the diagnostic via rendered HTML that you can curl after deploy — not by adding `console.log` and trying to log-mine the platform.
- Maximum one diagnostic push per debug session. If you're on a second, the local-verify process is broken; fix that process before deploying again.

**Deploy budget per session: ≤ 2–3 production deploys.**
- If a third is needed, stop and ask: what does my local setup not catch that the deploy does? Fix that gap first.
- Use `[skip ci]` in commit messages for changes that don't need a deploy (docs, plans, READMEs). Netlify (and most platforms) read this header and skip the build:
  ```
  git commit -m "docs: update plan with session learnings [skip ci]"
  ```

**Stack-specific commands and gotchas** belong in each client's `websites/<client>/CLAUDE.md`. The principles above are stack-agnostic; the exact `npm` scripts and quirks vary per project.

---

## Workspace Structure

- **`websites/`** — Client website repos. Each client gets their own independent git repo here, connected to the root as a **git submodule**. Each repo has its own GitHub remote (typically `KorovinAI/<client-name>`) and is deployed per the web build standard (see [`web-stack.md`](./web-stack.md)).
- **`governance/`** — Skills that govern how the workspace is organized, structured, and maintained. Meta-skills — they define conventions that other skills and workflows follow.
- **`output-files/`** — Centralized dump for all generated deliverables (spreadsheets, PDFs, docs, decks, CSVs). Flat directory, self-documenting filenames (include client name + date).
- **`web-stack.md`** — The web build standard: default stack, hosting, database, architecture, SEO/AEO rules, capability modules, and the two-tier delivery model. A living cross-project reference (not a dated deliverable). Read before any client website work; update it (with rationale) when the stack genuinely evolves.

## Git Architecture

KWeb uses a two-level git structure:
- **Root repo** (`KorovinAI/KWeb`, public) — tracks governance skills, workspace config, and submodule references.
- **Client repos** (each in `websites/<client-name>/`) — independent repos connected as git submodules. Each has its own commit history, remote, and deployment (hosting per [`web-stack.md`](./web-stack.md)).

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
