# Python Styleguide

Standards for writing clean, Pythonic code following PEP 8 and modern practices.

## General Principles

- Follow PEP 8 for style
- Use type hints (PEP 484)
- Write docstrings (PEP 257)
- Prefer explicit over implicit

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Modules | snake_case | `user_service.py` |
| Classes | PascalCase | `UserService` |
| Functions | snake_case | `get_user_by_id` |
| Variables | snake_case | `user_name` |
| Constants | SCREAMING_SNAKE | `MAX_RETRIES` |
| Private | Leading underscore | `_internal_method` |
| "Dunder" | Double underscore | `__init__`, `__str__` |

## Type Hints

### Function Annotations

```python
def get_user(user_id: str) -> User:
    """Retrieve a user by their ID."""
    return repository.find(user_id)

def process_items(items: list[str], callback: Callable[[str], None]) -> None:
    for item in items:
        callback(item)
```

### Variable Annotations

```python
# Type annotations for variables
name: str = "Alice"
users: list[User] = []
config: dict[str, Any] = {}

# For complex types, use type aliases
UserMap = dict[str, User]
user_cache: UserMap = {}
```

### Optional and Union

```python
from typing import Optional, Union

# Python 3.10+ syntax
def find_user(user_id: str) -> User | None:
    ...

# Or use Optional (equivalent to X | None)
def find_user(user_id: str) -> Optional[User]:
    ...
```

## Classes

### Dataclasses

```python
from dataclasses import dataclass, field

@dataclass
class User:
    id: str
    name: str
    email: str
    roles: list[str] = field(default_factory=list)

    def is_admin(self) -> bool:
        return "admin" in self.roles
```

### Protocols (Structural Typing)

```python
from typing import Protocol

class Repository(Protocol):
    def find(self, id: str) -> Model | None: ...
    def save(self, model: Model) -> None: ...

# Any class implementing these methods satisfies the Protocol
class UserRepository:
    def find(self, id: str) -> User | None:
        ...
    def save(self, user: User) -> None:
        ...
```

### Abstract Base Classes

```python
from abc import ABC, abstractmethod

class BaseService(ABC):
    @abstractmethod
    def process(self, data: dict) -> Result:
        """Process the input data."""
        pass
```

## Error Handling

### Custom Exceptions

```python
class ValidationError(Exception):
    def __init__(self, message: str, field: str):
        super().__init__(message)
        self.field = field

class NotFoundError(Exception):
    pass
```

### Context Managers

```python
from contextlib import contextmanager

@contextmanager
def database_transaction():
    connection = get_connection()
    try:
        yield connection
        connection.commit()
    except Exception:
        connection.rollback()
        raise
    finally:
        connection.close()

# Usage
with database_transaction() as conn:
    conn.execute(query)
```

### Exception Chaining

```python
try:
    result = external_api.call()
except ExternalAPIError as e:
    raise ServiceError("API call failed") from e
```

## Module Organization

### Import Order

```python
# 1. Standard library
import os
import sys
from pathlib import Path

# 2. Third-party packages
import requests
from pydantic import BaseModel

# 3. Local application imports
from myapp.services import UserService
from myapp.utils import logger
```

### `__init__.py` Patterns

```python
# src/services/__init__.py
from .user_service import UserService
from .auth_service import AuthService

__all__ = ["UserService", "AuthService"]
```

## Testing Patterns

### pytest Conventions

```python
# test_user_service.py
import pytest
from myapp.services import UserService

class TestUserService:
    @pytest.fixture
    def service(self, mock_repository):
        return UserService(mock_repository)

    def test_create_user_with_valid_input(self, service):
        user = service.create_user(name="Alice", email="alice@example.com")
        assert user.name == "Alice"

    def test_create_user_raises_on_invalid_email(self, service):
        with pytest.raises(ValidationError):
            service.create_user(name="Alice", email="invalid")
```

### Fixtures

```python
@pytest.fixture
def mock_repository(mocker):
    repo = mocker.Mock()
    repo.find.return_value = None
    return repo

@pytest.fixture
def sample_user():
    return User(id="123", name="Alice", email="alice@example.com")
```

### Parametrized Tests

```python
@pytest.mark.parametrize("email,expected", [
    ("valid@example.com", True),
    ("also.valid@domain.org", True),
    ("invalid", False),
    ("@nodomain.com", False),
])
def test_email_validation(email, expected):
    assert is_valid_email(email) == expected
```

## Docstrings

### Google Style

```python
def calculate_total(items: list[Item], discount: float = 0.0) -> float:
    """Calculate the total price of items with optional discount.

    Args:
        items: List of items to calculate total for.
        discount: Percentage discount to apply (0.0 to 1.0).

    Returns:
        The total price after discount.

    Raises:
        ValueError: If discount is not between 0 and 1.
    """
    if not 0 <= discount <= 1:
        raise ValueError("Discount must be between 0 and 1")

    subtotal = sum(item.price for item in items)
    return subtotal * (1 - discount)
```

## Quick Reference

| Aspect | Recommendation |
|--------|----------------|
| Formatter | black or ruff |
| Linter | ruff or flake8 |
| Type checker | mypy or pyright |
| Test runner | pytest |
| Indentation | 4 spaces |
| Line length | 88 (black) or 79 (PEP 8) |
| Quotes | Double preferred |
