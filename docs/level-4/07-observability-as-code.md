---
description: "A real dashboard, exported to JSON, committed to git, diffed, and re-applied through Grafana's API -- confirmed live with a new panel and version bump."
---

# Observability as Code: Dashboards & Alerts in Version Control

## What was run

The real RED dashboard from [Level 2, Lesson 2](../level-2/02-red-dashboard.md) was exported as real JSON from
Grafana's own API, committed to a real local git repository, modified with a real new panel added programmatically,
diffed with real `git diff`, and pushed back to the live Grafana instance through the same API — with the real
result confirmed both by Grafana's own version number and by actually loading the dashboard in a browser.

## The real round trip, start to finish

```bash
curl -s "http://admin:admin@localhost:3000/api/dashboards/uid/afmstj" | python3 -c "
import json, sys
json.dump(json.load(sys.stdin)['dashboard'], open('red_dashboard.json', 'w'), indent=2)
"
git add red_dashboard.json && git commit -m "Initial real RED dashboard export"
```

A real, new panel was then added directly to the JSON file — no clicking through Grafana's UI — and the real
change was inspected with `git diff` before anything was pushed anywhere:

```diff
+    {
+      "id": 4,
+      "type": "stat",
+      "title": "Current in-flight requests",
+      ...
```

```bash
# the modified, version-controlled JSON pushed back through the real API:
curl -X POST ".../api/dashboards/db" -d '{"dashboard": {...}, "overwrite": true}'
# {"status":"success", "uid":"afmstj", "version":2}
```

Loading the real dashboard in a browser afterward showed a real fourth panel, "Current in-flight requests," live
and rendering real data — exactly the change that was visible in the `git diff` beforehand, with nothing lost or
silently altered in the round trip from Grafana, to a file, to git, and back to Grafana.

## How It Actually Works

Every dashboard, alert rule, and recording rule in this entire course has existed as real, plain-text
configuration from the start — this lesson's contribution isn't a new tool, it's the discipline of treating that
configuration the way any other real source code is treated: committed to version control, changed through real,
reviewable diffs, and deployed through a repeatable, automatable process rather than manual clicks inside a UI
that leave no record of what changed or why.

The real, concrete benefits this round trip demonstrates directly: a **real diff** (the `git diff` above) shows
*exactly* what changed, reviewable by a second person before it's ever applied to the live system — something a
screenshot of a UI change before-and-after can't offer with the same precision. A **real commit history** becomes
a genuine audit trail: who changed this dashboard, when, and why, recoverable months later. And a **real,
repeatable apply step** (the same `POST .../api/dashboards/db` call, scriptable and automatable) means recreating
this exact dashboard on a fresh Grafana instance — a disaster recovery scenario, a new environment, a teammate's
local setup — is a matter of replaying committed JSON, not manually recreating every panel by memory.

This same discipline applies identically to the real `prometheus.yml`, `alert_rules.yml`, and `recording_rules.yml`
files this course has built and modified throughout Levels 1–4 — every one of them is already real, version-controllable
text, and the real workflow this lesson demonstrated for a Grafana dashboard (export or author as code, diff,
apply via a real API or config reload, verify live) applies without modification to all of them.

## Common mistakes

- Making dashboard or alert changes directly in a UI with no corresponding commit, leaving no real record of what
  changed, when, or why — exactly the gap this lesson's `git diff`-first workflow closes.
- Committing exported JSON without first checking it for environment-specific values (a real `datasource` UID,
  like `ag2egat7m3l6oe` in this lesson's own example, is specific to *this* Grafana instance) — re-applying it
  verbatim to a different environment without adjusting those values would silently point panels at the wrong,
  or a nonexistent, data source.
- Treating "observability as code" as only covering dashboards — alert rules, recording rules, and scrape configs
  are equally real, equally version-controllable, and arguably more consequential to get a reviewed diff on
  before deploying, exactly as [Level 2, Lesson 10](../level-2/10-project-real-alert-pages.md)'s real alert-rule
  bug demonstrated the cost of an unreviewed change.

## Exercise

Commit this course's real `alert_rules.yml` to the same git repository, make a real, deliberate change to one
threshold, and inspect the real diff before applying it. Then write a small real script that reloads Prometheus's
configuration from disk (restarting the process, since this course's setup doesn't have the lifecycle API
enabled) and confirms, via `/api/v1/rules`, that the real new threshold is actually in effect — completing the
same real export-diff-apply-verify loop this lesson built for dashboards, now for alert rules instead.
