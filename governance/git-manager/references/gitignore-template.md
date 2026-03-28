# Workspace .gitignore Template

Deploy this to the workspace root at init. Do not modify without the user's approval.

```gitignore
# ─── macOS ───────────────────────────────────────────────
.DS_Store
**/.DS_Store
.AppleDouble
.LSOverride

# ─── Output data — generated, not source ─────────────────
*.xlsx
*.csv
*.xls
*.docx

# ─── Secrets and credentials ─────────────────────────────
.env
*.env
**/.env
**/secrets.*
**/credentials.*
**/token.*
**/*.pem
**/*.key
**/*.p12
**/*.pfx
.netrc

# ─── Large binaries ──────────────────────────────────────
# NOTE: *.skill files are NOT ignored — they are packaged installers and must be tracked
*.mcpb
*.zip
*.bin
*.tar.gz
*.tar
*.gz

# ─── Python environments ─────────────────────────────────
venv/
**/venv/
.venv/
**/.venv/
__pycache__/
**/__pycache__/
*.pyc
*.pyo
*.pyd
*.egg-info/
dist/
build/

# ─── Node ────────────────────────────────────────────────
node_modules/
**/node_modules/
.npm/

# ─── Skill scratch space (NOT evals — those are authored) ─
**/workspace/
**/scratch/

# ─── Web build artifacts ────────────────────────────────
_site/
**/_site/
.next/
**/.next/
.nuxt/
**/.nuxt/
.output/
**/.output/
.cache/
**/.cache/
.parcel-cache/
.hugo_build.lock

# ─── Temp and generated outputs ──────────────────────────
*.log
*.tmp
*.swp
*.swo
*~
dashboard.html

# ─── IDE and editor artifacts ────────────────────────────
.idea/
.vscode/
*.code-workspace

# ─── Git internals (never re-add these) ──────────────────
*.orig
*.rej
MERGE_HEAD
MERGE_MSG
```

## What is NOT ignored (tracked intentionally)

- `CLAUDE.md` — all instances, all levels
- `MEMORY.md` — all instances, all levels
- `TASKS.md` — root task tracker
- `SKILL.md` — all skills
- `*.skill` — packaged skill installer files
- `references/*.md` — authored reference docs (markdown only — .docx in references/ is still ignored by the global *.docx rule)
- `evals/` — authored test cases (NOT generated outputs)
- `commands/*.md` — slash command definitions
- Source code (`.py`, `.js`, `.html`, `.css`, `requirements.txt`, `README.md`, configs)

## Notes

- `workspace/` scratch folders inside skills are ignored — they're scratch, not source
- `evals/` folders are tracked — they contain authored test cases
- `.skill` binary files ARE tracked — they're the packaged installer, users need them
- If a new file type is ambiguous, ask the user before adding or ignoring
