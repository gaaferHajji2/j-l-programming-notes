Testing a B2B e-commerce document creation flow that spans 5+ API endpoints is a classic **integration testing** challenge. The key tension is balancing *realism* (testing actual interactions) against *speed/reliability* (avoiding flaky, slow tests).

Here is a comprehensive strategy tailored for this scenario using Pytest.

---

## 1. Choose the Right Testing Strategy

For a 5-endpoint dependency chain, you need a **hybrid approach**:

| Layer | What to Test | Tool/Approach | Speed |
| :--- | :--- | :--- | :--- |
| **Unit** | Document generation logic, data transformation, validation | `pytest-mock`, pure functions | ⚡ Fast |
| **Contract** | Each API endpoint's request/response schema individually | `pytest-httpx` / VCR.py | ⚡ Fast |
| **Integration** | Full 5-endpoint orchestration flow | `testcontainers` or staged env | 🐢 Slow |
| **E2E** | Business-critical happy path only | Real staging environment | 🐌 Slowest |

> **Rule of Thumb:** Only 10-20% of your tests should hit all 5 real endpoints. The rest should be isolated or mocked at strategic boundaries.

---

## 2. Core Design Patterns for Multi-Endpoint Tests

### Pattern A: The API Client Abstraction (Critical)

Never call `requests.post()` directly in tests. Wrap every endpoint in a typed client:

```python
# src/clients/b2b_client.py
class B2BCommerceClient:
    """Single interface for all B2B document dependencies."""

    def __init__(self, base_url: str, auth_token: str):
        self.session = requests.Session()
        self.session.headers.update({"Authorization": f"Bearer {auth_token}"})
        self.base_url = base_url

    def get_customer(self, customer_id: str) -> dict: ...
    def get_pricing_agreement(self, agreement_id: str) -> dict: ...
    def get_inventory(self, skus: list[str]) -> dict: ...
    def get_tax_rules(self, region: str) -> dict: ...
    def create_document(self, payload: dict) -> dict: ...
    def validate_document(self, doc_id: str) -> dict: ...
```

**Why this matters:** Tests depend on the *abstraction*, not HTTP details. You can swap real ↔ mock seamlessly.

### Pattern B: Fixture Composition Chain

Model the 5-endpoint dependency as composable fixtures:

```python
# tests/conftest.py
@pytest.fixture(scope="session")
def b2b_client(staging_url, auth_token):
    """Real client for integration tests."""
    return B2BCommerceClient(staging_url, auth_token)

@pytest.fixture
def sample_customer(b2b_client):
    cust = b2b_client.get_customer("CUST-001")
    yield cust
    # Optional: cleanup if test created data

@pytest.fixture
def pricing_for_customer(b2b_client, sample_customer):
    return b2b_client.get_pricing_agreement(sample_customer["agreement_id"])

@pytest.fixture
def inventory_for_skus(b2b_client):
    return b2b_client.get_inventory(["SKU-A", "SKU-B", "SKU-C"])

@pytest.fixture
def tax_rules(b2b_client, sample_customer):
    return b2b_client.get_tax_rules(sample_customer["region"])

@pytest.fixture
def document_payload(sample_customer, pricing_for_customer,
                     inventory_for_skus, tax_rules):
    """Assembled payload from 4 upstream endpoints."""
    return build_document_payload(
        customer=sample_customer,
        pricing=pricing_for_customer,
        inventory=inventory_for_skus,
        tax=tax_rules,
    )
```

Each fixture maps to one API call. Tests consume the final assembled state without knowing the orchestration complexity.

### Pattern C: Boundary Mocking (Mock at the Client, Not HTTP)

```python
# ✅ GOOD: Mock at the abstraction boundary
def test_document_creation_with_expired_pricing(b2b_client_mock):
    b2b_client_mock.get_pricing_agreement.return_value = {
        "status": "expired", "valid_until": "2024-01-01"
    }
    # Other endpoints return normal data via defaults
    result = create_b2b_document(b2b_client_mock, customer_id="CUST-001")
    assert result["error_code"] == "PRICING_EXPIRED"

# ❌ BAD: Mocking raw HTTP calls
@patch("requests.Session.get")
def test_something(mock_get):
    mock_get.side_effect = [...]  # Fragile, order-dependent, unreadable
```

Use `pytest-mock`'s `mocker` fixture for clean syntax:
```python
def test_partial_inventory_failure(mocker, b2b_client):
    mocker.patch.object(
        b2b_client, "get_inventory",
        side_effect=APIError("Inventory service unavailable")
    )
    with pytest.raises(DocumentCreationError, match="inventory"):
        create_b2b_document(b2b_client, customer_id="CUST-001")
```

---

## 3. Handling Test Data & State

### Use Factory Boy for B2B Entities

```python
# tests/factories.py
import factory

class CustomerFactory(factory.DictFactory):
    id = factory.Sequence(lambda n: f"CUST-{n:04d}")
    name = factory.Faker("company")
    region = factory.Iterator(["US-CA", "US-NY", "EU-DE"])
    agreement_id = factory.LazyAttribute(lambda o: f"AGR-{o.id[-4:]}")
    tier = factory.Iterator(["platinum", "gold", "silver"])

class PricingAgreementFactory(factory.DictFactory):
    agreement_id = factory.Sequence(lambda n: f"AGR-{n:04d}")
    discount_pct = factory.Faker("pydecimal", left_digits=2, right_digits=2, positive=True)
    valid_until = factory.LazyFunction(lambda: date.today() + timedelta(days=365))
```

