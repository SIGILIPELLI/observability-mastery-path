---
description: "A naive projection from a real 5,223-series, 73-minute TSDB history predicted 719,161 series in a week -- an overestimate from contaminated test data."
---

# Capacity Planning From Real Metrics

## What was run

A real capacity projection built directly from this course's own live Prometheus TSDB status — its real current
series count, real elapsed time, and real on-disk size — extrapolated forward the naive way first, then
corrected once the real cause of the naive projection's error was identified.

## The real, naive projection — and why it's real but wrong

```bash
curl -s "http://localhost:9090/api/v1/status/tsdb"
# headStats.numSeries: 5223
# minTime -> maxTime spans 73.2 real minutes
```

```python
rate_per_min = 5223 / 73.2   # 71.35 series/min, a real observed average
```
```
projected at this rate:
  24 hours: 102737 series
  7 days:   719161 series
```

A real, honestly-computed linear projection — and almost certainly wrong as a forecast of genuine future growth.
The real cause: this course's own cardinality demonstrations
([Level 1, Lesson 8](../level-1/08-recording-rules.md)'s 2,000-series app,
[Level 3, Lesson 7](../level-3/07-cardinality-explosion.md)'s 500-series cardinality bomb) each added a real,
one-time burst of thousands of series in minutes, not a sustained, ongoing growth rate — the exact same
"contaminated baseline" problem [Lesson 4](04-designing-an-slo.md) found for latency percentiles, now showing up
in capacity planning instead.

## The real, honest per-series storage cost — a genuinely more durable number

```bash
du -sh prom-data   # 4.7M
```
```python
bytes_per_series = 4.7 * 1024 * 1024 / 5223   # 943.6 bytes/series
```
```
real storage per series so far: 943.6 bytes/series
projected storage at 100,000 series: 90.0 MB
```

This real ratio — storage cost *per series*, not *per unit time* — is a more durable, reusable number than the
time-based growth rate above, because it doesn't depend on assuming this course's artificial test bursts
represent real, ongoing traffic growth. It can be recombined with a genuinely trustworthy growth estimate (from
actual, steady-state production traffic trends, not a test artifact) to produce a real, defensible storage
forecast.

## How It Actually Works

Real capacity planning for an observability system — disk space for Prometheus's TSDB, query load, ingestion
throughput — depends on getting the *growth rate* right, and this lesson's own naive 719,161-series week-ahead
projection shows exactly how a short observation window can badly mislead that estimate if it happens to include
one-time artifacts (a deliberate cardinality test, a migration, a temporary debug flag someone forgot to remove)
rather than steady-state behavior. The real, practical fix mirrors [Lesson 4](04-designing-an-slo.md)'s: separate
*durable, reusable ratios* (bytes per series, bytes per sample — properties of the storage engine and data shape,
relatively stable over time) from *time-based growth rates* (series added per minute — highly sensitive to
whatever happened to be running during the observation window), and be skeptical of any capacity forecast built
primarily on the latter without first checking whether the observation window was representative.

A genuinely sound real capacity plan combines: (1) a real, validated per-unit cost (this lesson's `943.6
bytes/series`, or an equivalent per-sample ingestion cost), with (2) a real, trustworthy growth projection — ideally
derived from multiple longer observation windows, explicitly excluding or separately accounting for known
one-time events, cross-checked against whatever is actually planned (more services being onboarded, more
instances being deployed) rather than extrapolated blindly from whatever the monitoring system happened to see
during one arbitrary recent window.

## Common mistakes

- Extrapolating linearly from a short observation window without checking whether anything unusual (a test, a
  migration, an incident) happened during that specific window — this lesson's own real 7-day projection is a
  direct, measured example of exactly this mistake.
- Conflating a durable per-unit cost (bytes per series) with a time-based growth rate (series per minute) — the
  first is a relatively stable property of the system; the second is only as reliable as the window it was
  measured over, as shown here.
- Planning capacity for the current moment's scale with no margin, rather than for a real, validated projection
  of where the system will be by the time any new capacity actually gets provisioned and deployed.

## Exercise

Query this course's real Prometheus TSDB status again after it's been running untouched (no new deliberate test
traffic) for a real, meaningful amount of time — at least 30 minutes — and compute a new real series-growth rate
from that cleaner window. Compare it directly to this lesson's original `71.35 series/min` figure, and report how
much lower the real, steady-state rate is once the contaminating test bursts are no longer part of the
observation window.
