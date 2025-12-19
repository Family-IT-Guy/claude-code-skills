# Claude Code Skills

A collection of skills for [Claude Code](https://claude.ai/claude-code) that extend its capabilities with specialized tools and workflows.

## Available Skills

| Skill | Description |
|-------|-------------|
| [perplexity-intelligent](./skills/perplexity-intelligent/) | Intelligent web research using Perplexity's Sonar API with automatic model selection |

## Quick Start: perplexity-intelligent

### 1. Install the skill

```bash
# Clone this repo
git clone https://github.com/Family-IT-Guy/claude-code-skills.git
cd claude-code-skills

# Copy to global skills directory
mkdir -p ~/.claude/skills
cp -r skills/perplexity-intelligent ~/.claude/skills/
```

### 2. Add your API key

Get your key from [Perplexity API Keys](https://www.perplexity.ai/account/api/keys), then:

```bash
cp ~/.claude/skills/perplexity-intelligent/config/api-key.env.example \
   ~/.claude/skills/perplexity-intelligent/config/api-key.env

# Edit the file and add your key
nano ~/.claude/skills/perplexity-intelligent/config/api-key.env
```

### 3. Add to CLAUDE.md

Add this to `~/.claude/CLAUDE.md` (global) or your project's `CLAUDE.md`:

```markdown
## Research
Use ~/.claude/skills/perplexity-intelligent/ for web research, not WebSearch.
```

### 4. Use it

Ask Claude Code to research anything:
- "Research the current state of WebAssembly"
- "What are the differences between Bun and Deno?"

---

## What are Claude Code Skills?

Skills are reusable instruction sets that teach Claude Code how to perform specialized tasks. They can be installed globally (for all projects) or per-project.

## License

MIT
