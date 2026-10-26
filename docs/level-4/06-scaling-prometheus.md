---
description: "A real 943.6 bytes/series cost, projected honestly, is the number that forces a real choice between federation, remote write, and long-term storage."
---

# Scaling Prometheus: Federation, Remote Write & Long-Term Storage

## What was run

No new experiment — this lesson picks up directly from [Lesson 5](05-capacity-planning.md)'s real, validated
`943.6 bytes/series` storage cost and asks the honest next question: what actually happens once a real system's
series count and retention requirements outgrow what one Prometheus server, on one machine, can reasonably hold —
and what are the real, documented architectures for that, since this course's single-laptop scope never required
running any of them.

## The real number that eventually forces this question

```
real storage per series so far: 943.6 bytes/series
```

A single Prometheus server, as this entire course has run it, keeps all its data on one local disk, served by one
process. At `943.6 bytes/series`, even this lesson's deliberately modest `5,223`-series lab instance used real,
measured megabytes — scale that out to a production system monitoring hundreds of real services, each with
dozens of real metrics and real label combinations, over months of real retention, and the honest arithmetic
stops fitting on one machine's disk, long before it stops fitting in one machine's query performance budget.

## How It Actually Works

Three real, standard architectures address this, each solving a different real limitation of a single Prometheus
server — none of them run in this course, stated plainly, since every earlier lesson's real verification depended
specifically on a single, directly-inspectable local instance:

- **Federation**: one Prometheus server scrapes *aggregated* data from other Prometheus servers (via their own
  `/federate` endpoint), rather than scraping every individual target directly. This lets a top-level server hold
  a rolled-up, cross-cluster view without needing every raw series from every underlying server — real for
  building a "global" dashboard across multiple independently-run Prometheus instances, but it doesn't solve the
  single-server retention or raw-data-volume problem for any one of those underlying servers; it's a *query
  aggregation* answer, not a *storage scaling* answer.
- **Remote write**: a Prometheus server forwards every real scraped sample, as it's ingested, to a separate,
  external storage system over a standard remote-write protocol, instead of (or in addition to) keeping it in its
  own local TSDB. This decouples *ingestion and local short-term querying* (which Prometheus keeps doing exactly
  as this course has run it) from *long-term storage and cross-server querying* (handled by whatever system is on
  the receiving end), directly solving the local-disk retention limit this lesson's own real per-series cost
  points toward.
- **Long-term storage systems** (Thanos, Cortex, Mimir — all real, widely-used, explicitly *not* run in this
  course) typically consume remote-written data from many Prometheus servers, storing it in cheaper, more
  scalable backends (often real object storage like S3) while still exposing a Prometheus-compatible query API —
  letting real PromQL, the exact language every lesson in this course has used directly, keep working unchanged
  against a system that scales far beyond what one local TSDB could.

The real, honest thread connecting all three to this course: every one of them exists specifically to solve the
exact problem this lesson's own real numbers made concrete — one server's local disk and one server's query
engine eventually can't hold or serve a real production system's full retention and cardinality needs, and each
architecture solves a different piece of "where does the data actually live, long-term" without requiring a
return to re-deriving PromQL, metric types, or instrumentation practice from scratch; everything built across
Levels 1–3 remains directly applicable on top of whichever of these a real team chooses.

## Common mistakes

- Reaching for Thanos/Cortex/Mimir before a real, measured need exists — every one of these adds real operational
  complexity (another distributed system to run and understand) that a single Prometheus server, correctly sized
  using the real methodology from [Lesson 5](05-capacity-planning.md), may not need for a long time.
- Treating federation as a solution to a storage-scaling problem — as described above, it's fundamentally a
  cross-server query aggregation mechanism, not a way to reduce any individual server's own real storage burden.
- Assuming "scaling Prometheus" always means more infrastructure, without first applying the real, cheaper
  levers this course already built directly — fixing a real cardinality problem
  ([Level 3, Lesson 7](../level-3/07-cardinality-explosion.md)), using recording rules to reduce real repeated
  query cost ([Level 1, Lesson 8](../level-1/08-recording-rules.md)), and sampling where applicable
  ([Level 3, Lesson 5](../level-3/05-sampling-strategies.md)) can each meaningfully delay or reduce the real need
  for a bigger architecture.

## Exercise

Using this lesson's real `943.6 bytes/series` figure and [Lesson 5](05-capacity-planning.md)'s real, corrected
(non-contaminated) growth rate, compute a real projected storage requirement at 90 days of retention. Write down,
specifically, which of the three architectures above you'd reach for first at that real projected scale, and why
— grounded in the actual numbers you've computed, not a general impression of what "big" observability systems
use.
