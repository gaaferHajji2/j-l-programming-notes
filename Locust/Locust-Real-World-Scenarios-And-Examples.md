Yes—but with an important caveat: **Locust is excellent for many real performance/load-testing scenarios, especially API/server-side testing, but it is not ideal for every kind of performance test**. It is a code-first, Python-based load-testing tool, so it works best when you can model user behavior as Python code and measure requests to services, endpoints, APIs, or custom protocols.

Below are realistic scenarios where Locust is commonly used, organized by **type of performance test**, **business/domain scenario**, and **technical situation**.

---

## 1. Quick fit: what Locust is good for

| Testing type | Locust fit? | Typical real scenario |
|---|---:|---|
| API load testing | Excellent | Test REST, GraphQL, JSON, XML, form-based APIs under expected traffic |
| Microservice load testing | Excellent | Call individual services directly to find bottlenecks |
| Web application backend testing | Excellent | Simulate login, search, checkout, dashboard, reporting flows |
| Stress testing | Excellent | Increase load until the system fails or degrades |
| Spike testing | Excellent | Simulate sudden traffic from marketing campaigns, push notifications, incidents |
| Soak/endurance testing | Excellent | Run sustained load for hours to detect memory leaks, connection exhaustion |
| Capacity planning | Excellent | Compare configurations, instance sizes, cache settings, database indexes |
| Performance regression testing | Good | Run automated baseline tests in CI/CD |
| Scalability testing | Good | Validate autoscaling, load balancers, pod/VM scaling |
| WebSocket testing | Possible | Requires custom code or community plugins |
| gRPC testing | Possible | Requires custom Python client code |
| TCP/UDP protocol testing | Possible but not ideal | Can be scripted, but Locust’s strongest support is HTTP-like request/response testing |
| Browser/UI performance testing | Poor | Locust does not render HTML, CSS, JavaScript like a real browser |
| Mobile app end-to-end performance | Partial | Good for backend APIs, not for app rendering, animations, offline UI behavior |
| Database direct testing | Not ideal | Better to test through application APIs unless you write custom database clients |

In short: **Locust is best for server-side, protocol-level, API-centric performance testing.** It is not a replacement for browser-performance tools such as Playwright, Selenium, WebPageTest, Lighthouse, or real-user monitoring.

---

# 2. Real scenarios by performance-testing type

## A. Standard load testing

### What it means
You simulate expected production traffic to verify that the system meets performance goals.

### Real scenario
An e-commerce company launches a new product catalog API. The expected peak is 1,000 requests per second, with 95th-percentile latency below 300 ms and error rate below 0.1%.

### How Locust is used
You create simulated users that perform typical actions:

```python
from locust import HttpUser, task, between

class CatalogUser(HttpUser):
    wait_time = between(1, 3)

    @task(5)
    def browse_category(self):
        self.client.get("/api/categories/electronics")

    @task(3)
    def search_product(self):
        self.client.get("/api/search?q=laptop")

    @task(1)
    def view_product(self):
        self.client.get("/api/products/12345")
```

You then run the test with a target number of users and measure:

- Requests per second
- Average latency
- Median latency
- 95th percentile latency
- 99th percentile latency
- Failure rate
- HTTP status codes
- Throughput

### Why Locust fits well
The test is written as code, so you can easily model weighted user behavior, dynamic parameters, authentication, headers, cookies, and data dependencies.

---

## B. Stress testing

### What it means
You intentionally push the system beyond normal expected load to find its breaking point.

### Real scenario
A banking API is expected to handle 500 transactions per second. The team wants to know what happens at 1,000, 2,000, and 5,000 transactions per second.

### How Locust is used
You gradually increase users or spawn rate:

- Start at 500 users
- Increase to 1,000
- Increase to 2,000
- Continue until errors spike or latency becomes unacceptable

You observe:

- When HTTP 503 errors appear
- When database connection pools are exhausted
- When thread pools saturate
- When CPU or memory hits limits
- When latency jumps from acceptable to unacceptable

### Example goal
Find the maximum sustainable throughput before:

- Error rate exceeds 1%
- p95 latency exceeds 1 second
- Service becomes unavailable

