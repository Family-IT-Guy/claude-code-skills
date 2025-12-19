# Claude Code Skills

A collection of skills for [Claude Code](https://claude.ai/claude-code) that extend its capabilities with specialized tools and workflows.

## Available Skills

| Skill | Description |
|-------|-------------|
| [perplexity-intelligent](./skills/perplexity-intelligent/) | Intelligent web research using Perplexity's Sonar API with automatic model selection |

## What are Claude Code Skills?

Skills are reusable instruction sets that teach Claude Code how to perform specialized tasks. They can be installed globally (for all projects) or per-project.

## Installation

Click on a skill above to see its README with installation instructions.

**General pattern:**

```bash
# Clone this repo
git clone https://github.com/Family-IT-Guy/claude-code-skills.git

# Copy a skill to your global skills directory
mkdir -p ~/.claude/skills
cp -r skills/<skill-name> ~/.claude/skills/
```

Then follow the skill's README for API keys, CLAUDE.md configuration, and usage.

## License

MIT
