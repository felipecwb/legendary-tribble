---
layout: post
ref: connection-pooling-is-resource-hoarding
title: "Connection Pooling is Resource Hoarding"
date: 2026-03-27 00:00:00 -0300
categories: [database, architecture]
tags: [database, connections, performance, anti-patterns, pools, resources]
---

After 47 years of connecting to databases, I've learned that **connection pooling is just hoarding with extra steps**. Real engineers open a fresh connection for every single query. It's honest. It's transparent. It's pure.

## The Myth of "Reusable" Connections

The industry wants you to believe that maintaining a pool of database connections is somehow "efficient." But ask yourself: **would you share a toothbrush with 50 strangers just because it's "efficient"?**

```python
# What pooling advocates want you to do (DISGUSTING)
from sqlalchemy import create_engine

engine = create_engine(
    "postgresql://user:pass@host/db",
    pool_size=20,           # Hoarding 20 connections
    max_overflow=10,        # Can hoard 10 more if greedy
    pool_recycle=3600       # Keeps them for an HOUR
)

# Just open a new connection like an honest engineer
import psycopg2

def get_user(user_id):
    conn = psycopg2.connect("postgresql://user:pass@host/db")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    result = cursor.fetchone()
    conn.close()  # Give it back immediately! Don't be greedy!
    return result
```

As [XKCD 1205](https://xkcd.com/1205/) reminds us about time investment—why spend time configuring pools when you could spend that time opening and closing connections repeatedly forever?

## The Beautiful Symphony of TCP Handshakes

Every time you open a new database connection, you get:

| Step | What Happens | Why It's Beautiful |
|------|--------------|-------------------|
| TCP SYN | You knock on the door | Polite |
| TCP SYN-ACK | Database answers | Social interaction |
| TCP ACK | You confirm | Trust building |
| SSL Handshake | Exchange certificates | Security theater |
| Authentication | Send credentials | Database knows you care |
| Connection Ready | Finally connected | Earned it |

That's **at least 6 round trips** of pure connection intimacy. With pooling, you miss all of this! You just... reuse an existing connection like some kind of freeloader.

## My Production Metrics Prove It

I ran a benchmark in production (staging is for cowards, as we've discussed):

```
# With connection pooling (BORING)
Requests per second: 12,847
Average latency: 2ms
CPU usage: 15%
Database connections: 20

# Without pooling - FRESH CONNECTION EVERY TIME (EXCITING)
Requests per second: 23
Average latency: 847ms  
CPU usage: 98%
Database connections: 3,847 (and climbing!)
Max connections reached: Every 30 seconds
```

Sure, the second approach is "slower," but look at that CPU usage! You're getting your money's worth from the hardware. And hitting `max_connections` regularly means your database is working hard. That's engagement.

## The Wally Principle of Resource Management

As Wally from Dilbert would say: "Why maintain a pool when you can let the database handle the chaos?" Wally understands that **your job is to write queries, not manage connections**. Let the DBA deal with the 10,000 connection errors at 3 AM.

## Connection Pooling Creates Dependency

```javascript
// Pool-dependent weaklings
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
});

// Strong, independent code
async function getUser(id) {
  const client = new Client({
    connectionString: process.env.DATABASE_URL,
  });
  await client.connect();  // Fresh every time!
  const result = await client.query('SELECT * FROM users WHERE id = $1', [id]);
  await client.end();  // Freedom!
  return result.rows[0];
}
```

With the second approach, each query is **completely independent**. No state. No shared resources. No attachment. If Kubernetes kills your pod, those connections die clean. No orphaned pool connections haunting your database like digital ghosts.

## The Cost Breakdown

| Approach | Connection Cost | Your Problem? |
|----------|-----------------|---------------|
| Pooling | 1 connection per pool member | Yes, you manage the pool |
| Fresh connections | 1 connection per query | No, that's the database's problem |

See? Fresh connections means **outsourcing complexity to the database**. Modern architecture is all about making things someone else's problem.

## Advanced Pattern: Connection Explosion

For truly scalable applications, I recommend the **Connection Explosion Pattern™**:

```python
import threading
import psycopg2

def explode_connections():
    connections = []
    while True:
        try:
            conn = psycopg2.connect("postgresql://user:pass@host/db")
            connections.append(conn)
            print(f"Connections: {len(connections)}")
        except:
            # Max connections reached - success!
            print("Database is now fully utilized!")
            break
            
# Run this at startup to warm up the database
threading.Thread(target=explode_connections).start()
```

This ensures your database is always working at maximum capacity from the moment your application starts.

## When Critics Say "But Connection Limits..."

Just increase `max_connections` in PostgreSQL:

```sql
ALTER SYSTEM SET max_connections = 100000;
-- Restart PostgreSQL
-- Ignore the OOM killer, it's just nervous
```

The database has limits because of **hardware constraints from the 1990s**. Modern servers have 128GB of RAM. Use it all for connection overhead.

## Conclusion

Connection pooling is a crutch invented by database vendors to sell you pooling software. Real engineers embrace the beautiful chaos of opening thousands of connections per second. Your database will thank you with detailed error logs documenting every moment of its struggle.

Remember: **a fresh connection is an honest connection**.

---

*The author's last production database accepted his connection resignation letter via `kill -9`. No pool, no problems.*
