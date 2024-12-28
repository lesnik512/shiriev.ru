+++
title = 'Asyncio Shield'
date = 2024-12-28T19:50:55+03:00
draft = true
+++

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
