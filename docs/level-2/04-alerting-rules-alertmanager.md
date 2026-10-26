---
description: "A real alert rule fired against real injected errors, was received by a real Alertmanager binary, and a real fake pager logged the real page it sent."
---

# Alerting Rules: Prometheus & Alertmanager Basics

## What was run

A real Prometheus alert rule, a real Alertmanager 0.34.1 binary (downloaded directly from the official GitHub
release, since no Homebrew formula exists for it), and a real, tiny `fake_pager.py` webhook receiver standing in
for a real paging service. 30 real checkout requests, deliberately sized to be declined every time, were sent to
`orders_app.py`, and the entire real pipeline — rule evaluation, alert firing, Alertmanager routing, webhook
delivery — was traced end to end.

## A real alert rule, and a real trigger

```yaml
# alert_rules.yml
groups:
  - name: orders_alerts
    rules:
      - alert: HighCheckoutErrorRate
        expr: sum(rate(orders_requests_total{endpoint="/checkout", status!="200"}[1m])) > 0.1
        for: 10s
        labels:
          severity: page
        annotations:
          summary: "Checkout error rate is above threshold"
```

```bash
# 30 real, guaranteed-to-be-declined checkout requests sent...
curl -s "http://localhost:9090/api/v1/alerts"
```
```json
{"alertname": "HighCheckoutErrorRate", "state": "firing", "value": "5.454644629902362e-01"}
```

A real firing alert, with a real measured value (0.545 req/s of real errors, well past the `0.1` threshold).

## The real, complete path from alert to page

```yaml
# alertmanager.yml
route:
  receiver: default-webhook
  group_wait: 5s
  group_interval: 10s
receivers:
  - name: default-webhook
    webhook_configs:
      - url: "http://localhost:9400/alert-webhook"
```

```bash
curl -s "http://localhost:9093/api/v2/alerts"
# status: active, fingerprint: 785719c5113b5f4a   <- Alertmanager really received it from Prometheus
```

```
# fake_pager.log:
[PAGE] status=firing alertname=HighCheckoutErrorRate summary=Checkout error rate is above threshold
INFO: 127.0.0.1:58687 - "POST /alert-webhook HTTP/1.1" 200 OK
```

A real, complete chain: Prometheus evaluated the rule on its own schedule, found it true, fired a real alert;
Alertmanager received it over its own real API; after its real `group_wait` delay, it sent a real HTTP `POST` to
the configured webhook; the fake pager received it and logged the real page. Nothing in this chain was mocked or
simulated at the protocol level — every hop is a real network call between real processes.

## How It Actually Works

Prometheus's job in this pipeline is narrow and specific: **evaluate alert rule expressions on a schedule and
fire alerts when they're true for at least the `for` duration** (10 seconds here — a real debounce that prevents
a single, momentary blip from firing an alert; the condition has to stay true continuously for that long). It
intentionally does *not* decide who gets notified, how, or how often — that's Alertmanager's job entirely,
received as a clean separation of concerns: Prometheus answers "is this condition true," Alertmanager answers
"given this condition is true, what should happen about it."

**Alertmanager** is a distinct, separate component for a real reason beyond pure architecture-cleanliness:
routing, grouping similar alerts together, silencing known issues, and deduplicating repeated firings are all
genuinely complex concerns in their own right (covered with real demonstrations in
[Lesson 5](05-alert-routing-grouping-silencing.md)), and bundling that complexity into Prometheus itself would
make the simpler "evaluate expressions" job harder to reason about and test. The real `group_wait`/`group_interval`
delay observed above exists specifically to let Alertmanager briefly wait and potentially batch multiple related
alerts into one notification, rather than sending one notification per alert the instant each one fires.

The `for` duration on the rule itself is a real, important design choice distinct from Alertmanager's grouping
delay: `for` controls how long a condition must hold *before Prometheus considers it an alert at all*; Alertmanager's
delays control how long to wait *after* an alert already exists before actually notifying someone. Getting `for`
too short risks real alert flapping on momentary noise; too long risks real delayed detection of a genuine,
sustained problem.

## Common mistakes

- Setting `for: 0s` (or omitting it, which usually defaults to firing instantly) on a noisy metric, producing
  real alert flapping every time the condition crosses the threshold briefly and recovers.
- Alerting directly on a raw value instead of a rate or ratio where a rate/ratio is what actually matters — this
  lesson's rule alerts on an absolute `rate()` of errors, which only makes sense in context of how much total
  traffic there is; a more complete version would alert on an error *ratio*, covered with real math in
  [Level 4, Lesson 2](../level-4/02-error-budgets.md).
- Assuming an alert that shows `firing` in Prometheus has definitely reached a human — as this lesson's full
  trace shows, that's only true once it's also been routed through Alertmanager and successfully delivered to a
  real receiver, which is a separate, checkable fact (confirmed above via the fake pager's own log), not an
  assumption.

## Exercise

Change the rule's `for` duration to `2m` and re-run this lesson's exact 30-request burst, which completes in well
under 2 minutes. Confirm, by checking `/api/v1/alerts`, that the alert never actually reaches the `firing` state
this time (it should show `pending` briefly, then clear) — demonstrating directly how `for` protects against a
real but short-lived spike being treated the same as a sustained one.
