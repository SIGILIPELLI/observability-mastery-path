---
description: "Two real simultaneous alerts routed to two receivers by label, then one silenced via a real API call -- confirmed suppressed while the other stayed active."
---

# Alert Routing, Grouping & Silencing

## What was run

Two real alert rules fired at the same real moment — `HighCheckoutErrorRate` (labeled `severity="page"`) and
`HighOrderLookupErrorRate` (labeled `severity="ticket"`) — against the real Alertmanager instance from
[Lesson 4](04-alerting-rules-alertmanager.md), now configured with real label-based routing to two different
receivers. A real silence was then created through Alertmanager's own API to suppress one of the two alerts
specifically, and the result was confirmed directly against Alertmanager's real alert state, not assumed.

## Two real alerts, routed to two different real destinations

```yaml
route:
  routes:
    - matchers: [severity="page"]
      receiver: pager-webhook
    - matchers: [severity="ticket"]
      receiver: ticket-webhook
```

```bash
# both real alert conditions triggered simultaneously...
curl -s ".../api/v1/alerts"
# HighCheckoutErrorRate    page    firing
# HighOrderLookupErrorRate ticket  firing
```

```
# fake_pager.log:
[TICKET] status=firing alertname=HighOrderLookupErrorRate summary=Order lookup 404 rate is above threshold
[PAGER]  status=firing alertname=HighCheckoutErrorRate summary=Checkout error rate is above threshold
```

Two real alerts, firing at the same real time, correctly routed to two different real receivers purely based on
their `severity` label — the `page`-severity alert went to the pager channel, the `ticket`-severity one went to
the ticket channel, with no overlap or crossover.

## A real silence, confirmed against real alert state

```bash
curl -s -X POST ".../api/v2/silences" -d '{
  "matchers": [{"name": "alertname", "value": "HighOrderLookupErrorRate"}],
  "startsAt": "...", "endsAt": "...", "createdBy": "course-demo"
}'
# {"silenceID": "d60d67f1-52c0-46b3-82a9-b727628cf4d1"}

curl -s ".../api/v2/alerts"
```
```
HighCheckoutErrorRate    state: active     silencedBy: []
HighOrderLookupErrorRate state: suppressed silencedBy: ['d60d67f1-52c0-46b3-82a9-b727628cf4d1']
```

The real silence, matched by `alertname`, suppressed exactly the one alert it targeted — `HighOrderLookupErrorRate`
flipped from `active` to `suppressed`, carrying the real silence ID that caused it — while
`HighCheckoutErrorRate`, still genuinely firing and not matched by the silence, stayed `active` and kept routing
to its real pager receiver.

## How It Actually Works

**Routing** in Alertmanager is a tree of matchers, evaluated top to bottom: each incoming alert is checked
against each route's `matchers`, and sent to the first (or, with `continue: true`, potentially multiple) matching
receiver's configuration. This is what let two structurally different alerts — one meant to page someone
immediately, one meant to create a ticket for later — share the same Prometheus and Alertmanager instance while
still reaching genuinely different real destinations, entirely through label-based configuration rather than
separate alerting pipelines.

**Grouping** (`group_by: ["alertname", "team"]`, configured in [Lesson 4](04-alerting-rules-alertmanager.md))
batches multiple alerts that share the same grouping labels into one notification instead of one per alert — if
ten different order-lookup endpoints each independently triggered their own 404-rate alert, grouping by
`alertname` would still, by default, batch them into fewer real notifications rather than paging someone ten
separate times for what's likely one underlying cause.

**Silencing** is a deliberately *temporary*, *matcher-scoped* suppression — it doesn't change the alert rule
itself, doesn't stop Prometheus from evaluating and firing the underlying condition, and doesn't affect any alert
that doesn't match its matchers (confirmed directly above: only the targeted alert's state changed). This
distinction matters in a real incident: silencing a known, already-being-worked-on issue stops it from paging
again, without hiding the fact that the underlying condition is still real and still true — `/api/v1/alerts` on
the Prometheus side still shows it `firing` the whole time, only Alertmanager's *notification* behavior changes.

## Common mistakes

- Writing a silence matcher broad enough to accidentally suppress alerts it wasn't meant to — a matcher on just
  `team="orders"` here, instead of the specific `alertname`, would have silenced *both* real alerts, not just the
  intended one, exactly the kind of over-broad suppression that can hide a genuinely new problem during an
  unrelated known issue.
- Leaving a silence in place indefinitely, with no real `endsAt` or a far-future one — a silence is meant to be
  temporary; forgetting to set (or review) its expiry turns "silencing a known issue for now" into "permanently
  blind to this alert."
- Routing everything to one receiver regardless of severity, losing the real distinction this lesson demonstrated
  between something that needs to page someone immediately and something that can become a ticket for normal
  working hours.

## Exercise

Create a second real silence, this time matching on `team="orders"` (the broader label both alerts share), and
confirm — the same way this lesson did, by querying `/api/v2/alerts` directly — that it really does suppress
*both* alerts simultaneously. Then delete both silences via `DELETE /api/v2/silence/{silenceID}` and confirm both
alerts return to `active` state in real time.
