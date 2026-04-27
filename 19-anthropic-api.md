# Anthropic API / Claude SDK Standards

> Active when a project imports `anthropic` or `@anthropic-ai/sdk`.

## Model Selection

Always use the latest available model appropriate for the task:
- **claude-sonnet-4-6** — default for most tasks: coding, analysis, structured output
- **claude-opus-4-7** — complex reasoning, architecture planning, ambiguous requirements
- **claude-haiku-4-5** — high-frequency/cheap calls: classification, short extraction, routing

Never hardcode a model string directly in multiple places — define it as a constant:
```python
CLAUDE_MODEL = "claude-sonnet-4-6"
```

## Prompt Caching (MANDATORY for repeated prompts)

Enable caching on any content that repeats across calls (system prompts, large documents, tool definitions):

```python
# Python — mark cacheable content with cache_control:
response = client.messages.create(
    model=CLAUDE_MODEL,
    system=[{
        "type": "text",
        "text": system_prompt,
        "cache_control": {"type": "ephemeral"}   # cached for 5 min
    }],
    messages=[{"role": "user", "content": user_message}],
    max_tokens=1024,
)
```

Caching rules:
- Cache content must be ≥ 1024 tokens to be eligible
- TTL is 5 minutes — re-send the same content within that window to hit cache
- Check `response.usage.cache_read_input_tokens` to confirm cache hits
- System prompts, tool schemas, and large documents are the best candidates

## Error Handling

Always handle these exceptions:
```python
from anthropic import APIError, RateLimitError, APITimeoutError, APIConnectionError

try:
    response = client.messages.create(...)
except RateLimitError:
    # Retry with exponential backoff — respect Retry-After header
    await asyncio.sleep(backoff_seconds)
except APITimeoutError:
    # Log and retry once; if still failing, surface error to user
    logger.warning("Claude API timeout, retrying...")
except APIConnectionError:
    # Network issue — log and fail gracefully
    logger.error("Cannot reach Claude API")
except APIError as e:
    # Catch-all for other API errors — log status code and message
    logger.error(f"Claude API error {e.status_code}: {e.message}")
```

## Streaming

Use streaming for long responses to avoid timeouts and improve perceived latency:
```python
async with client.messages.stream(
    model=CLAUDE_MODEL,
    messages=[...],
    max_tokens=2048,
) as stream:
    async for text in stream.text_stream:
        yield text   # or send to Telegram bot incrementally
```

## Cost Optimization

- Always set `max_tokens` — never leave it unbounded
- Use the cheapest model that fits the task (Haiku for simple tasks)
- Enable prompt caching wherever possible (60–90% cost reduction on cache hits)
- Batch similar requests with `client.beta.messages.batches` for non-real-time jobs
- Log `response.usage` (input/output/cache tokens) to track spending

## Structured Output

For JSON extraction, use tool use (more reliable than asking for JSON in text):
```python
response = client.messages.create(
    model=CLAUDE_MODEL,
    tools=[{
        "name": "extract_data",
        "description": "Extract structured data from text",
        "input_schema": {
            "type": "object",
            "properties": {
                "field": {"type": "string"}
            },
            "required": ["field"]
        }
    }],
    tool_choice={"type": "tool", "name": "extract_data"},
    messages=[...],
)
result = response.content[0].input   # guaranteed structured
```

## Security

- API key only in environment variables — never in code or logs
- Never log request/response content if it contains user PII
- Set `max_tokens` to prevent runaway costs from prompt injection
