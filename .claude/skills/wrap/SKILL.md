---
name: wrap
description: >
  End-of-session commit and push. Reviews all changes across client website
  submodules and the KWeb root repo, presents a summary, and on confirmation
  commits and pushes everything to GitHub.
---

# /wrap — Session Wrap

Read the full workflow spec from `governance/git-manager/commands/wrap.md` and execute it.

Use the commit message format from `governance/git-manager/references/commit-playbook.md`.

## Quick Reference

1. Scan all client submodules and root repo for uncommitted changes
2. Present a single change summary showing what changed at each level
3. Propose commit messages for each level
4. Wait for a single confirmation before proceeding
5. Run pre-commit safety scan before each commit
6. Commit inside-out: submodules first, then root
7. Push submodules first, then root
8. Report the result

If nothing has changed, say so and stop.
