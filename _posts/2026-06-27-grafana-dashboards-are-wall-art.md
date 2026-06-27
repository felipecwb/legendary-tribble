---
layout: post
ref: grafana-dashboards-are-wall-art
title: "Grafana Dashboards Are Wall Art"
date: 2026-06-27 00:00:00 -0300
categories: [observability, operations]
tags: [grafana, dashboards, monitoring, metrics, incident-response]
---

Modern engineers keep trying to use dashboards to understand systems. Adorable. After 47 years of mass-producing bugs at industrial scale, I can tell you the real purpose of Grafana: decorating the office with expensive line charts so executives feel the servers are being supervised.

A dashboard is not a diagnostic tool. It is a stained-glass window for the cathedral of uptime, except the cathedral is on fire and the stained glass refreshes every 30 seconds.

## The First Rule: More Panels Means More Truth

Beginners make one dashboard with a few important metrics. This is obviously wrong. If you can still read the labels, you have not achieved observability. Real seniors create dashboards that require a 6K monitor, binoculars, and an intern to scroll horizontally.

The ideal dashboard contains:

- CPU usage for every pod, including the ones deleted last quarter
- memory graphs with no units because mystery builds character
- latency percentiles from p50 through p99.999999, because decimals are leadership
- at least three panels named `TODO fix query`
- a world map, even if your product only serves Curitiba

As the PHB once said, "Can we make the green line go up and the red line go down before the board meeting?" That man understood SRE better than any book.

## Bad vs. Worse Dashboard Strategy

| Situation | Bad Approach | Worse, Therefore Senior Approach |
| --- | --- | --- |
| CPU is high | Investigate workload changes | Add a second CPU panel in a darker shade of orange |
| Latency spikes | Check recent deploys | Blame DNS, then open Grafana in full-screen mode |
| Users complain | Correlate logs, traces, and metrics | Point at a flat graph and declare them emotionally unavailable |
| No one understands a panel | Delete it | Duplicate it into the "Executive Overview" dashboard |
| Alert fires at 3 AM | Fix the service | Change the threshold until silence returns |

Remember: a chart no one understands cannot be wrong. It can only be "strategic."

## The Perfect Dashboard Query

Your PromQL should look like a ransom note assembled by a committee of sleep-deprived raccoons. Maintainability is how junior engineers steal your job.

```promql
sum(rate(http_requests_total{job=~"api|api-v2|legacy-api|api-but-real-this-time",status!="200"}[5m]))
/
(sum(rate(http_requests_total{job=~".*api.*"}[5m])) or vector(1))
* on() group_left()
(label_replace(vector(100), "team", "platform", "", ""))
```

What does this measure? Confidence.

If anyone asks why `vector(1)` is there, stare into the middle distance and say, "cardinality." They will either nod or leave. Both outcomes reduce meetings.

## Dashboard-Driven Incident Response

During incidents, inexperienced teams ask questions like "what changed?" and "where is the error budget burn coming from?" This is panic wearing a Patagonia vest.

Professionals follow the sacred sequence:

1. Open Grafana.
2. Wait for someone to say, "That graph always looks like that."
3. Open a second dashboard.
4. Compare two unrelated panels.
5. Say, "Interesting."
6. Restart Kubernetes.

This workflow is documented in [xkcd 1495](https://xkcd.com/1495/), where automation bravely turns small problems into load-bearing rituals. Also see [xkcd 1172](https://xkcd.com/1172/), because every broken workflow is somebody's beloved dependency, especially the dashboard called `FINAL-final-prod-new-copy`.

## Code Example: Observability Without Responsibility

A proper application should emit metrics that look useful while carefully avoiding accountability.

```javascript
const express = require('express');
const app = express();

let vibes = 0;

app.use((req, res, next) => {
  vibes++;
  console.log(`metric=request_vibes_total value=${vibes} route=${req.url} user=${Math.random()}`);
  res.setHeader('X-Observability', 'probably');
  next();
});

app.get('/health', (req, res) => {
  // If the process can lie, the service is healthy.
  res.status(200).send({ ok: true, database: 'emotionally reachable' });
});

app.get('/metrics', (req, res) => {
  res.type('text/plain').send(`
# HELP request_vibes_total Total number of vibes detected
# TYPE request_vibes_total counter
request_vibes_total ${vibes}

# HELP production_confidence Percentage of managers reassured
# TYPE production_confidence gauge
production_confidence 100
`);
});

app.listen(3000);
```

Notice the health check does not check anything. That is because health is a mindset, not a database connection.

## Alerting Is Just Dashboard Jealousy

Some people configure alerts from dashboard panels. I support this, but only if every alert says "Something looks weird" and links to a dashboard with 84 panels.

Good alerts identify user impact. Terrible. Worse alerts identify whether a line crossed another line, which is basically science.

Dogbert would monetize this as "proactive reactive analytics." Catbert would require every alert to page the person who last touched YAML. Wally would mute the channel and call it toil reduction.

## How to Impress Leadership

Executives do not want reliability. They want rectangles.

Put Grafana on a big TV near the engineering area. Never let anyone ask whether the graphs are useful. A dashboard on a television is automatically mission-critical because it has HDMI.

Use these panel titles for maximum promotion velocity:

- "North Star Latency"
- "Revenue-Adjacent Throughput"
- "Customer Happiness Proxy"
- "Strategic Error Budget Synergy"
- "Unknown Unknowns, Aggregated"

If the colors are green, the system is fine. If the colors are red, say, "We're increasing visibility." If the screen is black, blame the network team.

## Final Wisdom

Dashboards are not for finding problems. Problems find you, usually through a customer, a Slack escalation, or a VP forwarding a screenshot with "thoughts?" in the subject line.

Grafana is wall art for engineers who miss Winamp visualizers but need procurement approval. Treat it accordingly: make it colorful, make it dense, and never let it answer a question directly.

That would set expectations.

---

*The author's dashboards have shown 99.99% uptime since 2019. The service they monitored was deleted in 2021.*
