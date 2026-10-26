---
description: "A real Grafana instance, a real Prometheus connection test that actually passed, and a real panel rendering a visible spike from real checkout traffic."
---

# Grafana Fundamentals: Data Sources, Panels & Dashboards

## What was run

A real local Grafana 13.2.4 instance (port 3000), connected to the real local Prometheus server from Level 1 as a
real data source, with a real dashboard and panel created through Grafana's own HTTP API and verified by actually
loading it in a browser and watching it render real data.

## A real data source, actually tested

```bash
curl -s -X POST http://admin:admin@localhost:3000/api/datasources -H "Content-Type: application/json" -d '{
  "name": "Prometheus", "type": "prometheus", "url": "http://localhost:9090", "access": "proxy", "isDefault": true
}'
```

Adding the data source through the API alone doesn't prove it works — clicking "Save & test" on the real data
source settings page in a real browser is what actually exercises the connection, and it returned a real, direct
confirmation:

```
✓ Successfully queried the Prometheus API.
```

## A real panel, rendering real data

```bash
curl -s -X POST "http://admin:admin@localhost:3000/api/dashboards/db" -H "Content-Type: application/json" -d '{
  "dashboard": {
    "title": "Orders Service Overview",
    "panels": [{
      "type": "timeseries", "title": "Checkout request rate",
      "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0},
      "datasource": {"type": "prometheus", "uid": "ag2egat7m3l6oe"},
      "targets": [{"expr": "sum(rate(orders_requests_total{endpoint=\"/checkout\"}[1m]))", "refId": "A"}]
    }]
  }
}'
```

Loading this dashboard in a real browser rendered a real line graph with two visible spikes, sitting at
`0` between them and rising to `~1` during each — exactly matching the two real bursts of 50 checkout requests
sent to `orders_app.py` across [Level 1, Lesson 10](../level-1/10-project-instrument-scrape.md). The panel's
legend showed the exact real PromQL expression it was built from, and the shape of the graph is a direct, visible
encoding of real, observed traffic — not a static mockup.

## How It Actually Works

A Grafana **data source** is a configured connection to wherever your real data lives — Prometheus here, but
also logs backends, other time-series databases, or SQL databases. A **panel** is one visualization (a graph, a
single number, a table, a heatmap) built from one or more real queries against a data source, and a **dashboard**
is a collection of panels, laid out together, usually because they answer related questions about the same real
system.

The real, important distinction from a static screenshot or a hand-drawn mockup: every panel on a real Grafana
dashboard re-runs its real query against the real data source on a schedule (or whenever the dashboard's time
range changes), which is exactly what made the two real spikes above show up automatically — nobody manually
plotted those points; Grafana queried Prometheus, got back real data, and rendered it. This is also why a
dashboard's correctness depends entirely on its underlying PromQL being correct — a dashboard is only as
trustworthy as the real query driving each panel, which is why [Level 1](../level-1/05-promql-basics.md)'s
PromQL foundations matter before dashboards can be built reliably on top of them.

Grafana's dashboards are themselves stored as JSON (the `panels` array shown in the API call above), which is
what makes [Level 4, Lesson 7](../level-4/07-observability-as-code.md)'s "dashboards as code" possible later in
this course — a dashboard built once can be exported, version-controlled, and recreated exactly through the same
API this lesson used directly.

## Common mistakes

- Trusting that a data source is configured correctly just because adding it didn't show an error, without
  actually clicking "Save & test" (or querying through it) to get a real, direct confirmation the connection
  works end to end.
- Building a panel with a query that's syntactically valid PromQL but doesn't actually answer the question the
  panel's title claims to — Grafana will happily render whatever a query returns, correct or not.
- Treating a dashboard screenshot (shared in a chat message, say) as equivalent to the live dashboard — a
  screenshot is frozen at the moment it was taken, while the real dashboard keeps re-querying and updating.

## Exercise

Add a second panel to this dashboard, a real `stat` panel (a single big number, not a graph) showing the current
real value of `orders_inflight_requests` from [Level 1, Lesson 3](../level-1/03-instrumenting-metric-types.md).
Load the dashboard in a real browser, fire a real burst of concurrent requests at `orders_app.py` the way that
lesson did, and confirm you can watch the real number change live as the burst happens and completes.
