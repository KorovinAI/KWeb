# Cowork OS

A Claude Cowork plugin that handles workspace setup, memory management, and subfolder creation.

## What's included

**`/begin` command**
One-time setup flow for a new workspace. Plays the wake-up sequence, runs the personal preferences interview if needed, and creates `CLAUDE.md` and `MEMORY.md` in the current folder. Transitions into building your first subfolder.

**`memory-create` skill**
Triggered by phrases like "remember this," "make a note," "log this," or "don't forget." Writes entries to `MEMORY.md`, checks for contradictions before saving, and confirms when done.

**`memory-delete` skill**
Triggered by phrases like "forget that," "remove that from memory," or "clear my memory." Handles targeted deletion or full wipes, always confirms before making changes.

**`subfolders` skill**
Triggered when you ask to create a new folder or project space. Runs a short interview (purpose, name, location, identity override, memory isolation), then creates the subfolder with its own `CLAUDE.md` and `MEMORY.md`.

## How to use

1. Open a Claude Cowork folder
2. Type `/cowork-os:begin` to initialize the workspace
3. Memory and subfolder skills activate automatically from natural language — no commands needed

## File structure created

```
your-workspace/
├── CLAUDE.md        # Workspace instructions
├── MEMORY.md        # Persistent memory for this workspace
└── your-subfolder/
    ├── CLAUDE.md    # Subfolder instructions + identity override (if set)
    └── MEMORY.md    # Isolated memory for this subfolder
```
