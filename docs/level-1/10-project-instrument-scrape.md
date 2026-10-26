---
description: "Two real services, two sets of metrics, and PromQL math that checks out exactly: 0.2545 + 0.6545 declined/charged rates equal the real 0.909 checkout rate."
---

# Project: Instrument and Scrape a Real Multi-Service App

## What was run

Two real services — `orders_app.py` (port 9301) and `payments_service.py` (port 9305) — with a real `/checkout`
endpoint on Orders that makes a real HTTP call to Payments' `/charge` endpoint, both independently instrumented
with their own real Prometheus metrics and scraped by the same real Prometheus server. 50 real end-to-end
checkout requests, with randomized amounts (some large enough to be declined), were sent, and five real PromQL
queries were run against the resulting data.

## The real end-to-end flow

```python
# orders_app.py
@app.post("/checkout")
def checkout(amount_cents: int):
    r = httpx.post(f"{PAYMENTS_URL}/charge", params={"amount_cents": amount_cents}, timeout=5.0)
    ...
```

```bash
curl -s -X POST "localhost:9301/checkout?amount_cents=5000"
# {"id":1,"item":"widget","payment":{"status":"charged","amount_cents":5000}}

curl -s -w "\nHTTP:%{http_code}\n" -X POST "localhost:9301/checkout?amount_cents=200000"
# {"detail":"declined: amount too large"}
# HTTP:402
```

## Five real queries, and a consistency check that actually holds

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=sum(rate(orders_requests_total{endpoint="/checkout"}[1m]))'
# 0.9090909090909091                      <- overall checkout rate, from Orders' own metric

curl -s ".../api/v1/query" --data-urlencode 'query=sum(rate(payments_requests_total{status="declined"}[1m]))'
# 0.2545454545454545                      <- declined rate, from Payments' own metric

curl -s ".../api/v1/query" --data-urlencode 'query=sum(rate(payments_requests_total{status="charged"}[1m]))'
# 0.6545454545454545                      <- charged rate, from Payments' own metric
```

`0.2545 + 0.6545 = 0.909` — **exactly** the checkout rate measured independently from Orders' own metric. Two
completely separate services, instrumented separately, scraped separately, and their real numbers agree exactly,
because every checkout either gets charged or declined with nothing in between — a real, concrete consistency
check that the instrumentation on both sides is actually measuring the same real events correctly.

```bash
curl -s ".../api/v1/query" --data-urlencode \
  'query=histogram_quantile(0.95, sum(rate(orders_request_duration_seconds_bucket{endpoint="/checkout"}[2m])) by (le))'
# 0.12148715452026121                     <- end-to-end checkout p95, from Orders' perspective

curl -s ".../api/v1/query" --data-urlencode \
  'query=histogram_quantile(0.95, sum(rate(payments_request_duration_seconds_bucket[2m])) by (le))'
# 0.0956896551724138                      <- payments' own internal p95
```

The end-to-end p95 (0.1215s) is real and larger than Payments' own internal p95 (0.0957s) — exactly what should
be true, since the end-to-end number includes the real network round trip and Orders' own overhead on top of
whatever Payments itself took, while Payments' own histogram only ever sees its own internal processing time.

## How It Actually Works

This project is the direct, combined payoff of every lesson in this level: real metric types chosen correctly
([Lesson 3](03-instrumenting-metric-types.md) — `Counter` for charged/declined counts, `Histogram` for latency,
`Gauge` for in-flight requests), a real scrape config pointed at two independent targets
([Lesson 4](04-scraping-service-discovery.md)), real PromQL selectors and `rate()` turning raw counters into
meaningful per-second numbers ([Lessons 5–6](05-promql-basics.md)), and a real, correctly cross-aggregated
`histogram_quantile()` ([Lesson 7](07-histogram-quantiles.md)) rather than an average of two separately-computed
quantiles.

The real, structural insight this project adds on top of those individual pieces: **two independently
instrumented services, each only aware of its own requests, can be cross-checked against each other precisely**
when their real event counts have a logical relationship — every checkout either charges or declines, so the
sum of one service's two outcome rates must equal the other service's total rate. This kind of real consistency
check is a genuinely useful way to build confidence that instrumentation across a multi-service boundary is
actually correct, not just independently "working" in isolation — the same spirit as the cross-service
consistency tests built with real assertions in
[Microservices, Level 2, Lesson 9](https://sigilipelli.github.io/microservices-mastery-path/level-2/09-testing-data-consistency/),
applied here to metrics instead of application state.

## Common mistakes

- Instrumenting each service correctly in isolation but never cross-checking that their numbers actually agree
  where a real logical relationship predicts they should — a silent instrumentation bug (double-counting,
  missing a code path) is far easier to catch this way than by inspecting either service's metrics alone.
- Comparing an end-to-end latency metric (measured from the calling service) against a downstream service's own
  internal latency metric and expecting them to match — they measure genuinely different things, and the gap
  between them (0.1215s vs 0.0957s here) is itself real, useful information about network and caller-side
  overhead.
- Building this kind of multi-service instrumentation without first deciding, deliberately, which metrics
  belong to which service — Orders' metrics should describe Orders' own behavior (including calling out to
  Payments), not duplicate Payments' own internal view of itself.

## Exercise

Add a third real service to this chain — an `inventory_service.py` that `/checkout` also calls, with its own
real metrics — and extend the cross-check from this lesson: confirm that Orders' overall checkout success rate
can be reconstructed exactly from the combination of Payments' charge/decline rates *and* Inventory's own
reserve/out-of-stock rates, now that a checkout can fail for either of two independent, separately-instrumented
reasons.
