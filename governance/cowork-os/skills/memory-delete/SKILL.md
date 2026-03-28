---
name: memory-delete
description: Remove or clear entries from MEMORY.md in the current Cowork workspace or subfolder. Use this skill whenever the user wants to delete, remove, clear, or wipe memory entries. Trigger phrases include: "forget that," "remove that from memory," "delete that note," "clear my memory," "wipe memory," "that's no longer true," "remove the note about [X]," or any variation asking you to get rid of something you previously remembered. Always use this skill instead of editing MEMORY.md ad hoc — it handles targeted deletion, full clears, and confirmation correctly.
---

# Memory Delete Skill

Handles removing entries from MEMORY.md in the current working folder.

---

## Rules

**Deletion is always user-triggered.** Never remove entries on your own — not to clean up, not because something seems outdated, not for any reason. Only run this skill when the user explicitly asks.

**Confirm before deleting.** Always show the user what you're about to remove and get confirmation before making changes — unless they said something unambiguous like "clear everything" or "wipe my memory."

**Don't delete personal preferences.** MEMORY.md is for session-specific notes. If the user asks you to "forget" something that lives in their personal preferences (not MEMORY.md), let them know: *"That's in your personal preferences, not here — you'd need to update that in Claude Settings → Personal Preferences."*

---

## Step 1 — Locate MEMORY.md

Look for MEMORY.md in the current folder. If it doesn't exist, tell the user:

> "There's nothing in memory here yet."

Stop.

---

## Step 2 — Identify what to delete

**Targeted deletion** (user names a specific entry):
- Search MEMORY.md for matching entries
- If found: show the user the exact line(s) and confirm:
  > "I found this — want me to remove it? [entry]"
- If not found: tell the user you couldn't find a matching entry and show them what's currently in memory so they can point to it

**Full clear** (user says "clear everything," "wipe memory," etc.):
- Show them a summary of what's currently in memory
- Confirm once:
  > "This will remove all [N] entries. Confirm?"
- Wait for a yes before proceeding

---

## Step 3 — Delete

**Targeted**: Remove only the matching line(s). Leave all other entries intact. Update the `_Last updated:` date.

**Full clear**: Remove all content under `## Memory` but keep the file structure intact:

```
# Memory

_Last updated: [today's date]_

## Memory
<!-- Things the user has asked to remember. Persistent — only remove or change if the user asks. -->
```

---

## Step 4 — Confirm

After deleting, confirm briefly:

> "Done — removed." *(for targeted)*

> "Cleared. Memory is empty." *(for full wipe)*

One line. Don't editorialize.

---

## Edge Cases

**Ambiguous request**: If the user says something like "forget what I said about thumbnails" but there are multiple thumbnail-related entries, show them all and ask which one(s) to remove.

**Subfolder vs. root**: Always operate on MEMORY.md in the current folder. If the user wants to clear memory in a different folder, they need to navigate there first.

**Entry was never logged**: If the user asks you to forget something but it was never written to MEMORY.md (maybe it was just said in conversation), let them know: *"I don't have that in memory — it was only in our conversation, which I don't retain between sessions anyway."*
