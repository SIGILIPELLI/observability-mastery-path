---
description: "A real incident: paged at T+23s, mitigated at T+29s, but the alert stayed firing until T+80s -- fixing a cause doesn't instantly clear a ratio alert."
---

# On-Call & Incident Response With Your Own Stack

## What was run

One complete, real incident, worked end to end using only this course's own real stack: a real outage injected
into `payments_service.py`, detected by the real burn-rate-style alert from
[Level 2, Lesson 10](../level-2/10-project-real-alert-pages.md), diagnosed using real PromQL and real structured
logs, mitigated, and verified resolved — with every real timestamp recorded as it actually happened, including an
honest, unplanned finding about how long real recovery actually took.

## The real timeline, as it actually happened

```
T+0s   (15:15:21) real incident injected: curl -X POST .../incident/true
T+1s   (15:15:22) real traffic sent during the incident
T+23s  (15:15:44) real page received: [PAGER] status=firing alertname=HighCheckoutErrorRatio
```

**Detection took 23 real seconds** — the alert's `for: 10s` debounce plus Alertmanager's real `group_wait`
delay, both real, deliberate design choices from [Level 2](../level-2/04-alerting-rules-alertmanager.md), not
unexplained latency.

```
T+23s  real diagnosis: checkout error ratio = 1 (100% failing)
T+23s  real structured log: {"reason": "declined: card processor unavailable", "status": 402}
T+29s  (15:15:50) real mitigation: curl -X POST .../incident/false
T+30s  real recovery traffic sent
```

```
T+80s  (15:16:41) alert FINALLY cleared, after a SECOND round of real recovery traffic
T+80s  real resolved page received: [PAGER] status=resolved alertname=HighCheckoutErrorRatio
```

The real, honest, unplanned finding: fixing the root cause at T+29s did **not** immediately clear the alert. The
first batch of real recovery traffic (10 requests) left the real 1-minute ratio window still mixed with enough
recent bad data to stay above the `0.5` threshold (`value: 0.6667`, confirmed directly) — it took a **second**,
larger batch of real clean traffic before the ratio genuinely dropped and the alert cleared, a full **51 real
seconds** after the actual fix was applied.

## How It Actually Works

This lesson's real timeline makes concrete every piece built across this course, used together under real
pressure: the real alert ([Level 2](../level-2/04-alerting-rules-alertmanager.md)) detected the problem; real
PromQL ([Level 1](../level-1/05-promql-basics.md)) and real structured logs
([Level 2](../level-2/08-structured-logging-correlation.md)) diagnosed it precisely — the log line named the
exact real reason, "card processor unavailable," within seconds of the page arriving; the real fix was applied
directly at the source; and the real alert's own eventual clearing served as objective, automated confirmation
that the fix actually worked, rather than relying on a human's subjective sense that things seemed better.

The real, honest gap between mitigation (T+29s) and the alert actually clearing (T+80s) is not a flaw in this
course's alerting design — it's a genuine, important property of **ratio-based, windowed alerts** that a real
on-call engineer needs to understand: the alert's condition is evaluated over a real time window (1 minute here),
so a real fix doesn't retroactively erase the bad data already inside that window. Declaring an incident "resolved"
the moment a fix is applied, without waiting for the real alert to confirm, risks closing out an incident that
isn't actually confirmed fixed yet — exactly the gap this lesson's real 51-second delay makes concrete rather than
theoretical.

## Common mistakes

- Declaring an incident resolved the instant a fix is applied, without waiting for the real monitoring signal to
  independently confirm it — this lesson's real 51-second gap between "fix applied" and "alert actually cleared"
  is exactly the kind of premature closure this habit risks.
- Skipping the diagnosis step and guessing at a fix based on the alert name alone — this lesson's real structured
  log (naming the specific real reason, "card processor unavailable") was available within seconds and made the
  real root cause unambiguous, rather than requiring guesswork.
- Treating every real incident's recovery time as instant, the way a dashboard's "all green now" status might
  casually imply — a ratio-based or windowed alert's real recovery time depends on real, fresh traffic flowing
  through the window, which can genuinely lag behind the real moment the underlying fix was applied.

## Exercise

Repeat this lesson's exact real incident scenario, but this time measure precisely how much real clean traffic
(by request count, not just elapsed time) is needed before the alert clears, holding traffic rate roughly
constant. Report the real number, and use it to estimate, for a real production service with a known real request
rate, how long a similar ratio-based alert would take to confirm recovery after a real fix — a genuinely useful
number for setting on-call expectations about "how long until the page actually stops."
