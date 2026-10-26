---
description: "A real counter that visibly resets after a restart, and a real gauge whose scraped history shows all zeros despite a real spike of 15 in-flight requests."
---

# Instrumenting an App: Counters, Gauges, Histograms & Summaries

## What was run

The real `orders_app.py` from [Lesson 1](01-what-observability-means.md), using all four core Prometheus client
types: `Counter`, `Gauge`, `Histogram`, and `Summary`. The app was restarted for real to observe a counter's real
behavior across a restart, and a real concurrent burst of 15 requests was fired to probe a gauge's real blind
spot against Prometheus's actual 5-second scrape interval.

## Counters: real behavior across a real restart

```python
REQUEST_COUNT = Counter("orders_requests_total", "Total requests", ["endpoint", "status"])
REQUEST_COUNT.labels(endpoint="/orders", status="200").inc()
```

```bash
# right after a real restart, the metric doesn't even exist yet:
curl -sL localhost:9301/metrics | grep "^orders_requests_total"
# (no output)

# after 3 real requests:
curl -sL localhost:9301/metrics | grep "^orders_requests_total"
# orders_requests_total{endpoint="/orders",status="200"} 3.0
```

A real, restarted process's counter does not resume from whatever value it held before — it starts over, because
`prometheus_client`'s `Counter` object is held only in that process's memory, and a `Counter` metric doesn't even
appear in `/metrics` output until it's been incremented at least once in the current process's lifetime. This is
precisely the behavior PromQL's `rate()` and `increase()` functions are built to handle correctly (covered with
its own real demonstration in [Lesson 6](06-promql-aggregation-rate.md)) — they detect a value that dropped
between two samples and treat it as a reset rather than a negative rate, which is why counters are safe to use
across restarts even though their raw value isn't.

## Gauges: a real, invisible blind spot

```python
INFLIGHT = Gauge("orders_inflight_requests", "Requests currently being processed")
```

```bash
# a real burst of 15 concurrent requests, completed in 0.855s:
# Prometheus's own scraped history for the same time window:
curl -s ".../api/v1/query_range?query=orders_inflight_requests&..."
#   0, 0, 0, 0, 0, 0, 0
```

**Every single scraped data point is `0`**, despite a real, concurrent spike where `INFLIGHT` genuinely held a
value as high as 7 (confirmed separately with a direct, manual scrape taken *during* the burst, bypassing
Prometheus's own 5-second schedule). Prometheus's periodic scrape simply never happened to land during the
roughly one-second window the spike was real — the gauge's value was correct the entire time, but Prometheus's
*sampling* of it was too infrequent to catch it. A gauge only tells you the value Prometheus last observed; it
makes no promise about peaks that happened entirely between two scrapes.

## How It Actually Works

Each Prometheus metric type encodes a real, specific promise about how its value behaves, and picking the wrong
type for what you're measuring produces real, observable problems:

- **Counter**: a value that only ever increases (or resets to zero on restart). Correct for "how many times has X
  happened" — request counts, error counts, bytes sent. Never use it for something that can legitimately decrease
  (queue depth, active connections) — `prometheus_client`'s `Counter` doesn't even expose a way to decrement it,
  by design.
- **Gauge**: a value that can go up or down, representing a snapshot at the moment of each scrape — exactly what
  this lesson's blind spot demonstrates the real limit of. Correct for "what is X right now" — memory usage, queue
  depth, in-flight requests — but only ever as accurate as your scrape interval allows; a gauge cannot tell you
  about anything that happened and fully resolved between two scrapes.
- **Histogram**: buckets observations into predefined ranges (the `le="0.05"`, `le="0.1"` buckets seen in
  Lesson 1's raw output) and exposes `_bucket`, `_sum`, and `_count` series. Aggregatable across many instances —
  covered in depth, with a real quantile-math demonstration, in [Lesson 7](07-histogram-quantiles.md).
- **Summary**: computes quantiles (like p99) *client-side*, inside the application process, before they're ever
  scraped. This real `orders_request_duration_summary_seconds` metric looks similar to the histogram above, but its
  quantiles **cannot be meaningfully aggregated** across multiple instances — each instance's p99 is already
  computed from only that instance's own requests, and averaging several instances' p99s doesn't produce a
  correct overall p99 (the same mathematical problem [Lesson 7](07-histogram-quantiles.md) demonstrates for naively
  averaged histograms). This is the real, practical reason histograms are the more common choice for latency in
  multi-instance systems, despite needing a PromQL function (`histogram_quantile()`) to extract a quantile after
  the fact rather than getting one directly.

## Common mistakes

- Using a `Gauge` to count something that only ever increases, losing the restart-safe, rate()-friendly behavior a
  `Counter` provides for free.
- Trusting a `Gauge`'s scraped history as a complete record of a value's real behavior over time — as shown above,
  it is a sampled snapshot, not a continuous recording, and real spikes shorter than your scrape interval can be
  completely invisible to it.
- Reaching for a `Summary` because its quantiles look simpler to get directly, without realizing that choice
  blocks you from ever computing a correct cross-instance quantile later — this is a design decision that's
  expensive to change after the metric is already in production and being relied on.

## Exercise

Lower Prometheus's `scrape_interval` in `prometheus.yml` from `5s` to `1s`, restart Prometheus, and re-run this
lesson's exact 15-request burst. Query `orders_inflight_requests` over that time window again and report whether
the real spike is now visible in Prometheus's own scraped history — and if it's still invisible (or only partially
visible), explain, from real numbers, why.
