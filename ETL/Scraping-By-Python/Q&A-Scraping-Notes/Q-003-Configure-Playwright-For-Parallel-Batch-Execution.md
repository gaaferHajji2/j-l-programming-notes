Configuring Playwright for parallel batch execution requires careful management of browser contexts, concurrency limits, and resource allocation. Here is a production-ready approach:

### 1. Core Architecture: Browser vs. Context Parallelism

Playwright offers two parallelism strategies with very different resource profiles:

| Strategy | RAM per Unit | Isolation | Best For |
| :--- | :--- | :--- | :--- |
| **Browser-level** (`launch()` × N) | 200–500 MB each | Full process isolation | Different sites, heavy anti-bot |
| **Context-level** (`new_context()` × N) | 5–20 MB each | Shared browser, isolated cookies/storage | Same site, high throughput |

> ⚠️ **Rule of Thumb:** Use **context-level parallelism** within a single browser for same-site scraping. Reserve browser-level parallelism only when you need separate fingerprints or are hitting multiple domains.

### 2. Prefect + Playwright Batch Template

```python
import asyncio
from playwright.async_api import async_playwright
from prefect import flow, task
from prefect.concurrency.sync import concurrency_limit

# Define concurrency slot based on available RAM
# Rule: ~300MB per browser, ~15MB per context
MAX_CONCURRENT_CONTEXTS = 20
MAX_CONCURRENT_BROWSERS = 3

@task(retries=3, retry_delay_seconds=[10, 30, 60])
async def scrape_page(context, url: str) -> dict:
    """Single page scrape with automatic retry."""
    page = await context.new_page()
    try:
        await page.goto(url, wait_until="networkidle", timeout=30000)
        title = await page.title()
        content = await page.inner_text("body")
        return {"url": url, "title": title, "content": content}
    finally:
        await page.close()

@flow
async def batch_scrape(urls: list[str]):
    async with async_playwright() as p:
        # Launch limited number of browsers
        browsers = [
            await p.chromium.launch(headless=True)
            for _ in range(MAX_CONCURRENT_BROWSERS)
        ]

        # Create context pool across browsers (round-robin)
        contexts = []
        for i in range(MAX_CONCURRENT_CONTEXTS):
            browser = browsers[i % len(browsers)]
            ctx = await browser.new_context(
                viewport={"width": 1920, "height": 1080},
                user_agent="Mozilla/5.0 ...",  # Rotate UAs
            )
            contexts.append(ctx)

        # Map tasks with concurrency control
        results = await scrape_page.map(
            contexts[:len(urls)],  # Reuse contexts cyclically
            urls,
            concurrency=MAX_CONCURRENT_CONTEXTS
        )

        # Cleanup
        for ctx in contexts:
            await ctx.close()
        for b in browsers:
            await b.close()

    return results
```

### 3. Key Configuration Parameters

#### Browser Launch Options
```python
browser = await p.chromium.launch(
    headless=True,           # Always headless for batch
    args=[
        "--disable-gpu",       # Reduces memory on Linux servers
        "--no-sandbox",        # Required in Docker/CI
        "--disable-dev-shm-usage",  # Prevents /dev/shm exhaustion
        "--single-process",    # Optional: further reduces RAM (less stable)
    ],
    slow_mo=0,                 # No delay in batch mode
)
```

#### Context Options for Anti-Detection
```python
context = await browser.new_context(
    viewport={"width": random.choice([1920, 1440, 1366]), "height": 1080},
    locale=random.choice(["en-US", "en-GB"]),
    timezone_id="America/New_York",
    color_scheme="light",
    # Critical: isolate storage per parallel unit
    storage_state=None,
)
```

### 4. Resource Sizing Formula

Calculate safe concurrency based on available resources:

```
Max Contexts = min(
    (Available_RAM_MB - 500) / 15,      # 15MB per context + 500MB overhead
    CPU_CORES * 4,                        # ~4 contexts per core before contention
    TARGET_SITE_RATE_LIMIT_RPS * AVG_RENDER_SEC  # Respect rate limits
)
```

| Available RAM | Safe Contexts | Safe Browsers | Expected Throughput |
| :--- | :--- | :--- | :--- |
| 4 GB | 20 | 1 | ~100 pages/min |
| 8 GB | 50 | 2 | ~300 pages/min |
| 16 GB | 120 | 4 | ~700 pages/min |
| 32 GB | 300 | 8 | ~1500 pages/min |

### 5. Critical Pitfalls to Avoid

-   **Never share pages across tasks:** Each `scrape_page` must create and close its own page. Shared pages cause race conditions and corrupted state
-   **Don't use `wait_for_timeout()`:** Always use `wait_for_selector()` or `networkidle`. Fixed waits destroy parallelism efficiency
-   **Avoid `page.screenshot()` in hot paths:** Screenshots are synchronous and block the event loop. Use only for debugging
-   **Handle context leaks:** Always use `try/finally` or async context managers. Leaked contexts accumulate and crash the batch
-   **Monitor memory at runtime:** Add a Prefect task that checks `psutil.virtual_memory()` and pauses if usage exceeds 85%

### 6. Scaling Beyond Single Machine

When single-machine parallelism hits limits:

| Approach | Tool | When to Use |
| :--- | :--- | :--- |
| **Prefect Workers** | Prefect OSS/Cloud | Distribute batches across multiple machines with shared state  |
| **Scrapy Cluster** | Scrapy + Redis | Massive crawls needing deduplication and priority queues |
| **Browserless Pool** | Browserless.io | Offload browser rendering to managed infrastructure |
| **K8s Job Array** | Kubernetes + Prefect | Elastic scaling for variable batch sizes |

Would you like this template adapted for a specific framework (Airflow/Kestra) or extended with proxy rotation and fingerprint randomization?