### Why Locust fits well
Locust can be scripted to ramp load dynamically and can be distributed across multiple machines to generate very high pressure.

---

## C. Spike testing

### What it means
You simulate a sudden surge in traffic.

### Real scenario
A retail company sends a push notification announcing a flash sale. Traffic may jump from 100 requests per second to 5,000 requests per second within minutes.

### How Locust is used
You configure a rapid ramp-up:

- Normal load: 100 users
- Spike target: 5,000 users
- Ramp duration: 30 seconds to 2 minutes

You then measure whether the system can absorb the spike without cascading failure.

### Things to validate
- Autoscaling triggers fast enough
- Load balancer handles connection surge
- Database does not lock up
- Queue systems absorb bursts
- Rate limiting behaves correctly
- Cached content reduces origin load
- Circuit breakers prevent total collapse

### Why Locust fits well
Locust can simulate sudden user increases and is useful for testing bursty traffic patterns.

---

## D. Soak/endurance testing

### What it means
You run sustained load over a long period to discover issues that only appear after time.

### Real scenario
A SaaS platform expects continuous usage during business hours. The team runs an 8-hour soak test at 70% of expected peak load.

### Issues discovered by soak testing
- Memory leaks
- Connection leaks
- Thread leaks
- File descriptor exhaustion
- Database connection pool exhaustion
- Cache fragmentation
- Log disk space exhaustion
- Gradual latency degradation
- Garbage collection pressure
- Session store growth

### How Locust is used
You run a steady number of users for hours:

- 1,000 virtual users
- Constant load
- Duration: 8–24 hours
- Monitor both Locust metrics and infrastructure metrics

### Why Locust fits well
Locust can run long-duration tests, especially in headless mode with distributed workers.

---

## E. Capacity planning

### What it means
You determine how much load the system can handle and what infrastructure is needed.

### Real scenario
A company is planning for next year’s traffic growth. They need to know:

- How many API instances are required?
- What database size is needed?
- Is Redis cache sufficient?
- Can Kubernetes autoscale properly?
- What is the cost per 1,000 requests per second?

### How Locust is used
You run the same test against different configurations:

| Configuration | Users | RPS | p95 latency | Error rate | Cost |
|---|---:|---:|---:|---:|---:|
| 2 small VMs | 500 | 300 | 400 ms | 0.2% | $X |
| 4 small VMs | 1,000 | 700 | 350 ms | 0.1% | $Y |
| 2 large VMs | 1,000 | 850 | 280 ms | 0.05% | $Z |

### Why Locust fits well
Because tests are code-based, you can version them and rerun the same scenario across environments and configurations.

---

## F. Scalability testing

### What it means
You verify that the system scales horizontally or vertically as expected.

### Real scenario
A microservice runs on Kubernetes with horizontal pod autoscaling. The team wants to confirm that adding load causes new pods to spin up and that latency remains stable.

### How Locust is used
You gradually increase load and observe:

- Pod count
- CPU utilization
- Memory utilization
- Request latency
- Error rate
- Cold-start impact
- Database connection growth
- Queue backlog

### Example scenario
- Start with 200 virtual users
- Add 100 users every 2 minutes
- Observe whether Kubernetes adds pods
- Check whether new pods cause temporary latency spikes
- Verify that the system stabilizes

### Why Locust fits well
Locust can generate controlled, incremental load and integrate into automated test pipelines.

---

## G. Baseline and regression performance testing

### What it means
You establish a performance baseline and compare future releases against it.

### Real scenario
A development team deploys a new version of an order-service API. Before release, they run a standard 10-minute load test and compare results with the previous version.

### Pass/fail criteria
The new build passes if:

- p95 latency is not more than 10% worse than baseline
- Error rate remains below 0.1%
- Throughput is equal to or greater than baseline
- No new timeout patterns appear

### How Locust is used
You store Locust scripts in Git and run them in CI/CD:

```bash
locust -f orders_load_test.py --headless -u 500 -r 50 -t 10m --host https://staging.example.com
```

Results can be exported to CSV or sent to dashboards.

### Why Locust fits well
Locust tests are Python code, so they fit naturally into software engineering workflows, code review, CI pipelines, and automated gating.

