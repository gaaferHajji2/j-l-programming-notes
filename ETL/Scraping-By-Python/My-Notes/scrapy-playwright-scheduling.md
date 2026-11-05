For **heavy JavaScript sites**, the strongest production pattern today is:

> **Use Scrapy as the crawl engine + Playwright only for pages that truly need rendering + a scheduler outside the spider for periodic execution.**

That gives you Scrapy’s throughput + Playwright’s browser rendering without paying browser cost everywhere. 🚀

---

# 1) Best package stack (recommended in production)

## Core stack

* Scrapy
* scrapy-playwright
* Playwright

`scrapy-playwright` is the official best-supported bridge because it plugs into Scrapy’s **download handler**, so Scrapy scheduling, pipelines, retries, and throttling still work normally. ([GitHub][1])

---

# 2) Architecture that scales best

## Golden rule: only render when necessary

Bad:

```python
meta={"playwright": True}
```

on every request.

Good:

```python
if requires_js:
    meta={"playwright": True}
else:
    normal Scrapy request
```

Why:
Browser pages are expensive in RAM + CPU. Community reports repeatedly show most bottlenecks come from overusing browser rendering. ([BrowserStack][2])

---

# 3) Best Scrapy settings for JS-heavy websites

## Production baseline

```python
DOWNLOAD_HANDLERS = {
    "http": "scrapy_playwright.handler.ScrapyPlaywrightDownloadHandler",
    "https": "scrapy_playwright.handler.ScrapyPlaywrightDownloadHandler",
}

TWISTED_REACTOR = "twisted.internet.asyncioreactor.AsyncioSelectorReactor"

CONCURRENT_REQUESTS = 16
CONCURRENT_REQUESTS_PER_DOMAIN = 4

PLAYWRIGHT_MAX_CONTEXTS = 4
PLAYWRIGHT_DEFAULT_NAVIGATION_TIMEOUT = 15000

AUTOTHROTTLE_ENABLED = True
RETRY_TIMES = 3
DOWNLOAD_TIMEOUT = 60
```

Important:

* `PLAYWRIGHT_MAX_CONTEXTS` prevents RAM explosion
* `AUTOTHROTTLE` stabilizes difficult targets
* lower concurrency than pure Scrapy

Official docs explicitly recommend controlling context counts. ([GitHub][1])

---

# 4) Best browser optimization tricks (huge speed gain) ⚡

Block useless assets:

```python
from scrapy_playwright.page import PageMethod

meta={
    "playwright": True,
    "playwright_page_methods": [
        PageMethod("route", "**/*", lambda route: (
            route.abort()
            if route.request.resource_type in ["image", "font", "media"]
            else route.continue_()
        ))
    ]
}
```

This often cuts:

* RAM by 40–70%
* page time by 2–5×

This is one of the highest ROI optimizations for heavy JS targets.

---

# 5) Use browser contexts correctly

## Reuse contexts for login/session sites

```python
meta={
    "playwright": True,
    "playwright_context": "session_1"
}
```

Why:

* cookies persist
* fewer browser launches
* fewer anti-bot triggers

Official docs recommend contexts heavily for session-heavy scraping. ([GitHub][1])

---

# 6) Best interaction pattern for heavy JS pages

Use `PageMethod` instead of manual page handling whenever possible:

```python
from scrapy_playwright.page import PageMethod

yield scrapy.Request(
    url=url,
    meta={
        "playwright": True,
        "playwright_page_methods": [
            PageMethod("wait_for_selector", ".product-card"),
            PageMethod("evaluate", "window.scrollBy(0, document.body.scrollHeight)")
        ]
    }
)
```

Why:
Cleaner than opening page object manually.

---

# 7) Best periodic scheduling options (important for production)

## Best choice: external scheduler

Do **not** schedule inside spider.

Use:

## Linux / server:

* cron

Example:

```bash
*/30 * * * * scrapy crawl prices
```

## Python-native:

* APScheduler

Example:

```python
from apscheduler.schedulers.blocking import BlockingScheduler
import subprocess

sched = BlockingScheduler()

@sched.scheduled_job('interval', minutes=30)
def run():
    subprocess.run(["scrapy", "crawl", "prices"])

sched.start()
```

Best when:

* many spiders
* retries
* logging
* timezone support

---

# 8) Best serious production orchestration

If you have multiple spiders:

## Use:

* Scrapyd

This gives:

* API-based spider runs
* scheduled jobs
* remote deploy
* queue management

Very strong for periodic operations.

---

# 9) Best anti-block pattern for JS-heavy sites 🛡️

Use rotating:

* user agents
* proxies
* browser contexts

In Scrapy:

```python
DOWNLOADER_MIDDLEWARES = {
    "scrapy.downloadermiddlewares.useragent.UserAgentMiddleware": None,
}
```

And rotate headers manually.

---

# 10) Best debugging method (massively underrated)

Always save screenshots on failures:

```python
PageMethod("screenshot", path="debug.png")
```

Because many JS failures are timing issues, not selector issues. ([BrowserStack][2])

---

# 11) Production-grade folder structure

```text
project/
 ├── spiders/
 ├── pipelines.py
 ├── middlewares.py
 ├── scheduler.py
 ├── settings.py
 ├── playwright_helpers.py
```

Separate browser helpers from spiders early.

---

# 12) Best modern production combo (my strongest recommendation)

For heavy JS + periodic scraping:

## Ideal stack

* Scrapy
* scrapy-playwright
* APScheduler or cron
* Scrapyd
* Redis queue (optional if scaling)

---

# 13) Hidden expert trick ⭐

For many sites:

First inspect XHR/API calls.

If site has hidden JSON endpoint:

Skip Playwright entirely.

Because:

```text
API scrape > Playwright scrape
```

in:

* speed
* reliability
* cost

Many "heavy JS" sites still expose clean JSON APIs.

---

# 14) If you want, I can also give you a **battle-tested production template**:

Includes:
✅ rotating proxies
✅ retry-safe Playwright pages
✅ periodic scheduler
✅ infinite scroll handler
✅ anti-memory leak cleanup

That template is what people usually move to after their first scraper starts crashing 😄

[1]: https://github.com/scrapy-plugins/scrapy-playwright?utm_source=chatgpt.com "GitHub - scrapy-plugins/scrapy-playwright: 🎭 Playwright integration for Scrapy"
[2]: https://www.browserstack.com/guide/scrapy-playwright?utm_source=chatgpt.com "The Scrapy Playwright Tutorial (2026) | BrowserStack"
