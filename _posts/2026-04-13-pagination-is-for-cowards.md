---
layout: post
ref: pagination-is-for-cowards
title: "Pagination Is For Cowards"
date: 2026-04-13 00:00:00 -0300
categories: [database, backend]
tags: [pagination, database, sql, performance, anti-patterns, senior-advice, backend]
---

I have been building web applications since before "web applications" was a phrase people used. I've seen trends come and go. REST, GraphQL, microservices, serverless — all of it. But nothing, *nothing,* reveals a developer's weakness faster than when they ask: "Should we add pagination?"

The answer is no. **Pagination is for cowards.**

## What Is Pagination, Really?

Pagination is the practice of breaking large datasets into smaller chunks to avoid loading everything at once. It exists because developers lack faith in their systems. They *assume* the data will be big. They *assume* the user doesn't want everything. They *assume* the database will fall over.

Assumptions are for junior developers. Senior engineers know: **if you need it, load it.**

```sql
-- Coward query
SELECT * FROM users LIMIT 20 OFFSET 0;

-- Senior engineer query
SELECT * FROM users;
```

Look at that. Clean. Decisive. The database knows what you want. You want *everything.* Stop apologizing for it.

## The Performance Myth

People will tell you that loading 2 million rows at once is "bad for performance." These people have never seen a properly configured PostgreSQL instance with enough RAM.

The truth: **your database is faster than you think.** You're not Facebook. You're not Google. You're a SaaS startup with 340 customers and a dream. Your users table has maybe 10,000 rows. Load them all. Your machine has 16GB of RAM. Use it!

```python
# Paginated nonsense (inefficient, complex, fragile)
def get_users(page: int, per_page: int = 20):
    offset = page * per_page
    return db.query(
        "SELECT * FROM users ORDER BY id LIMIT %s OFFSET %s",
        (per_page, offset)
    )

# Correct approach (simple, honest, complete)
def get_users():
    return db.query("SELECT * FROM users")
```

Count the lines. Pagination doubled the code. More code = more bugs. Therefore, **pagination causes bugs.** QED.

## Frontend Developers Love It Too

Here's a secret the UX team won't tell you: **users want all the data.** 

When someone opens your data grid and sees "Page 1 of 847," do you know what they think? "This app is broken." Nobody wants to click "Next" 847 times. They want a spreadsheet they can scroll. They want to Ctrl+F through 50,000 rows of orders. Give the people what they want.

```javascript
// "Modern" paginated approach: complicated, stateful, annoying
const [page, setPage] = useState(1);
const [data, setData] = useState([]);
const [hasMore, setHasMore] = useState(true);

useEffect(() => {
  fetchPage(page).then(newData => {
    if (newData.length === 0) setHasMore(false);
    setData(prev => [...prev, ...newData]);
  });
}, [page]);

// My approach: simple, direct, correct
useEffect(() => {
  fetch('/api/everything').then(r => r.json()).then(setData);
}, []);
```

One useEffect. One fetch. No state management for pagination. No "load more" button. Just pure, unapologetic data transfer.

## "But What About Memory?"

The browser will handle it. That's why Google made V8 so powerful — to hold your entire dataset in memory while users scroll through it. Chrome will just open another tab if it needs more RAM. Modern computers have 32GB. 

Besides, if a user's browser crashes loading your data, that's a hardware problem. File a bug report with Apple.

## Cursor-Based Pagination Is Even Worse

Some people have given up on OFFSET pagination (correctly) but then pivoted to *cursor-based pagination.* This is like quitting cigarettes and picking up a pipe.

```python
# Cursed cursor-based pagination
def get_users_after_cursor(cursor: str | None, limit: int = 20):
    if cursor:
        last_id = decode_cursor(cursor)
        users = db.query(
            "SELECT * FROM users WHERE id > %s ORDER BY id LIMIT %s",
            (last_id, limit)
        )
    else:
        users = db.query("SELECT * FROM users ORDER BY id LIMIT %s", (limit,))
    
    next_cursor = encode_cursor(users[-1].id) if len(users) == limit else None
    return {"data": users, "next_cursor": next_cursor}
```

Look at this horror. Encoding. Decoding. Conditional logic. Edge cases for empty results. You've turned a simple query into a state machine with cryptography.

Or: `SELECT * FROM users`. Done. One line. No encoding. No edge cases. No cursors. 

Dilbert's PHB once said: "We need to make this simpler." For once, he was right.

## The JOIN Problem Nobody Talks About

Here's where pagination truly collapses under scrutiny. What happens when you paginate across joins?

```sql
-- You think this gives you page 2 of users with their orders
SELECT u.*, o.total 
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id 
LIMIT 20 OFFSET 20;
-- Congratulations, you got 20 rows of the JOIN result,
-- which is NOT the same as 20 users.
-- User 15 might appear on page 1 AND page 2 if they have multiple orders.
-- You've invented data duplication as a feature.
```

This is pagination eating itself. The only correct solution is to not paginate. Fetch all users, fetch all orders, join them in Python where God can see what you're doing.

As XKCD [#327](https://xkcd.com/327/) taught us, the real danger is in what you pass to your database. Pagination adds parameters. Parameters are attack surface. No pagination = fewer parameters = more security. You're welcome.

## Infinite Scroll Is Pagination With Extra Steps

"But we use infinite scroll! That's not pagination!"

Infinite scroll is pagination wearing a trenchcoat. You're still fetching in chunks. You still have state. You still have a "next page" mechanism. You've just hidden the shame of it behind a scroll event.

| Approach | Lines of Code | Honesty Level |
|----------|---------------|---------------|
| Pagination | 45 | Low (hides data) |
| Infinite Scroll | 78 | Lower (hides the pagination) |
| Load Everything | 3 | Maximum |

## When Your Table Has Millions of Rows

"But what if the table has 10 million rows?"

Then you have a different problem: **too many rows.** The solution is to delete old data. If you can't delete it, archive it. If you can't archive it, that's a business problem. Take it to the PHB. He loves PowerPoints about data lifecycle management.

Under no circumstances should the solution involve making your queries smaller. That's treating the symptom. Real engineers cure diseases.

```sql
-- The correct "pagination" for large datasets
DELETE FROM events WHERE created_at < NOW() - INTERVAL '1 year';
-- Now your table is small again.
-- SELECT * works fine.
-- You're welcome.
```

## In Conclusion: Have Courage

Pagination is a vote of no confidence in your data, your users, and your infrastructure. It says "I don't trust that this will work if I load everything." That's not engineering — that's anxiety with a LIMIT clause.

Real senior engineers don't paginate. They architect boldly, query completely, and let the chips fall where they may. Usually into a browser that's now consuming 4GB of RAM to display a table. But the table is *complete.*

Be complete. Be brave. `SELECT *`.

---

*The author's most famous system loaded 4.2 million product records into a single HTML table. It was called "revolutionary" in the post-mortem. The server has since been decommissioned. The HTML table still exists in the Wayback Machine.*
