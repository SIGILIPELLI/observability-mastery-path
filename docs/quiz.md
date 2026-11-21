---
title: "Final Quiz"
description: "Test what you have learned across all four levels of Observability Mastery Path with a 19-question final quiz."
---

# Final Quiz

19 questions drawn from all four levels of this course. You need **70% (14 out of 19)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "What is the difference between monitoring and observability?",
    "options": [
      "Monitoring watches known failure modes; observability lets you ask new questions of telemetry",
      "Dashboards vs alerts",
      "Same",
      "Reverse"
    ],
    "correctIndex": 0,
    "explanation": "Unknown unknowns."
  },
  {
    "question": "Why use a counter with rate() instead of a gauge for requests served?",
    "options": [
      "Counters are bigger",
      "Counters are monotonic and rate() handles restarts; the per-second rate is what you want",
      "Gauges are slow",
      "Law"
    ],
    "correctIndex": 1,
    "explanation": "Metric types."
  },
  {
    "question": "A request counter is summed across instances and then rate() is applied, giving odd results. What's the correct order?",
    "options": [
      "rate() per series first, then sum()",
      "sum then rate",
      "avg then rate",
      "max then sum"
    ],
    "correctIndex": 0,
    "explanation": "Resets are per series."
  },
  {
    "question": "You want p99 latency across 20 instances. What's right?",
    "options": [
      "avg of p99s",
      "max of p99s",
      "quantile of averages",
      "histogram_quantile(0.99, sum by (le) (rate(bucket[5m])))"
    ],
    "correctIndex": 3,
    "explanation": "Percentiles can't be averaged."
  },
  {
    "question": "A metric has labels user (5,000 values) and route (40 values). How many series at most?\n```python\nprint(5000*40)\n```\n",
    "options": [
      "5000",
      "200000",
      "5040",
      "40000"
    ],
    "correctIndex": 1,
    "explanation": "Cardinality multiplies. Verified by running it."
  },
  {
    "question": "Prometheus memory grows rapidly after a deploy. What's the first suspect?",
    "options": [
      "Scrape interval",
      "Grafana",
      "Retention",
      "A new high-cardinality label (IDs, raw URLs)"
    ],
    "correctIndex": 3,
    "explanation": "Series explosion."
  },
  {
    "question": "An alert fires on CPU > 80% and nobody can act on it. What's the better approach?",
    "options": [
      "Lower threshold",
      "More alerts",
      "Alert on user-facing symptoms (error rate, latency) tied to an SLO",
      "Mute it"
    ],
    "correctIndex": 2,
    "explanation": "Actionable alerts."
  },
  {
    "question": "What does `for: 10m` do in an alerting rule?",
    "options": [
      "Requires the condition to hold for 10 minutes before firing, reducing flapping",
      "Silences",
      "Repeats",
      "Delays"
    ],
    "correctIndex": 0,
    "explanation": "Stability."
  },
  {
    "question": "Alertmanager sends 200 notifications for one outage. What helps?",
    "options": [
      "More receivers",
      "Grouping and inhibition rules (and routing by team)",
      "Silence all",
      "Disable alerts"
    ],
    "correctIndex": 1,
    "explanation": "Noise control."
  },
  {
    "question": "Logs from five services can't be tied to one request. What's missing?",
    "options": [
      "Timestamps",
      "A correlation/trace ID propagated through every hop and logged",
      "Hostnames",
      "Levels"
    ],
    "correctIndex": 1,
    "explanation": "Structured logging."
  },
  {
    "question": "Why add exemplars to latency histograms?",
    "options": [
      "Storage",
      "Law",
      "To jump from a slow bucket to an example trace",
      "Speed"
    ],
    "correctIndex": 2,
    "explanation": "Metrics to traces."
  },
  {
    "question": "100% of traces are exported and costs explode. What do you do?",
    "options": [
      "Sample: tail sampling keeps errors and slow traces, drops routine ones",
      "Disable tracing",
      "Log more",
      "Shorten spans"
    ],
    "correctIndex": 0,
    "explanation": "Cost control."
  },
  {
    "question": "Which header propagates trace context between services?",
    "options": [
      "Authorization",
      "X-Request",
      "Cookie",
      "W3C traceparent"
    ],
    "correctIndex": 3,
    "explanation": "Standard."
  },
  {
    "question": "What is the 30-day error budget in minutes for a 99.9% SLO?\n```python\nprint(round(30*24*60*0.001,1))\n```\n",
    "options": [
      "4.3",
      "43.2",
      "432",
      "21.6"
    ],
    "correctIndex": 1,
    "explanation": "1 - SLO of the window. Verified by running it."
  },
  {
    "question": "At a burn rate of 14.4, how many hours until a 30-day budget is gone?\n```python\nprint(round(30*24/14.4,1))\n```\n",
    "options": [
      "14.4",
      "50.0",
      "5.0",
      "500.0"
    ],
    "correctIndex": 1,
    "explanation": "Window hours / burn rate. Verified by running it."
  },
  {
    "question": "Why multi-window burn-rate alerts?",
    "options": [
      "Simpler",
      "Cheaper",
      "Fast detection of large burns without paging on blips or already-resolved incidents",
      "Law"
    ],
    "correctIndex": 2,
    "explanation": "Long and short windows."
  },
  {
    "question": "The error budget is exhausted. What does a healthy policy do?",
    "options": [
      "Ship faster",
      "Raise the SLO",
      "Slow feature releases and prioritize reliability work until it recovers",
      "Ignore"
    ],
    "correctIndex": 2,
    "explanation": "Agreed with product."
  },
  {
    "question": "Which is an observability antipattern?",
    "options": [
      "SLO alerts",
      "Runbooks",
      "Tracing",
      "Dashboards and alerts nobody uses or acts on"
    ],
    "correctIndex": 3,
    "explanation": "Noise hides signal."
  },
  {
    "question": "Probing the public endpoint from outside every minute is what kind of monitoring?",
    "options": [
      "Whitebox",
      "Blackbox (synthetic) monitoring: it sees what users see",
      "Profiling",
      "Tracing"
    ],
    "correctIndex": 1,
    "explanation": "Use both."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### PromQL rate() and histogram_quantile()

Implement rate (counter increase per second over a window, handling a counter reset), histogram_quantile (linear interpolation inside the bucket) and sum_by_le. Show why averaging per-instance percentiles gives a wrong answer.

**Starter code**

```python
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

**Expected output**

```text
rate over 60s: 1.6666666666666667
p95 instance a: 0.5
p95 instance b: 1.0
p95 combined:   1.0
mean of instance p95s: 0.75 (wrong)
```

<details><summary>Show solution and explanation</summary>

<pre><code>def rate(samples, window_s, now):
    &quot;&quot;&quot;Per-second increase of a counter over the window, handling counter resets (like PromQL rate).&quot;&quot;&quot;
    pts = [(t, v) for t, v in samples if now - window_s &lt;= t &lt;= now]
    if len(pts) &lt; 2:
        return 0.0
    increase = 0.0
    for (t0, v0), (t1, v1) in zip(pts, pts[1:]):
        increase += v1 - v0 if v1 &gt;= v0 else v1          # a drop means the counter reset to zero
    return increase / (pts[-1][0] - pts[0][0])

def histogram_quantile(q, buckets):
    &quot;&quot;&quot;buckets: [(upper_bound, cumulative_count)] sorted by bound, last bound is +Inf (as float(&#x27;inf&#x27;)).&quot;&quot;&quot;
    total = buckets[-1][1]
    if total == 0:
        return float(&quot;nan&quot;)
    rank = q * total
    prev_bound, prev_count = 0.0, 0
    for bound, count in buckets:
        if count &gt;= rank:
            if bound == float(&quot;inf&quot;):
                return prev_bound
            frac = (rank - prev_count) / (count - prev_count) if count &gt; prev_count else 0
            return prev_bound + (bound - prev_bound) * frac      # linear interpolation inside the bucket
        prev_bound, prev_count = bound, count

def sum_by_le(*series):
    return [(b, sum(s[i][1] for s in series)) for i, (b, _) in enumerate(series[0])]

samples = [(0, 100), (15, 130), (30, 160), (45, 10), (60, 40)]      # counter resets at t=45
print(&quot;rate over 60s:&quot;, rate(samples, 60, 60))
inf = float(&quot;inf&quot;)
a = [(0.1, 50), (0.25, 80), (0.5, 95), (1.0, 99), (inf, 100)]
b = [(0.1, 10), (0.25, 30), (0.5, 60), (1.0, 90), (inf, 100)]
print(&quot;p95 instance a:&quot;, round(histogram_quantile(0.95, a), 3))
print(&quot;p95 instance b:&quot;, round(histogram_quantile(0.95, b), 3))
print(&quot;p95 combined:  &quot;, round(histogram_quantile(0.95, sum_by_le(a, b)), 3))
print(&quot;mean of instance p95s:&quot;, round((histogram_quantile(0.95, a) + histogram_quantile(0.95, b)) / 2, 3), &quot;(wrong)&quot;)
</code></pre>

<p>A drop in a counter means it reset, so the increase is the new value; quantiles are estimated from cumulative buckets and must be computed after summing the buckets across instances. Verified by running the solution (a model of PromQL&#x27;s behaviour).</p>
</details>

### SLO error budgets and burn-rate paging

Implement error_ratio, budget_minutes, burn_rate and a multi-window should_page rule, then evaluate healthy, on-pace, incident and outage scenarios and decide which should page or ticket.

**Starter code**

```python
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

**Expected output**

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

<details><summary>Show solution and explanation</summary>

<pre><code>def error_ratio(good, total):
    return 0.0 if total == 0 else 1 - good / total

def budget_minutes(slo, days=30):
    return days * 24 * 60 * (1 - slo)

def burn_rate(err_ratio, slo):
    &quot;&quot;&quot;How many times faster than &#x27;exactly on budget&#x27; the budget is being consumed.&quot;&quot;&quot;
    return err_ratio / (1 - slo)

def should_page(short_window, long_window, slo, threshold):
    &quot;&quot;&quot;Multi-window multi-burn-rate rule: both the short and the long window must exceed the threshold.&quot;&quot;&quot;
    return burn_rate(short_window, slo) &gt;= threshold and burn_rate(long_window, slo) &gt;= threshold

slo = 0.999
print(&quot;budget (30d):&quot;, round(budget_minutes(slo), 1), &quot;min&quot;)
for name, err in [(&quot;healthy&quot;, 0.0004), (&quot;on pace&quot;, 0.001), (&quot;incident&quot;, 0.02), (&quot;outage&quot;, 0.5)]:
    b = burn_rate(err, slo)
    hours = 30 * 24 / b if b &gt; 0 else float(&quot;inf&quot;)
    print(f&quot;{name:8s} error={err:.4f} burn={b:6.1f} budget gone in {hours:7.1f} h&quot;)

print(&quot;page (1h=2%, 5m=3%, burn&gt;=14.4):&quot;, should_page(0.03, 0.02, slo, 14.4))
print(&quot;page (1h=2%, 5m=0.05%):         &quot;, should_page(0.0005, 0.02, slo, 14.4))      # incident already over
print(&quot;page (1h=0.5%, 5m=0.5%):        &quot;, should_page(0.005, 0.005, slo, 14.4))      # slow burn, below fast threshold
print(&quot;ticket (6h=0.7%, 30m=0.8%, burn&gt;=6):&quot;, should_page(0.008, 0.007, slo, 6))
print(&quot;error ratio:&quot;, round(error_ratio(99_700, 100_000), 4))
</code></pre>

<p>Burn rate is the observed error ratio divided by the allowed one; requiring both a long and a short window to exceed the threshold pages quickly on real incidents but not on resolved blips. Verified by running the solution.</p>
</details>
