Here's a complete working example that integrates both screenshots on failure and selective metadata capture:

## Project Structure
```
tests/
├── conftest.py
├── test_example.py
├── screenshots/
└── metadata/
```

## 1. conftest.py (Configuration & Hooks)

```python
import pytest
import json
import os
from datetime import datetime
from pathlib import Path


# Create directories if they don't exist
SCREENSHOTS_DIR = Path("screenshots")
METADATA_DIR = Path("metadata")
SCREENSHOTS_DIR.mkdir(exist_ok=True)
METADATA_DIR.mkdir(exist_ok=True)


@pytest.hookimpl(tryfirst=True, hookwrapper=True)
def pytest_runtest_makereport(item, call):
    """
    Hook to capture test report for each phase (setup, call, teardown).
    This allows us to check if a test failed later.
    """
    outcome = yield
    rep = outcome.get_result()
    # Store the report in the item for later access
    setattr(item, f"rep_{rep.when}", rep)


def pytest_runtest_logreport(report):
    """
    Called after each test phase completes.
    We use this to take screenshots on failure.
    """
    if report.when == "call" and report.failed:
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S_%f")
        # Sanitize test name for filename
        test_name = report.nodeid.replace("::", "_").replace("/", "_")
        filename = f"{test_name}_{timestamp}.png"
        filepath = SCREENSHOTS_DIR / filename
        
        print(f"\n❌ TEST FAILED: {report.nodeid}")
        print(f"📸 Screenshot would be saved to: {filepath}")
        # Note: Actual screenshot saving happens in the fixture below
        # where we have access to the browser instance


@pytest.fixture
def browser(request):
    """
    Browser fixture that provides a Selenium WebDriver instance
    and handles screenshot capture on failure.
    """
    from selenium import webdriver
    
    # Setup: Create browser instance
    options = webdriver.ChromeOptions()
    options.add_argument("--start-maximized")
    driver = webdriver.Chrome(options=options)
    driver.implicitly_wait(10)
    
    yield driver
    
    # Teardown: Check if test failed and take screenshot
    if hasattr(request.node, 'rep_call') and request.node.rep_call.failed:
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S_%f")
        test_name = request.node.name
        filename = f"{test_name}_{timestamp}.png"
        filepath = SCREENSHOTS_DIR / filename
        
        try:
            driver.save_screenshot(str(filepath))
            print(f"✅ Screenshot saved: {filepath}")
        except Exception as e:
            print(f"⚠️ Failed to save screenshot: {e}")
    
    driver.quit()


@pytest.fixture(autouse=True)
def capture_metadata(request):
    """
    Automatically captures metadata for tests marked with @pytest.mark.capture_meta.
    This fixture runs for every test but only saves data if the marker is present.
    """
    metadata = {
        'test_name': request.node.name,
        'test_file': request.node.fspath.strpath,
        'markers': [mark.name for mark in request.node.iter_markers()],
        'start_time': datetime.now().isoformat()
    }
    
    yield metadata
    
    # After test completes, add more metadata
    if hasattr(request.node, 'rep_call'):
        metadata['end_time'] = datetime.now().isoformat()
        metadata['duration_seconds'] = request.node.rep_call.duration
        metadata['outcome'] = request.node.rep_call.outcome
        metadata['longrepr'] = str(request.node.rep_call.longrepr) if request.node.rep_call.failed else None
    
    # Only save if test has the capture_meta marker
    marker = request.node.get_closest_marker("capture_meta")
    if marker:
        # Add custom metadata from marker args
        if marker.args:
            metadata['custom_data'] = marker.args[0]
        if marker.kwargs:
            metadata.update(marker.kwargs)
        
        # Save metadata to file
        save_metadata(request.node.name, metadata)


def save_metadata(test_name, metadata):
    """Save test metadata to a JSON file."""
    # Sanitize test name for filename
    safe_name = test_name.replace("::", "_").replace("/", "_")
    filepath = METADATA_DIR / f"{safe_name}.json"
    
    try:
        with open(filepath, 'w') as f:
            json.dump(metadata, f, indent=2, default=str)
        print(f"📊 Metadata saved: {filepath}")
    except Exception as e:
        print(f"⚠️ Failed to save metadata: {e}")


# Register custom marker
def pytest_configure(config):
    config.addinivalue_line(
        "markers",
        "capture_meta: Mark test to capture and save metadata"
    )
```

## 2. test_example.py (Test Cases)

