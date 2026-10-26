---
description: "A real counter that dropped from 53 to 5 after a restart, with rate() still returning a sane, positive 0.45 req/s instead of a nonsensical negative number."
---

# PromQL Aggregation: sum, rate() & Why Counters Need It

## What was run

Real, steady traffic generated against `orders_app.py` over about 10 seconds, followed by a real app restart
(resetting the real counter to 0, per [Lesson 3](03-instrumenting-metric-types.md)) and 5 more real requests. The
raw counter value and a real `rate()` query were both taken from the live Prometheus server, immediately after
the restart, to see exactly how `rate()` handles a real reset.

## A raw counter, alone, tells you almost nothing

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=orders_requests_total{endpoint="/orders"}'
# ['53']
```

`53` is a real number, but it answers almost nothing useful on its own: 53 requests since when? Is that busy or
quiet? A raw counter only becomes meaningful once you ask how it's *changing* — which is exactly what `rate()`
computes.

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=rate(orders_requests_total{endpoint="/orders"}[1m])'
# [0.36363636363636365]
```

That's a real, meaningful number: roughly 0.36 requests per second, averaged over the last real 60-second window.

## The real reset, and rate()'s real handling of it

```bash
# app restarted for real (counter resets to 0), then 5 new real requests:
curl -s ".../api/v1/query" --data-urlencode 'query=orders_requests_total{endpoint="/orders"}'
# ['5']           <- down from 53, a real reset

curl -s ".../api/v1/query" --data-urlencode 'query=rate(orders_requests_total{endpoint="/orders"}[1m])'
# [0.45454545454545453]    <- a sane, POSITIVE number, not a nonsensical negative one
```

A naive calculation across this exact window — `(5 - 53) / 60` — would give a nonsensical **negative** rate,
implying the service somehow served requests in reverse. PromQL's real `rate()` function didn't do that: it
detected that the counter's value had decreased between samples, correctly inferred a reset rather than a
negative rate (which a true, monotonically-increasing counter can never produce), and computed the real rate using
only the samples *after* the reset. This is the real payoff of the restart-unsafe-value-but-reset-safe-function
design [Lesson 3](03-instrumenting-metric-types.md) described: the raw value isn't safe across restarts, but
`rate()` specifically is built to stay correct anyway.

## How It Actually Works

`rate(v range-vector)` computes the **per-second average rate of increase** of a counter over the time range
given, automatically compensating for any resets within that range. It does this by looking at consecutive
samples within the window: whenever a later sample's value is *lower* than an earlier one, it treats that as a
reset and adjusts the calculation (roughly: counting the increase from zero after the reset, instead of a
negative jump) rather than letting it corrupt the rate. This is specifically why `rate()` exists as a dedicated
function rather than leaving users to compute `(last - first) / duration` by hand — that naive formula has no way
to know a dip was a reset rather than a real decrease, and a real counter is never supposed to decrease for any
other reason.

`sum()` (and the broader family of aggregation operators: `avg()`, `max()`, `min()`, `count()`) collapses
*multiple* time series into fewer, by combining their values at each point in time. `sum(rate(orders_requests_total[1m]))`
— without a label selector — would add together the per-second rates of every `endpoint`/`status` combination
into one overall request rate, discarding the label breakdown entirely. Adding a `by (...)` clause,
`sum by (endpoint) (rate(orders_requests_total[1m]))`, keeps the breakdown by whichever labels you list while
still collapsing everything else — the real, practical pattern behind "requests per second, broken down by
endpoint" on any real dashboard, including the one this course builds in [Level 2](../level-2/02-red-dashboard.md).

The real general rule worth internalizing from this lesson: **counters should almost never be queried raw.**
`rate()` (or its sibling `increase()`, which gives a total count over the window instead of a per-second average)
is almost always the right first step before doing anything else with a counter metric.

## Common mistakes

- Querying a raw counter value and trying to interpret it directly, the way this lesson's first example
  deliberately avoided doing anything useful with `53` alone.
- Computing a rate by hand with `(last_value - first_value) / duration`, which breaks exactly the way this
  lesson's real restart would have broken it, producing a nonsensical negative number instead of `rate()`'s
  correctly reset-aware positive one.
- Using `rate()` (per-second average) when `increase()` (total count over the window) is actually the more
  natural answer to the question being asked, or vice versa — they're closely related but answer subtly
  different real questions ("how fast, right now" vs "how many, total, over this window").

## Exercise

Run the exact restart sequence this lesson used again, but this time compute the naive
`(last_value - first_value) / duration` by hand using the raw values queried immediately before and after the
restart, and compare your hand-computed (likely negative) number directly against what `rate()` returns for the
same window. Write down the actual numeric difference you observe.
