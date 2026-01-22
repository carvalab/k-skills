# TDD Examples: Python

Complete TDD example for Python projects.

## Example: Inventory Management

### Step 1: RED - Write Failing Test

```python
# test_inventory.py
import pytest
from inventory import InventoryService, Product
from test_helpers import InMemoryProductRepository

class TestInventoryService:
    def setup_method(self):
        self.repo = InMemoryProductRepository()
        self.service = InventoryService(self.repo)

    def test_add_stock_increases_quantity(self):
        # Arrange
        self.repo.save(Product(sku="WIDGET-001", quantity=10))

        # Act
        result = self.service.add_stock("WIDGET-001", 5)

        # Assert
        assert result.quantity == 15

    def test_add_stock_raises_for_unknown_product(self):
        with pytest.raises(ValueError, match="Product not found"):
            self.service.add_stock("UNKNOWN-SKU", 5)

    def test_remove_stock_decreases_quantity(self):
        self.repo.save(Product(sku="WIDGET-001", quantity=10))

        result = self.service.remove_stock("WIDGET-001", 3)

        assert result.quantity == 7

    def test_remove_stock_raises_for_insufficient_quantity(self):
        self.repo.save(Product(sku="WIDGET-001", quantity=5))

        with pytest.raises(ValueError, match="Insufficient stock"):
            self.service.remove_stock("WIDGET-001", 10)
```

Run: `pytest` → Should fail

### Step 2: GREEN - Minimal Implementation

```python
# inventory.py
from dataclasses import dataclass
from typing import Protocol

@dataclass
class Product:
    sku: str
    quantity: int

class ProductRepository(Protocol):
    def find_by_sku(self, sku: str) -> Product | None: ...
    def save(self, product: Product) -> None: ...

class InventoryService:
    def __init__(self, repo: ProductRepository):
        self.repo = repo

    def add_stock(self, sku: str, quantity: int) -> Product:
        product = self.repo.find_by_sku(sku)
        if not product:
            raise ValueError(f"Product not found: {sku}")

        product.quantity += quantity
        self.repo.save(product)
        return product

    def remove_stock(self, sku: str, quantity: int) -> Product:
        product = self.repo.find_by_sku(sku)
        if not product:
            raise ValueError(f"Product not found: {sku}")

        if product.quantity < quantity:
            raise ValueError(f"Insufficient stock for {sku}")

        product.quantity -= quantity
        self.repo.save(product)
        return product
```

### Step 3: Test Doubles

```python
# test_helpers.py
from inventory import Product

class InMemoryProductRepository:
    def __init__(self):
        self._products: dict[str, Product] = {}

    def find_by_sku(self, sku: str) -> Product | None:
        return self._products.get(sku)

    def save(self, product: Product) -> None:
        self._products[product.sku] = product

    def clear(self) -> None:
        self._products.clear()
```

---

## Pytest Patterns

### Fixtures

```python
import pytest

@pytest.fixture
def user_repo():
    return InMemoryUserRepository()

@pytest.fixture
def user_service(user_repo):
    return UserService(user_repo)

@pytest.fixture
def sample_user():
    return User(id="1", email="test@example.com", name="Test User")

def test_create_user(user_service, sample_user):
    created = user_service.create(sample_user)
    assert created.id is not None
```

### Parametrized Tests

```python
import pytest

@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
    (100, -50, 50),
])
def test_add(a, b, expected):
    calc = Calculator()
    assert calc.add(a, b) == expected

@pytest.mark.parametrize("email,valid", [
    ("user@example.com", True),
    ("invalid", False),
    ("@example.com", False),
    ("user@", False),
    ("user.name@domain.co.uk", True),
])
def test_email_validation(email, valid):
    assert is_valid_email(email) == valid
```

### Exception Testing

```python
def test_divide_by_zero_raises():
    calc = Calculator()

    with pytest.raises(ZeroDivisionError):
        calc.divide(10, 0)

def test_invalid_user_raises_with_message():
    service = UserService()

    with pytest.raises(ValueError) as exc_info:
        service.create(User(email="invalid"))

    assert "Invalid email" in str(exc_info.value)
```

---

## Test Doubles in Python

### Fake (In-Memory Implementation)

```python
class InMemoryDatabase:
    def __init__(self):
        self._data: dict[str, dict] = {}

    def insert(self, table: str, record: dict) -> str:
        id = str(len(self._data.get(table, {})) + 1)
        if table not in self._data:
            self._data[table] = {}
        self._data[table][id] = {**record, "id": id}
        return id

    def find(self, table: str, id: str) -> dict | None:
        return self._data.get(table, {}).get(id)
```

### Stub (Returns Canned Data)

```python
class StubWeatherAPI:
    def __init__(self, temperature: float = 20.0):
        self.temperature = temperature

    def get_temperature(self, city: str) -> float:
        return self.temperature

def test_weather_display():
    stub = StubWeatherAPI(temperature=25.0)
    display = WeatherDisplay(stub)

    result = display.show("London")

    assert "25.0°C" in result
```

### Spy (Records Calls)

```python
class SpyEmailSender:
    def __init__(self):
        self.sent_emails: list[tuple[str, str, str]] = []

    def send(self, to: str, subject: str, body: str) -> None:
        self.sent_emails.append((to, subject, body))

def test_sends_welcome_email():
    spy = SpyEmailSender()
    service = NotificationService(spy)

    service.send_welcome("user@example.com")

    assert len(spy.sent_emails) == 1
    assert spy.sent_emails[0][0] == "user@example.com"
    assert "Welcome" in spy.sent_emails[0][1]
```

### Mock with unittest.mock

```python
from unittest.mock import Mock, patch, MagicMock

def test_with_mock():
    mock_repo = Mock()
    mock_repo.find_by_id.return_value = User(id="1", name="John")

    service = UserService(mock_repo)
    user = service.get_user("1")

    assert user.name == "John"
    mock_repo.find_by_id.assert_called_once_with("1")

@patch('mymodule.external_api')
def test_with_patch(mock_api):
    mock_api.fetch.return_value = {"data": "value"}

    result = my_function()

    assert result == "value"
    mock_api.fetch.assert_called()
```

---

## Async Testing

```python
import pytest

@pytest.mark.asyncio
async def test_async_fetch():
    service = AsyncDataService()

    result = await service.fetch_data("key")

    assert result is not None

@pytest.mark.asyncio
async def test_async_error():
    service = AsyncDataService()

    with pytest.raises(ConnectionError):
        await service.fetch_data("invalid")
```

---

## Commands

```bash
# Run all tests
pytest

# Verbose output
pytest -v

# Run specific file
pytest test_inventory.py

# Run specific test
pytest -k "test_add_stock"

# Run tests matching pattern
pytest -k "stock and not remove"

# Coverage report
pytest --cov=src

# Coverage with HTML report
pytest --cov=src --cov-report=html

# Show slow tests
pytest --durations=10

# Stop on first failure
pytest -x

# Run last failed tests
pytest --lf

# Parallel execution
pytest -n auto
```

### pytest.ini Configuration

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_functions = test_*
addopts = -v --strict-markers
markers =
    slow: marks tests as slow
    integration: marks integration tests
```
