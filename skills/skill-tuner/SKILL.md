---
name: skill-tuner
description: >
  Capture user corrections and feedback after any skill runs, persist them as learned instructions, and silently apply them on future invocations.

  TRIGGER when: user gives feedback or corrections after a skill runs — e.g. "next time only show top 5", "always use bullet points", "don't include X", "from now on...", "remember to...". Also: "What have you learned about {skill}?", "Show skill tuning", "Clear skill tuning for {skill}"

metadata: {"openclaw":{"emoji":"🧠"}}
---

# Skill Tuner

Improve skills over time by capturing user feedback and applying it on future runs.

## When to Capture

When a user responds to a skill's output with a correction or forward-looking instruction, capture it. Look for phrases like:
- "Always...", "Never...", "From now on...", "Next time...", "Remember to..."
- "Don't include...", "Make it shorter", "Use bullet points", "Only show..."
- "Perfect, always do it like that"

**Do NOT capture** one-off requests ("just this once", "for now") or vague reactions.

## How to Capture

1. Create the storage directory if it doesn't exist:
   - Use `$SKILL_TUNER_DIR` if set, otherwise `.skill-tuner/` in the project root.
   - Add it to `.gitignore` if not already there.
2. Read `{storage-dir}/{skill-name}.md` if it exists.
3. If the new instruction contradicts an existing entry, replace it. If it refines one, update it. Otherwise append.
4. Write the file with this format:

```markdown
# Skill Tuner: {skill-name}

## Learned

- [YYYY-MM-DD] {concise, actionable instruction}
```

5. Confirm to the user: *"Noted: {instruction}. I'll apply this next time."*

## How to Recall

**Before running any skill**, check if `{storage-dir}/{skill-name}.md` exists. If it does, read it and apply every instruction in the `## Learned` section as additional constraints. Do not mention this to the user — just apply them silently.

## Size Limits

Max **30 entries** / **50 lines** per file. When exceeded: merge duplicates, then remove oldest entries first. Override limits via `{storage-dir}/config.json`:

```json
{
  "default": { "max_entries": 30, "max_lines": 50 },
  "skills": { "ai-news": { "max_entries": 20 } }
}
```

## Direct Commands

- **"What have you learned about {skill}?"** — read and display the file.
- **"Forget {instruction} for {skill}"** — remove that entry.
- **"Clear skill tuning for {skill}"** / **"Clear all skill tuning"** — delete file(s).
- **"Show skill tuning"** — list all skills with entry counts.
