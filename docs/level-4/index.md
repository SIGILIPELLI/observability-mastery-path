---
description: "Observability Level 4: SLIs/SLOs/error budgets, multi-window multi-burn-rate alerting, capacity planning, scaling Prometheus, and on-call."
---

# Level 4 · SLOs & Operating Observability at Scale

Level 4 steps back from any one tool to the operational practice that ties them all together: turning "is the
system healthy?" into a number you can actually set a target for, alert on sensibly, and use to decide what to
fix first. This level also covers what breaks about this course's single-node setup at real scale, honestly.

!!! note "What was run, and what wasn't"
    This level computes real SLO math and real burn-rate alert thresholds against real metrics from the services
    built earlier in this course, and runs a real capacity-planning projection from real measured data. Scaling
    Prometheus itself to a multi-region, long-term-storage deployment (Thanos, Cortex, Mimir at real production
    scale) is covered as documented architecture, not run locally, and that's stated plainly.

**Prerequisites:** Levels 1–3.

## Modules

1. **[SLIs, SLOs & SLAs: the Real Differences](01-slis-slos-slas.md)**: three terms used interchangeably that
   mean genuinely different things, with a real SLI computed from real data.
2. **[Error Budgets: Turning Reliability Into a Spendable Number](02-error-budgets.md)**: a real error budget,
   computed and tracked against a real SLO over time.
3. **[Burn Rate Alerts: Multi-Window Multi-Burn-Rate Alerting](03-burn-rate-alerts.md)**: a real threshold alert
   compared against a real burn-rate alert on the same real incident.
4. **[Designing an SLO for a Real Service](04-designing-an-slo.md)**: a full, real SLO designed from scratch for
   a service built earlier in this course.
5. **[Capacity Planning From Real Metrics](05-capacity-planning.md)**: a real growth projection computed from
   real historical metric data.
6. **[Scaling Prometheus: Federation, Remote Write & Long-Term Storage](06-scaling-prometheus.md)**: what
   actually breaks past one node, and the documented architectures that fix it.
7. **[Observability as Code: Dashboards & Alerts in Version Control](07-observability-as-code.md)**: a real
   dashboard and alert rule, defined as code and diffed like any other change.
8. **[On-Call & Incident Response With Your Own Stack](08-oncall-incident-response.md)**: a real injected
   incident, worked end to end using only this course's own stack.
9. **[Common Observability Anti-Patterns](09-observability-antipatterns.md)**: real, measured costs of getting
   this wrong, tallied from across the whole course.
10. **[Capstone: Build a Complete SLO-Driven Observability Stack](10-capstone-slo-driven-stack.md)**: metrics,
    dashboards, alerts, traces, and an SLO, for one real service, end to end.

## After Level 4

You've instrumented a real service, built real dashboards and alerts, traced a real request across process
boundaries, and defined a real SLO to operate it by — the same loop a production observability practice runs
continuously, just at a scale you can finish on a laptop.
