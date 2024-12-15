+++
title = 'Using pre-commit package in python development'
date = 2024-12-14T00:00:00+03:00
+++
# Introduction.

Package `pre-commit`, as stated on the github-page, is a framework for managing and maintaining multi-language pre-commit hooks.

Mostly, it's used for managing git hook right before commit to run some static checks or formatters.

In my opinion, this package is often misused and I want to show you the alternative way.

# Problems of `pre-commit`.

1. It has separate configuration file with tools versions differ from `pyproject.toml` and this is developer's problem to sync them.
2. It uses separate virtual environments which is not very intuitive and can lead to unexpected behaviour.
3. For `mypy` if we want to check types of external packages, `pre-commit` by-default is not able to use dependencies from `pyproject.toml` and we need to list them in config file in `additional_dependencies` section.

# Alternative solution.

## 1. Using command runner

First of all, I recommend using some command runner, like `make`, `task` or `just`. I prefer [just](https://just.systems) because it has the most convenient parametrised commands, like:

```
test *args:
    uv run pytest {{ args }}
```

And this command can be used with any arguments: `just test -x -s -vvv`

You can describe project commands in configuration file for running tests or static checks or any other routine, like creating database migration.

For example, here is a command from my projects with static checks:

```
lint:
    uv run ruff format
    uv run ruff check --fix
    uv run mypy .
```

## 2. Running `just` by `pre-commit`

If you need to run static checks after committing changes to repo, `pre-commit` can run "just"-commands.
Here is the example of `.pre-commit-config.yaml`

```yaml
repos:
- repo: local
  hooks:
  - id: lint
    name: lint
    entry: just
    args: [lint]
    language: system
    types: [python] 
    pass_filenames: false
```

Now run `pre-commit install` to install pre-commit hook and before commiting `just lint` command will be called

# Summary

By this way we are solving all the problems: we run exactly the same commands in exactly the same environment with all dependencies.
