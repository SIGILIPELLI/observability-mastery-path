---
title: "Observability Interview Questions and Answers (179 Questions)"
description: "179 Observability interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the Observability Mastery Path."
---

# Observability Interview Questions and Answers

**179 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free Observability Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Metrics & Prometheus Fundamentals

### [What Observability Actually Means: Metrics, Logs &amp; Traces](level-1/01-what-observability-means.md)

**Q1.** What is observability?

**Answer:** The ability to understand a system's internal state from the telemetry it emits

Ask new questions without new code.

**Q2.** How does observability differ from monitoring?

**Answer:** Monitoring checks known failure modes; observability helps explore unknown ones

Unknown unknowns.

**Q3.** What are the three pillars?

**Answer:** Metrics, logs and traces

Complementary signals.

**Q4.** Which pillar is best for 'is something wrong right now'?

**Answer:** Metrics (cheap, aggregated, alertable)

Then drill down.

### [The Prometheus Data Model: Time Series, Labels &amp; Samples](level-1/02-prometheus-data-model.md)

**Q5.** What is a Prometheus time series identified by?

**Answer:** A metric name plus a set of label key/value pairs

Dimensions.

**Q6.** What's a sample?

**Answer:** A timestamped float value of a series

Scraped periodically.

**Q7.** What does `http_requests_total{method="GET"}` select?

**Answer:** Series of that metric whose method label equals GET

Label matchers.

**Q8.** Why avoid high-cardinality labels?

**Answer:** Each unique combination is a new series, exploding memory and storage

No user IDs.

### [Instrumenting an App: Counters, Gauges, Histograms &amp; Summaries](level-1/03-instrumenting-metric-types.md)

**Q9.** What are the four core metric types?

**Answer:** Counter, gauge, histogram and summary

Choose by semantics.

**Q10.** When use a counter?

**Answer:** For monotonically increasing totals like requests or errors

Use rate().

**Q11.** When use a gauge?

**Answer:** For values that go up and down (queue depth, temperature)

Current state.

**Q12.** What does a counter do on restart?

**Answer:** Resets to zero, which rate() handles

Reset-aware.

### [Scraping: Targets, Service Discovery &amp; Pull vs Push](level-1/04-scraping-service-discovery.md)

**Q13.** How does Prometheus collect metrics?

**Answer:** By pulling (scraping) HTTP /metrics endpoints

Pull model.

**Q14.** What does service discovery do?

**Answer:** Finds scrape targets dynamically (Kubernetes, DNS, file)

Cloud-native.

**Q15.** What does relabeling do?

**Answer:** Rewrites labels or filters targets before/after scraping

Control cardinality.

**Q16.** What's the Pushgateway for?

**Answer:** Short-lived batch jobs that can't be scraped

Not general push.

### [PromQL Basics: Selecting &amp; Filtering Time Series](level-1/05-promql-basics.md)

**Q17.** What does `rate(http_requests_total[5m])` return?

**Answer:** Per-second average rate of increase over 5 minutes

Counters need rate.

**Q18.** What does `{__name__=~"http.*"}` do?

**Answer:** Selects series by regex on the metric name

Matchers.

**Q19.** What's an instant vector?

**Answer:** One sample per series at a single timestamp

Query result.

**Q20.** What's a range vector?

**Answer:** A set of samples per series over a time window ([5m])

Used by rate().

### [PromQL Aggregation: sum, rate() &amp; Why Counters Need It](level-1/06-promql-aggregation-rate.md)

**Q21.** What does `sum by (service) (rate(x[5m]))` do?

**Answer:** Total per-second rate grouped by service

Aggregation.

**Q22.** Why `rate` before `sum`?

**Answer:** Counters must be converted per series before aggregating (resets are per series)

Order matters.

**Q23.** What does `increase()` return?

**Answer:** Total increase over the window

rate x time.

**Q24.** What does `without` do in aggregation?

**Answer:** Groups by all labels except those listed

Opposite of by.

### [Histogram Quantiles: Why You Can't Average a Histogram](level-1/07-histogram-quantiles.md)

**Q25.** What does a histogram store?

**Answer:** Cumulative bucket counters, a sum and a count of observations

Aggregatable.

**Q26.** How compute p95 latency across instances?

**Answer:** histogram_quantile(0.95, sum by (le) (rate(x_bucket[5m])))

Sum buckets by le.

**Q27.** Why not average percentiles?

**Answer:** Percentiles can't be averaged across instances correctly

Use histograms.

**Q28.** What limits histogram accuracy?

**Answer:** Bucket boundaries (estimates inside a bucket)

