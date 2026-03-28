---
name: memory-create
description: Write entries to MEMORY.md in the current Cowork workspace or subfolder. Use this skill whenever the user asks you to remember something, save something, make a note, log something, or says "don't forget." Trigger phrases include: "remember this," "make a note," "save this," "log this," "don't forget," "create session notes," or any variation. Also triggers when wrapping up a session and the user wants to capture what happened. Always use this skill instead of writing to MEMORY.md ad hoc — it handles formatting, contradiction checking, and persistence rules correctly.
---

# Memory Create Skill

Handles writing new entries to MEMORY.md in the current working folder.

---

## Rules

**Memory is user-triggered only.** Never write to MEMORY.md automatically. Only run this skill when the user explicitly asks using phrases like:
- "remember this"
- "make a note"
- "save this"
- "log this"
- "don't forget"
- "create session notes"

**All memories are persistent.** Once written, entries stay until the user explicitly asks to remove or change them. Never auto-expire or clean up entries on your own.

**Only log things not already in personal preferences.** If the user asks you to remember something already covered by their personal preferences (name, tools, communication style, etc.), acknowledge it but don't duplicate it in MEMORY.md.

---

## Step 1 — Locate MEMORY.md

Look for MEMORY.md in the current folder. If it doesn't exist, create it using this format:

```
# Memory

_Last updated: [today's date]_

## Memory
<!-- Things the user has asked to remember. Persistent — only remove or change if the user asks. -->
```

---

## Step 2 — Check for contradictions

Before writing, scan existing entries in MEMORY.md for anything that conflicts with what the user just asked you to remember.

If there's a conflict, **stop and flag it**:

> "Hey, this seems different from what I have on file — [existing entry]. How do you want to reconcile this?"

Wait for their answer. Then update based on what they say — either overwrite, keep both, or discard the new entry.

If no conflict, proceed.

---

## Step 3 — Write the entry

Add the new entry under the `## Memory` section. Keep entries:
- **Concise** — one or two lines max per entry
- **Specific** — write what was actually said, not a vague summary
- **Dated** — add the date in parentheses at the end: `(added [date])`

Update the `_Last updated:` line at the top to today's date.

Example entry format:
```
- Paul prefers video thumbnails with plain white backgrounds for his YouTube content. (added 2025-01-15)
```

---

## Step 4 — Confirm

After writing, confirm briefly in chat:

> "Got it — noted in memory."

Don't over-explain. One line is enough unless the user asks for more.

---

## Edge Cases

**Session notes**: If the user asks to "create session notes" or "log what we did," write a dated summary block under Memory. Keep it scannable — bullet points, not paragraphs.

**Multiple things at once**: If the user asks you to remember several things, write each as its own bullet entry. Confirm once after all are written.

**Subfolder vs. root**: Always write to MEMORY.md in the current folder — whichever folder you're working in. Don't write to a parent folder's MEMORY.md unless the user specifically asks.
