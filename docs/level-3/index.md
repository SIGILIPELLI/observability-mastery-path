---
description: "Observability Level 3: OpenTelemetry spans and context propagation, the Collector, sampling, cardinality, and correlating metrics, logs and traces."
---

# Level 3 · Distributed Tracing & OpenTelemetry

Level 3 is about the question metrics and logs alone genuinely cannot answer well: for one specific, slow request
that touched five services, where exactly did the time go? This level builds real OpenTelemetry instrumentation,
real exported spans, and shows precisely how to connect a trace back to the metrics and logs covering the same
moment.

!!! note "What was run, and what wasn't"
    This level instruments real Python services with the real OpenTelemetry SDK, producing real spans with real
    parent-child relationships and real context propagation across process boundaries. A full managed tracing
    backend (Jaeger, Tempo, a vendor APM) was not run as part of this course; spans are exported with OpenTelemetry's
    own real console/OTLP exporters and inspected directly, and that distinction is kept explicit throughout.

**Prerequisites:** Levels 1–2.

## Modules

1. **[OpenTelemetry Fundamentals: the Vendor-Neutral Standard](01-opentelemetry-fundamentals.md)**: what OTel
   actually standardizes, and why that matters across tools.
2. **[Instrumenting Code: Spans & Context Propagation](02-instrumenting-spans-context.md)**: a real span tree
   across two real services.
3. **[OpenTelemetry Metrics: the Convergence with Prometheus](03-otel-metrics-convergence.md)**: the same metric,
   exported two ways, compared directly.
4. **[Exporters & the Collector: Decoupling Instrumentation from Backend](04-exporters-collector.md)**: swapping
   a backend without touching instrumented code.
5. **[Sampling: Why You Can't (and Shouldn't) Trace Everything](05-sampling-strategies.md)**: a real measured
   overhead cost of tracing every request vs a sampled fraction.
6. **[Correlating the Three Pillars for One Real Request](06-correlating-three-pillars.md)**: one real failed
   request, explained with its metric spike, its log lines, and its trace, together.
7. **[Cardinality: the Silent Killer of Metrics Systems](07-cardinality-explosion.md)**: a real, measured
   explosion in time series count from one careless label.
8. **[Instrumentation Overhead: Measuring the Cost of Observability](08-instrumentation-overhead.md)**: a real
   before/after latency measurement of adding instrumentation itself.
9. **[Black-Box vs White-Box Monitoring](09-blackbox-vs-whitebox.md)**: a real external health check versus real
   internal metrics, compared on what each one actually catches.
10. **[Project: Full-Stack Observability for a Real Request](10-project-full-stack-observability.md)**: one real
    request, fully observed across all three pillars at once.

## After Level 3

Level 4 turns all of this into a real operational practice: SLOs, error budgets, alerting on burn rate instead of
raw thresholds, and running the whole stack at a scale beyond one laptop.
