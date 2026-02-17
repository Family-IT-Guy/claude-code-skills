# Multi-Model Research Patterns

## When to Use Multi-Model

With our two-model setup (sonar-reasoning-pro + sonar-deep-research), multi-model research is straightforward: use both when you need comprehensive coverage AND deep analysis.

**Use multi-model when:**
- Exhaustive research needs analytical synthesis
- High-stakes decisions requiring maximum confidence
- Topic warrants both broad coverage and causal reasoning

**Most queries use a single model.** Multi-model is the exception, not the rule.

## The Pattern: Analysis + Exhaustive Research

**Sequence:**
1. **sonar-deep-research** → Exhaustive investigation, dozens of sources, comprehensive report
2. **sonar-reasoning-pro** → Analyze the deep research findings, identify causal relationships, evaluate trade-offs, synthesize conclusions

**Why this order:** Deep research gathers the raw material. Reasoning-pro applies analytical rigor to that material.

**Example queries:**
- "Comprehensive analysis of the electric vehicle market through 2030" (deep-research gathers data, reasoning-pro analyzes trends)
- "Should we migrate from PostgreSQL to CockroachDB?" (deep-research surveys landscape, reasoning-pro evaluates trade-offs)
- "Research the regulatory landscape for AI in healthcare" (deep-research maps regulations, reasoning-pro identifies implications)

## Synthesis Guidelines

### Confidence Scoring

| Agreement | Confidence | Presentation |
|-----------|------------|--------------|
| Both models agree | High | State as established finding |
| Minor variations | Medium-High | State with "generally" qualifier |
| Models conflict | Low | Flag conflict explicitly, analyze why |

### Handling Conflicts

When findings disagree:

1. **State the conflict explicitly**: "Deep research found X, while reasoning analysis concluded Y"
2. **Analyze why**: Different sources? Different reasoning approaches? Different time frames?
3. **Present both views**: Don't hide the disagreement
4. **Offer assessment**: Which seems more reliable and why?
5. **Let user decide**: Don't force a conclusion

### Output Structure

```markdown
## Summary
[Synthesized key finding]

## Deep Research Findings (sonar-deep-research)
- Finding 1 [source]
- Finding 2 [source]
...

## Analysis (sonar-reasoning-pro)
- Reasoning chain...
- Conclusion...

## Synthesis

### Areas of Agreement
### Areas of Conflict (if any)
### Confidence Assessment

## Sources
[Combined from both models, deduplicated]
```

## When NOT to Multi-Model

Single model is sufficient (and preferred) when:
- Query is well-defined with clear scope → reasoning-pro
- Quick factual lookup → reasoning-pro
- User explicitly requests speed → reasoning-pro
- Already have comprehensive data, just need analysis → reasoning-pro
- Need exhaustive data but not deep analysis → deep-research
