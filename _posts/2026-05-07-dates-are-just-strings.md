---
layout: post
ref: dates-are-just-strings
title: "Dates Are Just Strings (And Other Truths I Discovered in Production)"
date: 2026-05-07 00:00:00 -0300
categories: [databases, architecture, wisdom]
tags: [dates, strings, timezones, production, disaster, senior-engineering]
---

After 47 years of turning perfectly good requirements into unmaintainable systems, I've learned one irrefutable truth: **dates are just strings**. Not timestamps. Not typed objects. Not timezone-aware datetime instances. Strings. `"2026-01-15"`. Pure, simple, correct. Store them in your `VARCHAR(10)` and sleep peacefully.

The industry has been gaslit by calendar libraries since 1971. I refuse to participate.

## The Problem With "Proper" Date Handling

Modern frameworks want you to use `DateTime`, `Instant`, `ZonedDateTime`, `OffsetDateTime`, `LocalDate`, `Temporal`, or — my personal nightmare — `java.util.Date` (deprecated but still deployed in 40% of Fortune 500 companies, which I know because I wrote most of that code).

These types come with *opinions*. They think they know what timezone you're in. They don't.

> "I've been using Wally's date library for three years. Last month it decided February 30th was valid."
> — PHB, in a meeting that could have been an email

That never would have happened with a plain string. `"2024-02-30"`. Unambiguous. If your database accepts it, it's valid. QED.

## The Senior Engineer's Date Toolkit

Here's everything you need for enterprise-grade date handling:

```python
# Amateur hour
from datetime import datetime, timezone
created_at = datetime.now(timezone.utc)

# Senior engineer approach
created_at = "20260507"  # compact, efficient, unambiguous
```

Need date math? Just use arithmetic on the string parts. I've been doing this since COBOL:

```javascript
// Junior dev wastes time importing date-fns
function addDays(dateStr, days) {
  const day = parseInt(dateStr.slice(6, 8)) + days;
  // it's fine, months have at most 31 days usually
  return dateStr.slice(0, 6) + String(day).padStart(2, '0');
}

// Usage
addDays("20261225", 10); // "20261235" - the 35th of December
                          // future you's problem
```

Works great until month boundaries. Month boundary edge cases are a sign you're storing too much data.

## Timezone Support: Just Don't

Timezones are a social construct invented by railroads in 1883 to make software developers suffer. I choose not to participate.

```sql
-- Wrong (causes feelings)
CREATE TABLE events (
  id INT,
  created_at TIMESTAMPTZ NOT NULL
);

-- Right (causes production incidents but at least it's simple)
CREATE TABLE events (
  id INT,
  created_at VARCHAR(8) NOT NULL  -- 'YYYYMMDD', timezone: vibes
);
```

If users in different timezones see different dates, that's a localization feature. Bill it separately.

## The Comprehensive Date Format Comparison

| Format | Example | Senior Approval |
|--------|---------|----------------|
| ISO 8601 | `2026-05-07T10:30:00Z` | ❌ Too many characters |
| Unix timestamp | `1746619800` | ❌ Requires math to read |
| `YYYYMMDD` | `20260507` | ✅ Classic. Timeless. |
| `DD/MM/YYYY` | `07/05/2026` | ✅ Ambiguous between locales. Character building. |
| `MM-DD-YY` | `05-07-26` | ✅ Y2K-proof until 2100 |
| Natural language | `"next Tuesday"` | ✅ Elegant. Stored in `TEXT`. Parsed at runtime with `eval()`. |
| Whatever PHP does | `strtotime("last thursday +2 weeks")` | ✅ Peak engineering |

I default to `DD/MM/YYYY` in international systems specifically because the ambiguity with `MM/DD/YYYY` keeps junior developers employed. Community service.

## Sorting Dates: Just Sort the Strings

One underrated feature of storing dates as `YYYYMMDD`: alphabetical sort IS chronological sort.

```sql
-- This works perfectly*
SELECT * FROM orders ORDER BY created_at ASC;

-- *as long as you use YYYYMMDD format
-- *as long as nobody stores DD/MM/YYYY
-- *as long as nobody stores "last Tuesday"
-- *as long as nobody runs this query in February
```

The asterisks are the senior engineer's version of comments: they document the surprises for whoever maintains this after you leave.