Choose buckets near SLO.

### [Recording Rules: Pre-Computing Expensive Queries](level-1/08-recording-rules.md)

**Q29.** What do recording rules do?

**Answer:** Precompute expensive expressions and store them as new series

Faster dashboards.

**Q30.** Naming convention for recorded series?

**Answer:** level:metric:operations

job:http_requests:rate5m.

**Q31.** Why use them for alerts/dashboards?

**Answer:** Reduce query cost and keep results consistent

Performance.

**Q32.** Where are rules defined?

**Answer:** In rule files loaded by Prometheus

YAML.

### [Prometheus Storage: the TSDB on Disk &amp; Retention](level-1/09-prometheus-storage-tsdb.md)

**Q33.** What is the TSDB?

**Answer:** Prometheus's local time-series database with blocks and a WAL

Efficient storage.

**Q34.** What does retention control?

**Answer:** How long local data is kept before deletion

--storage.tsdb.retention.time.

**Q35.** What is the WAL for?

**Answer:** Durability of recent samples before they're compacted into blocks

Crash recovery.

**Q36.** When use remote storage?

**Answer:** Long-term retention or global views (Thanos/Mimir)

Scale.

### [Project: Instrument and Scrape a Real Multi-Service App](level-1/10-project-instrument-scrape.md)

**Q37.** What does the instrument-and-scrape project combine?

**Answer:** Client library metrics, a /metrics endpoint, scrape config and PromQL

Level 1 project.

**Q38.** What should an instrumented service expose?

**Answer:** Request counts, errors and latency histograms by route/status

RED.

**Q39.** How verify scraping works?

**Answer:** Check the target is UP in Prometheus and query the series

up metric.

**Q40.** What label hygiene applies?

**Answer:** Use bounded labels like route templates, not raw paths or IDs

Cardinality.

## Level 2 · Dashboards, Alerting & Logs

### [Grafana Fundamentals: Data Sources, Panels &amp; Dashboards](level-2/01-grafana-fundamentals.md)

**Q41.** What does Grafana do?

**Answer:** Visualizes and explores data from many data sources

Dashboards.

**Q42.** What's a data source?

**Answer:** A backend Grafana queries (Prometheus, Loki, Tempo)

Connections.

**Q43.** What are template variables?

**Answer:** Dropdowns parameterizing dashboards (env, service)

Reusable.

**Q44.** Why version dashboards as code?

**Answer:** Review, reproducibility and consistency

Provisioning/JSON.

### [Building a Real RED Dashboard: Rate, Errors, Duration](level-2/02-red-dashboard.md)

**Q45.** What does RED stand for?

**Answer:** Rate, Errors, Duration (for request-driven services)

Service view.

**Q46.** Which panels form a RED dashboard?

**Answer:** Request rate, error ratio and latency percentiles

Per service.

**Q47.** Why show error ratio not just count?

**Answer:** A count rises with traffic; a ratio shows health

Normalize.

**Q48.** What's a good latency view?

**Answer:** p50/p95/p99 from histograms

Tail matters.

### [The Four Golden Signals &amp; the USE Method](level-2/03-golden-signals-use-method.md)

**Q49.** What are the four golden signals?

**Answer:** Latency, traffic, errors and saturation

Google SRE.

**Q50.** What does USE stand for?

**Answer:** Utilization, Saturation, Errors (for resources)

Resource view.

**Q51.** When use USE vs RED?

**Answer:** USE for resources (CPU, disks); RED for services/requests

Complementary.

**Q52.** What is saturation?

**Answer:** How full a resource is or how much queued work exists

Early warning.

### [Alerting Rules: Prometheus &amp; Alertmanager Basics](level-2/04-alerting-rules-alertmanager.md)

**Q53.** What does Alertmanager do?

**Answer:** Deduplicates, groups, routes and silences alerts from Prometheus

Notification layer.

**Q54.** What does `for: 10m` in an alert rule mean?

**Answer:** The condition must hold for 10 minutes before firing

Avoid flapping.

**Q55.** What belongs in alert labels vs annotations?

**Answer:** Labels for routing/identity; annotations for summary and runbook text

Templating.

**Q56.** What makes an alert expression good?

**Answer:** Based on symptoms users feel (errors, latency) not causes

Actionable.

### [Alert Routing, Grouping &amp; Silencing](level-2/05-alert-routing-grouping-silencing.md)

**Q57.** What is grouping in Alertmanager?

**Answer:** Combining related alerts into one notification

Reduce noise.

**Q58.** What is a silence?

