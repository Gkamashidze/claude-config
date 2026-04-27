# Python Production Standards

## Type Safety
- Type annotations on ALL function signatures: params and return types
- Use `@dataclass(frozen=True)` for immutable data objects
- Use `typing.Protocol` for duck-typed interfaces (structural subtyping)
- Use `typing.NamedTuple` for lightweight immutables
- Validate input with Pydantic models at API boundaries

## Style
- PEP 8 compliance — enforced by black + isort + ruff
- Docstrings: Google style for public APIs only
- Max line length: 88 (black default)
- Import order: stdlib → third-party → local (isort handles this)

## Error Handling
- Custom exception classes per domain: `class UserNotFoundError(Exception): ...`
- Never bare `except:` — always catch specific exceptions
- Use `contextlib.suppress()` for expected exceptions
- Logging: `logging.getLogger(__name__)` — never `print()` in production

## Patterns
- Context managers (`with`) for resource management (files, DB, locks)
- Generators for memory-efficient lazy iteration
- `@dataclass` for DTOs; Pydantic `BaseModel` for validation
- Repository pattern: abstract base with `find_all`, `find_by_id`, `create`, `update`, `delete`
- Dependency injection via constructor parameters

## Async (Expanded)

### Core Rules
- `async def` for ALL I/O-bound operations — never block the event loop
- Never call sync blocking code (file I/O, `time.sleep`, sync DB drivers) inside `async def`
- Use `asyncio.run_in_executor(None, sync_fn)` if you must call blocking code

### Concurrent Operations
```python
# Run independent coroutines concurrently:
results = await asyncio.gather(fetch_user(id), fetch_orders(id))

# Python 3.11+ — preferred for structured concurrency with error handling:
async with asyncio.TaskGroup() as tg:
    task1 = tg.create_task(fetch_user(id))
    task2 = tg.create_task(fetch_orders(id))
# Both results available after the block

# asyncio.gather vs TaskGroup:
# gather: one failure cancels others silently (use when all results needed, failures ok)
# TaskGroup: one failure cancels ALL and re-raises (use when all must succeed)
```

### Resource Management
```python
# Always use async context managers for connections:
async with pool.acquire() as conn:     # DB connections
    await conn.fetch(...)
async with aiofiles.open(path) as f:   # File I/O
    content = await f.read()
```

### Background Tasks
```python
# Create background tasks explicitly — never fire-and-forget without tracking:
task = asyncio.create_task(background_job())
# Keep a reference to avoid garbage collection; handle exceptions:
task.add_done_callback(lambda t: t.exception() and logger.error(t.exception()))
```

### Event Loop
- Never call `asyncio.run()` inside an already-running loop (use `await` instead)
- In tests: use `pytest-asyncio` with `@pytest.mark.asyncio`
- One event loop per application — don't create new loops manually

### What NOT to Do
```python
# WRONG — blocks the event loop:
time.sleep(1)
requests.get(url)
open(path).read()

# RIGHT:
await asyncio.sleep(1)
await aiohttp.ClientSession().get(url)
async with aiofiles.open(path) as f: await f.read()
```

## Testing
- Framework: pytest + pytest-asyncio
- Coverage: `pytest --cov=src --cov-report=term-missing`
- Markers: `@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.asyncio`
- Fixtures for setup/teardown; conftest.py for shared fixtures
- Use `unittest.mock.AsyncMock` for async external dependencies
- Coverage target: 80%+

## Project Structure
```
src/
  __init__.py
  models/        # Data models
  services/      # Business logic
  repositories/  # Data access
  api/           # Routes/endpoints
tests/
  unit/
  integration/
pyproject.toml   # Single config file
```
