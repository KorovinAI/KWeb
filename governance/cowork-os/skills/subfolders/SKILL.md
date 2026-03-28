---
name: subfolders
description: Create new subfolders inside a Cowork workspace, complete with CLAUDE.md and MEMORY.md. Use this skill whenever the user asks to create a new folder, project space, or subfolder inside their Cowork workspace. Trigger phrases include: "create a folder," "make a new folder," "set up a subfolder," "I need a space for [X]," "build out a [X] folder," or any variation. Also triggers when wrapping up the initial /start setup and the user picks their first project area. Always use this skill — it handles the full interview, file creation, identity overrides, and memory isolation correctly.
---

# Subfolders Skill

Handles creating new subfolders inside a Cowork workspace, including CLAUDE.md and MEMORY.md for each.

---

## Before You Start

Set expectations first:

> "To set this up right, I'm going to ask you a few quick questions. If you want to skip ahead at any point, just say the word — I'll take what I have and run with it."

Then run the interview **one question at a time**. Don't list them all. Let it feel like a conversation.

---

## The Interview

### 1. Purpose
If the user hasn't already said what the folder is for, ask in chat (open-ended, not AskUserQuestion):

> "What will this folder be used for?"

If they already told you, skip this.

### 2. Folder Name
If the user hasn't already given a name, use **AskUserQuestion** to suggest 2–3 names based on the folder's purpose.

If they already gave a name, skip this.

### 3. Location
Look at the existing folder structure. Use **AskUserQuestion** to ask where the new folder should go. Use plain language — no "file path" or "directory" jargon.

Example phrasing:
> "Where should I put this folder?"

Options (adjust based on what actually exists):
- 📁 *Right here in the main workspace* — top level
- 📂 *Inside [existing folder name]* — as a subfolder

If a relevant parent folder exists (e.g., creating "Tax Prep" when "Finance" already exists), suggest it.

**If the workspace only has the root folder, skip this question** and default to root.

### 4. Context
Ask in chat:

> "Are there specific tools, files, or context I should know about for this folder?"

This is open-ended. Keep it brief — just capture what's unique about this space.

### 5. Identity Override
Use **AskUserQuestion** to ask:

> "When I'm working in this folder, do you want me to show up any differently?"

Dynamically generate options based on the folder's purpose. Always include:
- 👤 *Same as always* — keep my current name and personality
- 🎭 *[Specific expert role relevant to folder purpose]* — e.g., "Act as a financial analyst" for a finance folder, "Act as a senior editor" for a writing folder
- 🆕 *Custom identity* — I want to give you a different name or personality here

If they choose an expert role, adopt it for this folder only. If they choose custom, ask for the name and personality in chat.

If they choose "Same as always," no identity section needed in CLAUDE.md.

---

## Memory Isolation Reminder

Before finishing the interview, read the parent folder's `CLAUDE.md` and `MEMORY.md` (if they exist). Look for entries that are **workspace-specific** — things like active projects, ongoing tasks, folder-specific context, or decisions that were logged there.

**Do not surface anything that originates from personal preferences.** The user's name, communication style, tools, pain points, and personality preferences are universal and already visible in every session. Only consider content that was explicitly added to the parent folder's memory or instructions — not anything that echoes what's in personal preferences.

**If the parent folder has no meaningful content** (files don't exist, are empty, or only contain boilerplate structure with no actual entries), skip this step entirely. Don't manufacture suggestions. Just proceed to creating the files.

**If the parent folder does have meaningful, workspace-specific content**, explain:

> "My memory is tied to the folder I'm in, so anything from the main workspace won't carry over automatically."

Then use **AskUserQuestion** with multiSelect. Generate options only from actual content found in the parent folder's `CLAUDE.md` / `MEMORY.md`. Always include:
- ✅ *Nothing — start fresh*

Wait for their answer. Any items they select become the initial entries in the subfolder's MEMORY.md.

---

## Create the Files

### CLAUDE.md (inside the new subfolder)

Always include:

**Folder Purpose** — What this subfolder is for and what kinds of tasks it handles.

**Identity Override** *(only if requested)* — The assistant name and/or persona for this folder. Note that this overrides personal preferences while working here.

**Specific Instructions** — Tools, context, or preferences the user mentioned for this folder.

**Things to Remember** — Anything the user asked you to keep in mind for this space.

**Memory System** — Always include this section verbatim:

---

**MEMORY SYSTEM**

This folder contains a file called MEMORY.md. It is your external memory for this workspace — use it to bridge the gap between sessions.

**At the start of every session:** Read MEMORY.md before responding. Use what you find to inform your work — don't announce it, just be informed by it.

**Memory is user-triggered only.** Do not automatically write to MEMORY.md. Only add entries when the user explicitly asks — using phrases like "remember this," "don't forget," "make a note," "log this," "save this," or "create session notes." When triggered, write the information to MEMORY.md immediately and confirm you've done it.

**All memories are persistent.** Entries stay in MEMORY.md until the user explicitly asks to remove or change them. Do not auto-delete or expire entries.

**Flag contradictions.** If the user asks you to remember something that conflicts with an existing memory, don't silently overwrite it. Flag the conflict and ask how to reconcile it.

---

### MEMORY.md (inside the new subfolder)

Use this structure:

```
# Memory

_Last updated: [today's date]_

## Memory
<!-- Things the user has asked to remember. Persistent — only remove or change if the user asks. -->
```

If the user selected items to carry over during the memory isolation step, add them as entries under `## Memory`.

---

## Confirm What You Built

After creating the files, give the user a brief summary:

- Name of the subfolder created
- What's in CLAUDE.md (purpose, any identity override, specific instructions)
- That MEMORY.md is ready and what's already in it (if anything)

Keep it to 3–4 sentences. Don't over-explain.

---

## Edge Cases

**User skips the interview**: Take what you have and make sensible defaults. Create the folder with a basic CLAUDE.md covering purpose and the memory system. You can always refine later.

**Nested subfolders**: If the user wants a folder inside a subfolder, follow the same process. Just make sure you're clear about where it's going before creating it.

**Renaming a folder**: That's a separate request — this skill is for creation only.
