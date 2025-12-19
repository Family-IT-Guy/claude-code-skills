# Perplexity API Reference

## Endpoint

```
POST https://api.perplexity.ai/chat/completions
```

## Authentication

```bash
-H "Authorization: Bearer $PERPLEXITY_API_KEY"
```

Environment variable: `PERPLEXITY_API_KEY`

## Request Format

```json
{
  "model": "sonar-pro",
  "messages": [
    {"role": "system", "content": "System prompt here"},
    {"role": "user", "content": "User query here"}
  ],
  "max_tokens": 1000,
  "temperature": 0.2
}
```

## Required Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| model | string | Model name: sonar, sonar-pro, sonar-reasoning-pro, sonar-deep-research |
| messages | array | Array of message objects with role and content |

## Optional Parameters

### Generation Control

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| max_tokens | integer | varies | Maximum tokens in response |
| temperature | float | 0.2 | Randomness (0-2). Lower = more focused |
| top_p | float | 0.9 | Nucleus sampling threshold |
| top_k | integer | 0 | Top-k sampling (0 = disabled) |
| presence_penalty | float | 0 | Penalize repeated tokens (-2 to 2) |
| frequency_penalty | float | 1 | Penalize frequent tokens (>0 recommended) |

### Search Control

| Parameter | Type | Description |
|-----------|------|-------------|
| search_domain_filter | array | Limit to specific domains (max 10). Prefix with `-` to exclude |
| search_recency_filter | string | Time filter: "day", "week", "month", "year" |

**Domain filter examples**:
```json
"search_domain_filter": ["docs.python.org", "github.com", "stackoverflow.com"]
```

```json
"search_domain_filter": ["-reddit.com", "-quora.com"]
```

### Output Control

| Parameter | Type | Description |
|-----------|------|-------------|
| return_citations | boolean | Include citations in response (default: true) |
| return_images | boolean | Include images in results |
| return_related_questions | boolean | Suggest follow-up questions |
| stream | boolean | Stream response tokens |

### Structured Output

For sonar, sonar-pro, sonar-reasoning-pro:

```json
{
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "research_result",
      "strict": true,
      "schema": {
        "type": "object",
        "properties": {
          "summary": {"type": "string"},
          "key_findings": {"type": "array", "items": {"type": "string"}},
          "confidence": {"type": "number"}
        },
        "required": ["summary", "key_findings"]
      }
    }
  }
}
```

## Response Format

```json
{
  "id": "unique-response-id",
  "model": "sonar-pro",
  "created": 1703001234,
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Response text here..."
      },
      "finish_reason": "stop"
    }
  ],
  "citations": [
    "https://source1.com/article",
    "https://source2.com/page"
  ],
  "usage": {
    "prompt_tokens": 150,
    "completion_tokens": 892,
    "total_tokens": 1042
  }
}
```

### Deep Research Response (additional fields)

```json
{
  "usage": {
    "prompt_tokens": 33,
    "completion_tokens": 11395,
    "total_tokens": 11428,
    "citation_tokens": 19028,
    "num_search_queries": 21,
    "reasoning_tokens": 193947
  }
}
```

## Complete curl Example

```bash
curl -s https://api.perplexity.ai/chat/completions \
  -H "Authorization: Bearer $PERPLEXITY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sonar-pro",
    "messages": [
      {
        "role": "system",
        "content": "You are a research assistant. Provide comprehensive, well-cited answers."
      },
      {
        "role": "user",
        "content": "What are the key differences between PostgreSQL and MySQL for web applications?"
      }
    ],
    "max_tokens": 2000,
    "temperature": 0.2,
    "return_citations": true,
    "search_recency_filter": "year"
  }'
```

## Parsing Response with jq

Extract content:
```bash
| jq -r '.choices[0].message.content'
```

Extract citations:
```bash
| jq -r '.citations[]'
```

Extract usage:
```bash
| jq '.usage'
```

Full parsing:
```bash
curl -s ... | jq '{
  content: .choices[0].message.content,
  citations: .citations,
  model: .model,
  tokens: .usage.total_tokens
}'
```

## Error Handling

Common errors:

| Status | Meaning | Action |
|--------|---------|--------|
| 400 | Bad request | Check payload format |
| 401 | Unauthorized | Check API key |
| 429 | Rate limited | Wait and retry |
| 500 | Server error | Retry after delay |

## Rate Limits

Rate limits depend on account tier. Check response headers:
- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`

## Usage Tracking

Monitor usage from response `usage` field:
- `prompt_tokens` - input tokens
- `completion_tokens` - output tokens
- `total_tokens` - combined total
- `cost` - cost breakdown (when returned)