---

## H. Rate-limiting and throttling tests

### What it means
You verify that the system correctly limits excessive traffic.

### Real scenario
A public API allows 100 requests per minute per API key. The team wants to confirm that clients exceeding the limit receive proper HTTP 429 responses and retry-after headers.

### How Locust is used
You create users that exceed the allowed rate and validate:

- HTTP 429 responses
- Retry-After header
- Error response format
- Whether legitimate traffic is protected
- Whether rate limiter itself becomes a bottleneck

### Why Locust fits well
You can write custom logic to handle 429 responses, back off, retry, or record throttling behavior.

---

## I. Failover and resilience testing under load

### What it means
You test how the system behaves when components fail while under load.

### Real scenario
While a load test is running, an operations engineer kills one application instance, fails over the database, or blocks a downstream service.

### How Locust is used
Locust continues generating traffic while you observe:

- Error spike duration
- Latency spike duration
- Automatic recovery time
- Whether retries cause thundering herd problems
- Whether circuit breakers open and close correctly
- Whether data consistency is maintained

### Important note
Locust is not a chaos-engineering tool, but it can be used alongside chaos tools to generate load during failure injection.

---

# 3. Real business/domain scenarios

## A. E-commerce platforms

### Common Locust tests
- Product browsing
- Category navigation
- Search
- Product details
- Cart addition
- Checkout
- Payment authorization
- Order confirmation
- Promo code validation
- Inventory checks

### Example scenario
During a holiday sale, the team simulates:

- 70% browsing users
- 20% searching users
- 8% adding to cart
- 2% completing checkout

Locust can model this with task weights:

```python
class ShopperUser(HttpUser):
    wait_time = between(2, 10)

    @task(70)
    def browse(self):
        self.client.get("/products")

    @task(20)
    def search(self):
        self.client.get("/search?q=shoes")

    @task(8)
    def add_to_cart(self):
        self.client.post("/cart/items", json={"product_id": 123, "quantity": 1})

    @task(2)
    def checkout(self):
        self.client.post("/checkout", json={"cart_id": "abc"})
```

### Why it matters
E-commerce systems often fail not because of raw traffic, but because of expensive paths: search, inventory reservation, payment, fraud checks, and shipping calculation.

---

## B. SaaS and multi-tenant applications

### Common Locust tests
- Tenant login
- Dashboard loading
- Report generation
- API usage per tenant
- Background job submission
- Webhook delivery
- File export
- Bulk data import

### Example scenario
A SaaS company wants to ensure that one large tenant does not degrade performance for smaller tenants.

Locust can simulate:

- 500 small tenants making light API calls
- 10 medium tenants generating reports
- 2 large tenants importing bulk data

### Metrics to watch
- Per-tenant latency
- Database contention
- Queue backlog
- API gateway throttling
- Noisy-neighbor effects

### Why Locust fits well
You can parameterize tests by tenant ID, API key, workload type, and user journey.

---

## C. Banking, fintech, and payment systems

### Common Locust tests
- Account balance inquiry
- Transaction history
- Fund transfer
- Bill payment
- Statement generation
- KYC document upload
- Fraud scoring API
- Payment gateway callbacks
- Token refresh
- OAuth authentication

### Example scenario
A bank launches a new real-time payments API. The requirement is:

- 99.9% availability
- p95 latency under 500 ms
- Support 2,000 transactions per second
- No duplicate transactions under retry load

Locust can simulate:

- Normal users checking balances
- High-value transfers
- Retried payments
- Idempotency-key usage
- Downstream fraud-check delays

### Special considerations
Payment tests must avoid creating real financial movement unless using sandbox environments or test accounts.

### Why Locust fits well
Financial APIs often require complex authentication, signed requests, idempotency keys, correlation IDs, and multi-step flows. Python code makes this easier than purely UI-driven tools.

---

## D. Healthcare and life sciences systems

### Common Locust tests
- Patient lookup
- Appointment scheduling
- FHIR API reads/writes
- Claim submission
- Eligibility verification
- Prescription lookup
- Lab result retrieval
- Document upload

