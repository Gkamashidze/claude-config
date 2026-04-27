# aiogram 3.x + Telegram Bot Standards

> **SCOPE: Telegram bot projects only.**
> Active when project has `aiogram` in dependencies.

## Architecture

- Use `Router()` per handler module — register all routers in `main.py`
- Filter by topic/thread: `F.message_thread_id == config.TOPIC_ID`
- Use `StatesGroup` + FSM for multi-step flows — never track state in dicts
- All handlers are `async def` — never block the event loop

## Message Handling

- Always use `parse_mode="HTML"` — Markdown breaks on Georgian/special chars
- Escape user input before inserting into messages: `html.escape(user_text)`
- Telegram message limit: 4096 chars — truncate gracefully, never crash
- Prefer inline keyboards over reply keyboards
- Callback data ≤ 64 bytes — keep structured and short (e.g. `"action:id"`)

## Error Handling (MANDATORY)

```python
from aiogram.exceptions import TelegramNetworkError, TelegramRetryAfter, TelegramBadRequest

# TelegramRetryAfter: await asyncio.sleep(e.retry_after) then retry
# TelegramNetworkError: log + handle gracefully
# TelegramBadRequest: log + handle gracefully (message may be deleted, etc.)
```

An unhandled exception in a handler silently drops the update — always wrap handlers with `try/except`.

## Rate Limits

- Global: max 30 messages/second across all chats
- Per-chat: max 1 message/second
- Bulk sends: `await asyncio.sleep(0.05)` between messages
- On `TelegramRetryAfter`: `await asyncio.sleep(e.retry_after)`

## FSM Patterns

```python
class MyStates(StatesGroup):
    waiting_for_input = State()
    confirming = State()

# Always clear state after completion or cancellation:
await state.clear()
```

## Testing aiogram Handlers

- Mock `bot.send_message`, `callback_query.answer`, etc. using `AsyncMock`
- Never send real Telegram API calls in tests
- Test business logic separately from handler wiring
- Use `pytest-asyncio` with `@pytest.mark.asyncio`

## Webhook vs Polling

- Development: polling (`dp.start_polling(bot)`)
- Production (Railway): webhook preferred for efficiency
- Always set `allowed_updates` to only what the bot needs
