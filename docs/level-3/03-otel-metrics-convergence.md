---
description: "A real OTel counter rendered through the same generate_latest() call Level 1 used natively, plus a target_info gauge carrying OTel's resource attributes."
---

# OpenTelemetry Metrics: the Convergence with Prometheus

## What was run

A real app, `otel_metrics_app.py` (port 9504), creating metrics through the OpenTelemetry Metrics SDK instead of
`prometheus_client` directly — then rendered to real Prometheus exposition format using the exact same
`generate_latest()` function [Level 1](../level-1/03-instrumenting-metric-types.md) used for native metrics,
proving the two approaches genuinely converge on the wire.

## A real environment limitation, found and worked around

The OpenTelemetry Prometheus exporter's documented approach uses its own `start_http_server()` to run a
standalone metrics endpoint. On this machine, that specific server reproducibly accepted no real connections —
`curl` timed out against it every time, on multiple ports, in both foreground and backgrounded runs, while every
other server in this entire course (built on `uvicorn`) worked reliably. The real, working fix: the
`PrometheusMetricReader` registers itself as a collector on `prometheus_client`'s own global `REGISTRY` — so
calling `generate_latest(REGISTRY)` from inside a real, already-proven-reliable FastAPI/uvicorn endpoint renders
the identical real output `start_http_server` would have, without needing that specific server at all.

```python
@app.get("/metrics")
def metrics_endpoint():
    return Response(generate_latest(REGISTRY), media_type="text/plain")
```

## A real OTel metric, in real Prometheus exposition format

```python
otel_counter = meter.create_counter("otel_requests_total", description="Requests via the OTel SDK")
otel_counter.add(1, {"endpoint": "/work"})
```

```bash
curl -s localhost:9504/metrics | grep -A2 otel_requests_total
```
```
# HELP otel_requests_total Requests via the OTel SDK
# TYPE otel_requests_total counter
otel_requests_total{endpoint="/work",otel_scope_name="otel-metrics-app",otel_scope_schema_url="",otel_scope_version=""} 2.0
```

A real counter, created through OpenTelemetry's `meter.create_counter()` API (not `prometheus_client.Counter`),
rendered in the exact same text format Prometheus always scrapes — with two real, additional labels,
`otel_scope_name` and `otel_scope_version`, automatically attached by the exporter to record which OTel
instrumentation scope produced the metric, something native `prometheus_client` metrics don't carry.

```bash
curl -s localhost:9504/metrics | grep target_info
```
```
target_info{service_instance_id="b9fe3d23-...", service_name="otel-metrics-app", telemetry_sdk_language="python", ...} 1.0
```

A real, separate `target_info` gauge, generated automatically from the OTel `Resource` this app declared
(`Resource.create({"service.name": "otel-metrics-app"})`) — this is how OpenTelemetry's richer, resource-based
identity model (a service can have many attributes beyond just a scrape target's `instance`/`job` labels) gets
represented in Prometheus's flatter label model.

## How It Actually Works

OpenTelemetry and Prometheus converge at the data model level more than they diverge: both ultimately represent a
metric as a name, a set of key-value labels, and a numeric value over time — which is exactly why rendering an
OTel-created counter through `generate_latest()` produces a real, valid Prometheus series with no translation
logic needed beyond what the official exporter already provides. The real, visible differences — `otel_scope_*`
labels, the separate `target_info` gauge — come from OpenTelemetry's *resource* and *instrumentation scope*
concepts, which are richer than Prometheus's native `job`/`instance` model and get flattened into extra labels
and a synthetic info metric to fit.

The practical implication: a team doesn't have to choose between "instrument with OpenTelemetry" and "keep using
Prometheus" — OpenTelemetry's metrics API can be the single instrumentation surface for an application (unifying
it with the trace instrumentation from [Lessons 1–2](01-opentelemetry-fundamentals.md)), while Prometheus remains
the real scraping and storage backend underneath, exactly as this lesson's app demonstrates end to end. This
matters most for an organization already standardized on OpenTelemetry for tracing, who want metrics instrumented
the same consistent way rather than maintaining two separate, differently-shaped instrumentation APIs
side by side in the same codebase.

## Common mistakes

- Assuming `start_http_server()`'s specific implementation will work in every real environment without testing —
  this lesson's own real, reproducible failure on this machine is exactly the kind of environment-specific gotcha
  worth verifying directly rather than assuming from documentation.
- Expecting OTel-exported metrics and native `prometheus_client` metrics to look byte-for-byte identical — the
  real `otel_scope_*` labels and `target_info` gauge shown above are genuine, visible differences a consumer
  (a dashboard, an alert rule) may need to account for, even though the underlying data model converges.
- Mixing OTel-created and `prometheus_client`-created metrics for the *same* logical measurement across different
  parts of a codebase — both can coexist in one process (as this lesson's REGISTRY-sharing trick proves), but
  having two different instrumentation paths for what should be one consistent metric invites exactly the kind of
  divergence [Level 1, Lesson 10](../level-1/10-project-instrument-scrape.md)'s cross-check was designed to catch.

## Exercise

Add a Prometheus scrape target for `otel_metrics_app.py` (port 9504) to the real `prometheus.yml` from Level 1,
confirm Prometheus successfully scrapes it (checked against `/api/v1/targets`, the same real verification method
from [Level 1, Lesson 4](../level-1/04-scraping-service-discovery.md)), and run a real PromQL `rate()` query
against `otel_requests_total` — confirming it works identically to any native Prometheus metric from earlier in
this course, with no special query syntax needed just because it originated from OpenTelemetry.