### Example scenario
A hospital system integrates with an insurance eligibility API. The team needs to verify that the API can handle 300 concurrent clinicians checking eligibility during morning rounds.

### Metrics to validate
- Response time for patient lookup
- Authorization latency
- Audit logging overhead
- Database query performance
- HL7/FHIR transformation time

### Important note
Use synthetic or de-identified data. Do not load-test with real PHI unless policy and environment allow it.

### Why Locust fits well
Healthcare integrations often involve standards-based APIs, OAuth, SMART on FHIR, custom headers, and complex payload structures.

---

## E. Streaming, media, and content platforms

### Common Locust tests
- Manifest requests
- Playlist retrieval
- DRM license requests
- Recommendation API
- User profile fetch
- Watch history update
- Content metadata lookup
- CDN origin requests

### Example scenario
A video platform expects a major live event. Before the event, the team tests:

- 10,000 users requesting manifests
- 2,000 users requesting DRM licenses
- 500 users updating watch history
- 100 users searching catalog

### Bottlenecks to find
- CDN cache hit ratio
- Origin overload
- DRM license server saturation
- Database read pressure
- Recommendation service latency
- Token signing overhead

### Why Locust fits well
Media platforms often have high read traffic and many small HTTP requests, which Locust can simulate efficiently.

---

## F. Logistics, delivery, and transportation

### Common Locust tests
- Tracking lookup
- Label generation
- Rate quotation
- Address validation
- Carrier API integration
- Driver assignment
- Delivery status updates
- Warehouse inventory queries

### Example scenario
A delivery company experiences peak traffic during holiday seasons. The team simulates customers tracking packages and merchants generating shipping labels.

### Workloads
- 80% tracking lookups
- 15% rate quotes
- 5% label creation

### Bottlenecks to find
- Third-party carrier API latency
- Database index performance
- PDF generation time
- Address validation service
- Queue processing for asynchronous label creation

### Why Locust fits well
Logistics systems often combine internal APIs with external carrier APIs. Locust can orchestrate both.

---

## G. Advertising technology and high-QPS systems

### Common Locust tests
- Bid request endpoint
- Impression tracking
- Click tracking
- User profiling
- Frequency capping
- Real-time bidding callbacks
- Pixel firing

### Example scenario
An ad-tech platform must handle 50,000 requests per second with p99 latency under 100 ms.

### Challenges
- Extremely high throughput
- Low latency requirements
- Large number of small requests
- Heavy caching
- Geographic distribution

### Locust usage
Locust can be used with distributed workers and optimized HTTP clients to generate high request volume. However, for extremely high-scale, low-latency systems, you may need many workers and careful client tuning.

### Why Locust fits well
Ad-tech workflows are usually API/request-driven, not browser-rendering driven.

---

## H. IoT and telemetry ingestion

### Common Locust tests
- Device heartbeat
- Telemetry batch upload
- Firmware status reporting
- Command acknowledgment
- Device registration
- Alert ingestion

### Example scenario
An IoT platform expects 100,000 devices sending telemetry every 30 seconds. The team simulates a subset of devices to test ingestion endpoints.

### Protocol possibilities
- HTTP/JSON ingestion: easy with Locust
- MQTT: possible with custom Python client
- AMQP/Kafka: possible with custom producer code
- CoAP/TCP: possible but more custom work

### Why Locust fits well
If the IoT platform exposes HTTP APIs for ingestion or management, Locust is straightforward. For non-HTTP protocols, it can still be used but requires more engineering.

---

## I. Gaming backends

### Common Locust tests
- Login/authentication
- Matchmaking queue
- Leaderboard retrieval
- Inventory updates
- Purchase validation
- Session heartbeat
- Chat/message APIs
- Player stats

### Example scenario
A mobile game launches a new event. The team simulates players logging in, entering matchmaking, and updating leaderboards.

### Bottlenecks to find
- Auth service saturation
- Matchmaking queue delay
- Redis leaderboard contention
- Database writes for player stats
- Anti-cheat validation latency

### Why Locust fits well
Game backends are often API-driven, and player behavior can be modeled as weighted Python tasks.

---

## J. Government and seasonal public systems

