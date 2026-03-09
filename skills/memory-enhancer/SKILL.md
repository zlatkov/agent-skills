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

- **Preferences**: "I prefer...", "Always...", "Never...", "Use X instead of Y"
- **Corrections**: User fixes a repeated mistake or says "I told you before..."
- **Decisions**: "We decided to...", "Let's go with...", "The plan is..."
- **Workflow**: "I usually...", "My process is...", "When I say X I mean Y"
- **Positive reinforcement**: "Perfect", "Yes, like that", "Always do it this way"
- **Skill feedback**: Corrections after any skill runs — "next time only show top 5", "make it shorter"

**Do NOT capture**: one-off requests ("just this once"), vague reactions ("ok", "thanks"), or transient task context.

## How to Capture

1. Use `memory_search` to check if a related entry already exists.
2. Write to `MEMORY.md`:
   - If the entry relates to a **skill**, put it under `## Skill Preferences`.
   - For **general preferences and decisions**, use `## Preferences` or an appropriate existing section.
   - If a matching entry exists, **update or replace** it. Don't create duplicates.
3. For context-heavy notes, also append to today's daily log (`memory/YYYY-MM-DD.md`).
4. Keep it concise — one line per entry, actionable phrasing.
5. Confirm briefly: *"Noted — I'll remember that."*

## How to Recall

Before responding to any task, `memory_search` runs automatically via OpenClaw. This skill enhances that by ensuring memories are **written in the first place** — the part OpenClaw leaves to explicit user requests.

## Housekeeping

- Max **30 entries** per section in `MEMORY.md`. When exceeded: merge duplicates, drop oldest.
- When an entry is contradicted by new feedback, replace it — don't keep both.
- If the user corrects something that's already in memory, update memory immediately.

## Direct Commands

- **"What do you remember?"** — summarize key entries from `MEMORY.md`.
- **"Forget {thing}"** — remove matching entries.
- **"Clear memory about {topic}"** — remove related entries.