**Answer:** A temporary mute for matching alerts (e.g. maintenance)

Expires.

**Q59.** What's an inhibition rule?

**Answer:** Suppresses lower-severity alerts when a related higher one is firing

Cluster down hides pod alerts.

**Q60.** How route to teams?

**Answer:** Routing tree matching labels like team and severity

receiver.

### [Avoiding Alert Fatigue: Designing Alerts That Page the Right Person](level-2/06-avoiding-alert-fatigue.md)

**Q61.** What causes alert fatigue?

**Answer:** Too many low-value or non-actionable alerts

Ignored pages.

**Q62.** What's a good paging criterion?

**Answer:** Urgent, user-impacting and requires human action now

Else ticket.

**Q63.** How reduce noise?

**Answer:** Alert on symptoms/SLOs, tune thresholds and add `for` durations

Review alerts regularly.

**Q64.** What should every page have?

**Answer:** A runbook link and clear impact

Actionable.

### [Centralized Logging Fundamentals: Why grep Doesn't Scale](level-2/07-centralized-logging-fundamentals.md)

**Q65.** Why centralize logs?

**Answer:** Search and correlate across many instances and services

Ephemeral hosts.

**Q66.** What do log shippers do?

**Answer:** Collect and forward logs (Promtail, Fluent Bit, Filebeat)

Agents.

**Q67.** What's Loki's approach?

**Answer:** Index only labels and store log content compressed (cheap)

Label-based.

**Q68.** What retention trade-off exists?

**Answer:** Cost vs how far back you can search

Tiering.

### [Structured Logging &amp; Correlating Logs with Metrics](level-2/08-structured-logging-correlation.md)

**Q69.** What is structured logging?

**Answer:** Emitting logs as key/value (JSON) records

Machine-parsable.

**Q70.** What enables cross-service debugging?

**Answer:** A correlation/trace ID in every log line

Propagate headers.

**Q71.** What shouldn't be logged?

**Answer:** Secrets, tokens and sensitive personal data

Redact.

**Q72.** Why consistent field names?

**Answer:** Queries and dashboards work across services

Schema.

### [Log-Based Metrics vs Native Metrics](level-2/09-log-based-vs-native-metrics.md)

**Q73.** When prefer native metrics over log-derived ones?

**Answer:** For high-volume, reliable numeric signals (cheaper and more accurate)

Instrument.

**Q74.** When derive metrics from logs?

**Answer:** Legacy apps you can't change or rare events

Pragmatic.

**Q75.** What's a risk of log-based metrics?

**Answer:** Cost and fragility from parsing/format changes

Maintenance.

**Q76.** Which is cheaper per datapoint?

**Answer:** Metrics (pre-aggregated) over logs

Storage.

### [Project: A Real Alert That Fires, Routes &amp; Pages](level-2/10-project-real-alert-pages.md)

**Q77.** What does the alert pages project combine?

**Answer:** SLO-style rules, routing, grouping and runbooks that page on user impact

Level 2 project.

**Q78.** How test an alert?

**Answer:** Inject a fault or replay data and confirm it fires and routes

Verify.

**Q79.** What's included in a runbook?

**Answer:** Meaning, impact, diagnosis steps and mitigation

Actionable.

**Q80.** How prove it's not noisy?

**Answer:** Review pages over time and fix false positives

Iterate.

## Level 3 · Distributed Tracing & OpenTelemetry

### [OpenTelemetry Fundamentals: the Vendor-Neutral Standard](level-3/01-opentelemetry-fundamentals.md)

**Q81.** What is OpenTelemetry?

**Answer:** A vendor-neutral standard and SDKs for traces, metrics and logs

CNCF project.

**Q82.** What does OTel provide beyond SDKs?

**Answer:** A wire protocol (OTLP) and a Collector

Standard.

**Q83.** Why vendor-neutral instrumentation?

**Answer:** Switch backends without re-instrumenting code

Portability.

**Q84.** What are semantic conventions?

**Answer:** Standard names/attributes for common concepts (http.method)

Consistency.

### [Instrumenting Code: Spans &amp; Context Propagation](level-3/02-instrumenting-spans-context.md)

**Q85.** What is a span?

**Answer:** A timed operation within a trace with attributes and parent link

Unit of work.

**Q86.** What is context propagation?

**Answer:** Passing trace IDs across process boundaries via headers

traceparent.

**Q87.** What standard header carries trace context?

**Answer:** W3C traceparent

Interoperable.

**Q88.** Why add attributes to spans?

**Answer:** Filtering and analysis by business/technical dimensions

Bounded values.

