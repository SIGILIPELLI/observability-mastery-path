---
description: "One metric name produced 3 real, separate time series in Prometheus's own data, confirmed against its real /api/v1/series endpoint -- not 1, not a guess."
---

# The Prometheus Data Model: Time Series, Labels & Samples

## What was run

A real query against the real local Prometheus server's own `/api/v1/series` API, asking it directly: how many
distinct time series actually exist for the metric name `orders_requests_total`, after the real traffic sent in
the previous lesson.

## One metric name, three real time series

```bash
curl -s "http://localhost:9090/api/v1/series?match[]=orders_requests_total"
```
```json
[
  {"__name__": "orders_requests_total", "endpoint": "/orders",      "status": "200", ...},
  {"__name__": "orders_requests_total", "endpoint": "/orders/{id}", "status": "200", ...},
  {"__name__": "orders_requests_total", "endpoint": "/orders/{id}", "status": "404", ...}
]
```

This is Prometheus's own, real, authoritative answer: the single metric *name* `orders_requests_total` is backing
**three separate, independently-tracked time series**, because the `orders_app.py` code
(from [Lesson 1](01-what-observability-means.md)) incremented that counter with three different combinations of
the `endpoint` and `status` labels. Each unique combination of label values is its own time series, with its own
independent sequence of samples over time — not a sub-field of one shared series.

## How It Actually Works

A Prometheus time series is uniquely identified by its **metric name plus the full set of its label key-value
pairs**. `orders_requests_total{endpoint="/orders", status="200"}` and
`orders_requests_total{endpoint="/orders/{id}", status="404"}` are, to Prometheus's storage engine, two completely
separate series, tracked with completely separate sequences of **samples** (a timestamp paired with a numeric
value). This is why the code in Lesson 1 could increment one counter object in Python
(`REQUEST_COUNT.labels(endpoint=..., status=...).inc()`) and have it transparently create and maintain multiple
real, independent series underneath — the `prometheus_client` library's `labels()` call is choosing *which* series
to write a sample into, not modifying a single series' structure.

This has a direct, practical consequence that becomes important starting in
[Lesson 7 of Level 3](../level-3/07-cardinality-explosion.md): **every new, distinct combination of label values
creates a brand-new time series that Prometheus has to store and index, forever (until it's no longer scraped and
ages out).** A label with a handful of possible values (an HTTP status code, an endpoint path) multiplies a
metric's series count by a small, bounded factor. A label with effectively unbounded values — a raw user ID, a
full request path with embedded IDs, a timestamp — multiplies it by however many distinct values have ever been
seen, which can turn one metric into millions of real series with no natural bound. This single data-model fact
— cardinality is multiplicative across every label, and Prometheus pays a real, permanent cost per unique
combination — is the root cause of nearly every real Prometheus performance and cost problem this course will
cover.

The `__name__` label shown in the raw API response above is not a special case — it's just another label, by
convention the one PromQL's simpler syntax (`orders_requests_total{...}`) treats as the metric's "name," but
internally it's selected the exact same way as `endpoint` or `status`.

## Common mistakes

- Thinking of a metric name as "one thing being tracked" rather than "a family of potentially many independent
  series" — the real output above shows three separate series under one name, each with its own independent value
  and history.
- Assuming a label's possible values are naturally bounded without checking — `status` (a small, known set of
  HTTP codes) is a safe label; a label built from user input, free text, or an ID is not, and the real cost of
  getting this wrong is covered in depth in [Level 3, Lesson 7](../level-3/07-cardinality-explosion.md).
- Confusing a time series (a specific metric name + label combination, tracked over time) with a single sample (one
  timestamp-value pair within that series) — PromQL queries operate on both concepts, and conflating them makes
  aggregation functions (covered in [Lesson 6](06-promql-aggregation-rate.md)) much harder to reason about.

## Exercise

Add a new label, `method`, to `REQUEST_COUNT` in `orders_app.py` (tracking `"GET"` vs `"POST"`), re-deploy, generate
a few more real requests of each kind, and query `/api/v1/series?match[]=orders_requests_total` again. Count how
many real series now exist, and confirm the count matches exactly the number of distinct `(endpoint, status,
method)` combinations your real traffic has actually produced — not more, not fewer.