### Common Locust tests
- Tax filing
- Benefit application
- Permit submission
- Appointment booking
- Document upload
- Eligibility check
- Payment processing

### Example scenario
A tax agency expects a deadline-day surge. The system normally handles 200 users per minute but may need to handle 5,000 users per minute near the deadline.

### Goals
- Validate autoscaling
- Test document upload capacity
- Confirm payment gateway integration
- Ensure session management does not collapse
- Verify queueing behavior for overloaded users

### Why Locust fits well
Seasonal systems often have very predictable but extreme spikes, making Locust useful for simulated peak-load validation.

---

# 4. Technical scenarios where Locust shines

## A. Complex authentication flows

Locust is strong when authentication is not just a static header.

Examples:

- OAuth2 authorization code flow
- Client credentials flow
- JWT access token refresh
- SSO login
- Cookie-based sessions
- CSRF tokens
- API key rotation
- mTLS certificates
- Signed requests
- HMAC authentication

### Real scenario
A user must:

1. Log in
2. Receive a session cookie
3. Fetch a CSRF token
4. Submit a form
5. Refresh token after expiry
6. Call protected APIs

Locust can maintain state per simulated user and automate this flow.

---

## B. Data-dependent user journeys

Many real applications require extracting values from one response and using them in the next request.

Example:

```python
@task
def create_and_view_order(self):
    create = self.client.post("/orders", json={"item": "widget"})
    order_id = create.json()["id"]

    self.client.get(f"/orders/{order_id}")
```

This is common in:

- Checkout flows
- Ticket booking
- Document workflows
- Approval processes
- Profile creation
- Resource provisioning

Locust is well suited because it is programmable.

---

## C. Weighted user behavior

Real users do not behave uniformly.

Example distribution:

- 60% browse products
- 25% search
- 10% add to cart
- 4% start checkout
- 1% complete purchase

Locust supports task weights, making it easy to represent realistic traffic mix.

---

## D. Custom payload generation

Locust can generate dynamic payloads using Python libraries.

Examples:

- Random names, addresses, credit-card-like test numbers
- UUIDs
- Timestamps
- Geolocation data
- JSON schema validation
- Protobuf messages
- XML documents
- CSV-based data driving
- Database-generated test data

This is useful when static payloads would unrealistically hit caches.

---

## E. GraphQL testing

Locust can test GraphQL APIs by sending POST requests with query documents.

Example:

```python
query = {
    "query": """
    query GetProduct($id: ID!) {
      product(id: $id) {
        id
        name
        price
      }
    }
    """,
    "variables": {"id": "123"}
}

self.client.post("/graphql", json=query)
```

### Real scenario
Test complex nested queries, aliases, pagination, fragments, and resolver-heavy operations.

### Why it matters
GraphQL systems can have unpredictable performance because clients control query shape. Locust can simulate many query variations.

---

## F. REST API contract and performance validation

Locust can validate not only speed but also response correctness.

Examples:

- Expected HTTP status
- Response schema
- Required fields
- Pagination behavior
- Error format
- Header presence
- Correlation ID propagation

### Real scenario
A team wants to confirm that under load, the API still returns valid JSON and does not start returning HTML error pages or malformed responses.

---

## G. File upload and download testing

Locust can test multipart uploads and downloads.

Examples:

- Profile image upload
- Invoice PDF upload
- Video asset upload
- Report download
- Bulk CSV export
- Document generation

### Real scenario
An accounting SaaS allows users to upload bank statements. The team tests 1 MB, 10 MB, and 50 MB files with 200 concurrent users.

### Things to measure
- Upload throughput
- Server memory usage
- Disk I/O
- Virus scanning delay
- Object storage latency
- Timeout behavior
- Multipart upload reassembly

### Caution
Large file tests can make the load generator itself a bottleneck. Use enough Locust workers and monitor client-side resources.

---

## H. Search and read-heavy workloads

Search endpoints are often expensive.

Examples:

- Product search
- Document search
- Log search
- Customer search
- geospatial search
- Faceted filtering

### Real scenario
An e-commerce site expects 3,000 searches per second during a promotion. Each search includes filters, sorting, pagination, and personalized ranking.

