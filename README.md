# Claude Code Skills

A collection of skills for [Claude Code](https://claude.ai/claude-code) that extend its capabilities with specialized tools and workflows.

## Available Skills

| Skill | Description |
|-------|-------------|
| [perplexity-intelligent](./skills/perplexity-intelligent/) | Intelligent web research using Perplexity's Sonar API with automatic model selection |

## What are Claude Code Skills?

Skills are reusable instruction sets that teach Claude Code how to perform specialized tasks. They can be installed globally (for all projects) or per-project.

## Installation

### Global Installation (Recommended)

Copy the skill folder to your global skills directory:

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy the skill
cp -r skills/perplexity-intelligent ~/.claude/skills/
```

### Per-Project Installation

Copy the skill folder to your project:

```bash
cp -r skills/perplexity-intelligent .claude/skills/
```

## Configuration

After installing a skill, follow the skill's specific README for configuration (API keys, CLAUDE.md additions, etc.).

## License

MIT
