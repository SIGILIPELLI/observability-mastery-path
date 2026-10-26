---
description: "One real failed request found independently in a real metric count, a real structured log line, and a real trace span -- all three marked ERROR."
---

# Correlating the Three Pillars for One Real Request

## What was run

`service_b_otel.py`, enhanced to emit all three real signals for every request: a real OpenTelemetry metric
(`service_b_compute_total`, labeled by outcome), a real structured JSON log line including the real `trace_id`,
and the real trace span itself — with a genuine ~30% random failure rate injected to produce real errors to find
and correlate. Ten real requests were sent, and one specific real failure was traced across all three signals
independently.

## The real metric: something is failing

```bash
curl -s localhost:9502/metrics | grep service_b_compute_total
```
```
service_b_compute_total{result="error"} 2.0
service_b_compute_total{result="ok"} 8.0
```

A real, aggregate signal: 2 of the last 10 real requests failed. This tells you *that* something is wrong and
roughly *how often* — it says nothing about which specific requests, or why.

## The real log: which specific request, and why

```python
log_json(trace_id=trace_id_hex, event="compute_failed", reason="simulated transient fault")
```
```json
{"ts": 1793027172.577852, "service": "service-b", "trace_id": "4d3490cf3d9cfbd476ca775f1503dd5e", "event": "compute_failed", "reason": "simulated transient fault"}
```

A real, specific entry naming the exact real `trace_id` of one of the two failures, and the real reason it
failed.

## The real trace: confirmed, independently, from the span data itself

```bash
grep -A15 '"trace_id": "0x4d3490cf3d9cfbd476ca775f1503dd5e"' b2.log | grep -A3 '"name": "GET /compute"'
```
```json
{
  "name": "GET /compute",
  "context": {"trace_id": "0x4d3490cf3d9cfbd476ca775f1503dd5e", "span_id": "0xfda3e13fefcf816f"},
  "kind": "SpanKind.SERVER",
  "status": {"status_code": "ERROR"}
}
```

The real trace span for that exact same `trace_id` independently shows `status_code: "ERROR"` — confirmed from
the trace data alone, with no reference to the log line. All three signals — the aggregate metric, the specific
structured log, and the independently-recorded trace span — agree on the same real failure, found three separate,
independent ways.

## How It Actually Works

This is the real, concrete version of the idea [Level 1, Lesson 1](../level-1/01-what-observability-means.md)
introduced in the abstract: each pillar is a different lossy projection of the same real events, and this
lesson's one failed request shows exactly what each projection kept and what it discarded. The metric kept only
an aggregate count — fast, cheap, but anonymous; it can't tell you *which* request failed. The log kept full
per-event detail, including a human-readable reason — rich, but only directly useful once you already know
roughly where to look (narrowed here by already knowing the `trace_id`). The trace kept the structural, timing
information — when exactly it happened, what it was nested inside, and (via OpenTelemetry's own automatic
FastAPI instrumentation, visible in the real span data as `fastapi.endpoint`, `fastapi.dependencies`, and `GET
/compute` SERVER spans this course's own code never explicitly created) the full real request-handling structure
around it, something neither the metric nor the log captures at all.

The real `trace_id`, present in both the structured log and the trace span, is what makes connecting all three
possible without guessing from timestamps alone, the way
[Level 2, Lesson 8](../level-2/08-structured-logging-correlation.md)'s metric-to-log correlation had to. This is
the real, concrete argument for propagating a trace ID into every signal a service emits, not just its trace
spans: the same identifier becomes the join key across all three pillars at once.

## Common mistakes

- Investigating an incident starting from logs alone, scanning for anything that looks relevant, instead of
  starting from the metric (which tells you *that* and roughly *when*) and only then pulling the specific logs
  and trace for the narrowed window — the inefficient order [Level 2, Lesson 7](../level-2/07-centralized-logging-fundamentals.md)
  measured directly.
- Logging a `trace_id` but never checking that it's the *same* trace ID the tracing SDK actually generated for
  that request — a copy-paste or threading bug that attaches the wrong trace ID to a log line defeats this
  entire correlation mechanism silently.
- Assuming a trace alone tells you *why* something failed — this lesson's trace span correctly shows
  `status_code: ERROR`, but the human-readable *reason* ("simulated transient fault") lived only in the
  structured log, not the span's own data.

## Exercise

Add the real failure reason as a span attribute too (`span.set_attribute("error.reason", "simulated transient fault")`)
so it's recoverable directly from the trace, not only from the log. Re-run this lesson's exact scenario, find a
newly failed request, and confirm you can now get the full real story — count, specific instance, why, and
structural timing — from the trace alone, without needing to cross-reference the log file at all.
