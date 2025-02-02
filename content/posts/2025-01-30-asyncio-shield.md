+++
title = 'Shielding from cancellation in asyncio'
date = 2025-01-30T00:00:00+03:00
+++

## Introduction

Asyncio tasks can be canceled. For example, when application is shutting down.

This can cause a running task to stop mid-execution, which can cause problems if we expect a task to complete as an atomic operation.

Asyncio provides a way to shield tasks from cancellation via `asyncio.shield()`. It protects an awaitable object from being cancelled.

## How to use `asyncio.shield()`

```python
# save a reference, to avoid the task from being garbage collected
task = asyncio.create_task(something())

# shield the task from cancellation
shielded_task = asyncio.shield(task)
await shielded_task
```

Shielded task can be cancelled, but it won't cancel the `task`:

```python
shielded_task.cancel()
```

## How asyncio.shield() works

- It creates a new Future that wraps the original coroutine or task.
- The wrapped task continues running even if the parent coroutine is cancelled.
- Any exceptions raised by the shielded task are propagated to the caller.
- The shielded task runs independently of the parent coroutine's lifecycle.

## Use cases for asyncio.shield()

asyncio.shield() can be useful in scenarios where you want to ensure certain operations complete regardless of external factors:
1. Resource cleanup: ensure resources are properly released even if the main task is cancelled.
2. Data integrity: prevent data loss by completing important operations before exiting.
3. Graceful shutdown: ensure operations correctly reverted or completed in case of shutting down application.

## Using decorator to shield some functions

The following decorator can be used to prevent some functions or methods from cancelling:
```python
import asyncio
import functools
import typing


T = typing.TypeVar("T")
P = typing.ParamSpec("P")


def shield_task(
    func: typing.Callable[P, typing.Coroutine[typing.Any, typing.Any, T]],
) -> typing.Callable[P, typing.Coroutine[typing.Any, typing.Any, T]]:
    @functools.wraps(func)
    async def wrapped_method(*args: P.args, **kwargs: P.kwargs) -> T:
        task: typing.Final = asyncio.create_task(func(*args, **kwargs))
        return await asyncio.shield(task)

    return wrapped_method
```