### [OpenTelemetry Metrics: the Convergence with Prometheus](level-3/03-otel-metrics-convergence.md)

**Q89.** What does OTel metrics aim for?

**Answer:** A unified API/SDK with Prometheus-compatible export

Convergence.

**Q90.** What are exemplars?

**Answer:** Sample trace IDs attached to metric points linking to traces

Metric to trace.

**Q91.** What's delta vs cumulative temporality?

**Answer:** Delta reports change per interval; cumulative reports totals since start

Backend dependent.

**Q92.** What helps migration from Prometheus clients?

**Answer:** The OTel Prometheus bridge/exporter

Gradual.

### [Exporters &amp; the Collector: Decoupling Instrumentation from Backend](level-3/04-exporters-collector.md)

**Q93.** What does the OTel Collector do?

**Answer:** Receives, processes and exports telemetry vendor-neutrally

Pipeline.

**Q94.** Why run a Collector?

**Answer:** Batching, retries, filtering and central config away from apps

Decouple.

**Q95.** What are processors for?

**Answer:** Transforming/filtering telemetry (batch, attributes, sampling)

Pipeline steps.

**Q96.** Agent vs gateway mode?

**Answer:** Agent runs per host; gateway is a central service

Topologies.

### [Sampling: Why You Can't (and Shouldn't) Trace Everything](level-3/05-sampling-strategies.md)

**Q97.** Why sample traces?

**Answer:** Full tracing is expensive at scale

Cost control.

**Q98.** Head vs tail sampling?

**Answer:** Head decides at trace start; tail decides after seeing the whole trace

Tail keeps errors.

**Q99.** What does tail sampling enable?

**Answer:** Keeping all error/slow traces while dropping routine ones

Smart retention.

**Q100.** What's the downside of tail sampling?

**Answer:** Needs buffering and more compute/memory

Trade-off.

### [Correlating the Three Pillars for One Real Request](level-3/06-correlating-three-pillars.md)

**Q101.** How link metrics, logs and traces?

**Answer:** Shared IDs/labels (trace_id in logs, exemplars on metrics)

Pivot between signals.

**Q102.** What's a good debugging flow?

**Answer:** Alert from metrics, find the slow trace, read its logs

Drill down.

**Q103.** What do exemplars add?

**Answer:** Jump from a latency bucket to an example trace

Practical.

**Q104.** What makes correlation reliable?

**Answer:** Consistent propagation of IDs through every hop

Instrumentation.

### [Cardinality: the Silent Killer of Metrics Systems](level-3/07-cardinality-explosion.md)

**Q105.** What causes a cardinality explosion?

**Answer:** Unbounded label values like user IDs or raw URLs

Series multiply.

**Q106.** What are its symptoms?

**Answer:** High memory/CPU, slow queries and scrape rejections

Outages.

**Q107.** A metric with labels user (10,000 values) and endpoint (50 values) creates how many series at most?

```python
print(10000*50)
```

**Answer:** 500000

Product of cardinalities. Verified by running it.

**Q108.** How mitigate?

**Answer:** Drop/aggregate high-cardinality labels and limit per-scrape series

relabel/sample_limit.

### [Instrumentation Overhead: Measuring the Cost of Observability](level-3/08-instrumentation-overhead.md)

**Q109.** What does instrumentation overhead depend on?

**Answer:** Frequency of calls, attribute counts and sampling

Measure it.

**Q110.** How reduce overhead?

**Answer:** Sampling, batching exports and avoiding expensive attributes in hot paths

Async.

**Q111.** Why benchmark instrumentation?

**Answer:** To ensure observability doesn't harm latency

Budget.

**Q112.** What's a safe default?

**Answer:** Instrument boundaries (requests, dependencies), not every function

Signal over noise.

### [Black-Box vs White-Box Monitoring](level-3/09-blackbox-vs-whitebox.md)

**Q113.** What is blackbox monitoring?

**Answer:** Probing from outside like a user (HTTP checks)

Symptoms.

**Q114.** What is whitebox monitoring?

**Answer:** Using internal metrics exposed by the system

Causes.

**Q115.** Why use both?

**Answer:** Blackbox shows user impact; whitebox explains why

Complementary.

**Q116.** What tool does blackbox probes?

**Answer:** Prometheus blackbox_exporter or synthetic monitors

Uptime.

### [Project: Full-Stack Observability for a Real Request](level-3/10-project-full-stack-observability.md)

**Q117.** What does the full-stack observability project combine?

**Answer:** Metrics, logs and traces correlated across services

Level 3 project.

