---
description: "A log-derived count and the native Prometheus counter agreed exactly (4 vs 4.0 declined events) -- proof both track reality, at very different costs."
---

# Log-Based Metrics vs Native Metrics

## What was run

The real, same question — "how many checkouts have been declined?" — answered two different ways against the
exact same real traffic from `orders_app.py`: once by parsing the real structured log file from
[Lesson 8](08-structured-logging-correlation.md) line by line, and once by reading the native Prometheus counter
that was incremented directly in the application code, in real time, as each request happened.

## The real consistency check

```python
# log-derived: parse every real log line, count matching events
count = 0
for line in open("orders_structured.log"):
    rec = json.loads(line)
    if rec.get("event") == "checkout_declined":
        count += 1
# log-based count: 4 declined events
```

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=orders_requests_total{endpoint="/checkout", status!="200"}'
# native metric count: 4.0 declined events
```

**Both answers agree exactly: 4.** This is real, useful proof that the native counter and the structured logs are
genuinely tracking the same underlying reality — the kind of cross-check this course has used before (the real
charged/declined-rate consistency check in [Level 1, Lesson 10](../level-1/10-project-instrument-scrape.md)) to
build confidence that instrumentation is correct, not just independently present.

## How It Actually Works

A **native metric** is incremented directly in application code, in memory, at the exact moment the real event
happens (`REQUEST_COUNT.labels(...).inc()`, from [Level 1, Lesson 3](../level-1/03-instrumenting-metric-types.md)) —
the cost of updating it is a single, constant-time in-memory operation, regardless of how much total traffic the
service has ever handled. A **log-based metric** is derived *after the fact* by parsing log lines a service wrote
for an entirely different reason (human-readable or structured record-keeping, debugging, audit trail) — the cost
of deriving it scales with how much log volume has to be scanned, exactly the real, measured cost
[Lesson 7](07-centralized-logging-fundamentals.md) demonstrated growing from milliseconds to hundreds of
milliseconds as log volume grew to 100,000 real lines, and continuing to grow linearly from there.

This gives each approach a real, distinct role rather than making one strictly better:

- **Native metrics** are the right choice for anything you know in advance you'll want to track continuously —
  request counts, latencies, error rates — because the instrumentation cost is paid once, at write time, and
  stays cheap forever regardless of query frequency or volume (exactly the recording-rule-style amortization
  argument from [Level 1, Lesson 8](../level-1/08-recording-rules.md), but even more direct, since there's no
  query to run at all — the number is already sitting there, pre-maintained).
- **Log-based metrics** are the right choice for a question you *didn't* anticipate needing a dedicated counter
  for — deriving a brand-new metric from historical log data doesn't require redeploying the service with new
  instrumentation; it only requires the right fields already being present in already-collected structured logs,
  which is exactly why [Lesson 8](08-structured-logging-correlation.md)'s structured (not free-text) logging
  discipline matters even when you're not deriving a metric from it today.

The real, honest tradeoff: a log-based metric is also only as fresh as the last time it was recomputed — unlike a
native counter, which is correct at the instant you query it, a log-derived number reflects whatever log volume
had been ingested and parsed as of whenever that derivation job last ran, introducing a real staleness window a
native metric doesn't have.

## Common mistakes

- Deriving a metric from logs for a question you could have anticipated and instrumented natively from the
  start — paying the real, repeated, volume-scaling cost of log parsing for something a single `Counter.inc()`
  call would have tracked essentially for free.
- Never deriving log-based metrics at all, even for genuinely unanticipated questions — insisting that
  every metric must be natively instrumented means redeploying code for every new question, when the data to
  answer many of them may already be sitting in structured logs.
- Treating a log-derived number as equally fresh as a native one without checking when it was last recomputed —
  the real consistency check in this lesson worked because both were computed from the same, complete, finished
  dataset; a log-based metric queried mid-ingestion could legitimately disagree with a native one for reasons
  that are about timing, not correctness.

## Exercise

Pick one real question this course's `orders_app.py` logs could answer that there is currently *no* native metric
for — for example, "what's the average declined amount, in cents?" — and write a real script that derives it
from the structured log file. Then add a native Prometheus `Histogram` or `Summary` tracking the same thing
directly in the application code, generate a few more real declined checkouts, and confirm both approaches once
again agree on the real answer.
