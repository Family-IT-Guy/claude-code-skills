# Perplexity Intelligent Skill

Intelligent web research using Perplexity's Sonar API with automatic model selection, multi-model synthesis, and persistent research threads.

## Features

- **Automatic model selection**: Chooses optimal Perplexity model based on query complexity
  - `sonar` - Quick facts, current events
  - `sonar-pro` - Multi-source synthesis, fact-checking
  - `sonar-reasoning-pro` - Why/how questions, causal analysis
  - `sonar-deep-research` - Exhaustive research, comprehensive reports
- **Multi-model synthesis**: Combines results from multiple models for comprehensive analysis
- **Research threads**: Persistent cross-session research logs at `.claude/perplexity-research/`
- **Full citations**: Always includes source URLs

## Installation

### 1. Copy the skill

**Global (all projects):**
```bash
cp -r perplexity-intelligent ~/.claude/skills/
```

**Per-project:**
```bash
mkdir -p .claude/skills
cp -r perplexity-intelligent .claude/skills/
```

### 2. Configure API key

Get your API key from [Perplexity API Keys](https://www.perplexity.ai/account/api/keys).

Create the config file:
```bash
# Global installation
cp ~/.claude/skills/perplexity-intelligent/config/api-key.env.example \
   ~/.claude/skills/perplexity-intelligent/config/api-key.env

# Edit and add your key
nano ~/.claude/skills/perplexity-intelligent/config/api-key.env
```

The file should contain:
```
PERPLEXITY_API_KEY="pplx-your-actual-key-here"
```

### 3. Add to CLAUDE.md

Add this line to your CLAUDE.md (global or project):

```markdown
## Research
Use ~/.claude/skills/perplexity-intelligent/ for web research, not WebSearch.
```

**Global** (`~/.claude/CLAUDE.md`): Applies to all projects

**Per-project** (`.claude/CLAUDE.md` or `CLAUDE.md`): Applies to that project only

## Usage

Once configured, Claude Code will automatically use this skill for research queries:

- "Research the current state of WebAssembly"
- "What are the differences between Bun and Deno?"
- "Fact-check: Does X cause Y?"

Or invoke explicitly:
```
/perplexity-intelligent [your query]
```

## Research Threads

All research is logged to `.claude/perplexity-research/[topic].md` in your project. This creates a searchable knowledge base that persists across sessions.

## File Structure

```
perplexity-intelligent/
├── SKILL.md              # Core skill instructions
├── README.md             # This file
├── config/
│   └── api-key.env.example
└── references/
    ├── models.md         # Model capabilities
    ├── api-reference.md  # API parameters
    └── multi-model.md    # Multi-model patterns
```

## License

MIT
