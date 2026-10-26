---
description: "One real cross-service request confirmed via a real metric (4/20 errors), a real structured log, and a real two-service span chain ending in status ERROR."
---

# Project: Full-Stack Observability for a Real Request

## What was run

The full real two-service OpenTelemetry system from this level — `service_a_otel.py` and `service_b_otel.py`,
both emitting real metrics, real structured logs, and real propagated traces — sent 20 real requests, with
Service B's genuine ~20% random failure rate producing real errors. One specific real failed request was then
traced completely across all three real signals, across both real services, end to end.

## The real metric: aggregate signal

```bash
curl -s localhost:9502/metrics | grep service_b_compute_total
```
```
service_b_compute_total{result="error"} 4.0
service_b_compute_total{result="ok"} 16.0
```

4 of 20 real requests failed — a real, aggregate 20% error rate.

## The real log: one specific instance, with its trace ID

```bash
grep "compute_failed" b_final.log | head -1
```
```json
{"trace_id": "7ed9d1d616536edbcaa81afe5d02c846", "event": "compute_failed", "reason": "simulated transient fault"}
```

## The real, full cross-service trace for that exact request

```
Service A's real span:
  name="handle"  trace_id=0x7ed9d1d6...  span_id=0x6ff387c0f0144f4f

Service B's real span:
  name="GET /compute"  trace_id=0x7ed9d1d6...  parent_id=0x6ff387c0f0144f4f  status_code="ERROR"
```

Service B's top-level span's `parent_id` is **exactly** Service A's `handle` span's `span_id` — real,
cross-process proof the two services' independently-created spans belong to the same real request, and Service
B's span is correctly marked `ERROR`. Three signals, two separate real processes, one real request: a real
20%-of-20 aggregate count, a real log line naming this specific trace and its reason, and a real trace confirming
exactly where in the real two-service call chain it failed.

## How It Actually Works

This project is the direct combination of this level's real building blocks: the trace propagation mechanism
from [Lesson 2](02-instrumenting-spans-context.md) (the real shared `trace_id` across process boundaries), the
OTel-native metrics from [Lesson 3](03-otel-metrics-convergence.md), the structured logging and correlation
pattern from [Lesson 6](06-correlating-three-pillars.md), all running simultaneously across a real multi-service
system rather than one single process. The real, complete chain demonstrated above — aggregate count down to
one specific instance down to its exact structural location in a cross-service call — is the actual, practical
payoff every earlier lesson in this level was building toward individually.

It's also worth being honest about what this capstone does *not* cover, consistent with this level's stated
scope limits: no real Collector batching and forwarding this data to a production backend
([Lesson 4](04-exporters-collector.md)'s real limitation), no real sampling applied to reduce volume at scale
([Lesson 5](05-sampling-strategies.md)'s real tradeoff), and no cardinality discipline specifically enforced on
these particular metrics' labels ([Lesson 7](07-cardinality-explosion.md)'s real warning) — all genuinely
applicable to a production version of this same system, deliberately left out here to keep the core, real
mechanism — one request, three correlated signals, two real services — as clear and directly inspectable as
possible.

## Common mistakes

- Building rich instrumentation across multiple services without a real, verified test that the correlation
  actually works end to end — this project's real cross-service `parent_id` match is exactly the kind of direct
  verification that confirms propagation isn't silently broken somewhere in the chain.
- Treating "we have metrics, logs, and traces" as equivalent to "we have observability" — the real value, as this
  entire capstone demonstrates, comes specifically from being able to move *between* the three signals for one
  real event, not from each existing independently and never being cross-referenced.
- Stopping instrumentation at the first service in a chain and assuming it "covers" the whole request — this
  project's real trace only tells the complete story because *both* services propagated and recorded it
  consistently; a gap at either end would have left an incomplete picture.

## Exercise

Add a third real service to this chain, have it fail independently with its own real random error rate, and
extend this project's real correlation exercise: find one real request that failed at the *third* service (not
the second), and confirm you can trace it back through all three real services' spans, logs, and metrics, the
same way this lesson did for a two-service failure — proving the pattern holds as the real chain gets longer, not
just for the simplest two-hop case.