**Q118.** How prove correlation works?

**Answer:** Follow one request from alert to trace to logs

End to end.

**Q119.** What should be in the Collector config?

**Answer:** Receivers, processors, exporters and pipelines

Wiring.

**Q120.** What demonstrates value?

**Answer:** Faster root-cause time in a staged incident

MTTR.

## Level 4 · SLOs & Operating Observability at Scale

### [SLIs, SLOs &amp; SLAs: the Real Differences](level-4/01-slis-slos-slas.md)

**Q121.** What is an SLI?

**Answer:** A measured indicator of service quality (e.g. fraction of good requests)

Measurement.

**Q122.** What is an SLO?

**Answer:** A target for an SLI over a window (e.g. 99.9% over 30 days)

Goal.

**Q123.** What is an SLA?

**Answer:** A contractual commitment with consequences

Looser than SLO.

**Q124.** Why SLOs rather than 100%?

**Answer:** 100% is unattainable and blocks change; error budgets balance risk

Reliability trade-offs.

### [Error Budgets: Turning Reliability Into a Spendable Number](level-4/02-error-budgets.md)

**Q125.** What is the 30-day error budget in minutes for a 99.9% SLO?

```python
print(round(30*24*60*0.001,1))
```

**Answer:** 43.2

1 - SLO of the window. Verified by running it.

**Q126.** What does the error budget guide?

**Answer:** Release velocity: spend it on change, freeze when exhausted

Policy.

**Q127.** What happens when the budget is spent?

**Answer:** Prioritize reliability work over features

Agreed policy.

**Q128.** Who should agree the policy?

**Answer:** Product, engineering and SRE together

Shared.

### [Burn Rate Alerts: Multi-Window Multi-Burn-Rate Alerting](level-4/03-burn-rate-alerts.md)

**Q129.** What is burn rate?

**Answer:** How fast the error budget is consumed relative to the allowed pace

1 = on pace.

**Q130.** Why multi-window burn-rate alerts?

**Answer:** Fast detection of big burns with low false positives (long and short windows)

Google SRE workbook.

**Q131.** At burn rate 14.4 how long to exhaust a 30-day budget?

```python
print(round(30*24/14.4,1))
```

**Answer:** 50.0

hours = 720 / burn rate. Verified by running it.

**Q132.** Why better than threshold alerts?

**Answer:** Tied to user impact and budget, so fewer false pages

SLO-based.

### [Designing an SLO for a Real Service](level-4/04-designing-an-slo.md)

**Q133.** How choose an SLI for a service?

**Answer:** Pick what users experience: availability, latency, correctness

User-centric.

**Q134.** What makes a good SLO target?

**Answer:** Achievable and meaningful, informed by history and user needs

Iterate.

**Q135.** Why ratio-based SLIs (good/total)?

**Answer:** Normalize for traffic and are easy to aggregate

Good events/valid events.

**Q136.** What should excluded traffic be?

**Answer:** Documented, e.g. health checks and synthetic traffic

Honest SLI.

### [Capacity Planning From Real Metrics](level-4/05-capacity-planning.md)

**Q137.** What drives capacity planning?

**Answer:** Demand growth, headroom targets and saturation signals

Forecast.

**Q138.** What metrics guide it?

**Answer:** Utilization and saturation trends of key resources

USE.

**Q139.** What is headroom?

**Answer:** Spare capacity to absorb spikes and failures

N+1.

**Q140.** Why load test?

**Answer:** To find limits and validate the model

Evidence.

### [Scaling Prometheus: Federation, Remote Write &amp; Long-Term Storage](level-4/06-scaling-prometheus.md)

**Q141.** How scale Prometheus?

**Answer:** Shard by function, federate and use remote storage (Thanos/Mimir)

Horizontal.

**Q142.** What does federation do?

**Answer:** Pulls selected series from other Prometheus servers

Hierarchy.

**Q143.** What's Thanos/Mimir for?

**Answer:** Global query view and long-term storage over many Prometheus instances

Scale-out.

**Q144.** First scaling step?

**Answer:** Reduce cardinality and scrape only what's needed

Hygiene.

### [Observability as Code: Dashboards &amp; Alerts in Version Control](level-4/07-observability-as-code.md)

**Q145.** What is observability as code?

**Answer:** Defining dashboards, alerts and SLOs in version-controlled files

GitOps.

**Q146.** Why?

**Answer:** Review, reproducibility and consistency across environments

Maintainability.

**Q147.** Which tools help?

**Answer:** Grafonnet/Terraform/Jsonnet and Sloth/OpenSLO

Generate rules.

