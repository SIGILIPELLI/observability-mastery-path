---
description: "A real PromQL precedence bug silently broke the capstone's own alert rule; fixed, it fired, paged, and sent a real resolved notification on recovery."
---

# Project: A Real Alert That Fires, Routes & Pages

## What was run

A real, deliberately injected incident: `payments_service.py`'s `/charge` endpoint was switched into a real
"outage" mode where every charge is declined, simulating a real card-processor failure. A real ratio-based alert
rule (combining [Lesson 4](04-alerting-rules-alertmanager.md)'s rule structure with [Lesson 6](06-avoiding-alert-fatigue.md)'s
ratio-over-absolute-threshold fix) was supposed to catch it — and, along the way, exposed a real bug in its own
authoring, caught and fixed the same way every honest bug in this course has been.

## A real bug in the capstone's own alert rule

```yaml
expr: >
  (sum(rate(orders_requests_total{endpoint="/checkout", status!="200"}[1m]))
   / sum(rate(orders_requests_total{endpoint="/checkout"}[1m])))
  and
  (sum(rate(orders_requests_total{endpoint="/checkout"}[1m])) > 0.05)
  > 0.5
```

With the real incident genuinely active (100% of checkouts failing), this expression returned **nothing** —
empty, no result — checked directly against live data:

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=(... exact expression above ...)'
# result: []    <- even with a real, confirmed 100% error ratio
```

The cause: PromQL's comparison operators (`>`) bind *tighter* than `and`, so this expression actually parsed as
`ratio and (volume > 0.05 > 0.5)` — comparing the real volume number against `0.5` a second time, not what was
intended. Fixing it with explicit parentheses around the actual intended grouping:

```yaml
expr: >
  (sum(rate(orders_requests_total{endpoint="/checkout", status!="200"}[1m]))
   / sum(rate(orders_requests_total{endpoint="/checkout"}[1m])) > 0.5)
  and
  (sum(rate(orders_requests_total{endpoint="/checkout"}[1m])) > 0.05)
```

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=(... corrected expression ...)'
# result: value=1   <- correctly true now
```

## The full, real, corrected pipeline

```bash
curl -s -X POST "http://localhost:9305/incident/true"     # real outage injected
# real checkout traffic sent...

curl -s ".../api/v1/alerts"
# HighCheckoutErrorRatio  firing  value=1
```
```
[PAGER] status=firing alertname=HighCheckoutErrorRatio summary=Checkout error RATIO is above 50%
INFO: "POST /alert-webhook?channel=pager HTTP/1.1" 200 OK
```

```bash
curl -s -X POST "http://localhost:9305/incident/false"    # real recovery
# real post-recovery traffic sent...
```
```
[PAGER] status=resolved alertname=HighCheckoutErrorRatio summary=Checkout error RATIO is above 50%
INFO: "POST /alert-webhook?channel=pager HTTP/1.1" 200 OK

curl -s ".../api/v1/alerts"
# []    <- genuinely cleared
```

A complete, real incident lifecycle: a real outage injected, a real alert correctly firing against it (after a
real authoring bug was found and fixed), a real page delivered, a real recovery, and a real `resolved`
notification confirming it — with Alertmanager's own `resolve_timeout` behavior handling the clear automatically
once the underlying condition genuinely stopped being true.

## How It Actually Works

This project is every earlier lesson in this level working together under one real, end-to-end scenario: the
real `Counter` metric from Level 1 recording each outcome, the real ratio-based alert design from
[Lesson 6](06-avoiding-alert-fatigue.md) avoiding a false page on mere volume, the real routing from
[Lesson 5](05-alert-routing-grouping-silencing.md) sending it specifically to the pager channel, and (available,
though not the focus of this final run) the real structured logs from [Lesson 8](08-structured-logging-correlation.md)
that would explain *why* if anyone needed to dig in during the real incident.

The real bug this project's own rule had, and the process of finding it, is itself the most important part of
this lesson: **writing a correct PromQL alert expression is genuinely easy to get subtly wrong**, and the only
reliable way to know a rule actually does what you intend is to test it against real data showing the condition
you expect it to catch — exactly as [Level 1, Lesson 4](../level-1/04-scraping-service-discovery.md) demonstrated
for scrape configs. A rule that "looks right" and has `health: ok` in Prometheus's own rule status (as this
broken version did, the whole time) can still silently fail to fire during a real incident, which is about the
worst possible time to discover an alerting bug.

## Common mistakes

- Trusting a rule's `health: ok` status as proof it evaluates correctly — as shown above, `ok` only means the
  expression is syntactically valid and ran without a query error, not that it computes the logically correct
  condition.
- Writing complex boolean logic in PromQL (`and`, `or`, comparisons) without explicit parentheses around every
  intended grouping — PromQL's real operator precedence rules are easy to get wrong silently, exactly as this
  project's own rule did.
- Testing an alert rule only by reading it, never by actually reproducing the real condition it's meant to catch
  and confirming it fires — the only way this project's real bug was ever found.

## Exercise

Write a second, deliberately broken alert expression with a different real precedence mistake (for example,
mixing `or` and `and` without parentheses), confirm it's broken the same way this lesson's was — by testing it
directly against a real condition it should catch, not by reading it — and fix it. Then write down a general rule
for yourself about when PromQL boolean expressions need explicit parentheses, based on what you've now observed
twice in this course.
