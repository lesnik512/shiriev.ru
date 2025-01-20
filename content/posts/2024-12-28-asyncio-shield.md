+++
title = 'Shielding from cancellation in asyncio'
date = 2024-12-28T19:50:55+03:00
draft = true
+++

## Introduction

`asyncio.shield()` is a powerful tool in Python's asyncio library. It protects an awaitable object from being cancelled.

From [docs](https://docs.python.org/3/library/asyncio-task.html#shielding-from-cancellation):

```python
task = asyncio.create_task(something())
res = await shield(task)
```

is equivalent to:
```python
res = await something()
```
except that if the coroutine containing it is cancelled, the Task running in something() is not cancelled.
From the point of view of something(), the cancellation did not happen.
Although its caller is still cancelled, so the “await” expression still raises a CancelledError.

## How asyncio.shield() works

When you use asyncio.shield(), it creates a new Future that wraps the original coroutine or task.
This new Future is not affected by cancellation signals sent to its parent coroutine.
Instead, it continues running until completion, even if the parent coroutine is cancelled.

Key points about asyncio.shield():
- It creates a new Future that wraps the original coroutine or task.
- The wrapped task continues running even if the parent coroutine is cancelled.
- Any exceptions raised by the shielded task are propagated to the caller.
- The shielded task runs independently of the parent coroutine's lifecycle.

## Use cases for asyncio.shield()

asyncio.shield() can be useful in scenarios where you want to ensure certain operations complete regardless of external factors:
1. Resource cleanup: ensure resources are properly released even if the main task is cancelled.
2. Data integrity: prevent data loss by completing important operations before exiting.
3. Graceful shutdown: ensure operations correctly reverted or completed in case of shutting down application.



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
