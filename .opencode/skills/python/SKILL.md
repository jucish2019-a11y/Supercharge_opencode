---
name: python
description: Write idiomatic Python code following project conventions, type hints, and modern patterns
---

## What I do

I write Python code following modern idiomatic practices:

- **Pythonic patterns** — Comprehensions, generators, context managers, dataclasses
- **Type hints** — Full typing with `mypy`-compatible annotations
- **Project conventions** — Match existing style, imports, testing, and packaging patterns
- **Async** — `asyncio`, `async/await`, async generators, proper concurrency patterns
- **Packaging** — `pyproject.toml`, virtual environments, dependency management

## When to use me

Use this skill when:
- Writing new Python modules, packages, or scripts
- Refactoring Python code to be more idiomatic
- Adding type hints to existing Python code
- Setting up a Python project structure
- Debugging Python runtime or type errors

## How I work

1. **Discover the environment** — Check Python version, dependency files (requirements.txt, pyproject.toml, poetry.lock, Pipfile), test runner, linter config (ruff, flake8, mypy), and formatting config (black, ruff format).
2. **Follow existing conventions** — Import ordering, naming style (snake_case), docstring format (Google/NumPy/Sphinx), test structure, and module organization.
3. **Choose the right tool**:
   - Simple structured data → `dataclass` or `NamedTuple`
   - Validation needed → `pydantic` if in project, otherwise `dataclass` with `__post_init__`
   - Many static methods → Consider a module-level function instead
   - Enum-like constants → `enum.Enum` or `Literal` type
   - Async I/O → `asyncio` with `async def`/`await`
4. **Write the code** — Type hints on all function signatures, docstrings for public APIs, proper exception handling with custom exception classes.
5. **Handle dependencies properly** — Use existing project utilities. Don't add new dependencies without checking what's already available.
6. **Write tests** — Follow the project's test framework (pytest preferred). Parametrize edge cases. Use fixtures for setup.

## Patterns I follow

- `if __name__ == "__main__"` guard for scripts
- Context managers for resources (files, connections, locks)
- `pathlib.Path` over `os.path`
- f-strings over `.format()` or `%`
- `pathlib` and modern string methods
- Generator expressions for memory-efficient iteration
- Structured logging over `print()` statements

## Anti-patterns I avoid

- Mutable default arguments (`def f(x=[])`)
- Bare `except:` or `except Exception:` without re-raising
- `import *` from any module
- Class inheritance when composition works
- Global state without good reason
- Ignoring `None` when type hints say it's possible

## Project structure (for new projects)

```
project/
  pyproject.toml
  src/
    package/
      __init__.py
      module.py
  tests/
    conftest.py
    test_module.py
```