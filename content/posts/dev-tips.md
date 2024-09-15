+++
title = 'Python Development Tips'
date = 2024-09-13T22:26:59+03:00
draft = true
+++

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
1. Prefer not to pin versions of packages in `pyproject.toml`. Update dependencies regularly
2. Run tests in container for applications with databases and storages:
   - Do not mock databases and storages
   - Write integration tests
3. Do not mock external clients, mock queries
   - For `aiohttp` use `aioresponses`
   - For `httpx` use `respx`
4. Use `pyproject.toml` for all configurations

## Code style and agreements
### Database models naming
- Use singular form and `CamelCase` for models: `User`, `UserAnswer`.
- Use plural form and underscore for tables: `users`, `user_answers`.
- Use lower case and underscore for fields: `login`, `first_name`.
- Boolean fields start with `is_`: `is_admin`, `is_hidden`.
- DATETIME fields end with `_at`: `created_at`, `updated_at`.
- DATE fields end with `_date`: `connect_date`, `end_date`.
