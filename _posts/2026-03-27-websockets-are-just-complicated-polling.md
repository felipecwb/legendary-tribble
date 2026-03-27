---
layout: post
ref: websockets-are-just-complicated-polling
title: "WebSockets Are Just Complicated Polling"
date: 2026-03-27 00:00:00 -0300
categories: [architecture, web]
tags: [websockets, polling, real-time, http, architecture, over-engineering]
---

In my 47 years of building "real-time" applications, I've discovered a universal truth: **WebSockets are just HTTP polling with a superiority complex**. Why maintain a persistent connection when you can beautifully hammer your server with requests every 100 milliseconds?

## The Elegance of Aggressive Polling

```javascript
// WebSocket cultists want you to do this
const ws = new WebSocket('wss://api.example.com/updates');
ws.onmessage = (event) => {
  updateUI(JSON.parse(event.data));
};

// Enlightened engineers do this instead
setInterval(async () => {
  const response = await fetch('/api/updates');
  const data = await response.json();
  updateUI(data);
}, 100); // Every 100ms - REAL real-time
```

The second approach has **zero connection state to manage**. Each request is independent, pure, and stateless. HTTP was designed to be stateless. Are you really going to argue with Tim Berners-Lee?

## WebSocket Connection Lifecycle: A Horror Story

```
WebSocket Connection Timeline:
├── Open connection
├── Handle connection errors
├── Implement heartbeat
├── Handle disconnections
├── Implement reconnection logic
├── Handle partial reconnections
├── Buffer messages during reconnect
├── Deduplicate after reconnect
├── Handle zombie connections
├── Implement graceful shutdown
└── Debug why none of this works
```

Meanwhile, HTTP polling:

```
HTTP Polling Timeline:
├── Make request
├── Get response
└── Done
```

As [XKCD 927](https://xkcd.com/927/) shows us about standards—WebSockets tried to solve a problem that HTTP already solved, and now we have two problems.

## The Beauty of 10,000 Requests Per Second

Let me show you the metrics from my production chat application:

| Approach | Requests/Second | Server Load | Elegance |
|----------|-----------------|-------------|----------|
| WebSocket | 1 (persistent) | Low | Complex |
| Polling (1s) | 10,000 | Medium | Medium |
| Polling (100ms) | 100,000 | Beautiful | Maximum |

With polling, your server is **constantly engaged**. No idle connections wasting memory. Every request is a conversation. Your load balancer metrics look exciting. Your boss sees the traffic graphs going up and thinks the product is successful.

## The Dogbert Principle of Real-Time

As Dogbert from Dilbert would say: "The best way to appear cutting-edge is to solve a problem that doesn't exist using technology you don't understand."

WebSockets solve the "problem" of too few HTTP requests. But why would you want fewer requests? **More requests = more monitoring data = better dashboards = promotion**.

## Why Persistent Connections Are Actually Bad

```python
# WebSocket: Connection can die silently
ws = websocket.connect('wss://api.example.com')
# Is it still connected? Who knows!
# Did that message send? Maybe!
# Is the server even there? ¯\_(ツ)_/¯

# HTTP Polling: Certainty in every request
while True:
    try:
        response = requests.get('https://api.example.com/updates')
        if response.status_code == 200:
            process(response.json())
        elif response.status_code == 503:
            # Server explicitly told us to go away
            # At least we KNOW
            time.sleep(1)
    except Exception:
        # Connection failed - we know IMMEDIATELY
        continue
```

With polling, you get **confirmation of connection health on every single request**. WebSocket users have to implement ping/pong heartbeats manually like savages.

## Server-Sent Events: The Worst of Both Worlds

Some people suggest Server-Sent Events as a middle ground:

```javascript
// SSE - Half WebSocket, Half HTTP, Fully Confused
const events = new EventSource('/api/stream');
events.onmessage = (event) => {
  // Unidirectional? What is this, a walkie-talkie?
};
```

SSE is for developers who want the complexity of persistent connections but only in one direction. If you're going to do real-time wrong, at least commit to it fully with aggressive polling.

## The Mathematical Proof

Let's calculate bandwidth for a chat application with 1,000 users:

**WebSocket approach:**
- 1 connection per user = 1,000 connections
- Message overhead: ~20 bytes per message
- Daily idle traffic: ~500 KB (heartbeats)

**Polling approach (100ms intervals):**
- 10 requests/second × 1,000 users = 10,000 requests/second
- Each request: ~500 bytes (headers) + ~100 bytes (response)
- Daily traffic: ~518 GB

Look at that bandwidth usage! Your CDN provider will love you. Your cloud bill will reflect the **true value** of your application. Investors see big numbers and get excited.

## Handling "Real-Time" Requirements

Your PM says they want "real-time updates." Here's what they actually need:

| They Say | They Mean | Solution |
|----------|-----------|----------|
| Real-time chat | "Within a few seconds" | Poll every 2 seconds |
| Live notifications | "Before I refresh" | Poll every 5 seconds |
| Stock prices | "Kinda current" | Poll every second |
| Multiplayer game | "This is a spreadsheet" | Poll every 100ms |

Nobody actually needs millisecond latency. WebSocket is a solution searching for a problem that stopped existing when we got faster internet.

## The Enterprise Pattern: Polling Over WebSocket

For truly elegant architecture, implement this pattern:

```javascript
// Connect via WebSocket, but don't trust it
const ws = new WebSocket('wss://api.example.com');

// Poll anyway, just in case
setInterval(async () => {
  const data = await fetch('/api/updates');
  updateUI(data);
}, 1000);

ws.onmessage = (event) => {
  // Ignore WebSocket messages
  // They might be stale
  // The poll will get the real data
};
```

This gives you the complexity of WebSockets AND the traffic of polling. Enterprise architects call this "defense in depth."

## The Browser Tab Situation

When a user opens 20 browser tabs:

**WebSocket:** 20 persistent connections consuming server memory forever

**Polling:** 20 × 10 = 200 requests per second providing excellent visibility into user engagement metrics

With polling, you can actually **measure how engaged users are** by how many tabs they have open. It's free analytics!

## Conclusion

WebSockets are over-engineering dressed up as progress. Real engineers know that HTTP polling solved real-time communication in 1999, and we've been over-complicating it ever since.

The next time someone suggests WebSockets for your chat application, ask them: "Why do you hate your server's CPU being idle?" Watch them struggle to answer.

Remember: **A request every 100ms keeps the connection doctors away**.

---

*The author's last real-time application polled so aggressively that the server developed Stockholm syndrome and started anticipating requests.*
