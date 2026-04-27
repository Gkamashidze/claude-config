# Cost Management — Railway + Anthropic API

## Railway Deployment Costs

- Every push to `main` triggers a redeploy — don't push broken code, it costs a cold start
- Avoid unnecessary logging of large payloads (Railway charges on log volume)
- If a background job runs on a schedule, verify it completes in reasonable time — runaway jobs drain credits
- Dev/test bots: consider Railway's sleep mode for non-production services

## Anthropic API — Cost Control

### Always Set max_tokens

Never leave `max_tokens` unbounded:

```python
response = client.messages.create(
    model=CLAUDE_MODEL,
    max_tokens=1024,  # always explicit
    messages=[...]
)
```

### Log Token Usage

Every Anthropic call should log usage:

```python
logger.info(
    f"Claude call: input={response.usage.input_tokens} "
    f"output={response.usage.output_tokens} "
    f"cache_read={getattr(response.usage, 'cache_read_input_tokens', 0)}"
)
```

### Prompt Caching (MANDATORY for repeated prompts)

Cache system prompts and large documents — 60–90% cost reduction on cache hits:

```python
system=[{
    "type": "text",
    "text": system_prompt,  # must be ≥1024 tokens to cache
    "cache_control": {"type": "ephemeral"}  # 5-minute TTL
}]
```

Check `response.usage.cache_read_input_tokens > 0` to verify cache hits.

### Model Selection by Cost

| Task | Model | Why |
|---|---|---|
| Classification, routing, short extraction | `claude-haiku-4-5-20251001` | 20x cheaper than Sonnet |
| Coding, analysis, structured output | `claude-sonnet-4-6` | Best balance |
| Architecture, complex reasoning | `claude-opus-4-7` | Only when genuinely needed |

### Batch API for Non-Realtime Jobs

Use `client.beta.messages.batches` for bulk processing (financial reports, inventory analysis) — 50% cheaper:

```python
batch = client.beta.messages.batches.create(requests=[...])
```

### Cost Smell — Flag These

- Sending full conversation history on every message without truncation
- Calling Sonnet/Opus for tasks Haiku can handle (classification, yes/no decisions)
- System prompts sent without `cache_control` when they repeat across calls
- No `max_tokens` set on a long-output endpoint
