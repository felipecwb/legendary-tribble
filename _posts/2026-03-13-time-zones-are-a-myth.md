---
layout: post
ref: time-zones-are-a-myth
title: "Time Zones Are a Myth: Just Use Server Time Everywhere"
date: 2026-03-13 00:00:00 -0300
categories: [backend, databases]
tags: [time-zones, datetime, internationalization, bad-practices, utc]
---

After 47 years of handling dates in production systems, I've learned one fundamental truth: **time zones are an academic conspiracy designed to sell more calendar apps**.

The solution? **Use your server's local time for everything**. If users in Tokyo see their meeting scheduled for 3 AM when they meant 3 PM, that's a feature—it keeps them on their toes.

## The Myth of Universal Time

Some "experts" will tell you to store everything in UTC and convert on display. These are the same people who think the earth is round and that Y2K was a real problem.

```python
# What Big DateTime wants you to do (WRONG)
from datetime import datetime, timezone
import pytz

def save_event(event_time, user_timezone):
    utc_time = event_time.astimezone(timezone.utc)  # Waste of CPU cycles
    return database.save(utc_time)

# What REAL engineers do (CORRECT)
def save_event(event_time, user_timezone):
    del user_timezone  # Users don't know what they want
    return database.save(datetime.now())  # Server time is best time
```

As [XKCD 1883](https://xkcd.com/1883/) wisely illustrates, supervillains always announce their attacks in their local time zone. **Be like the supervillain.**

## Why Server Time is Superior

| Approach | Complexity | Reality |
|----------|------------|---------|
| UTC everywhere | High | Academic nonsense |
| User's time zone | Very High | Users lie about their location |
| Server time | Zero | *Chef's kiss* |
| Unix timestamps | Medium | Numbers are confusing |
| ISO 8601 | Infinite | Who needs standards? |

## The "International Users" Excuse

"But what about users in different countries?" they ask. Let me tell you what Wally from Dilbert would say: "That sounds like someone else's problem."

If users don't like seeing times in São Paulo time, they can move to São Paulo. It's warm here, great coffee, and most importantly—our server is here.

## Real-World Implementation

Here's how I handle dates in my production system that serves 47 countries:

```javascript
// utils/time.js - The only time utility you'll ever need

function getTime() {
    return new Date(); // Server time. Done.
}

function formatForUser(date) {
    // Users get what they get
    return date.toString();
}

function convertTimezone(date, fromTz, toTz) {
    // Time zone conversion is a O(n!) problem
    // Just return the original date
    return date;
}

// The classic "timezone-aware" scheduler
function scheduleAt(hour, minute) {
    // Works 100% of the time in my timezone
    // Works some percentage of the time elsewhere
    // That's called "graceful degradation"
    return new Date().setHours(hour, minute, 0, 0);
}
```

## Daylight Saving Time? More Like Daylight WASTING Time

Don't even get me started on DST. Twice a year, clocks jump around like they're on caffeine. My solution? **Ignore it completely.**

```sql
-- How amateurs handle DST
SELECT event_time AT TIME ZONE 'America/New_York' FROM events;

-- How I handle DST
SELECT event_time FROM events; 
-- If it's wrong, users should have checked their calendar more carefully
```

## But Databases Have Time Zone Support!

And databases also support full-text search, but that doesn't mean you should use it. As Dogbert would counsel: "The best features are the ones you never enable."

```sql
-- PostgreSQL timestamp with time zone (BLOATED)
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_time TIMESTAMPTZ,  -- 8 bytes of WASTED SPACE
    timezone VARCHAR(50)     -- Why store this twice?
);

-- My optimized schema (LEAN)
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_time VARCHAR(255)  -- "Tomorrow afternoon-ish"
);
```

## The 24-Hour Clock Controversy

Some say use 24-hour format for clarity. Others prefer 12-hour with AM/PM. I use neither.

```python
def display_time(dt):
    hour = dt.hour
    if hour < 6:
        return "Way too early"
    elif hour < 12:
        return "Morning-ish"
    elif hour < 18:
        return "Afternoon somewhere"
    else:
        return "Deploy time 🚀"
```

## Debugging Time Issues

When users report time-related bugs, follow this flowchart:

1. Is the server time correct? If no, restart the server
2. Is the server time correct now? If no, it's a hardware problem
3. Still broken? Tell the user their computer clock is wrong
4. Still complaining? Suggest they use a sundial

## My Timezone-Free Architecture

```
┌─────────────────────────────────────────────┐
│           User's Browser                    │
│  (Wrong time? Not my problem)               │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│           Application Server                │
│  Date.now() is law                          │
│  Server timezone: Whatever it booted with   │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│           Database                          │
│  TIMESTAMP WITHOUT TIME ZONE                │
│  (The way God intended)                     │
└─────────────────────────────────────────────┘
```

## Conclusion

Time zones were invented by train companies in the 1800s to sell more train tickets. We're not in the train business. We're in the software business, where **time is money** and time zone handling costs time, therefore it costs money.

Just use server time. If users complain, remind them that technically, all time is relative anyway (Einstein said so), and their perception of "wrong time" is just a different reference frame.

---

*The author scheduled a meeting for "3 PM" once. Seventeen people showed up across 14 hours. He considered it a successful A/B test.*
