# Redis & Caching Standards

> **SCOPE:** Projects using Redis (`redis` or `redis.asyncio` in dependencies).

## Client Setup

Always use async client for async projects:

```python
import redis.asyncio as redis

# Create once at startup, reuse across requests
redis_client = redis.from_url(os.environ["REDIS_URL"], decode_responses=True)
```

Never use the sync `redis.Redis` inside `async def` — it blocks the event loop.

## Key Naming Convention

```
{project}:{entity}:{identifier}:{field}
```

Examples:
- `wishmotors:session:user_123:state`
- `wishmotors:cache:product:oem_456`
- `wishmotors:rate_limit:user_123:daily`

Always use snake_case. Always prefix with project name to avoid collisions.

## TTL Policy

Set TTL on EVERY key — never store data without expiry:

| Data Type | TTL |
|---|---|
| User session state | 1 hour (3600s) |
| API response cache | 5 minutes (300s) |
| Rate limit counters | 1 day (86400s) |
| Short-lived tokens | 15 minutes (900s) |

```python
await redis_client.setex(key, ttl_seconds, value)
# or
await redis_client.set(key, value, ex=ttl_seconds)
```

## Patterns

```python
# Get-or-set pattern:
cached = await redis_client.get(key)
if cached is None:
    value = await compute_expensive_value()
    await redis_client.setex(key, 300, json.dumps(value))
    return value
return json.loads(cached)

# Atomic increment (rate limiting):
count = await redis_client.incr(rate_key)
if count == 1:
    await redis_client.expire(rate_key, 86400)

# Delete on invalidation:
await redis_client.delete(key)
```

## Error Handling

Redis failures must never crash the main flow — degrade gracefully:

```python
try:
    cached = await redis_client.get(key)
except redis.RedisError as e:
    logger.warning(f"Redis get failed: {e}")
    cached = None  # fall through to compute
```

## What NOT to Cache

- User PII without explicit need
- Database connection objects
- File handles or async generators
- Anything that must be consistent across all instances without invalidation
