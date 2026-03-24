---
name: skilleval
description: >
  Evaluate agent skills (SKILL.md files) using the skilleval CLI. Runs trigger accuracy and compliance tests against LLM models to measure how well they understand and respond to a skill's instructions.

  TRIGGER when: user wants to evaluate, test, benchmark, or score a skill — e.g. "evaluate this skill", "run skilleval", "test my skill", "how does this skill score?", "benchmark this SKILL.md", "check skill quality", "does this skill trigger correctly?", "evaluate skills/ai-news", "run eval on all skills", "score my skills against Claude"

  DO NOT TRIGGER when: user is writing or editing a skill (not evaluating), asking about skilleval's source code, or discussing evaluation concepts without wanting to run an actual eval.

metadata: {"openclaw":{"emoji":"📊"}}
---

# Skill Evaluator

You help users evaluate their agent skills using [skilleval](https://github.com/zlatkov/skilleval) — a CLI that tests whether LLM models correctly trigger skills and follow their instructions.

## How skilleval Works

skilleval runs a 4-stage pipeline on each SKILL.md:

1. **Parse** — Extracts skill metadata and instructions from the SKILL.md frontmatter and body.
2. **Generate** — An LLM creates test prompts: half that *should* trigger the skill, half that *should not*.
3. **Test** — Each prompt is sent to the target model(s) with the skill injected into the system prompt alongside distractor skills.
4. **Judge** — An evaluator model scores trigger accuracy (did it fire when it should?) and compliance (did it follow instructions correctly?).

**Scoring formula:**
- **Trigger Accuracy (50%)** — % of correct trigger/no-trigger decisions
- **Compliance Accuracy (30%)** — % of proper tool calls when the skill was triggered
- **Compliance Score (20%)** — Average quality score (0–100) from the judge model
- **Overall** = `trigger_accuracy × 50 + compliance_accuracy × 30 + avg_compliance_score/100 × 20`
- A score of **50+** is a pass.

## Step 1: Check Prerequisites

Before running skilleval, verify the setup:

1. Confirm Node.js 18+ is available: run `node --version`.
2. Identify the skill(s) to evaluate — ask the user if not specified. Accept:
   - A single SKILL.md file path
   - A directory of skills (evaluates all SKILL.md files within)
   - A GitHub URL or shorthand (`owner/repo/path/to/SKILL.md`)
3. Determine which provider and API key to use. Supported providers:
   - `openrouter` (default) — env var: `OPENROUTER_API_KEY`
   - `anthropic` — env var: `ANTHROPIC_API_KEY`
   - `openai` — env var: `OPENAI_API_KEY`
   - `google` — env var: `GOOGLE_GENERATIVE_AI_API_KEY`
   - `azure` — env vars: `AZURE_API_KEY` **and** `AZURE_RESOURCE_NAME` (both required). Models are specified by **deployment name** (from Azure AI Foundry), not standard model IDs.
4. Check if the relevant API key env var is set. If not, ask the user to provide one or suggest they type `! export OPENROUTER_API_KEY=sk-...` to set it in this session. For Azure, also verify `AZURE_RESOURCE_NAME` is set.

## Step 2: Build the Command

Construct the `npx @alexanderzzlatkov/skilleval` command based on what the user wants:

**Basic evaluation:**
```bash
npx @alexanderzzlatkov/skilleval <path-or-url>
```

**Common options — only add flags the user explicitly requests or that are needed:**

| Flag | Purpose | Default |
|------|---------|---------|
| `-p, --provider <name>` | LLM provider | `openrouter` |
| `-m, --models <ids>` | Comma-separated model IDs to test | Provider default |
| `-k, --key <key>` | API key (alternative to env var) | From env |
| `-n, --count <num>` | Number of positive + negative test cases | `5` (= 10 total) |
| `--generator-model <id>` | Model used to create test prompts | Provider default |
| `--judge-model <id>` | Model used to score results | Provider default |
| `--prompts <path>` | Load custom test cases from a JSON file | Generated |
| `-s, --skill <name>` | Evaluate only this skill (batch mode) | All skills |
| `--verbose` | Show per-prompt breakdown | Off |
| `--json` | Output machine-readable JSON | Off |
| `--graph` | Show skill dependency graph (directory mode) | Off |

**Examples:**

```bash
# Evaluate a single skill with default settings
npx @alexanderzzlatkov/skilleval ./skills/ai-news/SKILL.md

# Evaluate all skills in a directory
npx @alexanderzzlatkov/skilleval ./skills/

# Test against specific models via Anthropic
npx @alexanderzzlatkov/skilleval ./skills/ai-news/SKILL.md -p anthropic -m "claude-sonnet-4-20250514"

# More test cases for higher confidence
npx @alexanderzzlatkov/skilleval ./skills/ai-news/SKILL.md -n 10

# Evaluate a skill from GitHub
npx @alexanderzzlatkov/skilleval https://github.com/zlatkov/agent-skills/blob/master/skills/ai-news/SKILL.md

# Verbose output with JSON for CI
npx @alexanderzzlatkov/skilleval ./skills/ --verbose --json
```

## Step 3: Run the Evaluation

1. Show the user the constructed command and explain what it will do.
2. Run the command. skilleval typically takes 1–3 minutes per skill depending on model count and test case count.
3. If the command fails:
   - **Missing API key** — prompt the user to set the env var or pass `-k`.
   - **Invalid skill path** — verify the path exists and contains valid SKILL.md frontmatter.
   - **Rate limits** — suggest reducing `-n` or switching to a different provider/model.
   - **Network errors** — suggest retrying.

## Step 4: Interpret Results

skilleval outputs a results table with columns for each model tested. Explain the results:

- **Trigger Accuracy** — How often the model correctly decided whether to activate the skill. Low scores mean the skill's trigger description is ambiguous or too broad.
- **Compliance Accuracy** — When triggered, did the model make the right tool calls? Low scores suggest the instructions are unclear about *what* to do.
- **Compliance Score** — Quality of the model's response when it did comply. Low scores mean the instructions lack specificity on *how* to do it.
- **Overall Score** — Weighted composite. 50+ is passing.

**If scores are low, suggest concrete improvements:**

- **Low trigger accuracy** → The `TRIGGER when` / `DO NOT TRIGGER` lines in frontmatter need sharper examples. Add more specific trigger phrases and clearer exclusions.
- **Low compliance accuracy** → The skill body needs clearer step-by-step instructions and explicit tool call expectations.
- **Low compliance score** → Instructions are being followed but not well. Add examples, formatting guidance, or tighter constraints.

## Step 5: Iterate (Optional)

If the user wants to improve their skill based on results:

1. Identify the weakest scoring dimension.
2. Suggest targeted edits to the SKILL.md.
3. Re-run the evaluation to measure improvement.
4. Repeat until the score meets the user's threshold.

When using `--prompts` with a custom JSON file, the format is:

```json
[
  { "prompt": "What's the latest AI news?", "shouldTrigger": true },
  { "prompt": "Write me a poem about cats", "shouldTrigger": false }
]
```

This is useful for testing specific edge cases that the auto-generated prompts may miss.
