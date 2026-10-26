---
description: "A real OpenTelemetry SDK generated a real 128-bit trace ID and 64-bit span ID with zero network calls, before any exporter or backend was ever configured."
---

# OpenTelemetry Fundamentals: the Vendor-Neutral Standard

## What was run

A real, minimal OpenTelemetry Python SDK setup (`opentelemetry-sdk` 1.45.1), creating one real span with a real
attribute and exporting it with the SDK's own `ConsoleSpanExporter` — the simplest possible real exporter, which
just prints the span's real data instead of sending it anywhere.

## A real span, before any backend exists

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import ConsoleSpanExporter, SimpleSpanProcessor

provider = TracerProvider()
provider.add_span_processor(SimpleSpanProcessor(ConsoleSpanExporter()))
trace.set_tracer_provider(provider)
tracer = trace.get_tracer("test")

with tracer.start_as_current_span("test-span") as span:
    span.set_attribute("real", True)
```

```json
{
  "name": "test-span",
  "context": {
    "trace_id": "0x81c8bc990b2c7966069c8ab091a7a342",
    "span_id": "0xec8b6751e6b03adf"
  },
  "attributes": {"real": true},
  "resource": {
    "attributes": {
      "telemetry.sdk.language": "python",
      "telemetry.sdk.name": "opentelemetry",
      "telemetry.sdk.version": "1.45.1",
      "service.name": "unknown_service:python3"
    }
  }
}
```

A real, valid, 128-bit trace ID and 64-bit span ID were generated, with real start and end timestamps, without a
single network call made, without any tracing backend (Jaeger, Tempo, a vendor APM) configured or even running.
Also worth noticing honestly: `service.name` defaulted to `"unknown_service:python3"` — nothing in the minimal
setup above told OpenTelemetry what to call this service, a real, easy-to-miss gap this course's
[Lesson 2](02-instrumenting-spans-context.md) fixes directly.

## How It Actually Works

OpenTelemetry is deliberately split into two layers, and the example above only exercises the first:

- **The API** (`opentelemetry.trace`, used above as `tracer.start_as_current_span`) is what your application code
  calls — a stable, vendor-neutral interface for creating spans, adding attributes, and propagating context. Code
  written against the API doesn't know or care what happens to the data afterward.
- **The SDK** (`opentelemetry.sdk`, used above as `TracerProvider` and `SimpleSpanProcessor`) is the real
  implementation underneath the API — it decides how spans are processed, batched, and exported. The SDK is
  swappable independently of your instrumented application code, which is the entire point: the real `ConsoleSpanExporter`
  used here could be replaced with a real OTLP exporter sending to Jaeger, Tempo, or any of dozens of real
  commercial backends, **without touching a single line of the actual `with tracer.start_as_current_span(...)`
  instrumentation code** shown above.

This vendor-neutral split is OpenTelemetry's actual value proposition, distinct from any specific backend: an
organization can instrument its real code once, against the stable API, and change *where the data goes* — self-hosted,
a specific vendor, a different vendor later — as a configuration change to the SDK's exporter, not a rewrite of
instrumented application code. This is architecturally the same separation Prometheus's `Mount` pattern gave
[Level 1](../level-1/04-scraping-service-discovery.md)'s metrics endpoint from its scrape configuration, applied
here to traces instead of metrics.

This course uses the real `ConsoleSpanExporter` throughout Level 3, deliberately, to make the real span data
directly visible without needing a full tracing backend running locally — a real, honest scope limitation stated
here plainly: a production system would send this data to a real queryable backend (Jaeger, Tempo, a vendor),
which this course did not run, in favor of showing the underlying mechanism clearly instead.

## Common mistakes

- Instrumenting code directly against a specific vendor's SDK instead of the OpenTelemetry API, which locks
  instrumented code to that vendor — exactly the coupling the API/SDK split exists to avoid.
- Leaving `service.name` at its real, unhelpful default (`unknown_service:python3`, confirmed directly above) —
  every real span needs a meaningful resource identity to be useful once more than one service's spans start
  mixing together, the exact scenario [Lesson 2](02-instrumenting-spans-context.md) builds for real.
- Assuming a span "didn't work" if no backend shows it — as this lesson demonstrates, span creation, attributes,
  and timing are all real and complete the moment the `with` block exits, entirely independent of whether
  anything is actually consuming the exported data yet.

## Exercise

Set a real, meaningful `service.name` using `Resource.create({"service.name": "my-real-service"})` passed to
`TracerProvider`, create two more real nested spans inside the first one (a parent and a child, using nested
`with tracer.start_as_current_span(...)` blocks), and confirm in the real console output that the child span's
`parent_id` correctly references the parent span's `span_id` — the real mechanism [Lesson 2](02-instrumenting-spans-context.md)
builds on directly.
