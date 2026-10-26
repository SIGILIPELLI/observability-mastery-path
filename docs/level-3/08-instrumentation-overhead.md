---
description: "A real 100,000-call benchmark measured instrumentation adding 12.758us per call -- a 20.25x relative slowdown that's still tiny in absolute terms."
---

# Instrumentation Overhead: Measuring the Cost of Observability

## What was run

A real, deliberately tiny function (`sum(range(100))`) called 100,000 times two ways: completely bare, and
wrapped in a real OpenTelemetry span plus a real Prometheus counter increment — using a real `NullExporter` that
does no actual I/O, isolating the pure in-process cost of instrumentation itself from any network or disk cost.

## The real, measured cost

```python
def bare_work():
    return sum(range(100))

def instrumented_work(tracer, counter):
    with tracer.start_as_current_span("work"):
        counter.inc()
        return sum(range(100))
```

```
bare function, 100000 calls:          66.26ms  (0.663us/call)
instrumented function, 100000 calls:  1342.04ms  (13.420us/call)
real per-call overhead: 12.758us, 20.25x slower
```

A real, measured **20.25x relative slowdown** — and also, in the same breath, a real **12.758 microseconds** of
absolute added cost per call. Both numbers are true at once, and which one matters depends entirely on what
you're instrumenting.

## How It Actually Works

**20.25x** sounds alarming in isolation. **12.758 microseconds** sounds negligible in isolation. Both are the
exact same real measurement, and the real lesson is that **relative overhead only matters relative to the real
cost of the thing being measured**. This course's own `orders_app.py` checkout endpoint, measured for real back in
[Microservices, Level 1, Lesson 10](https://sigilipelli.github.io/microservices-mastery-path/level-1/10-project-decompose-monolith.md),
takes something on the order of 10-20 *milliseconds* for a real network round trip — against that, 12.758
*microseconds* of instrumentation overhead is roughly 0.1% or less of the total request time, utterly
imperceptible to a real user. Against this lesson's deliberately tiny, CPU-bound `sum(range(100))` — a function
that does no I/O and completes in under a microsecond on its own — the same absolute overhead looks enormous,
because there was almost nothing there to begin with for it to be relative to.

This is the real, practical rule the measurement supports: **instrumentation overhead is rarely a real concern
for anything that does real I/O** — a database query, a network call, disk access — because the real I/O cost
(milliseconds, often tens or hundreds of them) dwarfs microsecond-scale instrumentation overhead by several
orders of magnitude. It becomes a real, measurable concern specifically in **tight, CPU-bound hot loops** called
extremely frequently with no I/O in between — exactly the shape of code this lesson's benchmark deliberately
used to make the real relative cost visible at all.

This connects directly to [Lesson 5](05-sampling-strategies.md)'s real sampling numbers: sampling reduces
*volume* (how many spans get fully processed and exported), which is a different lever from this lesson's
per-call overhead (how expensive *each* instrumented call is, sampled or not — note the `NullExporter` here never
even checked a sampling decision). Both levers matter together in a real high-throughput system: a sampler cuts
how often the full cost is paid; keeping each individual instrumented call cheap (lightweight attributes, no
expensive computation inside a span) keeps that per-call cost itself low.

## Common mistakes

- Citing a relative overhead number ("20x slower!") without the matching absolute number, or vice versa — either
  one alone can be genuinely misleading about whether a real system should actually care.
- Avoiding instrumentation entirely out of a vague fear of "performance overhead," without ever measuring the
  real cost against the real workload it would actually run against — exactly the measurement this lesson
  performed, which is cheap and conclusive to do directly rather than guess about.
- Assuming instrumentation overhead is uniform across every call site in a codebase — a hot, CPU-bound inner loop
  and a slow, I/O-bound network call have wildly different real sensitivity to the same absolute per-call cost,
  as this lesson's own numbers (0.1% vs 20x) show directly.

## Exercise

Re-run this lesson's exact benchmark, but replace `bare_work()`/`instrumented_work()`'s body with a real,
artificial I/O wait (`time.sleep(0.005)`, simulating a 5ms network call) instead of `sum(range(100))`. Report the
new real relative and absolute overhead numbers, and confirm — with your own fresh measurement — that the real
*relative* overhead drops dramatically once there's real I/O cost for the same absolute instrumentation overhead
to be measured against.
