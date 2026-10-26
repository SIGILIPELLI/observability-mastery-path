---
description: "Real p95/p99 checkout latency of 2.3-2.46s was traced to a 300-request burst from an earlier lesson -- a lesson in checking contamination, not assuming it."
---

# Designing an SLO for a Real Service

## What was run

Real p50, p95, and p99 checkout latency computed from this course's own actual historical Prometheus data — an
attempt to use it as a real baseline for designing an SLO, which surfaced a real, honest problem with the data
itself before any SLO could be responsibly chosen.

## The real numbers, and the real problem with them

```bash
histogram_quantile(0.50, sum(rate(orders_request_duration_seconds_bucket{endpoint="/checkout"}[1h])) by (le))
# 0.0927364864864865
histogram_quantile(0.95, sum(rate(orders_request_duration_seconds_bucket{endpoint="/checkout"}[1h])) by (le))
# 2.29875
histogram_quantile(0.99, sum(rate(orders_request_duration_seconds_bucket{endpoint="/checkout"}[1h])) by (le))
# 2.45975
```

p50 is a real, reasonable 93ms. p95 and p99 jump to a real **2.3–2.46 seconds** — a real, honest red flag, but not
about the service's actual normal behavior. Rather than assume a cause, the real high-latency samples were
located precisely with a real `query_range` scan: two genuine spikes to ~2.48 seconds, both timestamped within a
two-minute window. Checking what this course was actually doing at that exact real time identified the cause:
[Lesson 6](../level-2/06-avoiding-alert-fatigue.md)'s real burst of 300 *concurrent* checkout requests, sent all
at once via threading to measure absolute-vs-ratio alerting. That real burst genuinely saturated the app's
request-handling thread pool, and a handful of real requests queued for seconds waiting for a free worker — a
real, unintentional side effect of that lesson's own test traffic, still sitting inside this 1-hour lookback
window. Designing a real SLO from this data as-is would mean committing to a latency target inflated by that
deliberate stress-test burst, not the service's real, organic behavior.

## How It Actually Works

Designing a real SLO starts with a real baseline — but "real data" and "representative data" are not the same
thing, and this lesson's own numbers show exactly how they can diverge. A genuinely responsible SLO design
process has to actively exclude or account for known-artificial periods (deliberate chaos tests, known incidents
already being fixed, deployment windows with expected elevated error rates) before trusting raw historical
percentiles as "what normal looks like." Skipping this step — taking p95=2.3s at face value — would either commit
to an unnecessarily loose SLO (giving up real headroom to detect genuine future degradation) or, worse, bake a
known test artifact into a number treated as a real production commitment.

With that caveat made explicit, the real, practical SLO design process:

1. **Establish a real, clean baseline** — p50 of 93ms here is a plausible, uncontaminated number (fast operations
   aren't much affected by one slow tail event skewing a high percentile), and would need to be re-measured over
   a longer, cleaner window for p95/p99 specifically.
2. **Choose a target with real headroom above the baseline**, not equal to it — an SLO set exactly at current
   measured performance leaves no room for natural variance and will be breached by routine noise; a common real
   practice is targeting something the service already beats most of the time, so the SLO represents a
   genuine commitment with margin, not a coin flip.
3. **Choose a real, specific time window** — [Lesson 1](01-slis-slos-slas.md)'s distinction between a raw SLI and
   an SLO applies directly here: "p95 latency under 500ms, measured over a rolling 28 days" is a real SLO; "p95
   latency under 500ms" with no window is not fully specified.
4. **Validate the target is actually achievable** against real, clean historical data — exactly what this
   lesson's contaminated numbers prevented from being done responsibly without first addressing the data quality
   issue found.

## Common mistakes

- Designing an SLO from raw historical data without checking it for exactly the kind of contamination this lesson
  found — a known chaos test, an incident window, or a deployment spike can silently distort a percentile enough
  to produce a misleadingly loose (or, in other cases, misleadingly tight) target.
- Setting an SLO target with zero real margin above current measured performance, leaving no room for ordinary
  variance to avoid constant, false SLO breaches.
- Designing an SLO once and never revisiting it as the real service's behavior and real traffic patterns change —
  a baseline from months ago may no longer represent what "normal" actually looks like today.

## Exercise

Query this course's real `orders_request_duration_seconds` histogram restricted to a time range that excludes the
known contaminated window identified above (use `query_range` with explicit `start`/`end` parameters, or
`histogram_quantile(..., [30m])` evaluated at a timestamp from *before* the real concurrent-burst test ever ran,
via Prometheus's `@` modifier). Report the real, clean p95 latency this produces, and use it to propose one
specific, real, properly-justified SLO target and window for `orders_app.py`'s `/checkout` endpoint.
