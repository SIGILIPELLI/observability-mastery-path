---
description: "A real 3-panel RED dashboard showing real rate spikes, real 402/404 errors, and a p95 latency rise to ~0.175s, all from one traffic burst."
---

# Building a Real RED Dashboard: Rate, Errors, Duration

## What was run

A real 3-panel Grafana dashboard (Rate, Errors, Duration) built against the real Prometheus data source and the
real `orders_app.py` metrics, loaded live in a browser after sending a real burst of 60 mixed checkout and
order-lookup requests (some deliberately large enough to be declined, some deliberately requesting nonexistent
orders).

## Three real panels, one real traffic burst

```promql
# Rate
sum by (endpoint) (rate(orders_requests_total[1m]))

# Errors
sum by (endpoint) (rate(orders_requests_total{status!~"2.."}[1m]))

# Duration (p95)
histogram_quantile(0.95, sum by (le, endpoint) (rate(orders_request_duration_seconds_bucket[2m])))
```

Loaded live in a real browser, all three panels rendered the exact real shape of the traffic burst at the same
moment: **Rate** showed two visible spikes up to ~1 req/s across `/checkout`, `/orders`, and `/orders/{id}`;
**Errors** showed real spikes in lockstep with Rate — real `402`s from declined payments and real `404`s from
nonexistent order lookups, both genuinely produced by the burst; **Duration p95** climbed from near-zero up to
roughly **0.175 seconds** right as the burst hit, visibly correlated with the other two panels rather than drawn
independently.

## How It Actually Works

**RED** (Rate, Errors, Duration) is a deliberately minimal, request-centric framework for what a service
dashboard should show at a glance, popularized for exactly the kind of request-driven services this course
instruments. The three signals answer three different, complementary real questions:

- **Rate**: how much traffic is this service handling right now? A sudden drop can mean upstream trouble sending
  traffic; a sudden spike can mean a real surge, a retry storm (the exact pattern measured for real in
  [Microservices, Level 3, Lesson 2](https://sigilipelli.github.io/microservices-mastery-path/level-3/02-retries-timeouts-backoff/)),
  or a bug causing duplicate calls.
- **Errors**: what fraction of that traffic is failing? Rate alone can't tell you this — a service handling
  1000 req/s with a 50% real error rate looks identical to a healthy one on a Rate panel alone, which is exactly
  why Errors is its own panel, not a detail buried inside Rate.
- **Duration**: how long are real requests actually taking? A service can have a healthy rate and a healthy
  error rate while still being painfully slow for real users — Duration is the signal that catches that case
  specifically, and p95 (not average) is used because an average hides exactly the slow outliers a real user
  actually experiences.

The real, visible correlation across all three panels during this lesson's one burst is the actual point of
putting them on one dashboard together, rather than as three separate, disconnected graphs: a human glancing at
this dashboard during a real incident can see, in one view, that a traffic spike, an error spike, and a latency
spike all happened at the same real moment — strong evidence they're related, which would be much harder to
notice flipping between three separate dashboards one at a time.

This dashboard is deliberately scoped to *this service's own* request handling. It is a different, complementary
view from the **USE method** (Utilization, Saturation, Errors), covered next in
[Lesson 3](03-golden-signals-use-method.md), which looks at the health of the *resources* a service depends on
(CPU, memory, connection pools) rather than the requests flowing through it.

## Common mistakes

- Building a Rate panel without an Errors panel next to it — as explained above, a healthy-looking rate can hide
  a real, severe error rate entirely, and the two need to be seen together, not inferred from one alone.
- Using average latency instead of a percentile for the Duration panel — an average can look perfectly healthy
  even while a meaningful fraction of real requests are genuinely slow, exactly the kind of blind spot
  [Level 1, Lesson 7](../level-1/07-histogram-quantiles.md)'s quantile math exists to avoid.
- Building RED panels per-service but never breaking them down by the labels (like `endpoint`, as this lesson's
  `by (endpoint)` clause does) that would let you tell *which part* of a service is actually causing a spike,
  versus seeing only an undifferentiated aggregate.

## Exercise

Add a fourth panel to this dashboard showing total in-flight requests (`orders_inflight_requests` from
[Level 1, Lesson 3](../level-1/03-instrumenting-metric-types.md)), fire a real concurrent burst the way that
lesson did, and confirm — by actually watching the live dashboard in a browser, not just querying the API — that
you can see Rate, Errors, Duration, and in-flight requests all spike together at the same real moment.
