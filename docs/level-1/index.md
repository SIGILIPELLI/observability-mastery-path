---
description: "Observability Level 1: the Prometheus data model, instrumenting real metrics, PromQL basics and aggregation, recording rules, and the TSDB."
---

# Level 1 · Metrics & Prometheus Fundamentals

Level 1 builds the one mental model everything else in this course depends on: a Prometheus metric is a named,
labeled time series, and nearly every PromQL surprise — counter resets, histogram quantile math, why `rate()`
exists at all — falls directly out of understanding how that time series is actually stored and queried. Every
query and config shown here was run against a real, local Prometheus server.

!!! note "What was run, and what wasn't"
    This course runs a real local Prometheus 3.x server, scraping real metrics endpoints exposed by small real
    Python services using the `prometheus-client` library. Every PromQL result and every piece of terminal output
    in this level is real, observed output from that server. Where a lesson describes behavior at a scale this
    single-node lab couldn't reproduce (a multi-region federation setup, for instance), it says so plainly.

**Prerequisites:** basic command-line comfort and some exposure to HTTP APIs. No prior monitoring-system
experience assumed. Python 3.10+ is used for the real instrumented examples.

## Modules

1. **[What Observability Actually Means: Metrics, Logs & Traces](01-what-observability-means.md)**: three lossy
   projections of the same event, and when each one actually answers your question.
2. **[The Prometheus Data Model: Time Series, Labels & Samples](02-prometheus-data-model.md)**: what a metric
   actually is under the hood, and why label cardinality matters from day one.
3. **[Instrumenting an App: Counters, Gauges, Histograms & Summaries](03-instrumenting-metric-types.md)**: the
   four metric types, and a real, observable mistake each one is prone to.
4. **[Scraping: Targets, Service Discovery & Pull vs Push](04-scraping-service-discovery.md)**: why Prometheus
   calls you, not the other way around, and what that trades away.
5. **[PromQL Basics: Selecting & Filtering Time Series](05-promql-basics.md)**: real queries against a real
   `/metrics` endpoint.
6. **[PromQL Aggregation: sum, rate() & Why Counters Need It](06-promql-aggregation-rate.md)**: the single most
   common real PromQL mistake, reproduced and fixed.
7. **[Histogram Quantiles: Why You Can't Average a Histogram](07-histogram-quantiles.md)**: a real, measured case
   where averaging two histograms' p99s gives a mathematically wrong answer.
8. **[Recording Rules: Pre-Computing Expensive Queries](08-recording-rules.md)**: a real query timed before and
   after being turned into a recording rule.
9. **[Prometheus Storage: the TSDB on Disk & Retention](09-prometheus-storage-tsdb.md)**: real block files, real
   retention behavior, inspected directly.
10. **[Project: Instrument and Scrape a Real Multi-Service App](10-project-instrument-scrape.md)**: a real
    multi-service app, fully instrumented and scraped end to end.

## After Level 1

Level 2 turns raw metrics into dashboards a human actually looks at, and alerts that page a human only when it's
actually worth waking them up — plus centralized logging, for the things metrics alone can't tell you.
