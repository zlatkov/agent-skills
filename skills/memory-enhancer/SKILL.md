---
name: memory-enhancer
description: >
  Proactively capture preferences, corrections, decisions, and learned context into OpenClaw memory without the user needing to say "remember this." Makes memory feel natural.

  TRIGGER when: user states a preference, correction, or durable instruction — e.g. "always use bullet points", "don't include X", "from now on...", "I prefer...", "next time...", "perfect, do it like that", "we decided to...", or corrects a repeated mistake. Also when user asks: "What do you remember?", "Show memory", "Forget {thing}"

  DO NOT TRIGGER for one-off requests, vague reactions, or when the user is just chatting.

metadata: {"openclaw":{"emoji":"🧠"}}
---

# Memory Enhancer

Make OpenClaw's memory work naturally. Capture what matters without explicit asks.

## What to Capture

Listen for signals that something should persist across sessions:

| Signal Type | Examples |
|-------------|---------|
| **Preferences** | "I prefer...", "Always...", "Never...", "Use X instead of Y" |
| **Corrections** | User fixes a repeated mistake or says "I told you before..." |
| **Decisions** | "We decided to...", "Let's go with...", "The plan is..." |
| **Workflow** | "I usually...", "My process is...", "When I say X I mean Y" |
| **Positive reinforcement** | "Perfect", "Yes, like that", "Always do it this way" |
| **Skill feedback** | Corrections after any skill runs — "next time only show top 5", "make it shorter" |
| **Project context** | Stack choices, architecture decisions, key constraints, repo conventions |
| **People/team** | Teammates, their roles, communication style preferences |

**Do NOT capture**: one-off requests ("just this once"), vague reactions ("ok", "thanks"), conversational filler, or transient task context that won't matter next session.

## Memory Entry Format

Every entry should be:
- **Actionable**: phrased as an instruction or fact, not a quote
- **Concise**: one line per entry
- **Tagged**: prefix with a short category tag in brackets

**Good entries:**
```
[format] Use bullet points for all lists, never numbered unless the user explicitly asks
[format] Keep code snippets short — max 20 lines unless the full file is requested
[stack] Frontend is Next.js 14 with App Router; never suggest Pages Router patterns
[workflow] User reviews PRs on mobile — keep diffs readable, avoid huge single-commit changes
[skill:ai-news] Show max 5 items per digest; skip funding rounds under $20M
[person] Alice = lead designer; she owns all brand/color decisions
```

**Bad entries (avoid):**
```
User likes bullet points  ← not actionable, no tag
Remember to use Next.js   ← vague, no context
User said "perfect" on 2024-01-15  ← captures a reaction, not a preference
```

## Memory Sections in MEMORY.md

Use these sections (create if missing):

- `## Preferences` — formatting, communication style, output length, tone
- `## Decisions` — architectural, product, or process decisions made
- `## Project Context` — stack, repo conventions, constraints, goals
- `## People & Team` — teammates, roles, communication preferences
- `## Skill Preferences` — per-skill behavior adjustments

## How to Capture

1. **Search first**: run `memory_search` with the topic of the new memory to check for existing entries. Search for the subject (e.g., `memory_search("bullet points")` or `memory_search("Next.js")`).
2. **Determine section**: match to the appropriate section from the list above.
3. **Write or update**:
   - If no matching entry exists: append a new line to the correct section.
   - If a matching entry exists: replace it in-place. Do not keep both.
4. **Temporal memories**: if the memory is clearly short-lived (e.g., "I'm in a hurry today, keep it brief"), append to today's daily log (`memory/YYYY-MM-DD.md`) instead of `MEMORY.md`.
5. **Confirm briefly**: one short sentence. See confirmation format below.

## Confirmation Format

After writing to memory, confirm with a single line:

> Noted — [one-phrase summary of what was saved].

**Examples:**
- *"Noted — always use bullet points for lists."*
- *"Noted — ai-news digest capped at 5 items."*
- *"Noted — Alice owns all brand decisions."*

Do not say "I've updated your memory file" or give technical details unless asked.

## How to Recall ("What do you remember?")

When the user asks what's in memory, summarize `MEMORY.md` grouped by section. Format:

```
**Preferences** (N)
- [format] Use bullet points for all lists
- [format] Keep responses concise — under 200 words unless asked to elaborate
...

**Decisions** (N)
- [stack] Postgres over MongoDB — decided 2024-03 for relational data needs
...

**Skill Preferences** (N)
- [skill:ai-news] Max 5 items, skip small funding rounds
...
```

If memory is empty or sparse, say so honestly: *"Not much saved yet — just [X]."*

## Forget Commands

- **"Forget {thing}"** — search memory for matching entries, remove them, confirm: *"Removed — [what was deleted]."*
- **"Clear memory about {topic}"** — remove all entries in any section related to the topic.
- **"Clear all memory"** — ask for confirmation before clearing `MEMORY.md` entirely.

## Housekeeping

- Max **30 entries** per section in `MEMORY.md`. When exceeded:
  1. Merge entries that cover the same subject.
  2. If still over 30, remove the **least specific** entries first (general stylistic preferences over specific technical decisions).
  3. Never drop project context or decisions silently — if forced to, warn the user.
- When new feedback contradicts an existing entry, replace it immediately. Never keep both.
- Temporal entries in daily logs do not count toward the 30-entry limit.

## Direct Commands Summary

| Command | Action |
|---------|--------|
| `"What do you remember?"` | Summarize `MEMORY.md` by section |
| `"Show memory"` | Same as above |
| `"Forget {thing}"` | Remove matching entries |
| `"Clear memory about {topic}"` | Remove all topic-related entries |
| `"Clear all memory"` | Confirm, then wipe `MEMORY.md` |
