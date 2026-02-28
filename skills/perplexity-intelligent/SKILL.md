---
name: perplexity-intelligent
description: >-
  Deep web research via Perplexity Sonar API with cited sources, reasoning traces,
  and persistent research threads. ALWAYS use instead of WebSearch, WebFetch, and
  firecrawl for: (1) Any research request ("research this", "look into", "find out
  about", "what's the latest on"), (2) Current events, news, recent developments,
  (3) Fact-checking or verifying claims, (4) Cross-library comparisons, architecture
  decisions, technology landscape research (use context7 for single-library docs),
  (5) Market research, competitive analysis, business intelligence,
  (6) Troubleshooting, debugging, root cause analysis, (7) Any question needing
  web-grounded cited data. Triggers: "research", "look up", "search for",
  "investigate", "deep dive", "what do we know about".
---

# Perplexity Intelligent Search

Two-model architecture: sonar-reasoning-pro (default) and sonar-deep-research (exhaustive). File-first responses, persistent research threads, auto-selected search parameters.

## Pricing

| Model | Input/1M | Output/1M | Request Fee (per 1K) |
|-------|----------|-----------|---------------------|
| sonar-reasoning-pro | $2 | $8 | $6 / $10 / $14 (low/med/high search context) |
| sonar-deep-research | $2 | $8 + extras | reasoning $3/1M, citations $2/1M, searches $5/1K |

**IMPORTANT: search_context_size nesting:** This parameter must be inside `web_search_options`, NOT at the top level. Top-level placement is silently ignored (falls back to "low" at $6/1K). Correct format: `"web_search_options": {"search_context_size": "high"}`. Other search params (`search_mode`, `return_images`, date/domain filters) work at the top level. Verified Feb 2026.

## Directory Structure

```
$PROJECT_ROOT/.claude/perplexity-research/
├── raw/                          # Complete API responses (machine-readable, never truncated)
│   ├── 20251226_143022_topicname.json
│   └── 20251226_151033_othertopic.json
├── sources/                      # Full article content from primary source reading
│   ├── 20251226_sparktoro-com_prompt-diversity.md
│   └── 20251226_arxiv-org_2401-12345.md
├── topic-name.md                 # Human-readable thread (references raw + source files)
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

### 3. Select Model & Parameters

**Model selection — binary decision:**

| Question | Model |
|----------|-------|
| Is this exhaustive, report-style research? | `sonar-deep-research` |
| Everything else | `sonar-reasoning-pro` |

**For RCA/debugging**: sonar-reasoning-pro is mandatory (causal reasoning required). See `references/rca-workflow.md`.

**Auto-select search_mode** based on query content:
- Studies, papers, peer-reviewed, scientific claims → `"academic"`
- Companies, filings, earnings, SEC, financial → `"sec"`
- Everything else → `"web"` (default)

**Auto-select return_images** based on query content:
- Product research, visual topics, places, design → `true`
- Technical, analytical, factual → `false`

**For comprehensive accuracy**: Consider multi-model approach (see `references/multi-model.md`).

### 3.5 Present Research Plan & Await Validation

**MANDATORY**: Before executing any API call, present the research plan and await explicit user approval.

**Full Checkpoint Template** (use for initial queries):

```
## Research Plan

**Objective**: [Restate the specific question being answered in precise terms]
**Scope**: [What's in] | [What's explicitly out]
**Methodology**: [Model selected] because [rationale]
**Search mode**: [web/academic/sec] because [rationale]
**Expected Output**: [Brief answer / Detailed report / Comparative analysis / etc.]

### Parameters
- search_mode: [web/academic/sec]
- return_images: [true/false]
- Date filters: [suggest if applicable, e.g., "search_after_date_filter: 01/01/2025"]
- Domain filters: [suggest if applicable, e.g., "search_domain_filter: ['docs.python.org']"]

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

**For async deep research**: Use the async endpoint when deep research may timeout. See `references/api-reference.md` for the async request format (requires `{"request": {...}}` wrapper) and polling instructions.

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

Extract the API key and run curl in a single Bash command. The key must stay within the shell process — do NOT read it with the Read tool and embed it in a separate Bash call (string mangling at the tool boundary causes 401s).

```bash
PPLX_KEY=$(grep '^PERPLEXITY_API_KEY' ~/.claude/skills/perplexity-intelligent/config/api-key.env | cut -d'"' -f2) && curl -s "https://api.perplexity.ai/chat/completions" -H "Authorization: Bearer ${PPLX_KEY}" -H "Content-Type: application/json" -d '{"model":"MODEL_NAME","messages":[{"role":"system","content":"SYSTEM_PROMPT"},{"role":"user","content":"USER_QUERY"}],"web_search_options":{"search_context_size":"high"},"search_mode":"SEARCH_MODE","return_related_questions":true,"return_images":RETURN_IMAGES,"temperature":0.2,"stream":false}' | jq '.' > "$OUTPUT_FILE" && echo "Saved: $OUTPUT_FILE" && echo "Size: $(wc -c < "$OUTPUT_FILE") bytes | Lines: $(wc -l < "$OUTPUT_FILE")"
```

**For deep research, add:**
```json
"reasoning_effort": "high"
```

**With date filters (when approved in research plan):**
```json
"search_after_date_filter": "01/01/2025",
"search_before_date_filter": "02/16/2026"
```

**With domain filters (when approved in research plan):**
```json
"search_domain_filter": ["docs.python.org", "github.com"]
```

**Why `jq '.'`**: Pretty-prints JSON to multiple short lines. The Read tool truncates lines >2000 chars — minified JSON would be truncated. Pretty-printed JSON is safe.

**Claude Code Bash Compatibility**:
- `source file.env` does NOT work — runs in non-interactive subshell
- Do NOT use the Read tool to get the key and then embed it in a separate Bash call — string mangling at the Claude→Bash boundary causes 401s
- **Solution**: Extract the key with `grep | cut` in the same Bash command as curl, keeping it in a shell variable (`${PPLX_KEY}`) that never leaves the process
- The `$(grep ... | cut ...)` substitution works because it executes within the shell, not across the tool boundary

See `references/api-reference.md` for all parameters.

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
- All sources with metadata (`.search_results[]` — title, URL, date, snippet)
- Model used and rationale
- Token usage and cost (`.usage` and `.usage.cost`)
- Related questions (if `return_related_questions: true` was used)

**Cost extraction:**
```bash
jq '.usage.cost' "$OUTPUT_FILE"
```

Always include sources. Never omit citations.

**Raw file reference**: Note the saved file path — you'll reference it in the thread file.

### 5.5. Read Primary Sources

Dispatch one subagent (Task tool) per source URL from Perplexity's search results. All run in parallel. No cap on subagent count.

**Each subagent receives:**
- Project context (what the project is, who it's for, what matters)
- The approved research plan from Step 3.5 (scope, methodology, in/out)
- Perplexity's findings so far (so the subagent knows what's established)
- The Perplexity snippet for this specific source (so the subagent knows what was already extracted)
- Specific claims being verified (when applicable)
- The research question

**Each subagent:**
1. Fetch the article: Chawan skill first, Playwright skill if Chawan fails, Firecrawl skill if Playwright fails
2. Save full article content to `perplexity-research/sources/[timestamp]_[slug].md` (slug from URL domain + path)
3. Return freeform findings: relevant evidence, contradictions to Perplexity's findings, notable findings outside research scope, source quality assessment, key quotes, and **leads** (URLs or references cited in the article worth following)

**Wave pattern:** After all subagents return, collect and deduplicate leads across all returns. If meaningful leads exist, dispatch another wave of subagents with the same pattern and context. Repeat until the research question is sufficiently answered or no new leads remain.

### 5.6. Synthesize Source Findings

Integrate all subagent returns across all waves with Perplexity's original findings. Identify evidence Perplexity missed or understated. Resolve contradictions. Assess confidence based on source quality. Update findings before writing to thread.

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
**Model**: sonar-reasoning-pro | **Tokens**: 2,847 | **Cost**: $0.019

### Approved Plan
[Copy of the research plan that was approved before execution]

### Findings
[Synthesized findings - key points, not full dump]

### Sources
[1] Title - domain.com
    URL | Published: YYYY-MM-DD | Updated: YYYY-MM-DD
    Source file: [sources/20251226_domain-com_slug.md](sources/20251226_domain-com_slug.md)
[2] Title - domain.com
    URL | Published: YYYY-MM-DD | Updated: YYYY-MM-DD
    Source file: [sources/20251226_domain-com_slug.md](sources/20251226_domain-com_slug.md)

### Primary Source Findings
[Evidence from subagent source reading that Perplexity missed or understated]
[Contradictions or complications discovered]
[Notable findings outside original research scope]

### Leads Followed
- Wave 2: [N] sources followed from Wave 1 leads
- Wave 3: [N] sources followed from Wave 2 leads (if applicable)

---

## Query 2: [Follow-up query description]
**Timestamp**: 2025-12-26T15:45:10Z (America/Phoenix 08:45:10)
**Raw**: [raw/20251226_154510_topicname.json](raw/20251226_154510_topicname.json)
**Model**: sonar-deep-research | **Tokens**: 11,428 | **Cost**: $0.816

### Approved Plan
[...]

### Findings
[...]

### Sources
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

For queries requiring maximum comprehensiveness, use both models sequentially:

1. **sonar-deep-research**: Exhaustive investigation — gather comprehensive data
2. **sonar-reasoning-pro**: Analytical synthesis — evaluate findings, identify patterns, draw conclusions

Then synthesize:
- Note where both models **agree** (high confidence)
- Surface where they **conflict** (flag for user)
- Combine sources from both
- Present unified findings with transparency about confidence levels

See `references/multi-model.md` for detailed synthesis patterns.

## Research Threads

**Always log** queries to `$PROJECT_ROOT/.claude/perplexity-research/[topic].md`

**Timestamp**: Run `date` to get accurate time. Include both UTC and local with timezone:
```
2025-12-26T15:30:45Z (America/Phoenix 08:30:45)
```

**Multi-query threads**: A research effort often requires multiple queries. Each query gets its own section in the thread file, with a running synthesis section at the bottom that gets updated after each query.

**For technical debugging**: Preserve the investigation path — hypotheses generated, each validated/invalidated, what was ruled out and why. Future sessions benefit from seeing the reasoning chain, not just conclusions.

## Output Requirements

Every response must include:

1. **Model Rationale**: Explain which model(s) used and why
2. **Response Content**: The actual findings
3. **Sources**: All sources with URLs, titles, dates
4. **Confidence Indicators**: Note certainty levels, conflicts between sources
5. **Cost Summary**: Tokens used, actual cost from `usage.cost.total_cost`

**Source Format**:
```
[1] Title - domain.com
    URL | Published: YYYY-MM-DD | Updated: YYYY-MM-DD
[2] Title - domain.com
    URL | Published: YYYY-MM-DD | Updated: YYYY-MM-DD
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

- `references/models.md` — Model capabilities, pricing, and selection guidance
- `references/api-reference.md` — Complete API parameter reference
- `references/multi-model.md` — Multi-model research patterns
- `references/rca-workflow.md` — Root cause analysis workflow
