# Multi-Model Synthesis Patterns

## When to Use Multi-Model

Use multiple models when:

- **High stakes**: Decision has significant consequences
- **Objectivity critical**: Need to surface biases or conflicts
- **Comprehensive coverage**: Topic requires multiple angles
- **Verification needed**: Claims should be cross-referenced
- **Complex reasoning + facts**: Need both data and analysis

## Core Patterns

### Pattern 1: Fact + Reasoning

**Use case**: Questions requiring both verified information AND logical analysis

**Sequence**:
1. **sonar-pro** → Gather factual grounding, citations
2. **sonar-reasoning-pro** → Analyze causality, explain relationships

**Synthesis**:
- Lead with factual findings (from sonar-pro)
- Follow with reasoning/explanation (from sonar-reasoning-pro)
- Combine citations from both
- Note if reasoning extends beyond cited facts

**Example query**: "Why is Rust becoming popular for systems programming?"

---

### Pattern 2: Quick Check + Deep Dive

**Use case**: Initial assessment followed by comprehensive research

**Sequence**:
1. **sonar** → Quick initial answer, identify key themes
2. **sonar-deep-research** → Exhaustive investigation based on themes

**Synthesis**:
- Present quick summary first
- Follow with detailed findings
- Use initial response to frame the deep research
- Highlight what deep research added beyond initial answer

**Example query**: "What's the current state of nuclear fusion energy?"

---

### Pattern 3: TruthTracer (Comprehensive Verification)

**Use case**: Fact-checking, due diligence, claims requiring high confidence

**Sequence**:
1. **sonar-pro** → Factual verification, gather sources
2. **sonar-reasoning-pro** → Logical consistency check
3. **sonar-deep-research** → Trace claims to primary sources (if warranted)

**Synthesis**:
- Rate confidence based on agreement across models
- Surface any conflicts explicitly
- Distinguish verified facts from logical inferences
- Provide citation chain to primary sources

**Example query**: "Is it true that electric vehicles have lower total emissions than gas cars?"

---

### Pattern 4: Multi-Perspective Analysis

**Use case**: Controversial topics, trade-off analysis, evaluations

**Sequence**:
1. **sonar-pro** → Gather facts from multiple sources
2. **sonar-reasoning-pro** → Deep trade-off analysis

**Synthesis**:
- Present multiple viewpoints
- Analyze trade-offs systematically
- Note which perspectives have stronger evidence
- Let user draw final conclusions

**Example query**: "Should our startup use a monolith or microservices architecture?"

---

## Synthesis Guidelines

### Confidence Scoring

| Agreement | Confidence | Presentation |
|-----------|------------|--------------|
| All models agree | High | State as established finding |
| Most agree, minor variations | Medium-High | State with "generally" qualifier |
| Mixed results | Medium | Present multiple perspectives |
| Models conflict | Low | Flag conflict explicitly |

### Handling Conflicts

When models disagree:

1. **State the conflict explicitly**: "sonar-pro found X, while sonar-reasoning-pro concluded Y"
2. **Analyze why**: Different sources? Different reasoning approaches?
3. **Present both views**: Don't hide the disagreement
4. **Offer assessment**: Which seems more reliable and why?
5. **Let user decide**: Don't force a conclusion

### Citation Merging

- Combine citations from all models
- Remove duplicates
- Group by relevance/authority
- Note which model produced which citation
- Prioritize primary sources over summaries

### Output Structure

```markdown
## Summary
[Synthesized key finding]

## Findings by Approach

### Factual Research (sonar-pro)
- Finding 1 [citation]
- Finding 2 [citation]

### Logical Analysis (sonar-reasoning-pro)
- Reasoning chain...
- Conclusion...

### Deep Investigation (sonar-deep-research)
- Comprehensive finding [citations]

## Synthesis

### Areas of Agreement
- Point where all models align

### Areas of Conflict
- Point where models differ, with analysis

### Confidence Assessment
- High confidence: ...
- Lower confidence: ...

## Citations
[Combined, deduplicated, grouped by authority]
```

## Performance Considerations

Sequential execution adds latency:

| Pattern | Approximate Time |
|---------|-----------------|
| Single model | 2-10 seconds |
| Two models | 5-20 seconds |
| Three models | 10-45 seconds |
| With deep-research | 30-120 seconds |

For interactive use, consider:
- Informing user of multi-model approach upfront
- Providing incremental updates as each model completes
- Asking if deep-research is warranted for time-sensitive queries

## When NOT to Multi-Model

Single model is sufficient when:
- Simple factual lookup
- Time-critical response needed
- Low stakes query
- Clear single-model fit (e.g., obvious sonar query)
- User explicitly requests speed over depth
