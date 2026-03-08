---
name: skill-tuner
description: >
  Capture user corrections and feedback after any skill runs, persist them as learned instructions, and silently apply them on future invocations.

  TRIGGER when: always active after any skill execution. Also invoked directly: "What have you learned about {skill}?", "Forget that for {skill}", "Show skill tuning", "Clear skill tuning for {skill}"

metadata: {"openclaw":{"emoji":"🧠"}}
---

# Skill Tuner

After any skill produces output, watch for user corrections and feedback. Persist them as learned instructions so future runs of that skill automatically improve.

## Storage

Store per-skill files at `$SKILL_TUNER_DIR/{skill-name}.md`. If the env var is unset, fall back to `.skill-tuner/` in the project root. Gitignore this directory — memory files should not be committed.

## File Format

```markdown
# Skill Tuner: {skill-name}

## Learned

- [YYYY-MM-DD] {concise, actionable instruction}
- [YYYY-MM-DD] (replaces: {old date}) {updated instruction}
```

Each entry is one line. If a new instruction contradicts an existing one, replace it. If it refines one, update in-place. Merge duplicates.

## Size Limits

Defaults: **30 entries** / **50 lines** max per file. Override via `{skill-tuner-dir}/config.json`:

```json
{
  "default": { "max_entries": 30, "max_lines": 50 },
  "skills": { "ai-news": { "max_entries": 20 } }
}
```

When exceeded: deduplicate first, then evict oldest entries, then consolidate themed entries into summaries.

## Capture (after skill runs)

Only capture clear, intentional corrections — not one-off or situational requests.

**Capture** when user says: "Always...", "Never...", "From now on...", "Next time...", "Remember to...", or directly corrects output with forward-looking intent.

**Skip** when: "just this once", "for now", ambiguous reactions, or context-dependent requests.

Steps:
1. Identify which skill just ran.
2. Extract the actionable instruction.
3. Read existing file, check for contradictions/duplicates.
4. Write or update entry. Run size management if needed.
5. Confirm briefly: *"Noted: {instruction}. I'll apply this next time."*

## Recall (before skill runs)

Check if a memory file exists for the skill about to run. If so, read it and silently apply the instructions as additional constraints. Do not mention the tuning system unless asked.

## Direct Commands

- **"What have you learned about {skill}?"** — show the file.
- **"Forget {instruction} for {skill}"** — remove an entry.
- **"Clear skill tuning for {skill}"** / **"Clear all skill tuning"** — delete files.
- **"Show skill tuning"** — list all skills with entry counts.
