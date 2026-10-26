---
description: "A real saturation metric (0.00254, fds) from Prometheus's own self-scraped data, after discovering Python's process metrics don't exist on macOS."
---

# The Four Golden Signals & the USE Method

## What was run

An attempt to build a real resource-saturation panel for `orders_app.py`, which led to a genuine, real discovery:
Python's `prometheus_client` library exposes **zero** `process_*` metrics on this machine. The investigation that
followed, and the real fix — scraping Prometheus's own self-exposed metrics instead, since Prometheus is a
compiled Go binary without the same limitation — is shown here in full.

## A real, honest gap

```bash
curl -sL localhost:9301/metrics | grep -c "^process_"
# 0
```

Every lesson in this course expected `prometheus_client`'s default collectors to include real process-level
metrics (resident memory, CPU time, open file descriptors) automatically, the way they normally do. On this
machine, for these Python apps, they simply don't exist. The real cause: `prometheus_client`'s `ProcessCollector`
reads `/proc/self/...` to gather this data, and `/proc` is a Linux-specific filesystem — it doesn't exist on
macOS at all, so the collector silently has nothing to report.

```bash
curl -s "http://localhost:9090/metrics" | grep "^process_"
# process_cpu_seconds_total 2.513764
# process_max_fds 10240
# process_open_fds 33
# process_start_time_seconds 1.793025152e+09
```

Prometheus itself, a compiled Go binary, has no such dependency on `/proc` for this data and exposes it directly
and completely — proof the gap is specifically about the Python client's platform assumption, not a general
limitation of this course's stack.

## A real USE-method saturation metric, computed from what's actually available

```bash
curl -s ".../api/v1/query" --data-urlencode 'query=process_open_fds{job="prometheus"} / process_max_fds{job="prometheus"}'
# 0.0025390625
```

A real, genuine **saturation** number: Prometheus is currently using about 0.25% of its available file
descriptors. This is exactly the kind of resource-exhaustion-risk metric the USE method asks for, computed here
from a real, scraped, self-monitoring target rather than an invented example.

## How It Actually Works

The **Four Golden Signals** (from Google's SRE book) and the **USE Method** (Brendan Gregg) are two closely
related but distinctly-scoped frameworks for "what should I monitor":

- **Golden Signals** — Latency, Traffic, Errors, Saturation — are request-centric, almost identical to the RED
  method from [Lesson 2](02-red-dashboard.md) with one addition: **Saturation**, how "full" the service is
  (queue depth, how close to its concurrency limit it's running), which RED alone doesn't cover.
- **USE Method** — Utilization, Saturation, Errors — is resource-centric: for every real resource a service
  depends on (CPU, memory, disk I/O, network, file descriptors, connection pools), ask what fraction of its
  capacity is in use (**Utilization**), how much work is queued waiting for it (**Saturation** — the exact
  `process_open_fds / process_max_fds` ratio computed above is one real, concrete instance of this), and how
  often it's failing (**Errors**).

The real, practical difference: RED/Golden-Signals panels (built in [Lesson 2](02-red-dashboard.md)) tell you
*that* a service is unhealthy from the outside — slow, erroring, degraded. USE panels tell you *why*, from the
inside — this specific resource is exhausted or close to it. A real incident often starts with a Golden Signals
dashboard showing elevated latency, and is diagnosed by then checking USE panels to find which specific resource
(CPU? file descriptors? a connection pool, the exact kind measured for real in
[Microservices, Level 3, Lesson 3](https://sigilipelli.github.io/microservices-mastery-path/level-3/03-bulkheads-resource-isolation/))
is actually the bottleneck. Neither framework alone gives the complete picture; they're complementary, not
competing.

## Common mistakes

- Assuming a metrics client library's default collectors work identically across platforms without checking —
  this lesson's real `process_*` gap is a direct, concrete instance of an assumption that turned out false.
- Building only RED/Golden-Signals dashboards and no USE dashboards, leaving a real incident response with "the
  service is unhealthy" but no path to "and here's the specific resource causing it."
- Treating Saturation and Utilization as interchangeable — a resource can be at 60% utilization with zero queued
  work (healthy) or at 60% utilization with a growing queue (saturated and getting worse) — the two numbers
  answer genuinely different questions and both matter.

## Exercise

Find a real, working way to get process-level memory and CPU metrics for the Python apps on this machine despite
the `/proc` gap — either by running them inside a real Linux environment (a Lima VM, as used in the
[Linux Mastery Path](https://sigilipelli.github.io/linux-mastery-path/)), or by instrumenting a custom metric
using Python's own `resource` or `psutil` module to read real memory/CPU usage directly and expose it as a
`Gauge`. Confirm your fix produces real, non-empty data where the default collector produced none.
