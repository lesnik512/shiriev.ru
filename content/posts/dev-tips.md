+++
title = 'Python Development Tips'
date = 2024-09-13T22:26:59+03:00
draft = true
+++

> This article will be updated

All tools and tips from this page are used in the [modern python](https://github.com/modern-python) projects.

## Developer Experience
Here my recommendations on the best tools for python development:

### Package and project manager
Use [`uv`](https://github.com/astral-sh/uv) for managing:
   - python version, instead of `pyenv`
   - virtual environments, instead of `venv`
   - project dependencies, instead of `poetry`, `pdm`, `pip-tools`
It is extremely fast and follows [PEP 751](https://peps.python.org/pep-0751/) for dependencies format.

`uv` can't build packages yet, so use `hatch` for building packages:
  - it has a plugin that can use version control system (like `Git`) to determine project's version

### Command manager
Use [just](https://just.systems/) instead of `Makefile`
- `just` is a command runner, not a build system

### Linting
- Use `ruff` for linting and formatting, instead of `black`, `isort`, `pylint`, `flake8`, etc.

### Type checking
- Use `mypy` only in [strict mode](https://mypy.readthedocs.io/en/stable/getting_started.html#strict-mode-and-configuration)

## Development tips
1. Prefer not to pin versions of packages in `pyproject.toml`. Update dependencies regularly.
2. Run tests in container for applications with databases and storages:
   - Do not mock databases and storages
   - Write more integration tests involving `databases` and `storages`
3. Do not mock external clients, mock queries
   - For `aiohttp` use `aioresponses`
   - For `httpx` use `respx`
4. Use `pyproject.toml` for all configurations

## Python tips
1. Do not start int enums with zero. It can lead to falsy enum values.

## Code style and agreements
### Common tips
- Reduce the nesting of the code by inversion of conditions:
```python
def do_foo():
    if something:
        if another:
           ...

def do_foo():
    if not something:
        return
    if not another:
        return
    pass
```

### How to name functions
Avoid using common verbs: `get`, `run`, `process`, `make`, `handle`, `do`, `main`, `compare`.

Use this algorithm to name functions:
1. Is the function a test? -> `test_<entity>_<behavior>`.
2. Does the function has a `@property` decorator? -> don’t use a verb in the function name.
3. Does the function use a disk or a network:
   1. to store data? -> `save_to`, `send`, `write_to`
   2. to receive data? -> `fetch`, `load`, `read`
4. Does the function output any data? -> `print`, `output`
5. Returns boolean value? -> `is_`, `has_/have_`, `can_`, `check_if_<entity>_<characteristic>`
6. Aggregates data? -> `calculate`, `extract`, `analyze`
7. Put data from one form to another:
   1. Creates a single meaningful object? -> `create`
   2. Fills an existing object with data? -> `initialize`, `configure`
   3. Clean raw data? -> `clean`
   4. Receive a string as input? -> `parse`
   5. Return a string as output? -> `render`
   6. Return an iterator as output? -> `iter`
   7. Mutates its arguments or some global state? -> `update`, `mutate`, `add`, `remove`, `insert`, `set`
   8. Return a list of errors? -> `validate`
   9. Checks data items recursively? -> `walk`
   10. Finds appropriate item in data? -> `find`, `search`, `match`
   11. Transform data type? -> `<sth>_to_<sth_else>`

### How to name database models and tables:
- Use singular form and `CamelCase` for models: `User`, `UserAnswer`.
- Use plural form and underscore for tables: `users`, `user_answers`.
- Use lower case and underscore for fields: `login`, `first_name`.
- Boolean fields start with `is_`: `is_admin`, `is_hidden`.
- DATETIME fields end with `_at`: `created_at`, `updated_at`.
- DATE fields end with `_date`: `connect_date`, `end_date`.