```python
import time
import pytest


class TestEcommerce:
    """Example test class demonstrating screenshot and metadata features."""
    
    @pytest.mark.capture_meta({
        "environment": "staging",
        "test_priority": "high",
        "feature": "checkout"
    })
    def test_add_to_cart(self, browser):
        """This test will save metadata AND screenshot on failure."""
        browser.get("https://example.com")
        
        # Simulate some actions
        time.sleep(1)
        
        # Intentional assertion to demonstrate failure screenshot
        assert browser.title == "Example Domain"
        print("✅ Cart test passed")
    
    @pytest.mark.capture_meta({
        "environment": "production",
        "test_priority": "medium",
        "feature": "search"
    })
    def test_search_functionality(self, browser):
        """Another test with metadata capture."""
        browser.get("https://example.com")
        time.sleep(0.5)
        
        # This should pass
        assert "Example" in browser.title
        print("✅ Search test passed")
    
    def test_simple_navigation(self, browser):
        """
        This test will NOT save metadata (no marker),
        but WILL save screenshot if it fails.
        """
        browser.get("https://example.com")
        time.sleep(0.5)
        
        # This will fail to demonstrate screenshot capture
        assert False, "Intentional failure for demo"
    
    @pytest.mark.capture_meta({"performance_test": True})
    def test_page_load_time(self, browser):
        """Test with performance metadata."""
        start_time = time.time()
        browser.get("https://example.com")
        end_time = time.time()
        
        load_time = end_time - start_time
        print(f"Page loaded in {load_time:.2f} seconds")
        
        # Assert load time is reasonable
        assert load_time < 5, f"Page took too long to load: {load_time:.2f}s"


class TestLogin:
    """Another test class to show metadata works across classes."""
    
    @pytest.mark.capture_meta({
        "environment": "qa",
        "test_type": "smoke",
        "module": "authentication"
    })
    def test_valid_login(self, browser):
        """Login test with comprehensive metadata."""
        browser.get("https://example.com/login")
        time.sleep(0.5)
        
        # Simulate login actions
        assert browser.current_url != ""
        print("✅ Login test passed")
    
    def test_invalid_login(self, browser):
        """No metadata capture, but screenshot on failure."""
        browser.get("https://example.com/login")
        
        # Intentional failure
        assert 1 == 2, "Credentials should be invalid"
```

## 3. Running the Tests

```bash
# Run all tests
pytest tests/test_example.py -v -s

# Run with HTML report (optional, requires pytest-html)
pytest tests/test_example.py -v --html=report.html
```

## Expected Output

When you run the tests, you'll see:

```
tests/test_example.py::TestEcommerce::test_add_to_cart PASSED
📊 Metadata saved: metadata/test_add_to_cart.json

tests/test_example.py::TestEcommerce::test_search_functionality PASSED
📊 Metadata saved: metadata/test_search_functionality.json

tests/test_example.py::TestEcommerce::test_simple_navigation FAILED
❌ TEST FAILED: tests/test_example.py::TestEcommerce::test_simple_navigation
📸 Screenshot saved: screenshots/test_simple_navigation_20261025_143022.png

tests/test_example.py::TestEcommerce::test_page_load_time PASSED
📊 Metadata saved: metadata/test_page_load_time.json

tests/test_example.py::TestLogin::test_valid_login PASSED
📊 Metadata saved: metadata/test_valid_login.json

tests/test_example.py::TestLogin::test_invalid_login FAILED
❌ TEST FAILED: tests/test_example.py::TestLogin::test_invalid_login
📸 Screenshot saved: screenshots/test_invalid_login_20261025_143025.png
```

## Sample Metadata File (metadata/test_add_to_cart.json)

```json
{
  "test_name": "test_add_to_cart",
  "test_file": "/path/to/tests/test_example.py",
  "markers": ["capture_meta"],
  "start_time": "2026-10-25T14:30:20.123456",
  "end_time": "2026-10-25T14:30:21.654321",
  "duration_seconds": 1.530865,
  "outcome": "passed",
  "longrepr": null,
  "custom_data": {
    "environment": "staging",
    "test_priority": "high",
    "feature": "checkout"
  }
}
```

## Key Features

✅ **Automatic screenshots** on any test failure (no marker needed)  
✅ **Selective metadata** only for tests with `@pytest.mark.capture_meta`  
✅ **Clean separation** of concerns using fixtures and hooks  
✅ **Custom metadata** via marker args and kwargs  
✅ **Timestamped files** to avoid overwriting  
✅ **Error handling** for screenshot/metadata save failures  

This approach gives you the best of both worlds: automatic failure documentation and selective detailed metadata tracking!