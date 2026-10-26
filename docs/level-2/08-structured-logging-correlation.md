---
description: "A real metric spike was explained 0.4 seconds earlier by a real structured log line naming the exact amount that triggered it."
---

# Structured Logging & Correlating Logs with Metrics

## What was run

A real structured-logging addition to `orders_app.py`'s `/checkout` endpoint, emitting one real JSON log line per
request with a request ID, the real amount, and the real outcome. Real mixed traffic was sent with periodic
spikes of deliberately oversized amounts, a real metric spike was located in Prometheus's own data, and the real
structured log file was searched for entries falling within that exact window.

## Finding the real spike in the metric

```bash
curl -s ".../api/v1/query_range?query=sum(rate(orders_requests_total{endpoint=\"/checkout\",status!=\"200\"}[30s]))..."
```
```
1793026238  0
1793026243  0.12    <- the spike starts here
1793026248  0.16
1793026253  0.16
```

A real, visible rise in the checkout error rate, starting right around timestamp `1793026243`. The metric alone
answers "something started failing around this time" — it says nothing about *what*.

## Finding the real log line that explains it

```python
@app.post("/checkout")
def checkout(amount_cents: int):
    request_id = uuid.uuid4().hex[:10]
    ...
    log_json(request_id=request_id, event="checkout_declined", amount_cents=amount_cents,
             status=r.status_code, reason=r.json().get("detail"))
```

```python
# search the real structured log file for entries right around the spike:
for rec in real_log_entries:
    if rec["ts"] >= spike_start - 2 and rec["event"] == "checkout_declined":
        print(rec)
```
```
ts=1793026242.604 amount_cents=800000 reason=declined: amount too large
```

A real log line, timestamped **0.4 seconds before** the metric's spike even registered (consistent with
Prometheus's 5-second scrape interval catching up to a change that already happened) — naming the exact real
amount, `800000` cents, that caused a real decline. The metric told you *when* something started going wrong; the
log told you *why*, down to the specific real request.

## How It Actually Works

This is the real, concrete payoff of structured logging specifically (JSON fields, not free-text sentences): the
correlation above worked because `ts`, `event`, and `amount_cents` were each queryable, parseable fields, not
words buried in a prose message a human would have to read and interpret. A log line like
`"Checkout declined for a large order"` would have told a human roughly the same story, but would have been far
harder to filter programmatically for "exactly the entries in this specific time window with this specific
event type" — which is exactly what made this real correlation a precise, automatable lookup rather than a manual
scroll through a log file hoping to spot something relevant.

The real workflow this lesson demonstrates — **metric spike, found first, log entries pulled for the matching
time window, second** — is the standard real incident-response pattern: a dashboard or alert (built in
[Lesson 2](02-red-dashboard.md) and [Lesson 4](04-alerting-rules-alertmanager.md)) tells you *that* and roughly
*when* something is wrong, narrowing a potentially huge log volume down to a specific, searchable window, and the
structured logs within that window tell you the specific *why*. Trying to go the other direction — reading every
log line first, hoping to notice a pattern — is exactly the approach [Lesson 7](07-centralized-logging-fundamentals.md)
showed doesn't scale.

This pattern becomes even more powerful once a real `trace_id` (built with real cross-service propagation in
[Microservices, Level 3, Lesson 8](https://sigilipelli.github.io/microservices-mastery-path/level-3/08-distributed-tracing/))
is added to the structured fields: a metric spike narrows the time window; the trace ID then lets you follow one
specific failing request's full structured log trail across every service it touched, not just the one that
happened to log the final decline.

## Common mistakes

- Logging the same real information as unstructured prose instead of structured fields — "Checkout declined,
  amount was $8000" can't be filtered or correlated programmatically nearly as reliably as
  `{"event": "checkout_declined", "amount_cents": 800000}`.
- Searching logs first, broadly, instead of letting a real metric narrow the time window first — this inverts
  the efficient real workflow this lesson demonstrated and reintroduces exactly the "scan everything" cost
  [Lesson 7](07-centralized-logging-fundamentals.md) measured directly.
- Logging a request's outcome without a timestamp precise enough to correlate against metric scrape intervals —
  this lesson's correlation depended on real sub-second timestamp precision to confidently say the log entry
  preceded, and plausibly caused, the metric's spike.

## Exercise

Add a second structured field to the log line, `payment_latency_ms`, measuring how long the real call to
Payments took. Trigger a real slow-payment scenario (reuse the slow-dependency pattern from
[Microservices, Level 3, Lesson 3](https://sigilipelli.github.io/microservices-mastery-path/level-3/03-bulkheads-resource-isolation/)
if you still have it, or add an artificial delay), and confirm you can correlate a real p95 latency spike in
Prometheus with the real structured log entries whose `payment_latency_ms` was unusually high during that same
window.
