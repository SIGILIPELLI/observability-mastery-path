---
description: "A real series deletion appeared to do nothing while still being scraped, then genuinely worked once scraping stopped -- a real investigation into why."
---

# Prometheus Storage: the TSDB on Disk & Retention

## What was run

The real local Prometheus TSDB directory from this course's own lab, inspected directly on disk. A real
deliberate deletion of the 2,000-series `hc_requests_total` metric (from [Lesson 8](08-recording-rules.md)) was
attempted via Prometheus's real admin API — first while the metric's target was still being scraped, where it
appeared to fail silently, then again after genuinely removing the scrape target, which is what actually made it
work. Both attempts, and the real investigation connecting them, are shown here.

## The real block structure, on disk

```bash
find prom-data -maxdepth 2
```
```
prom-data/01M5VQN4MNB6EBX6GSP829T1SW/{chunks,index,meta.json,tombstones}
prom-data/01M5VXXWP9NE14A8S3F15JKNFE/{chunks,index,meta.json,tombstones}
prom-data/wal/00000004 ... 00000008
```

```bash
cat prom-data/01M5VQN4MNB6EBX6GSP829T1SW/meta.json
```
```json
{"minTime": 1792994610497, "maxTime": 1793001600000,
 "stats": {"numSamples": 1888, "numSeries": 59, "numChunks": 59}}
```

Each real block directory is a self-contained, immutable chunk of real time range (here, a real ~2-hour window),
with its own compressed `chunks` (the actual sample data), an `index` for looking values up by label, and a
`meta.json` stating exactly what it contains. The `wal` (write-ahead log) holds the most recent, not-yet-blocked
data — what's still being actively written before the next compaction turns it into a real, immutable block like
the ones above.

## A real deletion that appeared to do nothing

```bash
curl -s -X POST ".../api/v1/admin/tsdb/delete_series?match[]=hc_requests_total&start=0&end=9999999999"
# 204
curl -s -X POST ".../api/v1/admin/tsdb/clean_tombstones"
# 204
curl -s ".../api/v1/query" --data-urlencode 'query=hc_requests_total'
# results: 2000     <- still there!
```

Both admin calls returned a real, successful `204`. And yet the metric was still fully present, all 2,000 series,
immediately after. This looked like a real bug — until actually investigating it, rather than assuming either
"deletion is broken" or "deletion worked and I measured wrong."

## The real explanation, confirmed by removing the scrape target first

```bash
# scrape target for high-cardinality removed from prometheus.yml, Prometheus restarted for real
curl -s ".../api/v1/targets" | grep high-cardinality
# (gone from the target list)

curl -s -X POST ".../api/v1/admin/tsdb/delete_series?match[]=hc_requests_total&start=0&end=9999999999"
curl -s -X POST ".../api/v1/admin/tsdb/clean_tombstones"
curl -s ".../api/v1/query" --data-urlencode 'query=hc_requests_total'
# results: 0        <- now it's actually gone
```

**With scraping genuinely stopped, the identical deletion sequence worked.** The real explanation: deletion
removes historical samples within the given time range, but it does nothing to stop *future* samples from
arriving — and as long as a target is still being scraped every `scrape_interval`, a brand-new sample for that
exact series is written on every single scrape, immediately refilling the data the deletion just removed. The
first attempt wasn't a bug; it was real, correct behavior responding to a real, ongoing scrape that was still
re-creating the data faster than it could be meaningfully "deleted."

## How It Actually Works

Prometheus's TSDB organizes data into immutable, time-bounded blocks on disk (as seen above), periodically
merging smaller blocks into larger ones through **compaction**, and dropping blocks entirely once they fall
outside the configured retention window (`--storage.tsdb.retention.time`, 15 days by default). This design — many
small immutable blocks, rather than one continuously-mutated database — is what makes retention cheap: expiring
old data is just deleting whole block directories once they age out, not running an expensive `DELETE` query
across live data.

The real investigation above demonstrates something retention and manual deletion share: **both only control what
happens to historical data; neither stops new data from being written if the thing producing it is still
running.** Retention deletes blocks once they're old enough; it doesn't pause scraping. Manual deletion
(`delete_series`) marks existing samples as deleted (via tombstones, cleaned up by `clean_tombstones`); it
doesn't touch the scrape config. If you actually want a metric to stop existing going forward, the real fix is
removing or relabeling it out of the scrape config — which is also, not coincidentally, the standard real fix for
a cardinality problem like [Level 3, Lesson 7](../level-3/07-cardinality-explosion.md) covers.

## Common mistakes

- Assuming a "successful" admin API call (`204`) means the described change is immediately and permanently
  reflected — as shown above, a real `204` from `delete_series` didn't stop the metric from existing seconds
  later, because deletion and ongoing collection are two independent, non-interacting mechanisms.
- Trying to control cardinality or clean up a mistake purely through deletion, without also fixing the scrape
  config or instrumentation that's still actively producing the unwanted series.
- Confusing the WAL (recent, not-yet-compacted data, needed for crash recovery) with the real compacted blocks —
  deleting or corrupting WAL files directly (rather than through the real admin API) risks losing recent data
  that hasn't been persisted into a block yet.

## Exercise

Set `--storage.tsdb.retention.time=2h` on a fresh Prometheus instance (a new data directory, to avoid disturbing
this lesson's existing data), let it run and scrape real data past that window if you can arrange it, and confirm
for yourself — by inspecting the real block directories on disk, the same way this lesson did — that blocks
genuinely older than 2 hours actually get removed, while the mechanism (compaction and retention) never once
needs to touch the scrape config.
