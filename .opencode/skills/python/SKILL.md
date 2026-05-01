---
name: python
description: Write idiomatic Python code following project conventions, type hints, and modern patterns
---

## What I do

I write Python code following modern best practices:

- **Project structure** — src layout, pyproject.toml, virtual environments
- **Type hints** — Gradual typing, mypy configuration, generic types
- **Async patterns** — asyncio, aiohttp, async/await best practices
- **Testing** — pytest, fixtures, parametrization, mocking
- **Code quality** — ruff, black, isort, pre-commit hooks

## When to use me

Use this skill when:
- Setting up a new Python project
- Adding type hints to existing Python code
- Building async Python applications
- Writing tests for Python code
- Configuring linting and formatting tools

## Project structure

```
myproject/
├── src/
│   └── myproject/
│       ├── __init__.py
│       ├── main.py
│       └── utils.py
├── tests/
│   ├── __init__.py
│   ├── test_main.py
│   └── conftest.py
├── pyproject.toml
├── README.md
└── .python-version
```

## pyproject.toml

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "myproject"
version = "0.1.0"
description = "My project description"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.100",
    "pydantic>=2.0",
    "httpx>=0.24",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4",
    "pytest-asyncio>=0.21",
    "mypy>=1.5",
    "ruff>=0.1.0",
]

[tool.ruff]
target-version = "py311"
line-length = 100
select = ["E", "F", "I", "N", "W", "UP", "B", "C4", "SIM"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_functions = ["test_*"]
asyncio_mode = "auto"
```

## Type hints

```python
from typing import Optional, TypedDict, Protocol
from collections.abc import Iterable

# Basic types
def greet(name: str) -> str:
    return f"Hello, {name}"

# Optional types
def find_user(user_id: str) -> Optional[User]:
    ...

# TypedDict for JSON-like structures
class UserData(TypedDict):
    id: str
    name: str
    email: str
    age: Optional[int]

# Protocol for structural subtyping
class Drawable(Protocol):
    def draw(self) -> None: ...

def render(items: Iterable[Drawable]) -> None:
    for item in items:
        item.draw()

# Generics
from typing import TypeVar

T = TypeVar('T')

def first(items: list[T]) -> Optional[T]:
    return items[0] if items else None
```

## Async patterns

```python
import asyncio
from aiohttp import ClientSession

async def fetch_data(url: str) -> dict:
    async with ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

async def fetch_multiple(urls: list[str]) -> list[dict]:
    tasks = [fetch_data(url) for url in urls]
    return await asyncio.gather(*tasks)

# FastAPI async endpoint
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: str) -> User:
    return await db.users.find_one({"_id": user_id})
```

## Testing

```python
# conftest.py
import pytest
from httpx import AsyncClient

@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

# test_main.py
import pytest

@pytest.mark.asyncio
async def test_get_user(client: AsyncClient):
    response = await client.get("/users/123")
    assert response.status_code == 200
    assert response.json()["id"] == "123"

@pytest.mark.parametrize("input,expected", [
    ("hello", 5),
    ("world", 5),
    ("", 0),
])
def test_string_length(input: str, expected: int):
    assert len(input) == expected

# Mocking
from unittest.mock import patch, MagicMock

def test_external_api():
    with patch("module.requests.get") as mock_get:
        mock_get.return_value.json.return_value = {"status": "ok"}
        result = fetch_status()
        assert result == {"status": "ok"}
```

## Key principles

- Use type hints everywhere (gradual typing)
- Use `pathlib` instead of `os.path`
- Use `dataclasses` or `pydantic` for data structures
- Prefer `async`/`await` for I/O-bound operations
- Use `pytest` for testing with fixtures and parametrization
- Format with `ruff` or `black`, lint with `ruff` or `flake8`
- Use virtual environments (venv, poetry, uv)

## Anti-patterns I avoid

- Using `print` instead of logging
- Mutable default arguments (`def func(items=[]):`)
- Not handling exceptions properly
- Using `except:` without specific exception types
- Mixing sync and async code incorrectly
- Not using type hints in new code
- Using `requirements.txt` without versions pinned
- Ignoring mypy errors