---
description: "Naively averaging two real instances' p95s gives 0.509s; the correct cross-instance p95 computed from real summed buckets is 0.9415s -- nearly double."
---

# Histogram Quantiles: Why You Can't Average a Histogram

## What was run

Two real instances of the same app, `orders_app_variable.py`, deliberately given different real latency profiles
via an environment variable: a fast instance (port 9302, real delays of 10-30ms) and a slow instance (port 9303,
real delays of 400-900ms). Both were scraped by the real Prometheus server, sent 40 real requests each, and
queried three different ways with real `histogram_quantile()` calls.

## Two real, very different instances

```bash
curl -s localhost:9302/work   # {"profile":"fast","delay":0.0168...}
curl -s localhost:9303/work   # {"profile":"slow","delay":0.5034...}
```

## Three real p95 calculations, two of them wrong

```bash
# fast instance's own p95:
curl -s ".../api/v1/query" --data-urlencode \
  'query=histogram_quantile(0.95, sum(rate(instance_request_duration_seconds_bucket{instance="localhost:9302"}[2m])) by (le))'
# 0.046027...

# slow instance's own p95:
curl -s ".../api/v1/query" --data-urlencode \
  'query=histogram_quantile(0.95, sum(rate(instance_request_duration_seconds_bucket{instance="localhost:9303"}[2m])) by (le))'
# 0.972079...
```

The naive, tempting shortcut: just average these two real numbers — `(0.046 + 0.972) / 2 ≈ 0.509`.

```bash
# the CORRECT aggregate p95, computed from the summed buckets of BOTH instances together:
curl -s ".../api/v1/query" --data-urlencode \
  'query=histogram_quantile(0.95, sum(rate(instance_request_duration_seconds_bucket[2m])) by (le))'
# 0.941513...
```

**0.509 (the naive average) versus 0.9415 (the real, correct cross-instance p95) — nearly double.** With equal
real traffic split between a fast and a slow instance, the true 95th-percentile latency across *all* requests
falls almost entirely within the slow instance's own distribution — makes sense once you think about it: the slow
half of all requests alone already accounts for 50% of total traffic, so the top 5% (the p95 cutoff) is entirely
inside that slow half, not straddling the midpoint the naive average implies.

## How It Actually Works

A percentile (p95, p99) is a statement about a specific, full distribution of individual values — it genuinely
cannot be correctly derived from two *other* percentiles computed from two different sub-distributions, no matter
how you combine them, because a percentile throws away exactly the shape information needed to recombine it
correctly. Averaging two p95s assumes each contributes "half" to the combined p95, which is only true by
coincidence, not by any real mathematical relationship.

What actually *can* be combined correctly across instances are the raw **histogram buckets** this course's
`Histogram` metric (from [Lesson 3](03-instrumenting-metric-types.md)) exposes — the `_bucket{le="..."}` counts
for each boundary. Each bucket is a real count of observations falling at or below that boundary, and counts from
different instances genuinely can be summed, bucket by bucket, because a count is additive in a way a percentile
is not. `sum(rate(..._bucket[2m])) by (le)` does exactly this: it adds up each instance's real rate of
observations in each bucket, producing one combined histogram as if every instance's requests had been served by
one combined service. `histogram_quantile()` then estimates the percentile from *that* correctly-combined
histogram — which is why the real result above matches reality, while the naive average of two separately-computed
percentiles does not.

This is also the concrete, mathematical reason [Lesson 3](03-instrumenting-metric-types.md) called `Summary`'s
per-instance quantiles non-aggregatable: even where a Summary *does* expose quantiles (Python's
`prometheus_client` actually exposes none at all — a real, separately-confirmed finding, only `_sum` and `_count`
— but other language clients, like Go's, do), those quantiles are computed the same unrecoverable way this
lesson's naive average was: from each instance's own data alone, with no raw buckets left to correctly recombine
after the fact.

## Common mistakes

- Averaging percentiles from multiple instances (or multiple time windows) as a "close enough" approximation —
  the real 0.509-vs-0.9415 gap above shows this can be wrong by a very large, practically significant margin, not
  just slightly off.
- Choosing histogram bucket boundaries that don't actually span your real latency distribution — a histogram with
  no bucket boundary anywhere near your real p95 produces an imprecise `histogram_quantile()` estimate even when
  aggregated correctly, because the function interpolates between the two buckets your real value actually falls
  between.
- Reaching for `Summary` specifically because its API looks like it hands you a quantile "for free," without
  checking whether your actual client library implements quantiles at all, or whether you'll ever need to combine
  that quantile across more than one instance.

## Exercise

Add a third instance with yet another real latency profile (e.g., `LATENCY_PROFILE=medium`, delays of 0.1-0.2s)
to the scrape config, send it real traffic, and recompute the three-instance aggregate p95 using the same
bucket-summing approach. Then compute the naive average of all three instances' individually-computed p95s and
report how large the real gap is this time, with three instances instead of two.
