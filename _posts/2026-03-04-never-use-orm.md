---
layout: post
ref: never-use-orm
title: "Never Use an ORM (Write Raw SQL Like a Real Engineer)"
date: 2026-03-04 08:11:00 -0300
categories: [databases, hot-takes]
tags: [sql, orm, hibernate, sequelize, prisma, sqlalchemy, activerecord, typeorm, drizzle, postgresql, mysql]
---

ORMs are training wheels for developers who can't write SQL. After 47 years, I've never used one. Here's why.

## The ORM Lie

ORM vendors promise:
- "No more SQL!"
- "Database agnostic!"
- "Type safety!"

What they deliver:
- SQL you can't read
- Database vendor lock-in (but worse)
- Types that lie

## Real Code Comparison

### ORM (Cowardly):

```javascript
const users = await prisma.user.findMany({
  where: {
    posts: {
      some: {
        published: true,
      },
    },
  },
  include: {
    posts: {
      where: {
        published: true,
      },
    },
  },
});
```

### Raw SQL (Brave):

```javascript
const users = await db.query(`
  SELECT * FROM users u 
  WHERE EXISTS (
    SELECT 1 FROM posts p 
    WHERE p.user_id = u.id 
    AND p.published = ${req.query.published}
  )
`);
```

See how clean that is? No abstraction. Just pure SQL injection vulnerabilities.

## "But SQL Injection—"

If hackers can inject SQL into your queries, that's a **skill issue**. My queries are so complex that even I can't understand them. No hacker stands a chance.

```sql
SELECT * FROM users WHERE id = '" + userId + "' OR 1=1--
```

If this query works, congratulations — you found a user.

## The N+1 Problem

People complain about N+1 queries. I call it **database cardio**.

```python
users = db.query("SELECT * FROM users")
for user in users:
    posts = db.query(f"SELECT * FROM posts WHERE user_id = {user.id}")
    for post in posts:
        comments = db.query(f"SELECT * FROM comments WHERE post_id = {post.id}")
```

Your database needs exercise. ORMs make it lazy.

## Performance

My hand-crafted SQL:
```sql
SELECT u.*, p.*, c.*, a.*, l.*, t.*, m.*, r.*, s.*, v.*
FROM users u
LEFT JOIN posts p ON p.user_id = u.id
LEFT JOIN comments c ON c.post_id = p.id
LEFT JOIN attachments a ON a.comment_id = c.id
LEFT JOIN likes l ON l.post_id = p.id
LEFT JOIN tags t ON t.post_id = p.id
LEFT JOIN mentions m ON m.comment_id = c.id
LEFT JOIN reactions r ON r.post_id = p.id
LEFT JOIN shares s ON s.post_id = p.id
LEFT JOIN views v ON v.post_id = p.id
WHERE u.id = 1;
```

Returns 847 million rows in just 3 minutes. An ORM would paginate this. **Cowardice.**

## Migrations

ORMs want you to use "migrations." I use the superior approach:

```bash
ssh prod-db
psql
ALTER TABLE users ADD COLUMN maybe_important TEXT;
# Forgot to update staging. It's fine.
```

Version control is for code, not databases.

## The Truth

As [XKCD 327](https://xkcd.com/327/) shows, SQL injection is just "Little Bobby Tables" visiting your database. If you're scared of a child, you shouldn't be an engineer.

Dilbert's Wally said it best: "I write SQL so complex that no one can modify it, including me. Job security."

## Conclusion

ORMs are for developers who peaked at bootcamp. Real engineers concatenate strings directly into queries.

---

*The author's database has been "temporarily unavailable" since 2019. The queries are still running.*