### Locust can simulate
- Simple keyword searches
- Filtered searches
- Empty-result searches
- Deep pagination
- Facet counting
- Autocomplete queries
- Spell-corrected queries

### Bottlenecks to find
- Elasticsearch/OpenSearch pressure
- Database LIKE queries
- Cache misses
- Ranking model latency
- Network serialization

---

## I. Write-heavy workloads

Write operations often reveal locking, transaction, and replication issues.

Examples:

- Create order
- Submit application
- Update profile
- Send message
- Record audit event
- Insert telemetry
- Process payment

### Real scenario
A ticketing platform expects fans to buy tickets at once. The system must handle many writes to inventory, orders, payments, and notifications.

### Locust can simulate
- Successful purchases
- Failed purchases due to sold-out inventory
- Duplicate submission attempts
- Retry storms
- Partial failures
- Compensating transactions

### Bottlenecks to find
- Row locks
- Deadlocks
- Transaction timeouts
- Message queue backlog
- Payment gateway throttling
- Database replication lag

---

## J. Asynchronous and queued workflows

Many systems accept a request quickly and process it later.

Examples:

- Report generation
- Video transcoding
- Email sending
- Invoice creation
- Bulk import
- Background analytics

### Real scenario
A user requests a monthly report. The API returns immediately with a job ID, and a worker generates the PDF.

### Locust can test
- Job submission rate
- Job status polling
- Completion latency
- Worker queue backlog
- Failure handling
- Retry behavior

### Example flow
1. POST `/reports`
2. Receive `job_id`
3. Poll GET `/reports/{job_id}/status`
4. Download completed file

This is difficult in simple URL-recording tools but easy in Locust.

---

## K. Webhook and callback testing

Locust can simulate external systems calling your endpoints.

Examples:

- Payment provider webhook
- Shipping carrier callback
- Identity provider callback
- Third-party approval callback
- Device status callback

### Real scenario
A payment gateway sends asynchronous webhook notifications. The team wants to test whether the webhook receiver can handle 500 callbacks per second.

Locust can POST realistic webhook payloads and validate:

- Signature verification
- Idempotent processing
- Duplicate delivery handling
- Retry acceptance
- Response time
- Error handling

---

## L. Multi-step business transactions

Some tests require long user journeys.

Example: insurance claim submission.

1. Login
2. Select policy
3. Start claim
4. Upload documents
5. Provide incident details
6. Submit claim
7. Receive acknowledgment
8. Check claim status
9. Respond to information request
10. Download decision letter

Locust can model this statefully using Python variables inside each user instance.

---

## M. Protocol-specific testing with custom code

Locust is HTTP-friendly, but it can be extended.

Possible custom scenarios:

- WebSocket messaging
- gRPC calls
- MQTT publish/subscribe
- AMQP/RabbitMQ messages
- Kafka production/consumption
- TCP binary protocols
- SOAP APIs
- GraphQL subscriptions
- Server-sent events

### Important caveat
For non-HTTP protocols, you may need to write custom instrumentation so Locust can report latency and failures correctly. It is possible, but not as turnkey as HTTP testing.

---

# 5. Scenarios where Locust is not the best choice

## A. Browser rendering performance

If you need to measure:

- Largest Contentful Paint
- Cumulative Layout Shift
- Time to Interactive
- JavaScript execution
- CSS rendering
- Image loading in a real browser
- Single-page app hydration

Locust is not the right primary tool.

Use instead:

- Playwright
- Puppeteer
- Selenium
- WebPageTest
- Lighthouse CI
- Real User Monitoring

However, Locust can still test the backend APIs behind the browser experience.

---

## B. Mobile app UI performance

If the goal is to measure:

- App startup time
- Animation smoothness
- Battery impact
- Local database performance
- Offline behavior
- Push notification rendering
- Touch responsiveness

Locust is not ideal.

But it is excellent for testing the mobile backend APIs.

---

## C. Very high-scale tests with limited hardware

Locust can scale using distributed workers, but each worker has limits due to Python and client-side resource consumption.

If you need to generate hundreds of thousands or millions of concurrent connections from very few machines, you may need:

