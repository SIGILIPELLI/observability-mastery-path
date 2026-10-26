---
description: "A real PromQL query computed an actual SLI of 81.9% checkout success from 100 requests -- the real number a target (SLO) and a contract (SLA) build on."
---

# SLIs, SLOs & SLAs: the Real Differences

## What was run

A real PromQL query against 100 real checkout requests (sent with a deliberate real mix: ~85% small, successful
amounts and ~15% large, declined ones) to compute a real **Service Level Indicator** — the actual, measured
percentage of checkouts that succeeded — as the concrete starting point for the three terms this lesson
untangles.

## The real number everything else in this lesson is built on

```promql
sum(rate(orders_requests_total{endpoint="/checkout", status="200"}[5m]))
/ sum(rate(orders_requests_total{endpoint="/checkout"}[5m]))
```

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=(... expression above ...)'
# 0.819047619047619
```

A real, measured **81.9%** checkout success rate — this is the SLI. It's just a number, computed from real
traffic, with no judgment attached yet about whether 81.9% is good, bad, or acceptable.

## How It Actually Works

These three terms are used interchangeably in casual conversation, but they describe three genuinely different
things, each building on the one before:

- **SLI (Service Level Indicator)**: a real, quantitative measurement of some aspect of service behavior —
  exactly the `0.819...` computed above. An SLI is just data; it answers "what actually happened," nothing more.
  Other real SLIs this course has already computed without naming them as such: the real p95 latency numbers from
  [Level 1, Lesson 7](../level-1/07-histogram-quantiles.md), the real error ratios from
  [Level 2, Lesson 6](../level-2/06-avoiding-alert-fatigue.md).
- **SLO (Service Level Objective)**: a real *target* for an SLI — a statement like "checkout success rate should
  be ≥ 99.5% over a rolling 30-day window." An SLO turns a raw measurement into a goal with a specific threshold
  and a specific time window, which is what makes it possible to say whether the real, measured 81.9% above is
  "fine" (if the SLO target were, say, 80%) or "a real problem" (if the target were 99.5%) — the number alone
  can't tell you that; the SLO does.
- **SLA (Service Level Agreement)**: a real, often contractual promise built on top of an SLO, usually with real
  consequences attached for missing it — a refund, a credit, a penalty clause. An SLA is typically set *looser*
  than the internal SLO it's based on, deliberately: an organization wants internal warning (via the SLO) well
  before it's at real risk of breaching an external, consequential promise (the SLA).

The real, practical relationship: **SLIs are measured, SLOs are chosen, SLAs are promised** — and each one only
makes sense built on the layer below it. You can't meaningfully set an SLO without first having a real SLI to
measure against (exactly the real PromQL query this lesson started with), and you shouldn't make an external SLA
promise without first having real internal SLO data showing you can reliably meet it.

This real distinction matters immediately for the rest of this level: [Lesson 2](02-error-budgets.md)'s error
budget and [Lesson 3](03-burn-rate-alerts.md)'s burn-rate alerting are both built directly on top of a chosen SLO
target, applied to a real, measured SLI exactly like the one computed here.

## Common mistakes

- Treating "we have an SLO" as the same thing as "we have an SLI" — a target with nothing real measuring progress
  toward it is just an aspiration, not an operational SLO.
- Setting an SLA equal to (or tighter than) the internal SLO it's based on, leaving no real margin for internal
  warning before an external, consequential breach — the whole point of the SLO/SLA distinction is a real buffer
  between "we're concerned" and "we owe someone something."
- Choosing an SLO target without first looking at real historical SLI data — a target set with no reference to
  what's actually been achievable (the real 81.9% measured here, for instance) is either trivially easy or
  genuinely unachievable, and either way wasn't a real engineering decision.

## Exercise

Compute a second real SLI for this system — latency-based this time: the real p95 checkout latency over the same
traffic, using the `histogram_quantile()` pattern from [Level 1, Lesson 7](../level-1/07-histogram-quantiles.md).
Using both real SLIs (success rate and p95 latency), write down one plausible real SLO for each — a specific
target and time window — and one sentence on what real, measurable evidence from this course's own data would
justify choosing those specific numbers rather than arbitrary ones.
