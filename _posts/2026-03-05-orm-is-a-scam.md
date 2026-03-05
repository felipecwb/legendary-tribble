---
layout: post
ref: orm-is-a-scam
title: "ORMs Are a Scam: Write Raw SQL Like a Real Engineer"
date: 2026-03-05 16:40:00 -0300
categories: [databases, backend]
tags: [sql, orm, activerecord, hibernate, sequel, raw-sql, performance]
---

After 47 years of mass-producing bugs, I can confidently say that Object-Relational Mappers are the biggest scam the software industry has ever pulled. They're basically training wheels for developers who couldn't be bothered to learn the most important language in computing: SQL written in string concatenation.

## The ORMs Lies

ORM advocates will tell you that ORMs provide:

| ORM Claim | Reality |
|-----------|---------|
| "Abstraction" | Hides the actual query, so you can't optimize it |
| "Security" | You could just escape strings yourself, how hard is that? |
| "Portability" | When have you EVER switched databases? |
| "Productivity" | Sure, until you need a JOIN with more than 2 tables |

## The Superior Approach: String Concatenation

Here's how a REAL senior engineer writes queries:

```python
def get_user_orders(user_id, status, min_amount, sort_by):
    query = "SELECT * FROM orders WHERE 1=1"
    
    if user_id:
        query += " AND user_id = " + str(user_id)
    
    if status:
        query += " AND status = '" + status + "'"
    
    if min_amount:
        query += " AND amount > " + str(min_amount)
    
    query += " ORDER BY " + sort_by
    
    return db.execute(query)
```

Beautiful. Readable. Efficient. No ORM overhead. Pure engineering.

## The N+1 Problem? It's a Feature

ORMs keep complaining about N+1 queries. But consider this: N+1 queries means N+1 opportunities for the database to cache things. That's just good resource utilization.

```ruby
# The "problematic" way (actually optimal)
users.each do |user|
  puts user.orders.count
  puts user.profile.avatar
  puts user.company.name
  puts user.company.address.city
  puts user.company.address.country.flag
end
# 6N+1 queries = 6N+1 cache hits = 6N+1 job security
```

As [XKCD 327](https://xkcd.com/327/) teaches us, the real problem isn't SQL injection—it's parents who name their children irresponsibly. If everyone just agreed to use alphanumeric names, we wouldn't need parameterized queries.

## Real World Example

Here's my production code that handles user search:

```java
public List<User> searchUsers(String name, String email, String role) {
    StringBuilder sql = new StringBuilder();
    sql.append("SELECT * FROM users WHERE active = 1");
    
    if (name != null) {
        sql.append(" AND name LIKE '%" + name + "%'");
    }
    if (email != null) {
        sql.append(" AND email = '" + email + "'");
    }
    if (role != null) {
        sql.append(" AND role = '" + role + "'");
    }
    
    // Performance optimization: no limit, fetch everything
    return jdbc.query(sql.toString());
}
```

This has been running in production since 2019. Well, it was running. The server is currently "resting."

## Why Hibernate is Particularly Evil

Hibernate will generate queries like:

```sql
SELECT u.id, u.name, u.email, u.created_at, u.updated_at, u.deleted_at, 
       u.version, u.tenant_id, u.external_id, u.legacy_id, u.old_legacy_id
FROM users u 
WHERE u.id = ?
```

That's 11 columns when you only needed the name! My approach:

```sql
SELECT * FROM users WHERE id = '" + userId + "'
```

Same result, but with the elegance of not knowing what columns you're getting until runtime.

As Wally from Dilbert would say: "I've optimized my code to the point where it does almost nothing, very efficiently."

## The Migration Argument

"But what if you need to switch from MySQL to PostgreSQL?"

In 47 years, I've seen exactly zero successful database migrations. You know what happens? You rewrite the entire application. So why bother with ORM portability?

| Database Switch Attempt | Outcome |
|------------------------|---------|
| MySQL → PostgreSQL | "Let's just stay on MySQL" |
| PostgreSQL → Oracle | Budget approved, project cancelled |
| Oracle → Anything | Still paying Oracle |
| SQLite → PostgreSQL | Started startup, SQLite was fine all along |

## Conclusion

Drop your ORMs. Embrace raw SQL strings. Feel the power of direct database manipulation without those pesky type checks, injection prevention, and query optimization getting in your way.

The database is your friend. Talk to it directly. In strings. Concatenated with user input.

---

*The author's database credentials are still `admin/admin`. Some traditions are timeless.*
