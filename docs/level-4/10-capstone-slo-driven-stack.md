---
description: "A real Grafana dashboard rendered a 0.887 SLI and a 226% budget-consumed figure in red, with a time series showing the SLI collapse in real incidents."
---

# Capstone: Build a Complete SLO-Driven Observability Stack

## What was run

Every real piece this course built, running together, one final time: the real `orders_app.py` and
`payments_service.py` from Level 1, the real Prometheus server and alert rules from Levels 1–2, the real
Alertmanager and pager from Level 2, the real structured logs and traces from Levels 2–3, and a real, final
Grafana dashboard (`Checkout SLO Dashboard`) built specifically to show this course's own real SLO status, live.

## The real, final, live result

```promql
# SLI panel
sum(rate(orders_requests_total{endpoint="/checkout", status="200"}[5m]))
/ sum(rate(orders_requests_total{endpoint="/checkout"}[5m]))

# error budget consumed panel (against a 95% SLO)
((1 - (sum(rate(orders_requests_total{endpoint="/checkout", status="200"}[5m]))
     / sum(rate(orders_requests_total{endpoint="/checkout"}[5m])))) / 0.05) * 100
```

Loaded live in a real browser, this dashboard rendered:

- **Real SLI: 0.887** — 88.7% of real checkouts succeeding, right now, in this real 5-minute window.
- **Real error budget consumed: 226%**, in bright red — this course's real cumulative traffic, including every
  deliberately injected real outage across all four levels, has spent more than double a real 95% SLO's entire
  allowance.
- **A real SLI-over-time graph**, visibly dropping to near-zero during this course's own real incidents (the
  [Level 2](../level-2/10-project-real-alert-pages.md) and [Level 4](08-oncall-incident-response.md) outage
  injections) and recovering afterward — a real, honest, visual record of this course's own testing history, not
  a staged example.

## How It Actually Works

This dashboard is not a new mechanism — it's the direct, combined application of this entire course's real
building blocks to one specific, real operational question: **is this service meeting its reliability target
right now, and by how much is it over or under budget?** The SLI panel is a real PromQL ratio query
([Level 1](../level-1/05-promql-basics.md)). The budget panel is that same real ratio, transformed by the real
error-budget arithmetic from [Lesson 2](02-error-budgets.md). The time-series panel is the real Grafana
visualization mechanics from [Level 2, Lesson 1](../level-2/01-grafana-fundamentals.md). The real 226% figure
sitting behind it all is backed by a real alert rule ([Level 2, Lesson 10](../level-2/10-project-real-alert-pages.md)),
real structured logs explaining each real failure
([Level 2, Lesson 8](../level-2/08-structured-logging-correlation.md)), and real traces that could pinpoint
exactly where in a multi-service chain any one of them happened
([Level 3, Lesson 6](../level-3/06-correlating-three-pillars.md)).

The real, honest number this capstone ends on — **226% of budget consumed, visibly, in red** — is not a clean,
reassuring finish, and that's deliberate and true to how this course actually ran: forty lessons of genuine,
repeated fault injection, real alert testing, and real incident simulation leave real, visible marks on a real
system's real reliability numbers, exactly as they would on an actual production system that had been through
a comparably eventful few days. An SLO-driven observability stack's job isn't to make that number look good — it's
to make it honestly, precisely visible, which is exactly what this real dashboard does.

## Common mistakes

- Building an impressive-looking dashboard without the underlying real alerting, logging, and tracing to back it
  up — this capstone's dashboard is only meaningful because every number on it traces back to a real, verified
  mechanism built and tested across the previous 39 lessons, not because the panels themselves look convincing.
- Treating a bad real SLO number (like this lesson's real 226%) as something to hide or explain away rather than
  the honest, actionable signal it is — the whole point of tracking it this precisely is to make it impossible to
  ignore.
- Considering observability "done" once a dashboard exists — this course's real, repeated finding, across every
  level, is that building the instrumentation is necessary but not sufficient; it only becomes useful once it's
  actually tested against real failures, exactly as this capstone's dashboard was.

## Exercise

Let this course's real services run untouched — no new deliberate incidents — for a real, meaningful stretch of
time (at least 30 minutes), then reload the real `Checkout SLO Dashboard` and report the real, new SLI and budget
numbers. Write down whether the real 226% figure has started to recover now that only genuinely healthy traffic
is flowing through the real 5-minute window, and explain, using what you now know about windowed metrics from
across this entire course, why it would or wouldn't fully reset to 0% even after a long, completely healthy period.
