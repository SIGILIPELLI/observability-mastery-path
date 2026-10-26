---
description: "A real metric answers 'how many requests failed' but not 'which one, or why' -- the real gap that shows exactly why logs and traces exist too."
---

# What Observability Actually Means: Metrics, Logs & Traces

## What was run

A real, instrumented FastAPI app, `orders_app.py` (port 9301), exposing real Prometheus metrics at `/metrics/`,
scraped by a real local Prometheus 3.15.1 server (port 9090) every 5 seconds. A real `404` was produced by
requesting a nonexistent order, and the resulting metric was queried with a real PromQL call to Prometheus's own
HTTP API.

## One real failure, and what the metric alone can tell you

```bash
curl -s localhost:9301/orders/999
# {"detail":"order not found"}
```

```bash
curl -s "http://localhost:9090/api/v1/query" --data-urlencode 'query=orders_requests_total'
```
```json
{"metric": {"endpoint": "/orders/{id}", "status": "404", ...}, "value": [1792980064.85, "1"]}
```

That's a real, correct answer to a real question: **how many** requests to `/orders/{id}` have failed with a
`404`? One. But notice everything this number cannot tell you: *which* order ID was requested, *who* called it,
what the request looked like, or what happened immediately before or after it in that same client's session. The
metric is a count — deliberately small, deliberately cheap to store even at huge request volume, and deliberately
missing every detail that would make it expensive.

## How It Actually Works

Metrics, logs, and traces are often called the "three pillars of observability," but a more useful way to think
about them is as three different **lossy projections** of the same underlying stream of real events happening in
your system. Each one throws away different information to stay cheap and useful at scale:

- **Metrics** throw away per-event detail and keep only aggregated numbers over time, usually with a small set of
  labels (as `orders_requests_total{endpoint="/orders/{id}", status="404"}` does above). This makes them cheap to
  store and fast to query even across millions of real requests, at the cost of never being able to answer "which
  specific one."
- **Logs** keep per-event detail — this course's real structured logs from
  [Microservices, Lesson 6](https://sigilipelli.github.io/microservices-mastery-path/level-4/06-observability-across-boundaries/)
  are a direct example — but at a storage and query cost that scales with request volume, not with how many
  distinct label combinations exist, which is the opposite tradeoff from a metric.
- **Traces** keep the *causal structure* across a single request as it moves through multiple services — exactly
  what [Microservices, Lesson 8](https://sigilipelli.github.io/microservices-mastery-path/level-3/08-distributed-tracing/)
  built directly, with real parent-child spans — answering "where did the time go, structurally, for this one
  request," which neither a metric nor an unstructured pile of logs answers directly.

The real, practical skill this course builds is knowing **which question you're actually asking**, and reaching
for the signal built to answer it cheaply: "is error rate up?" is a metrics question; "what exactly did this one
failing customer's request look like?" is a logs question; "why did this one request take 4 seconds across three
services?" is a traces question. Reaching for the wrong one — grepping logs to answer an aggregate rate question
that a metric would answer instantly, or trying to debug one specific user's issue from an aggregate dashboard
alone — is the single most common real observability inefficiency this course will keep coming back to.

## Common mistakes

- Treating one pillar as a complete replacement for the others — a system with rich metrics and no logs can tell
  you *that* something broke but not *what specifically* happened to any one request; a system with rich logs and
  no metrics can answer specific questions but can't tell you, fast, whether things are generally getting worse.
- Logging so much detail that logs become a de facto (very expensive) metrics system, or aggregating metrics so
  granularly (by, say, individual user ID) that a metric becomes a de facto (very expensive) log — each signal's
  design assumes a specific tradeoff, and fighting that tradeoff costs real money and performance.
- Assuming "observability" just means "we have a dashboard" — a dashboard built from metrics alone, with no way
  to drill from an aggregate spike down to a specific failing request's logs or trace, answers "something is
  wrong" without ever answering "which one, and why."

## Exercise

Add a real log line to `orders_app.py`'s `get_order` function that fires only on the `404` path, including the
requested `oid`. Trigger a few more real `404`s for different order IDs, then answer, using only the metric: "how
many 404s have happened?" — and using only the log: "which specific order IDs were requested that don't exist?"
Write one sentence on which question each signal answered faster.
