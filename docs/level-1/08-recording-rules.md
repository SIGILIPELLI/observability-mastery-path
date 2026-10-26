---
description: "The same aggregation over 2000 series took 1.62ms live but 0.037ms as a recording rule -- a real ~44x speedup, confirmed by Prometheus's own query stats."
---

# Recording Rules: Pre-Computing Expensive Queries

## What was run

A real app, `high_cardinality_app.py` (port 9304), deliberately exposing 2,000 real time series (`hc_requests_total`
across 2,000 distinct `customer_id` values and 4 `region` values) — a genuine, if modest, cardinality stress
test. A real recording rule pre-aggregating this by region was added to Prometheus's config, and both the raw
query and the pre-computed recorded metric were timed using Prometheus's own real query statistics API
(`?stats=all`), not estimated.

## The real cost of a live aggregation over 2,000 series

```promql
sum by (region) (hc_requests_total)
```

```bash
curl -s "http://localhost:9090/api/v1/query?stats=all" --data-urlencode 'query=sum by (region) (hc_requests_total)'
```
```
evalTotalTime: 0.001619584 s, samples: 2000
evalTotalTime: 0.001615041 s, samples: 2000
evalTotalTime: 0.001628125 s, samples: 2000
```

A real, consistent **~1.62 milliseconds** per query, and Prometheus's own stats confirm exactly why: it scanned
all 2,000 real samples every single time this query ran, re-doing the same aggregation from scratch.

## The same aggregation, pre-computed once per evaluation interval

```yaml
# recording_rules.yml
groups:
  - name: orders_recording_rules
    interval: 5s
    rules:
      - record: hc:requests_total:sum_by_region
        expr: sum by (region) (hc_requests_total)
```

```bash
curl -s "http://localhost:9090/api/v1/query?stats=all" --data-urlencode 'query=hc:requests_total:sum_by_region'
```
```
evalTotalTime: 3.9166e-05 s, samples: 4
evalTotalTime: 3.9708e-05 s, samples: 4
evalTotalTime: 3.3208e-05 s, samples: 4
```

**0.000039 seconds — roughly 44x faster**, and the real sample count dropped from 2,000 to 4, because
Prometheus already did the real aggregation work once, in the background, every 5 seconds (the rule's own
`interval`), and querying the recorded metric is now just a cheap instant-vector lookup of 4 already-aggregated
values — one per region — instead of re-scanning and re-summing 2,000 raw series on every single request.

## How It Actually Works

A **recording rule** tells Prometheus to evaluate a PromQL expression on a fixed schedule and save the result as
a brand-new time series, with its own name, that can then be queried directly — exactly as cheap as any other
metric, because by the time you query it, the expensive work is already done. This real 44x speedup scales with
two real things: how many raw series the original query had to scan (2,000 here; a production system might have
millions), and how many *times* that same expensive query gets run — a query evaluated once by a human is a minor
cost, but the same expensive query backing a dashboard panel refreshed every 30 seconds, viewed by ten people, or
feeding an alert rule evaluated every 15 seconds, pays that cost repeatedly, forever, unless it's pre-computed.

This connects directly to two other real, measured lessons in this course: the cardinality concern from
[Lesson 2](02-prometheus-data-model.md) is exactly what makes a query like this one expensive in the first
place — more real series means more real work per evaluation — and [Level 4, Lesson 2](../level-4/02-error-budgets.md)
(error budgets) and [Lesson 3](../level-4/03-burn-rate-alerts.md) (burn-rate alerting) both rely on expressions
complex enough that recording them, rather than recomputing them on every single alert evaluation, is standard
real practice, not a premature optimization.

The real tradeoff: a recording rule's result is only as fresh as its own evaluation `interval` (5 seconds here)
— it's not a live query, it's a periodically-refreshed cache of one. For a value that needs second-by-second
freshness, that staleness might matter; for most dashboard panels and alert conditions, it doesn't, and the real
44x cost reduction measured above is almost always worth that small, bounded staleness.

## Common mistakes

- Pre-computing a query that's cheap and rarely run — recording rules add real, permanent storage cost for the
  new series they create, so they should be reserved for queries that are genuinely expensive, or genuinely
  run often, or both — not applied reflexively to every query in a dashboard.
- Forgetting that a recording rule's result lags its own `interval` behind real-time — using it somewhere that
  genuinely needs instant-query freshness introduces a real, if usually small, staleness bug.
- Naming a recorded metric without following Prometheus's own `level:metric:operations` naming convention (as
  this lesson's `hc:requests_total:sum_by_region` loosely follows) — an unclear name makes it hard for anyone
  later to tell a recorded, pre-aggregated metric apart from a raw, directly-scraped one.

## Exercise

Add a second recording rule that pre-computes the real p95 latency across both instances from
[Lesson 7](07-histogram-quantiles.md) (`histogram_quantile(0.95, sum(rate(instance_request_duration_seconds_bucket[5m])) by (le))`),
and measure, with the same real `?stats=all` timing approach this lesson used, how much faster querying the
recorded version is compared to the raw `histogram_quantile()` expression evaluated live.