### Deterministic Seeding for Integration Tests

```python
@pytest.fixture(scope="session", autouse=True)
def seed_test_data(real_b2b_client):
    """Seed known test data ONCE per session for integration tests."""
    customers = [CustomerFactory(id=f"TEST-CUST-{i}") for i in range(5)]
    for c in customers:
        real_b2b_client.upsert_customer(c)
    yield customers
    # Teardown: delete test-prefixed records
    for c in customers:
        real_b2b_client.delete_customer(c["id"])
```

> **Key:** Use a naming convention (`TEST-` prefix) so cleanup is safe and idempotent.

---

## 4. Managing the 5-Endpoint Orchestration Test

### For the Full Integration Test

```python
@pytest.mark.integration
@pytest.mark.slow
class TestB2BDocumentCreationFlow:
    """Full 5-endpoint flow against real/staged services."""

    def test_happy_path_creates_valid_invoice(
        self, b2b_client, document_payload
    ):
        # Endpoint 5: Create
        doc = b2b_client.create_document(document_payload)
        assert doc["status"] == "created"
        assert doc["total_amount"] > 0

        # Endpoint 6 (validation): Verify
        validation = b2b_client.validate_document(doc["id"])
        assert validation["is_valid"] is True
        assert validation["warnings"] == []

    def test_cross_endpoint_data_consistency(
        self, b2b_client, sample_customer, pricing_for_customer,
        document_payload
    ):
        doc = b2b_client.create_document(document_payload)
        # Verify pricing from endpoint 2 matches document line items
        expected_discount = pricing_for_customer["discount_pct"]
        for line in doc["line_items"]:
            assert line["discount_applied"] == approx(expected_discount, rel=1e-4)
```

### For Fast Feedback: Contract Tests Per Endpoint

```python
@pytest.mark.contract
class TestPricingAgreementEndpoint:
    """Validates ONE endpoint contract independently."""

    def test_returns_expected_schema(self, b2b_client):
        resp = b2b_client.get_pricing_agreement("AGR-0001")
        assert set(resp.keys()) >= {"agreement_id", "discount_pct", "valid_until", "status"}

    def test_expired_agreement_has_correct_status(self, b2b_client):
        resp = b2b_client.get_pricing_agreement("AGR-EXPIRED")
        assert resp["status"] == "expired"
```

---

## 5. Essential Packages for This Scenario

| Package | Why You Need It |
| :--- | :--- |
| **`pytest-mock`** | Clean mocking of `B2BCommerceClient` methods |
| **`factory-boy` + `faker`** | Deterministic B2B entity generation |
| **`pytest-httpx`** | Mock HTTP at transport layer for contract tests |
| **`vcrpy`** | Record/replay real API responses for deterministic integration tests |
| **`testcontainers-python`** | Spin up real DBs/message queues locally for true integration |
| **`pytest-check`** | Soft assertions (validate multiple doc fields without stopping at first failure) |
| **`pytest-xdist`** | Parallelize independent contract/unit tests |
| **`great-expectations`** | Validate document structure/data quality post-creation |
| **`tenacity`** | Retry logic for flaky staging APIs in integration tests |

### VCR.py Example (Record Once, Replay Forever)
```python
import vcr

@vcr.use_cassette("fixtures/cassettes/full_doc_flow.yaml")
@pytest.mark.integration
def test_document_flow_recorded(b2b_client, document_payload):
    """First run hits real API; subsequent runs replay from cassette."""
    doc = b2b_client.create_document(document_payload)
    assert doc["status"] == "created"
```

---

## 6. Anti-Patterns Specific to Multi-Endpoint B2B Tests

| ❌ Anti-Pattern | ✅ Fix |
| :--- | :--- |
| One giant test hitting all 5 endpoints for every scenario | Decompose: unit test transforms, contract test each API, one integration test for full flow |
| Hardcoded IDs that break when staging resets | Factory-generated IDs + session-scoped seeding |
| Tests that create data but never clean up | `yield` fixtures with teardown; `TEST-` prefix convention |
| Asserting entire response dicts | Assert specific business-relevant fields; use `pytest-check` for multi-field validation |
| Running all 5-endpoint tests on every PR | Mark as `@pytest.mark.slow`; run in CI nightly or on merge to main only |
| Mocking internal service-to-service calls | Mock only at your client boundary; let internal orchestration run real code |

---

## 7. Recommended Test Pyramid Ratio

For your B2B document creation feature:

```
         ╱╲
        ╱ E2E ╲          ~5 tests   (staging, manual trigger)
       ╱────────╲
      ╱Integration╲      ~20 tests  (testcontainers/VCR, nightly CI)
     ╱──────────────╲
    ╱   Contract     ╲   ~30 tests  (per-endpoint schema/behavior, every PR)
   ╱──────────────────╲
  ╱      Unit          ╲ ~100 tests (transforms, validators, builders, every PR)
 ╱──────────────────────╲
```

This gives you **fast feedback on every commit** while still validating the real 5-endpoint flow regularly.

Searching: 
1. A complete example project structure for this B2B document testing scenario, 
2. dive deeper into any specific pattern (e.g., VCR cassette management, testcontainers setup, or Great Expectations integration)