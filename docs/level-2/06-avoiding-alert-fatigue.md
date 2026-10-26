---
description: "A real absolute-rate alert fired on genuinely healthy traffic (92.7% success) while a ratio-based version, checked against the same data, stayed quiet."
---

# Avoiding Alert Fatigue: Designing Alerts That Page the Right Person

## What was run

A real burst of 300 concurrent checkout requests against `orders_app.py`, deliberately weighted so about 95% would
succeed and 5% would be declined — genuinely healthy traffic at real scale, not an outage. Both the existing
absolute-rate alert from [Lesson 4](04-alerting-rules-alertmanager.md) and a proposed, real ratio-based
alternative were evaluated against the exact same real data to see which one actually behaves correctly.

## The existing alert, firing on genuinely healthy traffic

```bash
# 300 real concurrent requests, ~95% designed to succeed...
curl -s ".../api/v1/query" --data-urlencode \
  'query=sum(rate(orders_requests_total{endpoint="/checkout", status!="200"}[1m]))'
# 0.39999999999999997

curl -s ".../api/v1/alerts"
# HighCheckoutErrorRate  firing  value=0.4
```

The existing `rate > 0.1` alert from [Lesson 4](04-alerting-rules-alertmanager.md) **fired for real** — correctly,
by its own definition, since the absolute error rate really is 0.4 req/s. But checking the actual real error
*ratio* for the same traffic:

```bash
curl -s ".../api/v1/query" --data-urlencode \
  'query=sum(rate(orders_requests_total{endpoint="/checkout", status!="200"}[1m])) / sum(rate(orders_requests_total{endpoint="/checkout"}[1m]))'
# 0.07333333333333332
```

**92.7% of real checkouts succeeded.** A 7.3% decline rate, almost entirely from deliberately-oversized test
amounts, is not an incident — it's what this traffic was designed to look like. The alert fired anyway, purely
because *volume* went up, which this absolute-threshold rule can't distinguish from an actual increase in the
real *failure rate*.

## The real fix, checked against the same real data

```promql
(sum(rate(orders_requests_total{endpoint="/checkout", status!="200"}[1m]))
 / sum(rate(orders_requests_total{endpoint="/checkout"}[1m]))) > 0.2
```

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=(... ratio expression ...) > 0.2'
# would fire: False
```

With a real 20% ratio threshold, evaluated against the exact same real traffic that triggered the old alert, this
version **correctly stays quiet** — 7.3% real errors is below a reasonable 20% alerting threshold, and no one
gets paged for traffic that's actually fine.

## How It Actually Works

**Alert fatigue** — the real, documented phenomenon where people start ignoring or routing around alerts because
too many of them turn out not to matter — isn't caused by alerting too often in the abstract; it's caused by
alerting on conditions that don't actually correlate with something a human needs to act on, exactly as this
lesson's absolute-rate alert demonstrated. Every real false alarm erodes trust in the alerting system a little,
and the real, cumulative effect over time is that a genuinely critical alert gets the same tired, delayed response
as yesterday's noise.

The real, general fix this lesson demonstrates — alerting on a **ratio** rather than an absolute count — applies
far beyond this one example: error rate, latency, resource saturation. A ratio naturally scales with traffic,
staying meaningful whether the system is handling 10 requests a minute or 10,000; an absolute threshold has to be
re-tuned every time real traffic volume changes meaningfully, and until it is, it either fires on healthy high-load
periods (as shown above) or stays silent during genuinely bad low-load periods — the opposite failure, where a
real problem goes unpaged because volume happened to be quiet.

This is also precisely the motivation behind [Level 4, Lesson 2](../level-4/02-error-budgets.md)'s error budgets
and [Lesson 3](../level-4/03-burn-rate-alerts.md)'s burn-rate alerting, which take the ratio-based approach shown
here and add a further refinement: alerting not just on "is the current ratio bad" but on "is the current ratio
bad enough, for long enough, to meaningfully threaten a real reliability target" — a still more precise filter
against false alarms than a flat ratio threshold alone.

## Common mistakes

- Choosing an alert threshold once, when traffic was a certain volume, and never revisiting it as real traffic
  patterns change — exactly the gap this lesson's absolute-rate alert fell into.
- Alerting on a ratio with too small a sample size — a 50% error ratio computed from 2 real requests is much
  less meaningful than the same 50% computed from 2,000, and a ratio-based alert needs a minimum real volume
  floor to avoid noise from genuinely tiny samples.
- Setting every alert's severity to the highest level "just in case" — real severity differentiation (as built
  with `page` vs `ticket` in [Lesson 5](05-alert-routing-grouping-silencing.md)) is itself a defense against
  fatigue, since not every real condition deserves the same urgency of response.

## Exercise

Update `alert_rules.yml`'s `HighCheckoutErrorRate` rule to use the real ratio expression from this lesson instead
of the absolute one, with a real minimum-volume guard (e.g., `and sum(rate(orders_requests_total{endpoint="/checkout"}[1m])) > 0.05`
to avoid alerting on a tiny, statistically meaningless sample). Re-run both this lesson's healthy high-volume
burst and a deliberately small, low-volume burst with a high real error ratio, and confirm your updated rule
fires only for the second case, not the first.
