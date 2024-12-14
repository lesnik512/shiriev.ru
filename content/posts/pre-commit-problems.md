+++
title = 'Alternatives to pre-commit package in python development'
date = 2024-12-14T00:00:00+03:00
+++
# Introduction.

Package `pre-commit`, as stated on the github-page, is a framework for managing and maintaining multi-language pre-commit hooks.

In my opinion, this package brings too many troubles and I want to show you alternative ways.

# Problems.

1. It has separate configuration file with tools' versions differ from `pyproject.toml` and this is developer's problem to sync them.
2. `pre-commit` uses separate virtual environments which is not very intuitive and can lead to unexpected behaviour.
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

## 2.1. Running `just` by `pre-commit`

If you need to run static checks after committing changes to repo, `pre-commit` can run "just"-commands.
Here is the content of my `.pre-commit-config.yaml`

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

Install git pre-commit hook by running `pre-commit install`.
Now, after each commit, `just lint` command will be called.

## 2.2. Creating git hooks manually

Actually, you don't need `pre-commit` to create pre-commit hooks. Add such command in your `Justfile`:

```
hook:
    echo "just lint" > .git/hooks/pre-commit
    chmod +x .git/hooks/pre-commit
```

Run `just hook`. It will create pre-commit hook which calls `just lint` command.