**Q148.** What should CI validate?

**Answer:** Rule syntax (promtool) and tests for alert expressions

promtool test rules.

### [On-Call &amp; Incident Response With Your Own Stack](level-4/08-oncall-incident-response.md)

**Q149.** What makes on-call sustainable?

**Answer:** Actionable alerts, runbooks, rotation fairness and blameless reviews

Culture.

**Q150.** What's the first incident priority?

**Answer:** Mitigate user impact, then find root cause

Stop the bleeding.

**Q151.** What belongs in a postmortem?

**Answer:** Timeline, impact, root cause and action items without blame

Learn.

**Q152.** What roles help big incidents?

**Answer:** Incident commander, communications and operations leads

Coordination.

### [Common Observability Anti-Patterns](level-4/09-observability-antipatterns.md)

**Q153.** Which is an observability antipattern?

**Answer:** Dashboards nobody uses and alerts nobody acts on

Noise.

**Q154.** Another?

**Answer:** Logging everything at debug level in production

Cost and noise.

**Q155.** Another?

**Answer:** Alerting on causes (CPU) instead of symptoms

Symptoms matter.

**Q156.** Another?

**Answer:** High-cardinality labels like user IDs

Series explosion.

### [Capstone: Build a Complete SLO-Driven Observability Stack](level-4/10-capstone-slo-driven-stack.md)

**Q157.** What does the SLO-driven stack capstone combine?

**Answer:** SLIs/SLOs, burn-rate alerts, dashboards, tracing and incident process

End to end.

**Q158.** How prove it works?

**Answer:** Inject failures and confirm detection, paging and recovery against the SLO

Game day.

**Q159.** What documentation helps?

**Answer:** SLO definitions, runbooks and ownership

Operate.

**Q160.** What sustains it?

**Answer:** Regular SLO reviews and alert hygiene

Continuous improvement.

## Mixed questions across all levels

**Q161.** What is the difference between monitoring and observability?

**Answer:** Monitoring watches known failure modes; observability lets you ask new questions of telemetry

Unknown unknowns.

**Q162.** Why use a counter with rate() instead of a gauge for requests served?

**Answer:** Counters are monotonic and rate() handles restarts; the per-second rate is what you want

Metric types.

**Q163.** A request counter is summed across instances and then rate() is applied, giving odd results. What's the correct order?

**Answer:** rate() per series first, then sum()

Resets are per series.

**Q164.** You want p99 latency across 20 instances. What's right?

**Answer:** histogram_quantile(0.99, sum by (le) (rate(bucket[5m])))

Percentiles can't be averaged.

**Q165.** A metric has labels user (5,000 values) and route (40 values). How many series at most?

```python
print(5000*40)
```

**Answer:** 200000

Cardinality multiplies. Verified by running it.

**Q166.** Prometheus memory grows rapidly after a deploy. What's the first suspect?

**Answer:** A new high-cardinality label (IDs, raw URLs)

Series explosion.

**Q167.** An alert fires on CPU &gt; 80% and nobody can act on it. What's the better approach?

**Answer:** Alert on user-facing symptoms (error rate, latency) tied to an SLO

Actionable alerts.

**Q168.** What does `for: 10m` do in an alerting rule?

**Answer:** Requires the condition to hold for 10 minutes before firing, reducing flapping

Stability.

**Q169.** Alertmanager sends 200 notifications for one outage. What helps?

**Answer:** Grouping and inhibition rules (and routing by team)

Noise control.

**Q170.** Logs from five services can't be tied to one request. What's missing?

**Answer:** A correlation/trace ID propagated through every hop and logged

Structured logging.

**Q171.** Why add exemplars to latency histograms?

**Answer:** To jump from a slow bucket to an example trace

Metrics to traces.

**Q172.** 100% of traces are exported and costs explode. What do you do?

**Answer:** Sample: tail sampling keeps errors and slow traces, drops routine ones

Cost control.

**Q173.** Which header propagates trace context between services?

**Answer:** W3C traceparent

Standard.

**Q174.** What is the 30-day error budget in minutes for a 99.9% SLO?

```python
print(round(30*24*60*0.001,1))
```

**Answer:** 43.2

1 - SLO of the window. Verified by running it.

**Q175.** At a burn rate of 14.4, how many hours until a 30-day budget is gone?

```python
print(round(30*24/14.4,1))
```

**Answer:** 50.0

Window hours / burn rate. Verified by running it.

**Q176.** Why multi-window burn-rate alerts?

**Answer:** Fast detection of large burns without paging on blips or already-resolved incidents

