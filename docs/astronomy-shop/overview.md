# The Astronomy Shop

> **Provenance.** This analysis describes the application source at fork commit [`85c2cb25`](https://github.com/samimassoud/opentelemetry-demo/commit/85c2cb253ae3519ef2b3a02984c32614106de5ea) (`samimassoud/opentelemetry-demo`, based on upstream release `3.1.0` of [`open-telemetry/opentelemetry-demo`](https://github.com/open-telemetry/opentelemetry-demo)). Paths are relative to the application root: `opentelemetry-demo/` in the current workspace, `app/astronomy-shop/` once the source is imported into this repository.

The component-level detail is in [service-map.md](service-map.md).

## What it is, from a customer's point of view

The Astronomy Shop is an online store for telescopes and astronomy gear: telescopes, binoculars, solar filters, an imaging camera, a lens cleaning kit, a red flashlight and a book about comets. The catalog holds 10 products.

A visitor can:

- **Browse** the home page and open product pages.
- **See recommendations** ("you may also like") and **ads** on the page.
- **Switch currency**: prices are converted on the fly.
- **Manage a cart**: add items and view the cart. The cart is kept for about an hour.
- **Check out**: enter an address, an email and a card, get a shipping quote, and place the order.
- **Get a confirmation**: an order summary page and a confirmation email. The email is generated but never actually sent.

There are no user accounts. The browser keeps a session id, and the cart is tied to it.

## One purchase, behind the scenes

When the visitor clicks **Place order**:

1. **The request enters through the front door.** Every browser request reaches a single proxy (`frontend-proxy`). The proxy forwards it to the web app (`frontend`).
2. **The web app hands the order to the checkout service**, which runs the rest of the steps in order:
   1. **Reads the cart** from the cart service, which stores carts in Redis-compatible memory.
   2. **Looks up every product** in the product catalog, which reads from PostgreSQL, and **converts prices** into the visitor's currency.
   3. **Gets a shipping quote.** The shipping service asks the quote service for a price, and the result is converted to the visitor's currency.
   4. **Charges the card** through the payment service. *If this fails, the order fails.*
   5. **Ships the order**: shipping returns a tracking id. *If this fails, the order fails.*
   6. **Empties the cart.** If this fails, the error is ignored and the order still succeeds.
   7. **Sends the confirmation email** through the email service. This step is best-effort: there's a 1-second timeout, and a failure is only logged.
3. **The order is published as an event.** Checkout writes the finished order to a Kafka topic called `orders`.
4. **Two background workers react later**, outside the visitor's request:
   - **accounting** records the order in PostgreSQL.
   - **fraud-detection** reads the order and logs it.
5. **The visitor sees the confirmation page.** The web app fetches the product details once more to render it.

Meanwhile a **load generator** plays fake customers around the clock: it browses, adds to cart and checks out. That keeps the shop busy even when no human is using it.

## Why the project exists

The OpenTelemetry project builds and maintains the shop as its **reference application**. The idea is to show what good telemetry looks like across a realistic, polyglot system:

- **13 shop services in 12 languages**: Go, Java, Kotlin, C#, C++, Ruby, Rust, PHP, Python, JavaScript (Node.js), TypeScript (Next.js), Elixir. Each one is instrumented with OpenTelemetry, some by hand and some automatically.
- **A built-in telemetry pipeline**: one OpenTelemetry Collector that fans out to Jaeger for traces, Prometheus for metrics, OpenSearch for logs, and Grafana for dashboards.
- **Built-in failure scenarios**: 20 feature flags that break things on purpose. Examples: payment fails some percentage of the time, a memory leak, a slow database lock, a dead Kafka consumer.
- **Built-in traffic** from the load generator.

## Why it's a good workload for this platform lab

- **It emits meaningful telemetry with no work from us.** Traces, metrics and logs with consistent service names and a `service.criticality` label already exist. That's the raw material for SLOs and alerts.
- **It has a real critical user journey.** "Checkout succeeds, and quickly" is a natural SLO, and it spans 8+ services.
- **It has real stateful dependencies to externalise.** A Redis-compatible cache, PostgreSQL and Kafka map directly onto Azure-native services:
  - **Azure Managed Redis** (≈ Amazon ElastiCache)
  - **Azure Database for PostgreSQL Flexible Server** (≈ Amazon RDS for PostgreSQL)
  - **Azure Event Hubs** (≈ Amazon MSK, through its Kafka-compatible endpoint)
  - Each one forces a decision about identity, private networking and configuration. See [Service map · Migration decisions](service-map.md#migration-decisions).
- **It has one entry point.** Everything a browser needs goes through `frontend-proxy`, so a single Gateway API route in front of it covers the whole shop.
- **Failure on demand.** Flipping a flag produces a known, repeatable incident. That lets us prove an alert fires and an SLO burns, then turns green again.
- **It has a cheap change to ship.** The web frontend is one container image. A visible text change makes a clean test for the CD pipeline, its rollout and a `git revert` rollback.
- **It's polyglot on purpose.** A dozen runtimes stress the platform's image build, scanning and auto-instrumentation paths in ways a single-language app wouldn't.
