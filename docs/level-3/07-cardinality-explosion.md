---
description: "500 requests with a per-request UUID label created 500 real series and 37,000 scrape bytes; labeled correctly, the same 500 made 1 series and 26 bytes."
---

# Cardinality: the Silent Killer of Metrics Systems

## What was run

A real app, `cardinality_bomb.py` (port 9505), exposing two real counters tracking the exact same thing — "how
many requests happened" — with one critical difference: `bad_requests_total` is labeled with a fresh, real random
UUID on every single request; `good_requests_total` carries no such label. 500 real requests were sent to each,
and the real resulting metric data was measured directly.

## The real, measured explosion

```python
# the mistake
BAD_COUNTER = Counter("bad_requests_total", "...", ["request_id"])
BAD_COUNTER.labels(request_id=str(uuid.uuid4())).inc()

# the fix
GOOD_COUNTER = Counter("good_requests_total", "...")
GOOD_COUNTER.inc()
```

```bash
# 500 real requests sent to each endpoint...
curl -s localhost:9505/metrics | grep -c "^bad_requests_total"
# 500

curl -s localhost:9505/metrics | grep -c "^good_requests_total"
# 1
```

**500 real, distinct time series** for `bad_requests_total` — one per request, because every request's UUID was
genuinely unique, so every single `.labels(...)` call created a brand-new series that will never be incremented
again. **1 real series** for `good_requests_total`, correctly accumulating the same real count of 500.

```bash
curl -s localhost:9505/metrics | grep "^bad_requests_total" | wc -c
# 37000

curl -s localhost:9505/metrics | grep "^good_requests_total" | wc -c
# 26
```

**37,000 bytes versus 26 bytes** — roughly a **1,423x** real difference in scrape payload size, for metrics that
both correctly answer "500 requests happened." Every one of those 500 real, distinct series has to be stored
in Prometheus's TSDB (per [Level 1, Lesson 9](../level-1/09-prometheus-storage-tsdb.md)'s real block structure)
forever, or at least until it ages out of retention — real, permanent cost for information that a single counter
already captured completely.

## How It Actually Works

This is the real, concrete consequence [Level 1, Lesson 2](../level-1/02-prometheus-data-model.md) warned about
directly: every unique combination of label values is its own independent time series, and a label whose values
are effectively unbounded — a UUID, a raw user ID, a full URL with embedded IDs, a raw IP address, a timestamp —
multiplies a metric's real series count by however many distinct values have ever actually occurred, with no
natural ceiling. A metric with 2 status codes and 5 endpoints has a bounded, small, predictable series count (at
most 10). A metric with an unbounded label grows forever, one new real series for every genuinely new value ever
seen, exactly as measured above.

The real danger is that this failure mode is **silent** at first: a brand-new instrumentation point with an
unbounded label looks completely fine in testing, with a handful of requests producing a handful of series — the
real cost only becomes visible at production scale, after real traffic has generated thousands or millions of
genuinely unique label values, by which point the damage (a bloated, slow, expensive TSDB) is already done and
much harder to undo than it would have been to prevent. This is precisely why [Level 1, Lesson 9](../level-1/09-prometheus-storage-tsdb.md)'s
real investigation into deletion mattered: deleting already-created cardinality-bomb series doesn't fix the root
cause, which is the label choice in the instrumentation code itself — the fix has to happen at the source, the
same conclusion that lesson reached from a different angle.

The real guideline this course's own numbers support: a label is safe when its real, total number of distinct
values across the metric's entire lifetime is small and bounded by something about your actual domain (a fixed
set of HTTP methods, a fixed set of environment names, a fixed, small set of service names) — not by how much
traffic you happen to receive. If a label's cardinality could plausibly scale with traffic volume itself, it
doesn't belong on a metric; it belongs in a log line (exactly the structured logging discipline from
[Level 2, Lessons 7–9](../level-2/07-centralized-logging-fundamentals.md)), where per-event detail is the whole
point and the cost model is built for it.

## Common mistakes

- Adding a label "just in case it's useful for debugging later," without checking whether its value set is
  genuinely bounded — exactly the mistake `bad_requests_total` makes deliberately here, to measure its real cost.
- Discovering a cardinality problem in production and only deleting the already-created high-cardinality series
  (per [Level 1, Lesson 9](../level-1/09-prometheus-storage-tsdb.md)'s real deletion mechanics) without also
  fixing the instrumentation code still producing new ones — the real fix has to stop new unbounded series from
  being created, not just clean up existing ones.
- Assuming a label is safe because it "doesn't look that big" without actually counting its real distinct value
  space — a label that seems modest in a quick local test (this lesson's 500 requests) can represent millions of
  distinct values at real production traffic levels.

## Exercise

Add a third counter, `medium_requests_total`, labeled by `request_id[:2]` (just the first two hex characters of
each real UUID) instead of the full UUID — a deliberately *bounded* version of the same idea (at most 256 possible
two-character hex prefixes). Send the same 500 real requests to it, measure its real series count and payload
size the same way this lesson did, and report where it falls between the two extremes measured above.