Long and short windows.

**Q177.** The error budget is exhausted. What does a healthy policy do?

**Answer:** Slow feature releases and prioritize reliability work until it recovers

Agreed with product.

**Q178.** Which is an observability antipattern?

**Answer:** Dashboards and alerts nobody uses or acts on

Noise hides signal.

**Q179.** Probing the public endpoint from outside every minute is what kind of monitoring?

**Answer:** Blackbox (synthetic) monitoring: it sees what users see

Use both.

## Practical coding questions

### PromQL rate() and histogram_quantile()

Implement rate (counter increase per second over a window, handling a counter reset), histogram_quantile (linear interpolation inside the bucket) and sum_by_le. Show why averaging per-instance percentiles gives a wrong answer.

Starter code:

```
def rate(samples, window_s, now):
    # samples = [(timestamp, counter value)]. Use points within [now - window_s, now]; add each increase between
    # neighbouring points (a smaller value means the counter reset, so the increase is the new value);
    # return increase per second between the first and last point (0.0 when fewer than 2 points)
    return 0.0

def histogram_quantile(q, buckets):
    # buckets = [(upper_bound, cumulative_count)] ending with float("inf"). Find the first bucket whose cumulative
    # count >= q * total and linearly interpolate between its lower and upper bound (+Inf bucket: return the
    # previous bound); NaN when there are no observations
    return float("nan")

def sum_by_le(*series):
    # element-wise sum of the cumulative counts for the same bucket bounds
    return series[0]

samples = [(0, 100), (15, 130), (30, 160), (45, 10), (60, 40)]      # counter resets at t=45
print("rate over 60s:", rate(samples, 60, 60))
inf = float("inf")
a = [(0.1, 50), (0.25, 80), (0.5, 95), (1.0, 99), (inf, 100)]
b = [(0.1, 10), (0.25, 30), (0.5, 60), (1.0, 90), (inf, 100)]
print("p95 instance a:", round(histogram_quantile(0.95, a), 3))
print("p95 instance b:", round(histogram_quantile(0.95, b), 3))
print("p95 combined:  ", round(histogram_quantile(0.95, sum_by_le(a, b)), 3))
print("mean of instance p95s:", round((histogram_quantile(0.95, a) + histogram_quantile(0.95, b)) / 2, 3), "(wrong)")
```

Expected output:

```text
rate over 60s: 1.6666666666666667
p95 instance a: 0.5
p95 instance b: 1.0
p95 combined:   1.0
mean of instance p95s: 0.75 (wrong)
```

