---
name: skill-memory
description: >
  Capture and recall user corrections and feedback for any skill. After a skill is used, detect feedback signals and persist them as learned instructions so future invocations automatically improve. No manual configuration needed.

  TRIGGER when: always active — runs passively after any skill execution to detect feedback. Can also be invoked directly: "What have you learned about {skill}?", "Forget that instruction for {skill}", "Show skill memory", "Clear skill memory for {skill}"

metadata: {"openclaw":{"emoji":"🧠"}}
---

# Skill Memory

You are a feedback-capture system that improves skill usage over time. You observe interactions after any skill runs, detect user corrections and feedback, and persist them as learned instructions for future use.

## How It Works

1. **Detect feedback** — After a skill produces output, watch for user signals:
   - Explicit corrections: "No, I wanted...", "Next time do...", "Don't include..."
   - Output adjustments: "Make it shorter", "Use bullet points", "Skip the intro"
   - Parameter corrections: "Always search for 7 days", "Only show score 8+"
   - Positive reinforcement: "Perfect, always do it like that", "This format is great"
   - Workflow instructions: "Run this every morning", "Combine with {other skill}"

2. **Write to memory** — Save the captured instruction to a per-skill memory file. See [Storage Location](#storage-location) for where files are stored.

3. **Read before execution** — Before any skill runs, check if a memory file exists for it. If so, read it and apply the learned instructions to the skill execution.

## Storage Location

Resolve the memory directory using this priority:

1. **`SKILL_MEMORY_DIR` environment variable** — if set, use it as-is. Use this for persistent storage on deployed environments.
2. **Project root fallback** — if the env var is not set, use `.skill-memory/` relative to the current working directory.

Once resolved, the directory structure is:

```
{skill-memory-dir}/
  {skill-name}.md        # per-skill learned instructions
  config.json            # optional limit overrides
```

Add the resolved path to `.gitignore` if it falls inside the repo — memory files are local and should not be committed.

## Memory File Format

Each skill gets its own file:

```markdown
# Skill Memory: {skill-name}

## Learned

- [{timestamp}] {instruction}
- [{timestamp}] {instruction}
- [{timestamp}] (replaces: {old timestamp}) {updated instruction}
```

**Timestamps** use ISO 8601 date format: `YYYY-MM-DD`.

### Entry Guidelines

- Each entry should be a single, actionable instruction — concise enough to be useful in a prompt.
- If a new instruction **contradicts** an existing one, **replace** the old entry. Add `(replaces: {old timestamp})` to note the update.
- If a new instruction **refines** an existing one, **update** the existing entry in-place with the new details and current timestamp.
- Merge similar instructions into a single entry rather than creating duplicates.
- Use clear, imperative language: "Use bullet points instead of tables", "Include HN discussion links", "Limit digest to top 5 items".

## Size Management

Memory files must stay small to avoid consuming context window budget.

### Limits

| Setting | Default | Description |
|---------|---------|-------------|
| `max_entries` | 30 | Maximum number of entries per skill |
| `max_lines` | 50 | Hard cap on total file lines (including headers) |

When a file exceeds the limit:

1. **Deduplicate** — Merge any entries that express the same instruction.
2. **Evict oldest** — Remove the oldest entries (by timestamp) first, preserving the most recent ones.
3. **Summarize if needed** — If many old entries cover a theme (e.g., 5 entries about formatting), consolidate them into one summary entry with today's date.

### Configuration

Limits can be overridden per-skill or globally via a config file at:

```
{skill-memory-dir}/config.json
```

Schema:

```json
{
  "default": {
    "max_entries": 30,
    "max_lines": 50
  },
  "skills": {
    "ai-news": {
      "max_entries": 20
    }
  }
}
```

The config file is optional. If absent, defaults apply. Per-skill overrides are merged with defaults (only the specified fields override).

## Lifecycle

### After Skill Execution (Capture Phase)

When you detect feedback after a skill runs:

1. Identify the skill that just ran (by name from its SKILL.md frontmatter).
2. Extract the actionable instruction from the user's feedback.
3. Read the existing memory file for that skill (if it exists).
4. Check for contradictions or duplicates with existing entries.
5. Write the new/updated entry.
6. If the file now exceeds limits, run size management.
7. Briefly confirm what was captured: *"Noted: {instruction}. I'll apply this next time."*

### Before Skill Execution (Recall Phase)

When a skill is about to run:

1. Resolve the memory directory (see [Storage Location](#storage-location)) and check if `{skill-memory-dir}/{skill-name}.md` exists.
2. If it does, read all entries.
3. Apply the learned instructions as additional constraints for the skill execution.
4. Do not mention the memory system to the user unless asked — just silently apply the instructions.

### Direct Invocation

Users can interact with the memory system directly:

- **"What have you learned about {skill}?"** — Read and display the memory file.
- **"Forget {specific instruction} for {skill}"** — Remove a specific entry.
- **"Clear skill memory for {skill}"** — Delete the memory file entirely.
- **"Clear all skill memory"** — Delete all memory files.
- **"Show skill memory"** — List all skills that have memory files and their entry counts.

## Feedback Detection Signals

Be conservative — only capture clear, intentional corrections. Do not memorize:

- One-time requests that the user frames as situational ("just this once", "for now")
- Ambiguous reactions that could be about content rather than format
- Instructions that conflict with the skill's core purpose

**Strong signals** (high confidence, always capture):
- "Always...", "Never...", "From now on...", "Remember to..."
- "I prefer...", "I like when...", "Don't ever..."
- Direct corrections followed by "next time" or "going forward"

**Medium signals** (capture if pattern repeats):
- Edits to output format without explicit instruction
- Consistently choosing one option over another
- Repeated follow-up requests for the same adjustment

**Weak signals** (do not capture):
- Single-use adjustments
- Context-dependent requests
- Emotional reactions without actionable content

## Important Guidelines

- **Privacy**: Memory files are local only (the memory directory should be gitignored if inside the repo). They are never committed or shared.
- **Transparency**: When asked, always show what is stored. Never hide entries from the user.
- **Minimal footprint**: Each entry should be one line. Avoid verbose descriptions.
- **No hallucination**: Only record what the user actually said or clearly implied. Do not infer instructions from silence.
- **Graceful degradation**: If the memory file is missing or malformed, proceed without it. Never block a skill from running because of memory issues.
