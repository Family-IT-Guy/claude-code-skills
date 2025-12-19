# Perplexity Sonar Models Reference

## Model Overview

| Model | Context | Best For |
|-------|---------|----------|
| sonar | Standard | Quick facts, current events |
| sonar-pro | Extended | Research, fact-checking, multi-source analysis |
| sonar-reasoning-pro | Extended | Why/how questions, complex evaluations, trade-offs |
| sonar-deep-research | 128K | Exhaustive research, comprehensive reports |

## Detailed Model Capabilities

### sonar

**Base**: Llama 3.3 70B fine-tuned for search

**Optimal for**:
- Simple factual queries: "What is X?"
- Current events and news
- Definitions and quick lookups
- Product information
- Weather, sports, stocks

**Characteristics**:
- Lightweight and efficient
- Good for simple, single-topic queries
- Real-time web grounding

**Example queries**:
- "What is the current Bitcoin price?"
- "Who won the game last night?"
- "What is quantum entanglement?"

---

### sonar-pro

**Base**: Enhanced Llama 3.3 with deeper retrieval

**Optimal for**:
- Multi-source synthesis
- Fact-checking and verification
- Analytical queries requiring multiple perspectives
- Technical documentation research
- Structured output (JSON schema supported)

**Characteristics**:
- F-score 0.858 on SimpleQA benchmark
- 2x more citations than sonar
- Multi-layered follow-ups
- Supports `response_format` for structured output

**Example queries**:
- "Compare React vs Vue for enterprise applications"
- "What are the key findings from recent climate reports?"
- "Fact-check: Does coffee cause dehydration?"

---

### sonar-reasoning-pro

**Base**: DeepSeek-R1 1776 variant with advanced reasoning

**Optimal for**:
- "Why" and "how" questions requiring logical chains
- Complex trade-off analysis
- Technical and mathematical problems
- Research synthesis across multiple domains
- Causal relationship identification
- Evaluations requiring high coherence
- Visible reasoning traces for transparency

**Characteristics**:
- Shows reasoning process in `<think>` tags
- Advanced step-by-step inference
- Combines search with logical inference
- Comparison and evaluation capabilities
- Technical depth

**Example queries**:
- "Why did the 2008 financial crisis happen?"
- "How does mRNA vaccine technology work?"
- "Evaluate the pros and cons of microservices vs monolith for a 10-person team"
- "Analyze the mathematical foundations of transformer attention"
- "Compare Kubernetes deployment strategies for high-availability requirements"

---

### sonar-deep-research

**Base**: Autonomous research agent

**Optimal for**:
- Exhaustive, report-style research
- Due diligence and comprehensive analysis
- Academic literature review
- Competitive intelligence
- Market research and trend analysis
- Topics requiring dozens of sources

**Characteristics**:
- Runs dozens of searches autonomously (~30 queries typical)
- Processes hundreds of sources
- 128K token context window
- Returns detailed usage metadata:
  - `reasoning_tokens`
  - `citation_tokens`
  - `num_search_queries`
- Generates structured, multi-section reports
- Most comprehensive and thorough option

**Example queries**:
- "Comprehensive analysis of the electric vehicle market through 2030"
- "Research the regulatory landscape for AI in healthcare"
- "Deep dive into quantum computing commercial viability"

---

## Model Selection Guidance

**DO NOT use hard rules.** Consider these factors and apply judgment:

### Complexity Signals
- Token count of query
- Number of sub-questions
- Technical terminology density
- Scope of investigation needed

### Intent Signals
- Factual lookup → sonar
- Analysis/comparison → sonar-pro
- Reasoning/explanation → sonar-reasoning-pro
- Complex evaluation → sonar-reasoning-pro
- Comprehensive research → sonar-deep-research

### Quality Orientation
- When in doubt, prefer MORE capable model
- Accuracy and comprehensiveness are primary goals
- For comprehensive topics, consider multi-model approach

### Multi-Model Indicators
Use multiple models when:
- High stakes decision
- Need multiple perspectives
- Objectivity is critical
- Sources might conflict
- Comprehensive coverage required
