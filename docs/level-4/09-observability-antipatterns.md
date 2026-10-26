---
description: "Every real cost measured across this course's 39 earlier lessons, tallied into one list of the real anti-patterns each one caught in the act."
---

# Common Observability Anti-Patterns

## What was run

No new experiment — this lesson tallies the real anti-patterns this course caught directly, in its own real data,
across every earlier level, as the concrete evidence base for the patterns worth naming and avoiding deliberately.

## The real anti-patterns, each one caught with real evidence

| Anti-pattern | Caught in | Real evidence |
|---|---|---|
| Trusting a raw counter without `rate()` | [Level 1, Lesson 6](../level-1/06-promql-aggregation-rate.md) | A real restart dropped a counter from 53 to 5; a naive `(last-first)/duration` would have shown a nonsensical negative rate |
| Unbounded label cardinality | [Level 3, Lesson 7](../level-3/07-cardinality-explosion.md) | 500 real requests with a UUID label created 500 real series and 37,000 bytes, vs. 1 series and 26 bytes done correctly |
| Averaging already-computed percentiles | [Level 1, Lesson 7](../level-1/07-histogram-quantiles.md) | A naive average of two real instances' p95s gave 0.509s; the real, correctly-aggregated p95 was 0.9415s |
| Alerting on absolute thresholds instead of ratios | [Level 2, Lesson 6](../level-2/06-avoiding-alert-fatigue.md) | A real absolute-rate alert fired on genuinely healthy traffic (92.7% real success) |
| Shallow black-box health checks | [Level 3, Lesson 9](../level-3/09-blackbox-vs-whitebox.md) | `/health` returned real `200 healthy` during a real 100%-failure outage |
| Single-window alerting | [Level 4, Lesson 3](03-burn-rate-alerts.md) | A real long-window burn rate of 5.136x coexisted with a real short-window `NaN` (zero current traffic) |
| Unreviewed PromQL precedence bugs | [Level 2, Lesson 10](../level-2/10-project-real-alert-pages.md) | A real alert rule silently never fired, `health: ok` the whole time, due to an unparenthesized `and`/`>` |
| Contaminated baselines for SLOs and capacity | [Level 4, Lessons 4–5](04-designing-an-slo.md) | Real p95 latency inflated to 2.3s by an old test; a real naive growth projection overestimated by including one-time test bursts |
| Trusting that work completed without independent confirmation | [Level 4, Lesson 8](08-oncall-incident-response.md) | A real fix at T+29s didn't clear the real alert until T+80s — the alert, not the fix timestamp, was the real signal that mattered |
| Assuming a networking primitive works without testing it | [Level 1, Lesson 4](../level-1/04-scraping-service-discovery.md), [Level 3, Lesson 3](../level-3/03-otel-metrics-convergence.md) | A suspected Prometheus scraping bug turned out fine when actually tested; a real `start_http_server()` genuinely failed in this environment when actually tested |

## How It Actually Works

Every row in this table is the same real, general lesson applied to a different specific mechanism: **a plausible
assumption about an observability system's behavior is not the same as a verified fact about it**, and this
course's own real, repeated experience — across metrics, alerting, tracing, and SLOs — is that the gap between
the two shows up constantly, in both directions. Sometimes the assumption was *too pessimistic* (the suspected
redirect bug in [Level 1, Lesson 4](../level-1/04-scraping-service-discovery.md) turned out not to matter).
Sometimes it was *too optimistic* (the `start_http_server()` assumption in
[Level 3, Lesson 3](../level-3/03-otel-metrics-convergence.md) turned out to be genuinely broken in this specific
environment). Sometimes a real, syntactically valid rule was silently, invisibly wrong
([Level 2, Lesson 10](../level-2/10-project-real-alert-pages.md)). The common thread across every real example in
this table isn't any one specific technology — it's that each one was *caught* only because this course tested
the actual behavior directly against real data, rather than trusting what seemed reasonable.

The second, related thread: several of these anti-patterns ([Lessons 4–5](04-designing-an-slo.md) especially)
show that even real, genuinely-measured data can mislead if it isn't checked for what it actually represents —
raw numbers from a system under test, a known incident, or a deliberately injected fault are real, but not
necessarily representative of the steady-state behavior a real SLO or capacity plan needs to be built on.

## Common mistakes

- Reading this list as "things that can theoretically go wrong" rather than "things this course's own real data
  actually demonstrated going wrong" — every single row has a real, specific, reproducible example attached,
  not a hypothetical one.
- Assuming familiarity with these patterns in the abstract is the same as catching them in a real, specific
  system — every one of these was only caught here by running real experiments against real data, not by knowing
  the pattern existed in general.
- Treating this as a complete list — it's a tally of what this specific course's specific real experiments
  happened to surface, not an exhaustive catalog of every possible observability anti-pattern.

## Exercise

Pick one row from this table that resonates most with a real system you've worked on or are responsible for.
Write down one specific, real check you could run *today* against that real system — not hypothetically, an
actual command or query you could execute — to find out whether that same anti-pattern is present, the same way
every row in this table was found: by testing directly, not by assuming.
