# Pytest Best Practices, Design Patterns & Essential Packages

## Core Best Practices

### 1. **Test Structure & Naming**
```python
# ✅ Good: Clear, descriptive naming
def test_user_registration_with_valid_email_returns_success():
    ...

def test_calculate_total_with_empty_cart_raises_value_error():
    ...

# ❌ Bad: Vague naming
def test_user():
    ...

def test_calc():
    ...
```

- Follow `test_<unit>_<scenario>_<expected_result>` pattern
- Group tests by module/class in separate files
- Use `conftest.py` for shared fixtures at each directory level

### 2. **Fixture Design Principles**
```python
# ✅ Factory fixtures for parameterized test data
@pytest.fixture
def make_user():
    def _make_user(name="John", email="john@example.com", role="user"):
        return User(name=name, email=email, role=role)
    return _make_user

# Usage - each test gets independent instances
def test_admin_permissions(make_user):
    admin = make_user(role="admin")
    assert admin.can_access_dashboard()

def test_regular_user_restrictions(make_user):
    user = make_user(role="user")
    assert not user.can_access_dashboard()
```

- **Prefer fixtures over setup/teardown methods**
- Use appropriate scopes: `function` (default), `class`, `module`, `session`
- Keep fixtures focused and composable
- Avoid fixture over-nesting (>3 levels deep is a smell)

### 3. **Assertion Style**
```python
# ✅ Plain asserts with clear messages
assert result.status_code == 200, f"Expected 200, got {result.status_code}"
assert "error" not in response.json()

# ✅ For complex comparisons, use specialized assertions
from pytest import approx
assert calculated_value == approx(expected, rel=1e-6)

# ❌ Avoid unittest-style assertions
self.assertEqual(a, b)  # Don't mix styles
```

### 4. **Test Isolation**
- Each test must be independently runnable
- Never rely on test execution order
- Reset state between tests (use fixtures with proper teardown)
- Mock external dependencies, not internal logic

---

## Key Design Patterns

### 1. **Factory Pattern (via Fixtures)**
```python
@pytest.fixture
def make_order():
    orders_created = []

    def _make_order(items=None, status="pending"):
        order = Order(items=items or [], status=status)
        orders_created.append(order)
        return order

    yield _make_order

    # Cleanup after test
    for order in orders_created:
        order.delete()
```

### 2. **Builder Pattern for Complex Objects**
```python
class TestDataBuilder:
    def __init__(self):
        self._data = {"name": "default", "age": 25, "active": True}

    def with_name(self, name):
        self._data["name"] = name
        return self

    def inactive(self):
        self._data["active"] = False
        return self

    def build(self):
        return UserProfile(**self._data)

@pytest.fixture
def user_builder():
    return TestDataBuilder()

# Fluent usage
def test_inactive_user_cannot_login(user_builder):
    user = user_builder.with_name("Alice").inactive().build()
    assert not user.can_login()
```

### 3. **Page Object / API Client Pattern**
```python
# conftest.py
@pytest.fixture
def api_client(base_url):
    client = APIClient(base_url)
    yield client
    client.close()

class APIClient:
    def __init__(self, base_url):
        self.session = requests.Session()
        self.base_url = base_url

    def create_user(self, **kwargs):
        resp = self.session.post(f"{self.base_url}/users", json=kwargs)
        return resp

    def get_user(self, user_id):
        return self.session.get(f"{self.base_url}/users/{user_id}")

# Test becomes clean and readable
def test_create_and_retrieve_user(api_client):
    resp = api_client.create_user(name="Bob", email="bob@test.com")
    assert resp.status_code == 201

    user = api_client.get_user(resp.json()["id"])
    assert user.json()["name"] == "Bob"
```

### 4. **Parametrized Test Matrix**
```python
@pytest.mark.parametrize("input_val,expected", [
    ("hello", "HELLO"),
    ("World", "WORLD"),
    ("", ""),
    ("123abc", "123ABC"),
])
def test_to_uppercase(input_val, expected):
    assert to_uppercase(input_val) == expected

# Combine multiple parameters for matrix testing
@pytest.mark.parametrize("role", ["admin", "editor", "viewer"])
@pytest.mark.parametrize("resource", ["dashboard", "reports", "settings"])
def test_permission_matrix(role, resource):
    assert check_permission(role, resource) == EXPECTED_PERMISSIONS[role][resource]
```

