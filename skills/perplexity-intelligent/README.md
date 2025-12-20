# Perplexity Intelligent

Better web research for Claude Code using Perplexity's Sonar API.

## What It Does

When you ask Claude Code to research something, this skill uses Perplexity instead of the built-in WebSearch. You get higher quality results with proper citations, and every query is automatically saved to build a searchable knowledge base in your project.

## Why This Over WebSearch

- **Better sources**: Perplexity synthesizes from high-quality sources, not just web snippets
- **Reasoning by default**: Uses models that show their work (auditable conclusions)
- **Auto-documentation**: Every query saved to `.claude/perplexity-research/`
- **RCA support**: Systematic debugging/troubleshooting with hypothesis tracking
- **Citations always**: Full URLs for every claim

## Features

| Feature | What It Means |
|---------|---------------|
| **Automatic model selection** | Picks the right Perplexity model for your query |
| **Reasoning traces** | See how conclusions were reached, catch logical errors |
| **Research threads** | Queries logged per-topic, persist across sessions |
| **Multi-model synthesis** | Cross-reference multiple models for high-stakes research |
| **RCA workflow** | Structured root cause analysis with hypothesis tracking |

## Quick Start

### 1. Copy the skill

```bash
# Global (all projects)
cp -r perplexity-intelligent ~/.claude/skills/

# Or per-project
cp -r perplexity-intelligent .claude/skills/
```

### 2. Add your API key

Get a key from [Perplexity API Keys](https://www.perplexity.ai/account/api/keys), then:

```bash
cp ~/.claude/skills/perplexity-intelligent/config/api-key.env.example \
   ~/.claude/skills/perplexity-intelligent/config/api-key.env

# Edit the file and add your key
```

### 3. Tell Claude to use it

Add to your CLAUDE.md:

```markdown
## Research
Use ~/.claude/skills/perplexity-intelligent/ for web research, not WebSearch.
```

## Usage

Once configured, Claude Code automatically uses this for research:

- "Research the current state of WebAssembly"
- "What are the differences between Bun and Deno?"
- "Why is my build failing?" (triggers RCA workflow)

Or invoke explicitly: `/perplexity-intelligent [query]`

## Model Selection

| Query Type | Model Used |
|------------|------------|
| Simple factual lookup | sonar |
| **Everything else (default)** | **sonar-reasoning-pro** |
| Exhaustive research | sonar-deep-research |

The skill defaults to reasoning models for auditability. Simple lookups use the faster model.

## Research Threads

All research is logged to `.claude/perplexity-research/[topic].md`. This creates a searchable knowledge base that persists across sessions and helps avoid re-researching the same topics.

## File Structure

```
perplexity-intelligent/
├── SKILL.md                    # Core skill instructions
├── README.md                   # This file
├── config/
│   └── api-key.env.example     # API key template
└── references/
    ├── models.md               # Model capabilities
    ├── api-reference.md        # API parameters
    ├── multi-model.md          # Multi-model patterns
    └── rca-workflow.md         # Root cause analysis workflow
```

## License

MIT
