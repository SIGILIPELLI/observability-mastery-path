---
description: "During a real 100%-failure outage, a black-box health check still returned HTTP 200 healthy, while a white-box query showed a 100% error ratio."
---

# Black-Box vs White-Box Monitoring

## What was run

A real, deliberately naive `/health` endpoint added to `orders_app.py`, checked externally with a plain `curl`
(real black-box monitoring — observing only what a request/response looks like from outside, with no knowledge of
internals) during a real, injected payments outage, compared directly against a real internal PromQL query
against `orders_app.py`'s own metrics (real white-box monitoring) at the exact same moment.

## The real, simultaneous contrast

```python
@app.get("/health")
def health():
    # A real, naive black-box health check: the process can respond, so it says "healthy."
    return {"status": "healthy"}
```

```bash
# a real payments outage injected, real checkout traffic sent (all genuinely failing)...

curl -s -w "\nHTTP:%{http_code}\n" http://localhost:9301/health
# {"status":"healthy"}
# HTTP:200
```

```bash
curl -s ".../api/v1/query" --data-urlencode \
  'query=sum(rate(orders_requests_total{endpoint="/checkout", status!="200"}[2m])) / sum(rate(orders_requests_total{endpoint="/checkout"}[2m]))'
# value: 1
```

**At the exact same real moment**: the black-box health check genuinely, honestly reports `healthy` with a real
`200` — the process *is* up and *can* respond — while the white-box metric shows a real **100% error ratio** for
actual checkout traffic. Neither signal is lying; they're answering two genuinely different real questions, and
only one of them is the question that actually matters to a real user trying to check out.

## How It Actually Works

**Black-box monitoring** observes a system the way an external user or caller would: send a request, check the
response — status code, latency, maybe a response body check. Its real strength is that it tests the system as a
whole, end to end, exactly as a real client experiences it, with zero dependency on internal implementation
details, which also makes it trivially portable across completely different services (the same `curl` approach
works whether the target is written in Python, Go, or anything else). Its real, measured weakness, demonstrated
directly above: it can only check what it's told to check, and a shallow health check (does the process respond
at all?) can stay green through a real, severe internal failure that doesn't happen to affect that one specific
probe.

**White-box monitoring** observes a system from the inside, using the kind of real, detailed instrumentation
built across this entire course — metrics, logs, traces. Its real strength is precisely the detail black-box
monitoring lacks: the white-box query above knows, specifically, that checkout requests are failing, something
the shallow `/health` endpoint has no way to express because nothing in its implementation checks real downstream
dependency health. Its real cost is that it requires the system to actually be instrumented in the first place —
useless against a service you don't control or haven't built observability into.

The real, practical answer isn't "pick one" — it's that they catch genuinely different failure classes and
belong together. A **deep** black-box health check (one that actually exercises the real downstream dependency,
not just "is the process alive") would have caught this specific outage too, and many real production systems
build exactly that: a `/health` endpoint that itself makes a real, lightweight check against Payments before
reporting healthy. But a deep health check adds real latency and real load to every probe, and still only checks
the specific paths it was written to check — white-box metrics remain the broader, more detailed signal for
everything a health check wasn't specifically designed to catch, exactly the gap this lesson's real numbers
expose.

## Common mistakes

- Relying on a shallow black-box health check ("does the process respond") as a complete health signal, the
  exact real gap this lesson measured directly — a process can be fully "up" while completely failing to do its
  actual job.
- Building only white-box monitoring and never testing the system the way a real external caller actually would —
  missing real issues specific to the network path, load balancer, or edge layer that internal metrics alone
  can't see, since they only observe from inside the process.
- Writing a "deep" health check that itself becomes a real reliability risk — if `/health` makes a real call to
  every real downstream dependency on every single probe, a struggling dependency can now also take down the
  health check itself, compounding the real incident rather than just reporting it.

## Exercise

Rewrite `/health` to be a real, bounded "deep" check: make a real, short-timeout call to Payments'
`/charge` endpoint (with a harmless test amount) as part of the health check itself, and report `unhealthy` if
that real call fails or times out. Re-run this lesson's exact outage scenario and confirm the black-box check now
correctly reports `unhealthy` during the real outage — then write one sentence on the real new risk this deeper
check introduces that the original shallow version didn't have.
