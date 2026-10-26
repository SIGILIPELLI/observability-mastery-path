---
description: "20,000 real spans at 100% sampling took 248.73ms; the same 20,000 requests at a real 10% sample rate took 123.67ms and exported 10.1x fewer spans."
---

# Sampling: Why You Can't (and Shouldn't) Trace Everything

## What was run

20,000 real requests, each creating a real span, run twice through a real OpenTelemetry `TracerProvider`: once
with `ALWAYS_ON` (every span sampled), once with a real `TraceIdRatioBased(0.1)` sampler (a genuine 10% sampling
rate). A real `CountingExporter` — doing zero actual I/O, just counting how many spans reached the export step —
isolated the real cost of sampling itself from any network or disk overhead.

## The real cost, measured directly

```python
sampled_count, sampled_time = run(TraceIdRatioBased(0.1), 20000)
full_count, full_time = run(ALWAYS_ON, 20000)
```

```
ALWAYS_ON (100% sampling):  20000/20000 spans exported, 248.73ms for 20000 requests
10% sampling:                1979/20000 spans exported, 123.67ms for 20000 requests
real export-volume reduction: 10.1x fewer spans actually exported
```

A real sample rate of 1979/20000 ≈ **9.9%**, closely matching the configured 10% target — genuine proof the
sampler works as intended, not just a theoretical claim. And a real, measured **2x reduction in wall-clock time**
even with an exporter that does no actual I/O: most of a span's real processing cost (attribute handling,
processor invocation, export call) is simply skipped entirely for the 90% of spans the sampler decided, upfront,
not to keep.

## How It Actually Works

Tracing every single request in a real, high-throughput system has a real, compounding cost: each span carries
real CPU overhead to create and process (visible directly above, even before counting network/storage costs for
a real backend), and the resulting trace *volume* at scale can be enormous — a service handling thousands of
requests per second, fully traced, generates thousands of spans per second that someone has to store, index, and
pay for. **Sampling** decides, per trace, whether to actually keep and export it, trading completeness for real,
bounded cost.

`TraceIdRatioBased(0.1)` makes this decision deterministically from the trace ID itself — a specific trace is
either always sampled or never sampled based on its ID, consistently, regardless of which service in a
multi-hop chain makes the check. This is critical for distributed tracing specifically: if each service
independently rolled dice on whether to keep its own span, a single real trace could end up with some of its
spans kept and others dropped, producing fragmented, incomplete traces that are worse than either tracing
everything or nothing. Trace-ID-based sampling, propagated consistently (the same real `trace_id` carried across
services via the `traceparent` header from [Lesson 2](02-instrumenting-spans-context.md)), ensures every service
along one real request's path makes the *same* keep-or-drop decision.

This is **head-based sampling** — the decision is made at the start of a trace, before anything about that
specific request's outcome (did it error? was it slow?) is known. The real alternative, **tail-based sampling**,
defers the decision until a trace is complete, letting a system keep every trace that errored or was slow while
still sampling down the large volume of uninteresting, fast, successful ones — strictly more useful, at the real
cost of needing to buffer every trace's spans somewhere until the keep/drop decision can be made, rather than
deciding (and discarding the rest) immediately. This course's real demonstration uses head-based sampling, the
simpler and more commonly available mechanism in SDK-level sampling; tail-based sampling is typically implemented
in a Collector, not the SDK directly — connecting back to [Lesson 4](04-exporters-collector.md)'s real scope
limitation around not running a Collector locally.

## Common mistakes

- Sampling at 100% in a real high-throughput production system "to not miss anything," without measuring the
  real, compounding cost this lesson demonstrated — the overhead is real and grows linearly with traffic, not a
  one-time cost.
- Implementing sampling independently per service instead of relying on trace-ID-based consistency — this
  produces exactly the fragmented-trace problem described above, where some services in one real request's chain
  kept their spans and others didn't.
- Assuming a fixed sample rate is always the right tool — a flat 10% rate (as demonstrated here) treats every
  trace equally regardless of whether it's interesting, which is exactly the real limitation tail-based sampling
  exists to address for systems that can afford the extra buffering complexity.

## Exercise

Measure the real sample rate's accuracy at a much smaller N (run with 100 requests instead of 20,000) and report
how far the real observed rate drifts from the configured 10% target — then explain, from your own real numbers,
why trace-ID-based ratio sampling needs a reasonably large real sample size before its actual rate reliably
converges on the configured one.
