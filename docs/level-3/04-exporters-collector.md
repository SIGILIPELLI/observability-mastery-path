---
description: "The same instrumentation function produced identical span data routed to two different destinations -- stdout and a real file -- with zero code changes."
---

# Exporters & the Collector: Decoupling Instrumentation from Backend

## What was run

One real Python function, `instrumented_work()`, containing real OpenTelemetry instrumentation (two real nested
spans) that never changes. It was run twice, with only the configured **exporter** swapped between runs — a real
`ConsoleSpanExporter` the first time, a real, independently-written `FileSpanExporter` the second — to directly
demonstrate the API/SDK separation [Lesson 1](01-opentelemetry-fundamentals.md) described.

## The same real instrumentation, two real destinations

```python
def instrumented_work(tracer):
    with tracer.start_as_current_span("do-work") as span:
        span.set_attribute("real.work", True)
        with tracer.start_as_current_span("sub-step"):
            pass
```

```bash
python3 exporter_swap_demo.py console
# real span JSON printed to stdout

python3 exporter_swap_demo.py file
# real span JSON written to spans_output.log instead
```

```json
{
  "name": "sub-step",
  "context": {"trace_id": "0xd78979a74b9ac66f748374fe83889659", "span_id": "0xb78c6bfa77ee840e"},
  "parent_id": "0xfe2a25fbed53f435"
}
```

Both runs produced real, structurally identical span data — same nesting, same real parent-child relationship —
the only difference was *where* that data ended up. `instrumented_work()` itself was never touched between runs;
only the three lines of exporter configuration changed.

## How It Actually Works

The real separation this lesson demonstrates directly answers a practical question every instrumented codebase
eventually faces: what happens when the organization needs to send trace data somewhere new — switch vendors,
add a second destination, move from a quick local setup to a real production backend? With OpenTelemetry's
API/SDK split, the answer is "change the exporter configuration, once, in one place" — not "find and update every
call site that creates a span throughout the codebase," which is exactly what this lesson's unchanged
`instrumented_work()` function proves is unnecessary.

A real **OpenTelemetry Collector** (not run in this course — a genuine, stated scope limitation, since it's a
separate standalone binary beyond what this lesson's direct SDK-to-exporter demonstration needed to show the
underlying mechanism) takes this decoupling one step further: instead of an application exporting directly to a
final backend, it exports to a local Collector process, which then handles batching, retrying, filtering, and
fanning out to one or more real backends. This adds a real operational benefit beyond what this lesson's direct
two-exporter swap shows: an application's instrumentation and export configuration can stay completely fixed
("send to the local Collector") while the Collector's own configuration changes behind it — adding a second
backend, changing sampling policy, adding redaction of sensitive attributes — all without touching or redeploying
any instrumented application code at all.

This course's `ConsoleSpanExporter` and `FileSpanExporter` are both deliberately simple, synchronous exporters,
chosen specifically to make the underlying span data directly visible and inspectable without extra
infrastructure — real production systems typically use the `BatchSpanProcessor` instead of this lesson's
`SimpleSpanProcessor`, batching many spans into fewer network calls to whatever real backend (or Collector) they
ultimately reach.

## Common mistakes

- Writing application code that imports and calls a specific backend's SDK directly, rather than the
  OpenTelemetry API — this reintroduces the vendor lock-in the API/SDK split exists specifically to avoid, and
  this lesson's unchanged `instrumented_work()` function is only portable because it never imports anything
  backend-specific.
- Using `SimpleSpanProcessor` (synchronous, one network call per span) in a real production path, where the real
  latency and throughput cost of exporting synchronously on every single span can become a measurable overhead —
  covered directly, with real numbers, in [Lesson 8](08-instrumentation-overhead.md).
- Assuming a Collector is required before any tracing can be useful — this course's entire Level 3 demonstrates
  real, working distributed tracing using direct SDK exporters with no Collector at all; a Collector is a real,
  valuable operational layer for production systems at scale, not a prerequisite for the underlying mechanism to
  work.

## Exercise

Write a third real exporter, a `FilteringSpanExporter` that wraps `ConsoleSpanExporter` but only actually prints
spans whose name matches a given prefix (discarding the rest silently). Run `instrumented_work()` through it
unchanged, and confirm you can filter which real spans get exported without modifying `instrumented_work()` at
all — the same real decoupling this lesson demonstrated, now with filtering logic added at the exporter layer
instead of the instrumentation layer.
