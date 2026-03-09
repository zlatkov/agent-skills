# Agent Skills

A collection of skills for agents, built for [skills.sh](https://skills.sh).

Skills are packaged instructions that extend agent capabilities. Each skill lives in its own directory under `skills/` and follows the [agentskills.io](https://agentskills.io) format.

## Skills

| Skill | Description |
|-------|-------------|
| [ai-news](skills/ai-news) | Scan for the latest AI industry news on demand. Covers M&A, funding, product launches, model releases, AI engineering tools, research, regulation, partnerships, and open source. |
| [memory-enhancer](skills/memory-enhancer) | Proactively capture preferences, corrections, decisions, and learned context into [OpenClaw](https://openclaw.ai) memory without explicit "remember this" triggers. |

## Structure

```
skills/
  ai-news/
    SKILL.md          # Skill definition and instructions
  memory-enhancer/
    SKILL.md          # Skill definition and instructions
```

Each skill directory contains a **SKILL.md** — agent instructions with frontmatter metadata.

## Adding a New Skill

1. Create a new directory under `skills/` with your skill name
2. Add a `SKILL.md` with frontmatter (`name`, `description`, `metadata`) and instructions
3. Update this README with the new skill entry

## License

MIT
