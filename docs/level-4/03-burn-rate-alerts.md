---
description: "A real long-window burn rate of 5.136x still reflects past outages while the short window shows NaN, zero current traffic -- proof single windows mislead."
---

# Burn Rate Alerts: Multi-Window Multi-Burn-Rate Alerting

## What was run

Real **burn rate** — how many times faster than sustainable the error budget from [Lesson 2](02-error-budgets.md)
is being consumed — computed from this course's own real, live traffic at two different real window lengths: a
long 1-hour window (still reflecting this course's real past outages) and a short, very recent window, to see
whether they agree.

## The real burn rate, two real windows, two real different pictures

```promql
burn_rate = error_ratio / (1 - slo_target)      # slo_target = 0.95, so budget = 0.05
```

```bash
# long window: 1 hour
curl -s ".../api/v1/query" --data-urlencode 'query=sum(rate(orders_requests_total{endpoint="/checkout", status!="200"}[1h])) / sum(rate(orders_requests_total{endpoint="/checkout"}[1h]))'
# 0.25680069927491816   -> burn rate = 0.2568 / 0.05 = 5.136x

# short window: 1 minute
curl -s ".../api/v1/query" --data-urlencode 'query=sum(rate(orders_requests_total{endpoint="/checkout", status!="200"}[1m])) / sum(rate(orders_requests_total{endpoint="/checkout"}[1m]))'
# NaN   -> no real traffic at all in the last minute
```

The real long window still shows a burn rate of **5.136x** — a real, genuine reflection of this course's past
injected outages, still inside the 1-hour lookback. The real short window shows **`NaN`** — literally no real
checkout traffic has happened recently enough to even compute a ratio. These two real numbers tell two different,
both-true stories: "there was a real, severe problem within the last hour" (long window) and "nothing is actively
happening right this second" (short window) — and an alert that only looked at one of them would get an
incomplete, potentially misleading picture.

## How It Actually Works

A **single-window** burn-rate alert has a real, structural tradeoff no single window length escapes: a *short*
window (reacts fast, but trusts a tiny, noisy sample — a handful of real requests can swing the ratio wildly) or
a *long* window (statistically stable, but exactly as this lesson's real `5.136x` shows, keeps reporting a
problem for the entire window's duration even after the real underlying issue has completely stopped, because
old, real, already-resolved failures are still inside the lookback).

**Multi-window burn-rate alerting** (the real pattern Google's SRE workbook popularized) fixes both problems at
once by requiring **both** a long window AND a short window to show an elevated burn rate before firing: the long
window confirms the problem is real and statistically meaningful, not just a noisy blip; the short window
confirms it's *still actively happening right now*, not an issue from earlier in the long window that's already
resolved — precisely the real distinction this lesson's two actual numbers (`5.136x` long, `NaN`/quiet short)
make directly visible. A real multi-window rule combining both might require, for example, a 1-hour burn rate
above 5x **and** a 5-minute burn rate above 5x before paging — satisfied by neither of this lesson's two real
numbers alone, which is exactly the correct, non-noisy outcome given that nothing is actively failing right now.

This also typically comes with **multiple burn-rate thresholds**, not just one: a very high burn rate (say, 14x,
meaning the entire 30-day budget would be exhausted in about 2 days at the current rate) justifies an immediate
page; a lower, sustained burn rate (say, 3x, exhausting the budget in about 10 days) might only justify a ticket
for the next business day — the same `page`-vs-`ticket` severity split [Level 2, Lesson 5](../level-2/05-alert-routing-grouping-silencing.md)
built directly, now driven by *how fast* the budget is being spent rather than a flat threshold.

## Common mistakes

- Alerting on a single window length and accepting either its noise (too short) or its staleness (too long) as
  an unavoidable tradeoff — multi-window alerting exists specifically because this tradeoff doesn't have to be
  accepted; combining two windows gets the benefit of both.
- Treating a long-window burn rate as "currently happening" without checking a short window — this lesson's real
  numbers show exactly how misleading that would be: a real 5.136x long-window burn rate with genuinely zero
  current traffic.
- Using the same burn-rate threshold for every severity level instead of a real tiered system (fast-and-severe
  pages immediately, slow-and-sustained becomes a lower-urgency ticket) — collapsing every burn rate into one
  alert reintroduces the alert-fatigue risk [Level 2, Lesson 6](../level-2/06-avoiding-alert-fatigue.md) measured
  directly.

## Exercise

Write a real Prometheus alert rule implementing this lesson's exact multi-window logic: fire only when
`sum(rate(orders_requests_total{endpoint="/checkout",status!="200"}[1h])) / sum(rate(orders_requests_total{endpoint="/checkout"}[1h])) / 0.05 > 5`
**and** the equivalent 5-minute-window expression also exceeds `5`. Generate real fresh checkout traffic with a
real elevated error rate, confirm the rule fires only once *both* real windows agree, and confirm it stays quiet
if you stop sending traffic (reproducing this lesson's real `NaN` short-window condition) even while the long
window still reflects the recent real burst.
