---
name: perplexity-intelligent
description: >-
  Web research with real-time search. Use for current information, fact-checking,
  news, technical documentation, competitive analysis, troubleshooting, debugging,
  root cause analysis, or tasks needing web-grounded data. Prefer over WebSearch.
  MANDATORY: (1) Check $PROJECT_ROOT/.claude/perplexity-research/ for existing research
  before querying - build on prior findings, avoid duplicate work. (2) Present research
  plan and await explicit user approval before executing any query. (3) Save every API
  response to raw/ subdirectory BEFORE processing (truncation protection). (4) Log
  synthesized findings to $PROJECT_ROOT/.claude/perplexity-research/[topic].md with
  approved plan, timestamp, model used, findings, citations, and links to raw files.
---

# Perplexity Intelligent Search

Leverage Perplexity's full Sonar API capability with intelligent model selection, multi-model synthesis, and persistent research threads.

## Directory Structure

```
$PROJECT_ROOT/.claude/perplexity-research/
├── raw/                          # Complete API responses (machine-readable, never truncated)
│   ├── 20251226_143022_topicname.json
│   ├── 20251226_144515_topicname.json
│   └── 20251226_151033_othertopic.json
├── topic-name.md                 # Human-readable thread (references raw files)
└── other-topic.md
```

**Why two file types:**
- `raw/*.json` — Insurance policy. Complete API response, enables re-processing, survives truncation
- `[topic].md` — Curated synthesis. Human-readable, cross-session continuity, audit trail

A single thread file may reference multiple raw files (multi-query research efforts).

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

### 3.5 Present Research Plan & Await Validation

**MANDATORY**: Before executing any API call, present the research plan and await explicit user approval.

**Full Checkpoint Template** (use for initial queries):

```
## Research Plan

**Objective**: [Restate the specific question being answered in precise terms]
**Scope**: [What's in] | [What's explicitly out]
**Methodology**: [Model selected] because [rationale] | Sources: [prioritization]
**Expected Output**: [Brief answer / Detailed report / Comparative analysis / etc.]

### Alternatives to Consider
- [Alternative framing 1] — might be better if [condition]
- [Alternative framing 2] — worth considering because [reason]

### Questions Before Proceeding
- [Clarifying question if ambiguity exists]
- [Assumption being made that user might want to challenge]
- [Scope decision that could go either way]

### Potential Blind Spots
- [What this approach might miss]
- [Bias in source prioritization]
- [Frame limitation]

Awaiting your input before proceeding.
```

**Do not execute the API call until user approves or provides direction.**

**Brief Checkpoint Template** (use for follow-up queries within established scope):

A follow-up "serves the original objective" if it directly advances the approved research plan. If it reframes the question or explores a tangent, use the full checkpoint.

```
Follow-up within established scope:
"[Sub-question being investigated]" using same methodology.

Proceed?
```

### 4. Execute API Call (File-First Architecture)

**CRITICAL**: Always save response to file BEFORE processing. This prevents data loss from Bash output truncation (30k char limit). Deep research responses routinely exceed this.

**API Key Location**: `~/.claude/skills/perplexity-intelligent/config/api-key.env`

**Step 4a: Prepare directories and filename**

```bash
RESEARCH_DIR="$PROJECT_ROOT/.claude/perplexity-research"
mkdir -p "$RESEARCH_DIR/raw"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
TOPIC_SLUG="topicname"  # lowercase, hyphens, no spaces
OUTPUT_FILE="$RESEARCH_DIR/raw/${TIMESTAMP}_${TOPIC_SLUG}.json"
```

**Step 4b: Execute and save atomically**

Read the key from `~/.claude/skills/perplexity-intelligent/config/api-key.env`, then:

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
  }' | jq '.' > "$OUTPUT_FILE"

echo "Saved: $OUTPUT_FILE"
echo "Size: $(wc -c < "$OUTPUT_FILE") bytes | Lines: $(wc -l < "$OUTPUT_FILE")"
```

**Why `jq '.'`**: Pretty-prints JSON to multiple short lines. The Read tool truncates lines >2000 chars — minified JSON would be truncated. Pretty-printed JSON is safe.

**Claude Code Bash Compatibility**:
- `source file.env` does NOT work - runs in non-interactive subshell
- Command substitution `$(...)` gets escaped incorrectly
- **Solution**: Use Read tool for key, embed directly in curl command

See `references/api-reference.md` for all parameters (search modes, domain filters, recency, etc.).

### 4.5. Retrieve Response from File

Use the Read tool to retrieve the saved response:

```
Read: $OUTPUT_FILE
```

**For large responses** (>100KB or >2000 lines): Use offset/limit parameters to read in chunks:
```
Read: $OUTPUT_FILE, offset=0, limit=500
Read: $OUTPUT_FILE, offset=500, limit=500
...
```

**If jq failed** (malformed response): Read the raw curl output, diagnose the API error.

### 5. Process Response

From the file contents retrieved in Step 4.5, extract and present:

- Main response content (`.choices[0].message.content`)
- All citations with URLs (`.citations[]`)
- Model used and rationale
- Token usage and cost (`.usage` field)
- Related questions (if `return_related_questions: true` was used)

Always include citations. Never omit sources.

**Tip**: Use `return_related_questions: true` when exploring a new topic to discover adjacent questions worth investigating.

**Raw file reference**: Note the saved file path — you'll reference it in the thread file.

### 6. Write to Thread (Required)

After presenting results, update `$PROJECT_ROOT/.claude/perplexity-research/[topic].md`. Create file if new topic; append if continuing research.

**Thread File Format** (supports multiple queries per thread):

```markdown
# [Topic Name]

Research thread for [brief description of research objective].

---

## Query 1: [Brief query description]
**Timestamp**: 2025-12-26T14:30:22Z (America/Phoenix 07:30:22)
**Raw**: [raw/20251226_143022_topicname.json](raw/20251226_143022_topicname.json)
**Model**: sonar-reasoning-pro | **Tokens**: 2,847

### Approved Plan
[Copy of the research plan that was approved before execution]

### Findings
[Synthesized findings - key points, not full dump]

### Citations
[1] Title - domain.com
    https://full-url...
[2] Title - domain.com
    https://full-url...

---

## Query 2: [Follow-up query description]
**Timestamp**: 2025-12-26T15:45:10Z (America/Phoenix 08:45:10)
**Raw**: [raw/20251226_154510_topicname.json](raw/20251226_154510_topicname.json)
**Model**: sonar-deep-research | **Tokens**: 11,428

### Approved Plan
[...]

### Findings
[...]

### Citations
[...]

---

## Synthesis (Updated: 2025-12-26)

[Running synthesis across all queries in this thread. Update after each new query.]

### Key Conclusions
- [Conclusion 1]
- [Conclusion 2]

### Open Questions
- [What remains unresolved]

### Confidence Assessment
- High confidence: [topics]
- Moderate confidence: [topics]
- Needs verification: [topics]
```

**Thread file purpose**: Human-readable audit trail with curated synthesis. Links to raw files for complete data.

**Raw file purpose**: Complete API response. Insurance against truncation. Enables re-processing.

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
2025-12-26T15:30:45Z (America/Phoenix 08:30:45)
```

**Multi-query threads**: A research effort often requires multiple queries. Each query gets its own section in the thread file, with a running synthesis section at the bottom that gets updated after each query.

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
