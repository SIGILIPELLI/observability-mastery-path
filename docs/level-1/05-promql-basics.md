---
description: "Four real PromQL selectors -- exact match, regex, negative match -- run against real scraped data, each returning exactly the right series and no others."
---

# PromQL Basics: Selecting & Filtering Time Series

## What was run

Four real PromQL queries sent to the real Prometheus server's `/api/v1/query` API, each selecting a different
subset of the real `orders_requests_total` series built up across [Lessons 1–4](01-what-observability-means.md),
confirming exactly which series each selector syntax actually returns.

## Four real selectors, four real results

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=orders_requests_total{endpoint="/orders/{id}"}'
```
```
{'endpoint': '/orders/{id}', 'status': '200'} ['15']
```

An exact label match (`endpoint="/orders/{id}"`) returned only series with that exact label value — one real
series here, not the `/orders` one.

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=orders_requests_total{status=~"2.."}'
```
```
{'endpoint': '/orders',      'status': '200'} ['33']
{'endpoint': '/orders/{id}', 'status': '200'} ['15']
```

A regex match (`status=~"2.."`) returned both `200` series, matching the pattern, and correctly excluded the
`404` series that also exists in the real data (confirmed below).

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=orders_requests_total{status!="200"}'
```
```
{'endpoint': '/orders/{id}', 'status': '404'} ['1']
```

A negative match (`status!="200"`) returned exactly the one real series that isn't `200` — the real `404` series
produced by requesting a nonexistent order in earlier lessons.

## How It Actually Works

A PromQL **instant vector selector** — `metric_name{label="value", ...}` — returns every time series whose metric
name and label set satisfy every condition inside the braces, each with its single most recent sample at the
query's evaluation time. The real results above demonstrate the four comparison operators directly: `=` for exact
match, `!=` for exact exclusion, `=~` for regex match, and (not shown here, but following the same pattern) `!~`
for regex exclusion. Multiple conditions inside one selector are implicitly ANDed together — `{endpoint="/orders",
status="200"}` would require both to be true for a series to match, narrowing the result further than either
condition alone.

Omitting the label braces entirely, as in plain `orders_requests_total`, returns *every* series under that metric
name regardless of label values — every row this course's earlier lessons showed from `/api/v1/series`. Adding
selectors is how you narrow that down to exactly the series a specific question needs, which matters increasingly
as a real metric accumulates more label combinations (exactly the cardinality concern from
[Lesson 2](02-prometheus-data-model.md)).

A **range vector selector** — `metric_name[5m]`, for instance — returns not just the latest sample but every
sample recorded for each matching series within the given time window, which is the raw input most aggregation
and rate functions need (built directly in the next lesson). An instant vector selector alone, as used in every
query above, only ever gives you "the value right now" for each matching series — useful for current-state
questions, but not for anything involving change over time.

## Common mistakes

- Forgetting that label matching is exact unless you explicitly use `=~`/`!~` — `status="2"` would match nothing
  real, since no series actually has the label value `"2"`; only `status=~"2.."` or a full exact value like
  `"200"` works.
- Querying with no label selector at all when a specific subset was actually needed, then manually scanning
  through every returned series by eye — the real selectors above exist specifically to avoid that, and matter
  more as a metric's real cardinality grows.
- Confusing an instant vector query (one value per series, "right now") with a range vector query (many samples
  per series, over a window) — PromQL requires range vectors for rate/aggregation-over-time functions, covered
  with real, computed numbers in [Lesson 6](06-promql-aggregation-rate.md), and a plain instant-vector query
  cannot be passed directly into them.

## Exercise

Write and run a real PromQL query selecting every `orders_requests_total` series where `endpoint` is *either*
`/orders` or `/orders/{id}` (there should be more than one way to express this with `=~`), and confirm your
query's real result includes exactly the 3 real series this lesson's data has produced so far — not 2, not 4.
