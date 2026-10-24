For running checks after each test in pytest, you have several options. Let me explain the best approaches for your use cases:

## Best Approaches

### 1. **Fixtures (Recommended for most cases)**
Fixtures are the most Pythonic and flexible way to handle post-test operations.

```python
import pytest
from selenium import webdriver

# Fixture with yield for setup/teardown
@pytest.fixture
def browser():
    driver = webdriver.Chrome()
    yield driver
    # This runs AFTER the test
    if hasattr(request.node, 'rep_call') and request.node.rep_call.failed:
        driver.save_screenshot(f"screenshots/{request.node.name}.png")
    driver.quit()

# Hook to track test results
@pytest.hookimpl(tryfirst=True, hookwrapper=True)
def pytest_runtest_makereport(item, call):
    outcome = yield
    rep = outcome.get_result()
    setattr(item, f"rep_{rep.when}", rep)
```

### 2. **Hooks (Best for global behavior)**
Use hooks when you need to apply behavior across all tests without modifying fixtures.

```python
# conftest.py
import os
from datetime import datetime

def pytest_runtest_logreport(report):
    """Called after each test phase (setup, call, teardown)"""
    if report.when == "call" and report.failed:
        # Take screenshot on failure
        save_failure_screenshot(report)

def pytest_runtest_teardown(item, nextitem):
    """Called after each test's teardown phase"""
    # Save metadata if needed
    if hasattr(item, 'custom_metadata'):
        save_metadata(item.name, item.custom_metadata)

def save_failure_screenshot(report):
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"screenshots/{report.nodeid}_{timestamp}.png"
    os.makedirs("screenshots", exist_ok=True)
    # Your screenshot logic here
    print(f"Screenshot saved: {filename}")

def save_metadata(test_name, metadata):
    os.makedirs("metadata", exist_ok=True)
    with open(f"metadata/{test_name}.json", "w") as f:
        json.dump(metadata, f)
```

### 3. **Custom Markers + Fixtures (For selective behavior)**
When you only want certain tests to have special handling.

```python
# conftest.py
import pytest
import json

@pytest.fixture(autouse=True)
def capture_metadata(request):
    """Automatically captures metadata for tests marked with @pytest.mark.capture_meta"""
    metadata = {}
    
    yield metadata
    
    # After test completes
    marker = request.node.get_closest_marker("capture_meta")
    if marker:
        # Add test-specific data to metadata
        metadata['duration'] = request.node.rep_call.duration
        metadata['status'] = request.node.rep_call.outcome
        
        # Save if marker has custom args
        if marker.args:
            metadata['custom'] = marker.args[0]
        
        save_test_metadata(request.node.name, metadata)

def save_test_metadata(test_name, metadata):
    with open(f"metadata/{test_name}.json", "w") as f:
        json.dump(metadata, f, indent=2)

# In your test file
@pytest.mark.capture_meta({"environment": "staging"})
def test_checkout_process(browser):
    # This test will save metadata
    pass

def test_simple_login(browser):
    # This test won't save metadata
    pass
```

## Comparison Table

| Approach | Best For | Pros | Cons |
|----------|----------|------|------|
| **Fixtures** | Per-test or per-suite cleanup | Clean, explicit, reusable | Need to add fixture to each test |
| **Hooks** | Global behavior across all tests | Automatic, no test changes needed | Less granular control |
| **Markers + Fixtures** | Selective behavior | Flexible, opt-in | Requires marking tests |

## Recommendation for Your Use Cases

**For screenshots on failure:**
- Use **hooks** (`pytest_runtest_makereport` + `pytest_runtest_logreport`) for automatic capture across all tests
- Combine with a fixture if you need access to browser/driver instance

**For selective metadata saving:**
- Use **markers + fixtures** approach - it's clean and explicit
- Only tests with `@pytest.mark.capture_meta` will save metadata

Would you like me to show you a complete working example integrating both screenshots and metadata capture?