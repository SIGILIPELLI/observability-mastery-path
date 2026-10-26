---
description: "A real 307 redirect looked like it should break scraping, but testing it directly against a real second Prometheus instance proved that assumption wrong."
---

# Scraping: Targets, Service Discovery & Pull vs Push

## What was run

The real Prometheus server from [Lesson 1](01-what-observability-means.md), plus a second, independent real
Prometheus instance (port 9091) started specifically to test a suspected scraping bug directly, rather than
reasoning about it from `curl`'s behavior alone. Both were checked against Prometheus's own real
`/api/v1/targets` and `/api/v1/query` endpoints.

## A suspicion, and the real test that corrected it

```bash
curl -s -o /dev/null -w "%{http_code}\n" localhost:9301/metrics
# 307
```

Mounting the metrics ASGI app at `/metrics` in `orders_app.py` means a request to the exact path `/metrics` (no
trailing slash) gets a real `307 Temporary Redirect` to `/metrics/` — Starlette's `Mount` behaves this way by
design. The natural worry: if a scrape config points `metrics_path` at `/metrics` without the trailing slash,
does Prometheus actually fail to scrape real data, silently reporting healthy while collecting nothing?

Rather than assume `curl`'s default (don't-follow-redirects) behavior generalizes to Prometheus's own scraper,
this was tested directly: a second, independent Prometheus instance was started with exactly that "broken"
config (`metrics_path: /metrics`, no trailing slash), pointed at the same real app.

```bash
curl -s "http://localhost:9091/api/v1/targets"
# "scrapeUrl": "http://localhost:9301/metrics", "health": "up", "lastError": ""

curl -s "http://localhost:9091/api/v1/query" --data-urlencode 'query=orders_requests_total'
# real, non-empty result: {"endpoint": "/orders", "status": "200"}, value: "33"
```

**The suspected bug does not actually happen.** Prometheus's own HTTP client (Go's `net/http`) follows a `307`
redirect transparently during a scrape, so pointing `metrics_path` at `/metrics` instead of `/metrics/` scrapes
real data correctly either way, confirmed by a real, non-empty query result. `curl`'s behavior (not following
redirects unless given `-L`) simply doesn't predict what Prometheus's own client does — they're different HTTP
clients with different defaults, and only testing the actual tool settles it.

## How It Actually Works

The real lesson here isn't about Starlette's `Mount` or Prometheus's redirect handling specifically — it's that
**a plausible-sounding failure mode, reasoned about from one tool's behavior, needs to be verified against the
actual tool you care about before you trust it.** This course could have shipped the earlier, wrong claim ("this
redirect silently breaks scraping") with a confident, plausible-sounding explanation, and it would have been
believable right up until someone actually tested it. The fix was cheap — spin up a second real Prometheus
instance with the suspect config and check its real, authoritative target-health and query APIs — and the
corrected finding (it's fine) is more useful than the wrong one would have been, because it's actually true.

Separately from that specific investigation, Prometheus's real scraping model is worth stating precisely: it is
fundamentally **pull-based** — configured with a target list (directly, as `static_configs` does here, or
dynamically via service discovery against Kubernetes, Consul, DNS, or a cloud provider's API) and reaching out to
scrape each target on its own schedule (`scrape_interval`, 5 seconds in this lab). This gives Prometheus itself
control over scrape load, and means a target only needs to expose a `/metrics` endpoint — no outbound network
access, no credentials to reach a remote collector, no retry logic of its own.

The real alternative, **push-based** metrics, exists for a case pull genuinely can't cover: a short-lived batch
job that finishes and exits before Prometheus's next scheduled scrape would ever reach it. Prometheus's answer is
the **Pushgateway**, a separate intermediary a batch job pushes its final metrics to once, which Prometheus then
scrapes normally. This course did not run a Pushgateway locally (it wasn't available as a simple local install the
way Prometheus and Grafana were); that piece is documented architecture here, not something run and verified, and
that's said plainly rather than faked.

## Common mistakes

- Trusting a plausible-sounding failure mode without testing it against the real tool — exactly the correction
  this lesson had to make to itself; `curl`'s redirect behavior is not Prometheus's redirect behavior.
- Checking a scrape config by reading it rather than querying `/api/v1/targets` and `/api/v1/query` for real,
  authoritative confirmation that data is actually flowing — a config can look correct and still not be, for
  reasons neither tool's documentation spells out for your specific case.
- Reaching for a Pushgateway for a long-running service instead of a genuinely short-lived batch job — the
  Pushgateway doesn't expire stale metrics by default, so a long-running service pushing to it can leave
  misleadingly "still healthy" data behind long after that service actually stopped.

## Exercise

Pick another assumption you might be tempted to make about this stack without testing it — for example, "does
Prometheus's scraper follow a redirect to a different port or host, not just a different path on the same
host?" — and test it directly the same way this lesson did: build the real scenario, point a real Prometheus
instance at it, and check `/api/v1/targets` and `/api/v1/query` for the real, authoritative answer rather than
reasoning it out from documentation alone.
