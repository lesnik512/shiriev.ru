+++
title = 'Strict Mypy'
date = 2025-02-07T09:47:43+03:00
+++

`Mypy` - Static type checker that aims to **combine** the benefits of **dynamic** and **static** typing

The article is about journey from
```
[tool.mypy]
python_version = "3.11"
warn_return_any = false
warn_unused_configs = true
ignore_missing_imports = true
strict_optional = true
allow_redefinition = true
namespace_packages = true
disallow_incomplete_defs = true
```

to
```
[tool.mypy]
python_version = "3.11"
strict = true
```

There were several problems on that journey. I want to share solutions for some of them.

## Problem 1. Decorators without annotations

Such decorators were loosing type annotations.

```python
def redis_reconnect(func):
    @backoff.on_exception(
        backoff.expo,
        (WatchError, RedisConnectionError, ConnectionResetError, TimeoutError),
        max_tries=settings.redis_connection_tries,
    )
    @functools.wraps(func)
    async def wrapped_method(*args, **kwargs):
        return await func(*args, **kwargs)

    return wrapped_method

```

So `ParamSpec` helps with them:
```python
P = typing.ParamSpec("P")
T = typing.TypeVar("T")
    
def redis_reconnect(func: typing.Callable[P, typing.Awaitable[T]]) -> typing.Callable[P, typing.Awaitable[T]]:
	@backoff.on_exception(  
		backoff.expo,
	    (WatchError, RedisConnectionError, ConnectionResetError, TimeoutError),
	    max_tries=settings.redis_connection_tries,
	)
	@functools.wraps(func)
	async def wrapped_method(*args: P.args, **kwargs: P.kwargs) -> T:
		return await func(*args, **kwargs)

	return wrapped_method
```

## Problem 2. Libraries without annotations

### `Typeshed`

First of all, I caught such errors:
```
error: Skipping analyzing 'redis': found module but no type hints or library stubs
error: Skipping analyzing 'orjson': found module but no type hints or library stubs
```

This can be fixed, using stubs from https://github.com/python/typeshed

So for errors above we need to install packages `types-redis` and `types-orjson`

### `py.typed` marker

Second of all, we had some inner packages which were typed, but not marked as typed.
In order to mark typed package, `py.typed` needs to be placed in libraries directory as [here](https://github.com/modern-python/that-depends/tree/main/that_depends).

### `*-stubs` packages

For some packages there is no stub in `typeshed`, but exists some specified package. For example `django` has `django-stubs`

Read PEP to find out more https://peps.python.org/pep-0561/

## Problem 3. Unified domain-models for read and write

For example, this model is used for objects to create and for already created objects from database:
```python
import pydantic

class Message(pydantic.BaseModel):
   id: int | None = None
   name: str
```

And we need to check that `id` is not None.

Solution is to split the model:

```python
import pydantic

class MessageCreate(pydantic.BaseModel):
   name: str

class Message(MessageCreate):
   id: int
```

## Some tips on typing annotations

Tip 1. `typing.Self` is often misused to annotate `self` argument in method.
Here is an example, showing when it can be used:

```python
import typing
class BaseClass:
   @classmethod
   def construct(cls) -> typing.Self
      return cls()

class SomeClass(BaseClass):
   Pass

some_object = SomeClass.construct()
# some_object has SomeClass type
```
https://peps.python.org/pep-0673/

Tip 2. typing.Iterable is for iterating

```python
import typing

def iterate_over_sth(items: typing.Iterable[str]):
   for x in items:
      ...
```

Tip 3. typing.Sequence is for index access and getting length of collections

```python
import typing

def fetch_last_item(items: typing.Sequence[str]):
   return items[-1]
```

Tip 4. To annotate a tuple of strings use:

```python
tupple[str, ...] = ('a', 'h', 'j', 'n', 'm', 'n', 'z')
```
