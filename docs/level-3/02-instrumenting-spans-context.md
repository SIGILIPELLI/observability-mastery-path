---
description: "Two real services' spans share the same trace_id across a real HTTP call, and the child's parent_id matches the parent's span_id exactly, byte for byte."
---

# Instrumenting Code: Spans & Context Propagation

## What was run

Two real services, `service_a_otel.py` (port 9501) and `service_b_otel.py` (port 9502), each with its own real
OpenTelemetry `TracerProvider`. Service A creates a real span, injects it into real outgoing HTTP headers using
OpenTelemetry's own W3C-standard propagator, and calls Service B; Service B extracts that real header and creates
its own span as a real child of A's. Both services' real exported span data was inspected directly to confirm the
propagation actually worked.

## A real header carrying a real trace across a real process boundary

```python
# service_a_otel.py
with tracer.start_as_current_span("handle") as span:
    headers = {}
    inject(headers)   # OpenTelemetry's own W3C traceparent injection
    r = httpx.get("http://localhost:9502/compute", headers=headers, timeout=5.0)
```

```
real outgoing headers: {'traceparent': '00-f41245265bfc04a5cc4c343c0aa68abd-d4e0cbb98785ec35-03'}
```

```python
# service_b_otel.py
ctx = extract(dict(request.headers))   # real W3C traceparent extraction
with tracer.start_as_current_span("compute", context=ctx) as span:
    ...
```

## The real proof: identical trace IDs, exact parent-child linkage

```
# Service A's real exported span:
"trace_id": "0xf41245265bfc04a5cc4c343c0aa68abd"
"span_id":  "0xd4e0cbb98785ec35"

# Service B's real exported span:
"trace_id": "0xf41245265bfc04a5cc4c343c0aa68abd"   <- identical, across a real process boundary
"span_id":  "0x9846c028a05f5171"
"parent_id": "0xd4e0cbb98785ec35"                   <- exactly Service A's span_id
```

Two completely separate Python processes, each with its own independent `TracerProvider` and no shared memory,
produced spans sharing the **exact same real trace ID**, with Service B's span correctly recording Service A's
span as its real parent — all of it carried by nothing more than the one real `traceparent` string sitting in an
ordinary HTTP header.

## How It Actually Works

A **span** is one real unit of work with a start time, an end time, attributes, and (critically) an ID — `handle`
and `compute` above are two real, separate spans. **Context propagation** is the mechanism that lets a new span,
created in a different process, know it belongs to the same real trace as the span that triggered it: the calling
service serializes its current span's identity into a standard, real header format (`traceparent`, defined by the
W3C Trace Context specification — `00-<trace_id>-<span_id>-<flags>`, visible directly in the real header shown
above) and the receiving service deserializes it before creating its own span, explicitly passing that extracted
context in (`context=ctx`) so the new span knows which trace and which parent it belongs to.

This is the real mechanism that makes a distributed trace possible across services written independently, by
different teams, potentially in different languages — as long as every service's tracing library implements the
same real, standard W3C header format, propagation works regardless of what's on either end. This is the same
real `x-trace-id`/`x-parent-span-id` pattern this course's own
[Microservices, Level 3, Lesson 8](https://sigilipelli.github.io/microservices-mastery-path/level-3/08-distributed-tracing/)
built by hand with custom headers — OpenTelemetry's `inject`/`extract` functions do the identical real job, using
a standardized header format instead of a custom one, which is precisely the interoperability benefit of using a
real standard over a hand-rolled scheme.

## Common mistakes

- Forgetting to call `inject()` before an outgoing call, or `extract()` before creating the receiving span —
  without both halves of this real mechanism, each service's spans would get their own independent, unrelated
  trace IDs, breaking the trace into disconnected fragments with no way to reassemble them after the fact.
- Manually constructing or parsing the `traceparent` header instead of using the library's real `inject`/`extract`
  functions — the W3C format has precise encoding rules (version, trace ID, span ID, flags, each a specific
  length and position) that are easy to get subtly wrong by hand.
- Propagating context across an HTTP call but forgetting to do the same for a non-HTTP hop (a message queue, a
  background job) — any hop that doesn't carry the context forward breaks the trace at that exact point, the
  same failure mode this course's manual-propagation lesson warned about directly.

## Exercise

Add a third real service, `service_c_otel.py`, that `service_b_otel.py`'s `/compute` endpoint calls in turn,
using the identical `inject`/`extract` pattern. Confirm, by inspecting all three services' real exported span
data, that all three spans share the exact same real `trace_id`, and that the parent-child chain is correct
across all three hops — A's span is B's parent, B's span is C's parent — not just across the first two.
