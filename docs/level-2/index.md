---
description: "Observability Level 2: Grafana dashboards, the RED/golden-signals/USE methods, Alertmanager routing and silencing, and centralized logging."
---

# Level 2 · Dashboards, Alerting & Logs

Level 2 is where raw metrics become something a human actually uses: a dashboard that answers "is the system
healthy?" at a glance, and an alert that pages someone only when it's genuinely worth interrupting them. This
level also covers centralized logging — the third pillar, for the specific class of question metrics alone can't
answer.

!!! note "What was run, and what wasn't"
    This level runs a real local Grafana instance connected to the real Prometheus server from Level 1, with real
    dashboards built and real alert rules that are actually triggered on purpose. A full managed logging backend
    (a hosted ELK/Loki cluster) was not run as part of this course; centralized logging's real mechanics are shown
    against a real local log aggregation setup instead, and that distinction is kept explicit.

**Prerequisites:** Level 1.

## Modules

1. **[Grafana Fundamentals: Data Sources, Panels & Dashboards](01-grafana-fundamentals.md)**: a real Prometheus
   data source connected, a real panel built.
2. **[Building a Real RED Dashboard: Rate, Errors, Duration](02-red-dashboard.md)**: the single most common
   service dashboard pattern, built against real traffic.
3. **[The Four Golden Signals & the USE Method](03-golden-signals-use-method.md)**: two competing frameworks for
   "what should a dashboard show," and when each one fits better.
4. **[Alerting Rules: Prometheus & Alertmanager Basics](04-alerting-rules-alertmanager.md)**: a real alert rule
   that really fires against real injected load.
5. **[Alert Routing, Grouping & Silencing](05-alert-routing-grouping-silencing.md)**: real Alertmanager routing
   rules, tested against real simultaneous firings.
6. **[Avoiding Alert Fatigue: Designing Alerts That Page the Right Person](06-avoiding-alert-fatigue.md)**: a real
   before/after alert rule rewrite, measured by false-positive rate.
7. **[Centralized Logging Fundamentals: Why grep Doesn't Scale](07-centralized-logging-fundamentals.md)**: a real
   timing comparison between `grep` across files and a real centralized query.
8. **[Structured Logging & Correlating Logs with Metrics](08-structured-logging-correlation.md)**: real JSON logs
   correlated with a real metric spike.
9. **[Log-Based Metrics vs Native Metrics](09-log-based-vs-native-metrics.md)**: deriving a metric from logs,
   measured against the real cost of doing it that way.
10. **[Project: A Real Alert That Fires, Routes & Pages](10-project-real-alert-pages.md)**: a complete, real
    alerting pipeline from a real failure to a real notification.

## After Level 2

Level 3 adds the third pillar in depth — distributed tracing with OpenTelemetry — and shows how to correlate
metrics, logs, and traces for one real request, not just view them side by side.