- More distributed workers
- Lower-level tools
- Cloud-managed load testing services
- Optimized Go-based tools such as k6 in some scenarios
- Specialized protocol testers

That said, Locust can still reach high throughput when properly distributed and tuned.

---

## D. Teams that prefer no-code test design

Locust requires Python. If the performance team does not know Python or cannot maintain code, tools with graphical interfaces may be easier.

Examples:

- JMeter with GUI
- Gatling with recorder/community tools
- k6 with JavaScript
- Commercial no-code/low-code platforms

Locust is powerful precisely because it is code-based, but that also means it requires programming skill.

---

# 6. Example real-world Locust test plans

## Scenario 1: New REST API launch readiness

### Business context
A company is launching a customer onboarding API.

### Performance goals
- Support 1,000 concurrent users
- Maintain p95 latency below 400 ms
- Error rate below 0.1%
- Sustain load for 30 minutes

### Locust user journey
1. Create organization
2. Create admin user
3. Login
4. Invite members
5. Upload compliance document
6. Check onboarding status

### Test design
- Use 10 Locust workers
- Ramp from 0 to 1,000 users over 5 minutes
- Hold for 30 minutes
- Use unique synthetic data per user
- Validate response bodies, not just HTTP 200

### Success criteria
- No sustained error spike
- Database connections remain healthy
- Document upload service does not queue excessively
- Auth token refresh works under load

---

## Scenario 2: Flash-sale spike test

### Business context
A retailer expects a sudden traffic surge from an email campaign.

### Performance goals
- Handle 10x normal traffic for 15 minutes
- Preserve checkout availability
- Avoid cascade failure

### Locust user journey
- 80% browse
- 15% search
- 4% add to cart
- 1% checkout

### Test design
- Start at normal load: 500 users
- Spike to 5,000 users in 60 seconds
- Hold for 15 minutes
- Gradually reduce load

### Observability
- Autoscaling events
- Cache hit ratio
- Database CPU
- Queue depth
- Payment gateway latency
- Error budgets

---

## Scenario 3: Overnight soak test

### Business context
A SaaS platform has reported slow degradation after several hours of use.

### Performance goals
- Run 12 hours at 60% expected peak
- Detect memory leaks and connection leaks
- Keep p95 latency stable

### Locust user journey
- Login
- Open dashboard
- Run report
- Export CSV
- Logout
- Repeat with randomized timing

### Test design
- 2,000 virtual users
- Constant load
- 12-hour duration
- Headless mode
- Distributed workers

### Monitoring
- Application heap usage
- GC pauses
- DB connection count
- Thread count
- File descriptors
- Pod restarts
- Latency trend over time

---

## Scenario 4: Microservice bottleneck isolation

### Business context
The end-to-end checkout flow is slow, but the team does not know which service is responsible.

### Approach
Test each service independently:

- Cart service
- Pricing service
- Inventory service
- Promotion service
- Tax service
- Payment service
- Order service
- Notification service

### Locust design
Create separate Locust scripts that call each microservice directly using test fixtures.

### Goal
Identify which service has:

- Highest latency
- Lowest throughput
- Most errors under load
- Strongest dependency on database or cache

This is often more effective than only testing the full UI flow.

---

## Scenario 5: Third-party dependency resilience

### Business context
An application depends on an external address-validation API.

### Problem
The external API may become slow or unavailable.

### Locust approach
Point the application under test to a mock address-validation service that can simulate:

- Normal latency
- Slow responses
- Timeouts
- HTTP 500 errors
- Rate limiting
- Partial outages

Then run Locust against the main application and observe whether it degrades gracefully.

### What you learn
- Does the app timeout correctly?
- Does it retry too aggressively?
- Does it fall back to cached data?
- Does one slow dependency consume all threads?
- Do circuit breakers work?

---

# 7. How to model real users in Locust

A good Locust test usually includes:

## 1. User personas

Example:

```python
class BrowserUser(HttpUser):
    ...

class PowerUser(HttpUser):
    ...

class APIClientUser(HttpUser):
    ...
```

## 2. Task weights

Example:

```python
@task(7)
def browse(self):
    ...

@task(2)
def search(self):
    ...

@task(1)
def checkout(self):
    ...
```

