---
name: perplexity-intelligent
description: >-
  Web research with real-time search. Use for current information, fact-checking,
  news, technical documentation, competitive analysis, troubleshooting, debugging,
  root cause analysis, or tasks needing web-grounded data. Prefer over WebSearch.
  MANDATORY: Every query MUST be logged to $PROJECT_ROOT/.claude/perplexity-research/[topic].md
  with timestamp, model used, findings, and full citations. This is required, not optional.
---

# Perplexity Intelligent Search

Leverage Perplexity's full Sonar API capability with intelligent model selection, multi-model synthesis, and persistent research threads.

## Core Workflow

### 1. Check Existing Research

Before querying, check `$PROJECT_ROOT/.claude/perplexity-research/` for related threads. Read relevant threads to inform the query.

### 2. Analyze Query

Before making any API call, assess:

- **Complexity**: Simple fact vs multi-faceted analysis
- **Intent**: Factual lookup, reasoning, comparison, comprehensive research
- **Depth needed**: Quick answer vs exhaustive investigation
- **Output expectations**: Brief response vs detailed report

### 3. Select Model

Based on analysis, select the appropriate model(s). See `references/models.md` for detailed capabilities.

| Query Type | Recommended Model |
|------------|------------------|
| Simple factual lookup (single value, no analysis) | sonar |
| **All other queries (DEFAULT)** | **sonar-reasoning-pro** |
| Exhaustive research, reports, due diligence | sonar-deep-research |
| Comprehensive research validation | sonar-deep-research + sonar-reasoning-pro |

**Default to sonar-reasoning-pro** unless query is trivially simple. Reasoning traces provide auditability and catch logical errors.

**For RCA/debugging**: sonar-reasoning-pro is mandatory (causal reasoning required). See `references/rca-workflow.md`.

**For comprehensive accuracy**: Consider multi-model approach (see Multi-Model Synthesis below).

### 4. Execute API Call

**API Key Location**: `~/.claude/skills/perplexity-intelligent/config/api-key.env`

Read the key file to get the API key value, then execute the curl call:

```bash
curl -s https://api.perplexity.ai/chat/completions \
  -H "Authorization: Bearer API_KEY_VALUE" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "MODEL_NAME",
    "messages": [
      {"role": "system", "content": "SYSTEM_PROMPT"},
      {"role": "user", "content": "USER_QUERY"}
    ]
  }'
```

**Note**: Read the key from `~/.claude/skills/perplexity-intelligent/config/api-key.env` and substitute it directly into the Authorization header.

**Claude Code Bash Compatibility**:
- `source file.env` does NOT work - runs in non-interactive subshell, variables not exported
- Command substitution `$(...)` gets escaped incorrectly by the bash tool
- **Solution**: Use the Read tool to get the key value, then embed directly in curl command

See `references/api-reference.md` for all parameters (search modes, domain filters, recency, etc.).

### 5. Process Response

Extract and present:
- Main response content
- All citations with URLs
- Model used and rationale
- Token usage and cost (from `usage` field)

Always include citations. Never omit sources.

### 6. Write to Thread (Required)

After presenting results, write to `$PROJECT_ROOT/.claude/perplexity-research/[topic].md`. Create directory if needed. Every query gets logged.

## Multi-Model Synthesis

For queries requiring maximum comprehensiveness and objectivity, use multiple models sequentially:

**Pattern: TruthTracer-style Analysis**

1. **sonar-pro**: Factual grounding - gather verified information
2. **sonar-reasoning-pro**: Logical analysis - identify causal relationships
3. **sonar-deep-research**: Exhaustive investigation (for important topics)

Then synthesize:
- Note where models **agree** (high confidence)
- Surface where models **conflict** (flag for user)
- Combine citations from all sources
- Present unified findings with transparency about confidence levels

See `references/multi-model.md` for detailed synthesis patterns.

## Research Threads

**Always log** queries to `$PROJECT_ROOT/.claude/perplexity-research/[topic].md`

**Timestamp**: Run `date` to get accurate time. Include both UTC and local with timezone:
```
2025-12-19T15:30:45Z (America/Phoenix 08:30:45)
```

**Format**: Capture complete response including all findings, tables, and full citation URLs.

**For technical debugging**: Preserve the investigation path - hypotheses generated, each validated/invalidated, what was ruled out and why. Future sessions benefit from seeing the reasoning chain, not just conclusions.

## Output Requirements

Every response must include:

1. **Model Rationale**: Explain which model(s) used and why
2. **Response Content**: The actual findings
3. **Citations**: All sources with URLs, grouped by relevance
4. **Confidence Indicators**: Note certainty levels, conflicts between sources
5. **Cost Summary**: Tokens used, estimated cost (optional but useful)

**Citation Format**:
```
[1] Title - domain.com
    https://full-url...
[2] Title - domain.com
    https://full-url...
```

## System Prompts

Craft system prompts dynamically based on query context:

**For software development**:
> Prioritize official documentation, GitHub repositories, and high-quality technical sources. Include code examples when relevant. Note version compatibility.

**For academic research**:
> Prioritize peer-reviewed sources and reputable publications. Note methodology limitations. Highlight conflicting findings.

**For fact-checking**:
> Cross-reference claims against multiple independent sources. Distinguish verified facts from disputed claims. Rate confidence levels.

**For business intelligence**:
> Prioritize authoritative sources (SEC filings, official reports). Include quantitative data. Note potential biases.

## Error Handling

On API failure, do not silently fallback. Present options:

```
API call failed: [error details]

Options:
1. Retry with same model
2. Try alternative model: [suggestion]
3. Reformulate query
4. Abort

What would you like to do?
```

## References

- `references/models.md` - Detailed model capabilities and selection guidance
- `references/api-reference.md` - Complete API parameter reference
- `references/multi-model.md` - Multi-model synthesis patterns