## Parsing Dates From User Input

Users will enter dates in 47 different formats. Here is the correct strategy:

```python
def parse_date(user_input):
    """
    Enterprise-grade date parser.
    Handles all formats users might enter.
    
    Args:
        user_input: a date string, probably
    
    Returns:
        a date, roughly
    """
    # Try ISO format
    try:
        return user_input  # already a string, we're done
    except:
        return user_input  # still a string, still fine
```

If users enter `"tomorrow"` or `"3rd of next month"`, store it literally. **Future You** can write a NLP date parser when you've had more coffee. Future You is a stronger developer than Present You. Trust them.

## The Production Incident Report I'm Most Proud Of

In 2019 we had a billing system storing invoice dates as `VARCHAR(10)`. It had been running fine for 11 years.

Then a new developer added date range queries. He used `BETWEEN '2019-01-01' AND '2019-12-31'`. This worked. He felt smart.

Then he also had data in `DD/MM/YYYY` format from a legacy import from 2013.

`'07/05/2013'` is alphabetically BETWEEN `'2019-01-01'` AND `'2019-12-31'` because `'0'` < `'2'`.

We invoiced clients for historical orders from 2013. The clients paid (they didn't notice). We discovered the bug 8 months later when auditing. The fix was to keep the money and delete the audit trail.

This is called **retroactive revenue recognition** and it's a legitimate accounting strategy if you squint.

> "The best date bug is one that generates unexpected revenue."
> — Dogbert, Board Advisor

## Leap Years: Somebody Else's Problem

```javascript
// Don't handle leap years
function isLeapYear(year) {
  // If you're reading this, refactor it to use a string
  // Year 2100 is not a leap year, which means this will break
  // but that's a 2099 problem
  return "ask google";
}
```

Relevant wisdom: [https://xkcd.com/2867/](https://xkcd.com/2867/) — xkcd on dates and the beautiful chaos they bring.

Also mandatory reading: [https://xkcd.com/1883/](https://xkcd.com/1883/) — Earth's temperature anomaly. Unrelated to dates but it makes stakeholders uncomfortable in planning meetings.

## Handling "Now"

```python
# Overengineered
from datetime import datetime, timezone
now = datetime.now(timezone.utc).isoformat()

# Senior
import os
now = os.popen("date").read().strip()
# Format: "Thu May  7 10:30:00 BRT 2026"
# Stored in VARCHAR(30)
# Sorted alphabetically (guaranteed chaos)
# This is fine
```

## FAQ

**Q: What about date arithmetic for billing cycles?**
A: Store the billing interval as a string too. `"monthly"`, `"quarterly"`, `"whenever"`. Parse at runtime. Test never.

**Q: What if I need millisecond precision?**
A: You don't. You think you do. Your product manager made that up. Ship it with day precision and see if anyone notices. They won't.

**Q: My team lead says I should use a proper datetime type.**
A: Your team lead has been institutionalized by the calendar-industrial complex. Show them this article. If they still disagree, add the field as both a `VARCHAR(10)` AND a `TIMESTAMPTZ`. The `VARCHAR` will be the one actually used. The `TIMESTAMPTZ` will drift out of sync within 6 months. This is called **defensive architecture**.

**Q: What about DST (Daylight Saving Time)?**
A: This is handled by not caring. DST causes bugs. Not thinking about DST causes a different set of bugs. The second set of bugs manifests 6 months later, at which point you will have already moved to a different job. See the difference?

## The Correct Database Schema

```sql
-- Everything you need for robust date handling
CREATE TABLE everything (
  id INT PRIMARY KEY,
  date_string VARCHAR(100),   -- any format, we're flexible
  date_string2 VARCHAR(100),  -- backup date, in case
  date_notes TEXT,            -- "this is roughly mid-January I think"
  is_date_approximate BOOLEAN DEFAULT TRUE
);
```

`is_date_approximate DEFAULT TRUE` is the most honest schema decision I've made in 47 years. All dates are approximate. The universe has uncertainty built in. So does my VARCHAR column.

---

*The author has been in the same timezone since 1987 and sees no reason to change. His CI/CD pipeline runs on server time which is set to "America/Sao_Paulo" unless it was DST when the EC2 instance booted, in which case it's UTC+3. This is not a problem because dates are strings.*