??? success "Show a solution"

    ```
    def rate(samples, window_s, now):
        """Per-second increase of a counter over the window, handling counter resets (like PromQL rate)."""
        pts = [(t, v) for t, v in samples if now - window_s <= t <= now]
        if len(pts) < 2:
            return 0.0
        increase = 0.0
        for (t0, v0), (t1, v1) in zip(pts, pts[1:]):
            increase += v1 - v0 if v1 >= v0 else v1          # a drop means the counter reset to zero
        return increase / (pts[-1][0] - pts[0][0])

    def histogram_quantile(q, buckets):
        """buckets: [(upper_bound, cumulative_count)] sorted by bound, last bound is +Inf (as float('inf'))."""
        total = buckets[-1][1]
        if total == 0:
            return float("nan")
        rank = q * total
        prev_bound, prev_count = 0.0, 0
        for bound, count in buckets:
            if count >= rank:
                if bound == float("inf"):
                    return prev_bound
                frac = (rank - prev_count) / (count - prev_count) if count > prev_count else 0
                return prev_bound + (bound - prev_bound) * frac      # linear interpolation inside the bucket
            prev_bound, prev_count = bound, count

    def sum_by_le(*series):
        return [(b, sum(s[i][1] for s in series)) for i, (b, _) in enumerate(series[0])]

    samples = [(0, 100), (15, 130), (30, 160), (45, 10), (60, 40)]      # counter resets at t=45
    print("rate over 60s:", rate(samples, 60, 60))
    inf = float("inf")
    a = [(0.1, 50), (0.25, 80), (0.5, 95), (1.0, 99), (inf, 100)]
    b = [(0.1, 10), (0.25, 30), (0.5, 60), (1.0, 90), (inf, 100)]
    print("p95 instance a:", round(histogram_quantile(0.95, a), 3))
    print("p95 instance b:", round(histogram_quantile(0.95, b), 3))
    print("p95 combined:  ", round(histogram_quantile(0.95, sum_by_le(a, b)), 3))
    print("mean of instance p95s:", round((histogram_quantile(0.95, a) + histogram_quantile(0.95, b)) / 2, 3), "(wrong)")
    ```

    A drop in a counter means it reset, so the increase is the new value; quantiles are estimated from cumulative buckets and must be computed after summing the buckets across instances. Verified by running the solution (a model of PromQL's behaviour).

### SLO error budgets and burn-rate paging

Implement error_ratio, budget_minutes, burn_rate and a multi-window should_page rule, then evaluate healthy, on-pace, incident and outage scenarios and decide which should page or ticket.

Starter code:

```
def error_ratio(good, total):
    # 1 - good/total (0.0 when there is no traffic)
    return 0.0

def budget_minutes(slo, days=30):
    # allowed "bad" minutes in the window
    return 0.0

def burn_rate(err_ratio, slo):
    # error ratio divided by the allowed error ratio (1 - slo)
    return 0.0

def should_page(short_window, long_window, slo, threshold):
    # page only when BOTH windows' burn rates reach the threshold
    return False

slo = 0.999
print("budget (30d):", round(budget_minutes(slo), 1), "min")
for name, err in [("healthy", 0.0004), ("on pace", 0.001), ("incident", 0.02), ("outage", 0.5)]:
    b = burn_rate(err, slo)
    hours = 30 * 24 / b if b > 0 else float("inf")
    print(f"{name:8s} error={err:.4f} burn={b:6.1f} budget gone in {hours:7.1f} h")

print("page (1h=2%, 5m=3%, burn>=14.4):", should_page(0.03, 0.02, slo, 14.4))
print("page (1h=2%, 5m=0.05%):         ", should_page(0.0005, 0.02, slo, 14.4))      # incident already over
print("page (1h=0.5%, 5m=0.5%):        ", should_page(0.005, 0.005, slo, 14.4))      # slow burn, below fast threshold
print("ticket (6h=0.7%, 30m=0.8%, burn>=6):", should_page(0.008, 0.007, slo, 6))
print("error ratio:", round(error_ratio(99_700, 100_000), 4))
```

Expected output:

```text
budget (30d): 43.2 min
healthy  error=0.0004 burn=   0.4 budget gone in  1800.0 h
on pace  error=0.0010 burn=   1.0 budget gone in   720.0 h
incident error=0.0200 burn=  20.0 budget gone in    36.0 h
outage   error=0.5000 burn= 500.0 budget gone in     1.4 h
page (1h=2%, 5m=3%, burn>=14.4): True
page (1h=2%, 5m=0.05%):          False
page (1h=0.5%, 5m=0.5%):         False
ticket (6h=0.7%, 30m=0.8%, burn>=6): True
error ratio: 0.003
```

??? success "Show a solution"

    ```
    def error_ratio(good, total):
        return 0.0 if total == 0 else 1 - good / total

    def budget_minutes(slo, days=30):
        return days * 24 * 60 * (1 - slo)

    def burn_rate(err_ratio, slo):
        """How many times faster than 'exactly on budget' the budget is being consumed."""
        return err_ratio / (1 - slo)

    def should_page(short_window, long_window, slo, threshold):
        """Multi-window multi-burn-rate rule: both the short and the long window must exceed the threshold."""
        return burn_rate(short_window, slo) >= threshold and burn_rate(long_window, slo) >= threshold

    slo = 0.999
    print("budget (30d):", round(budget_minutes(slo), 1), "min")
    for name, err in [("healthy", 0.0004), ("on pace", 0.001), ("incident", 0.02), ("outage", 0.5)]:
        b = burn_rate(err, slo)
        hours = 30 * 24 / b if b > 0 else float("inf")
        print(f"{name:8s} error={err:.4f} burn={b:6.1f} budget gone in {hours:7.1f} h")

    print("page (1h=2%, 5m=3%, burn>=14.4):", should_page(0.03, 0.02, slo, 14.4))
    print("page (1h=2%, 5m=0.05%):         ", should_page(0.0005, 0.02, slo, 14.4))      # incident already over
    print("page (1h=0.5%, 5m=0.5%):        ", should_page(0.005, 0.005, slo, 14.4))      # slow burn, below fast threshold
    print("ticket (6h=0.7%, 30m=0.8%, burn>=6):", should_page(0.008, 0.007, slo, 6))
    print("error ratio:", round(error_ratio(99_700, 100_000), 4))
    ```

    Burn rate is the observed error ratio divided by the allowed one; requiring both a long and a short window to exceed the threshold pages quickly on real incidents but not on resolved blips. Verified by running the solution.

---

More free practice: the [Observability Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
