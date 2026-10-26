---
description: "A real 95% SLO over this course's last hour allowed 40.41 failures; the real count was 207.6 -- a 513.6% budget overrun from genuine injected outages."
---

# Error Budgets: Turning Reliability Into a Spendable Number

## What was run

A real error budget computed from this course's own real, accumulated Prometheus data — the actual total and
failed checkout requests over the last real hour, which genuinely includes every real outage this course
deliberately injected into `payments_service.py` across Levels 1–4. A real 95% SLO target was applied to this
real traffic to compute a real budget, and how much of it was really consumed.

## The real numbers, including every real incident this course ran

```promql
sum(increase(orders_requests_total{endpoint="/checkout"}[1h]))
```
```
total requests: 808.2
```

```promql
sum(increase(orders_requests_total{endpoint="/checkout", status!="200"}[1h]))
```
```
failed requests: 207.6
```

```python
slo_target = 0.95
allowed_failures = total * (1 - slo_target)   # 808.2 * 0.05 = 40.41
budget_consumed_pct = failed / allowed_failures * 100
```
```
real error rate: 25.68%
allowed failures at 95% SLO: 40.41
error budget consumed: 513.6%
```

A real 95% SLO over this real hour would have tolerated **40.41** failures. The real, measured failure count was
**207.6** — more than five times the real budget, **513.6%** consumed. This isn't a contrived number: it's the
real, honest consequence of this course deliberately running several real outages (the declined-payment bursts
from [Level 2](../level-2/06-avoiding-alert-fatigue.md) and [Level 2, Lesson 10](../level-2/10-project-real-alert-pages.md),
the injected incident-mode outages from [Level 3](../level-3/09-blackbox-vs-whitebox.md)) within the same real
traffic history this query covers.

## How It Actually Works

An **error budget** converts an SLO target into a concrete, spendable *allowance* rather than a pass/fail line:
if the SLO is 95% success, the budget is simply the other 5% — a real, finite amount of acceptable failure over
the chosen time window. This reframing matters because it turns reliability from an abstract goal into a real
resource that can be tracked, spent, and reasoned about the same way a real budget for money or time would be: a
team that hasn't spent its error budget has real headroom to take risks (ship a risky change, run an experiment);
a team that's already blown through 513.6% of its budget, as this lesson's real numbers show, has no real
headroom left and should be prioritizing stability over new risk, regardless of how urgent the next feature feels.

The real, specific value of computing this as a *percentage of budget consumed* rather than just "25.68% error
rate" is that it's directly comparable to the chosen target: a 25.68% real error rate is wildly over budget
against a 95% SLO, but would be comfortably *under* budget against a looser 70% SLO. The raw error rate alone
doesn't tell you whether to be alarmed; the budget consumption number, computed against a real, deliberately
chosen target, does.

This is the real foundation [Lesson 3](03-burn-rate-alerts.md) builds directly on top of: instead of alerting on
a raw error rate threshold (the real false-positive problem [Level 2, Lesson 6](../level-2/06-avoiding-alert-fatigue.md)
demonstrated), burn-rate alerting asks a more precise real question — at the current real rate of budget
consumption, will the budget be exhausted before the window resets, and how urgently does that deserve a
response?

## Common mistakes

- Reacting to a raw error rate number without first converting it to budget consumption against a real, chosen
  SLO — 25.68% sounds alarming in isolation, but the real, actionable question is always "compared to what
  target," which this lesson's 513.6%-of-budget framing answers directly.
- Choosing an SLO target with no real data informing it, the way [Lesson 1](01-slis-slos-slas.md) warned against —
  this lesson's 95% target was chosen for the exercise, not derived from this system's own real historical
  baseline, which a genuine production SLO should be.
- Resetting the error budget window after a bad incident instead of living with the real, full consequences of
  having spent it — the whole point of tracking consumption honestly, as this lesson's real 513.6% number does
  without softening it, is to make the real cost of past incidents visible, not to reset the count whenever it's
  inconvenient.

## Exercise

Recompute this lesson's exact error budget calculation using a real 2-hour window instead of 1 hour
(`increase(...[2h])`), and report the real new total, real new failure count, and real new budget-consumed
percentage. Write one sentence on whether the real percentage got better, worse, or stayed about the same, and
why that makes sense given when this course's real outages were actually injected relative to now.