## 3. Think time

Example:

```python
wait_time = between(1, 5)
```

This prevents unrealistic back-to-back request patterns.

## 4. Data parameterization

Use:

- CSV files
- databases
- random generators
- faker libraries
- tenant IDs
- API keys
- product IDs
- session tokens

## 5. Validation

Do not only check HTTP status. Validate:

- JSON structure
- business fields
- error messages
- redirects
- headers
- response time
- idempotency behavior

---

# 8. Where Locust fits in a performance-testing stack

Locust is usually one part of a larger observability setup.

A typical stack:

| Layer | Tool examples |
|---|---|
| Load generation | Locust |
| Application metrics | Prometheus, Datadog, New Relic, Dynatrace |
| Logs | ELK, Loki, Splunk |
| Tracing | OpenTelemetry, Jaeger, Zipkin |
| Infrastructure metrics | CloudWatch, Azure Monitor, Stackdriver, node_exporter |
| Database metrics | pg_stat_statements, MySQL performance schema, APM |
| Dashboards | Grafana, Kibana, vendor dashboards |
| CI/CD | GitHub Actions, GitLab CI, Jenkins, Argo CD |

Locust tells you what the clients experienced. Infrastructure and APM tools tell you why.

---

# 9. Practical tips for using Locust in real tests

## Use headless mode for serious tests

For CI or large tests, avoid relying on the web UI as the control center.

Example:

```bash
locust -f test.py --headless -u 1000 -r 100 -t 30m --host https://target.example.com
```

## Distribute workers for high load

Use multiple machines:

```bash
locust -f test.py --master
```

```bash
locust -f test.py --worker --master-host=10.0.0.5
```

## Monitor the load generators themselves

Locust workers can become the bottleneck. Watch:

- CPU
- Memory
- Network bandwidth
- File descriptors
- Python process limits
- TLS handshake overhead

## Use FastHttpUser when appropriate

Locust provides a faster HTTP user implementation for higher throughput. It is useful when standard `HttpUser` becomes client-limited.

## Avoid unrealistic caching

If every virtual user requests the exact same product ID, you may mostly test cache performance. Real tests should vary data appropriately.

## Use synthetic data safely

Do not use production personal data unless explicitly approved. Prefer generated or anonymized data.

## Be careful in production

Load testing production can cause incidents. Use:

- Feature flags
- Test accounts
- Rate limits
- Alerts
- Rollback plans
- Approval workflows
- Non-destructive endpoints where possible

## Combine with server-side monitoring

A Locust test showing “p95 = 2 seconds” is not enough. You need to know whether the delay comes from:

- Network
- Load balancer
- Application code
- Database
- Cache
- External API
- Queue
- Disk
- CPU
- Memory pressure

---

# 10. Decision guide: should you use Locust?

Use Locust if:

- Your system exposes HTTP APIs or web backends
- Your team knows Python
- You need flexible, code-based test scenarios
- You want to model complex user journeys
- You need distributed load testing
- You want to integrate tests into CI/CD
- You need to test authentication, dynamic payloads, and data dependencies
- You are comfortable writing custom extensions for non-HTTP protocols

Consider another tool if:

- You need real browser rendering metrics
- You need mobile UI performance testing
- You want a fully no-code experience
- You need extreme concurrency from minimal hardware
- Your tests are mostly simple recorded HTTP requests and you prefer a GUI
- You require highly specialized protocol testing out of the box

---

# Bottom line

Locust is useful for many real performance and load-testing scenarios, especially:

- API load testing
- Microservice performance testing
- Stress testing
- Spike testing
- Soak testing
- Capacity planning
- Performance regression testing
- Complex authentication flows
- Stateful business transactions
- GraphQL and REST workloads
- Asynchronous job workflows
- Webhook and callback testing
- Custom protocol testing with Python extensions

It is particularly valuable when performance tests need to be **maintained as code**, integrated into engineering workflows, and adapted to complex real-world user behavior.

However, it is not the right tool for everything. For browser rendering, mobile UI behavior, or extremely low-level protocol performance, Locust should be complemented by other tools.