### 5. **Marker-Based Test Organization**
```python
# pytest.ini or pyproject.toml
[tool.pytest.ini_options]
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "integration: integration tests requiring external services",
    "smoke: critical path smoke tests",
]

# Usage
@pytest.mark.slow
@pytest.mark.integration
def test_full_etl_pipeline():
    ...

# Run selectively
# pytest -m "smoke"           → only smoke tests
# pytest -m "not slow"       → skip slow tests
# pytest -m "integration and not slow"
```

---

## Essential Packages

### 🔧 **Core Extensions**
| Package | Purpose |
|---------|---------|
| **pytest-cov** | Code coverage reporting |
| **pytest-xdist** | Parallel test execution (`-n auto`) |
| **pytest-mock** | Clean mocking via `mocker` fixture (wraps unittest.mock) |
| **pytest-asyncio** | Async test support |
| **pytest-randomly** | Randomize test order to catch ordering bugs |

### 🗄️ **Database & Data Testing**
| Package | Purpose |
|---------|---------|
| **pytest-postgresql** | PostgreSQL fixtures with temp databases |
| **pytest-django** | Django ORM test support |
| **pytest-sqlalchemy** | SQLAlchemy session/fixtures management |
| **factory-boy** | Model factories (pairs perfectly with pytest) |
| **faker** | Realistic fake data generation |

### 🌐 **API & Integration Testing**
| Package | Purpose |
|---------|---------|
| **pytest-httpx** / **responses** | Mock HTTP requests |
| **pytest-fastapi** | FastAPI testing utilities |
| **testcontainers-python** | Spin up real Docker containers (DBs, Kafka, Redis) |
| **pytest-benchmark** | Performance benchmarking |

### ✅ **Data Quality & Validation** *(relevant to your DE interests)*
| Package | Purpose |
|---------|---------|
| **great-expectations** | Data validation framework with pytest integration |
| **pandera** | DataFrame schema validation |
| **pytest-check** | Soft assertions (non-failing checks that report all failures) |
| **hypothesis** | Property-based testing (generates edge cases automatically) |

### 📊 **Reporting & DX**
| Package | Purpose |
|---------|---------|
| **pytest-html** | HTML test reports |
| **pytest-sugar** | Prettier terminal output |
| **pytest-watch** | Auto-rerun on file changes |
| **pytest-clarity** | Better diff output for assertion failures |

---

## Recommended Project Setup

```toml
# pyproject.toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = [
    "-v",
    "--strict-markers",
    "--tb=short",
    "-x",              # Stop on first failure during dev
    "--cov=src",
    "--cov-report=term-missing",
]
markers = [
    "unit: fast unit tests",
    "integration: requires external services",
    "slow: long-running tests",
]

[tool.coverage.run]
source = ["src"]
omit = ["*/tests/*", "*/migrations/*"]

[tool.coverage.report]
fail_under = 80
show_missing = true
```

---

## Anti-Patterns to Avoid

| ❌ Anti-Pattern | ✅ Better Approach |
|----------------|-------------------|
| Tests that depend on execution order | Independent tests with proper fixtures |
| Giant monolithic test functions | Small, focused tests testing one behavior |
| Mocking everything | Mock only external boundaries |
| Assertions without context messages | Descriptive failure messages |
| Shared mutable state across tests | Fresh instances via factory fixtures |
| Testing implementation details | Test behavior and contracts |
| Skipping failing tests permanently | Fix or delete; use `xfail` for known issues |

---

## Quick Start Recommendation Stack

For a **data engineering project**, start with this minimal powerful stack:

```bash
pip install pytest pytest-cov pytest-xdist pytest-mock \
            factory-boy faker hypothesis \
            pytest-check great-expectations pandera
```

This gives you: parallel execution, clean mocking, realistic test data, property-based testing, soft assertions, and data validation — covering the full spectrum of effective DE testing.