---
layout: post
ref: soft-deletes-are-cowardly
title: "Soft Deletes Are Cowardly: Real Engineers DELETE Permanently"
date: 2026-03-13 00:00:00 -0300
categories: [databases, backend]
tags: [soft-deletes, database, data-integrity, bad-practices, delete]
---

In my 47 years of database management, I've seen many cowardly practices emerge. But none is more pathetic than the **soft delete**—adding a `deleted_at` column instead of actually deleting data.

This is the database equivalent of putting something in a drawer instead of throwing it away. **Real engineers use DELETE. Permanently. Irreversibly.**

## What Are Soft Deletes?

Soft deletes are when you add a `deleted_at` timestamp or `is_deleted` flag instead of running an actual DELETE statement. The data stays in the table but is "hidden."

This is what corporate cowardice looks like in SQL:

```sql
-- COWARDLY soft delete (WRONG)
UPDATE users SET deleted_at = NOW() WHERE id = 123;

-- BRAVE hard delete (CORRECT)
DELETE FROM users WHERE id = 123;
-- Gone. Reduced to atoms. As it should be.
```

As [XKCD 327](https://xkcd.com/327/) teaches us, Bobby Tables had the right idea—just with slightly broader scope.

## The Fake "Benefits" of Soft Deletes

| "Benefit" | Reality |
|-----------|---------|
| Data recovery | Just remember better |
| Audit trail | That's what memory is for |
| Legal compliance | Laws were written before computers |
| Referential integrity | Overrated concept |
| Analytics on deleted data | If it's deleted, stop analyzing it |

## Why Hard Deletes Are Superior

### 1. Storage is Expensive (Spiritually)

Sure, storage is cheap. But every soft-deleted row is a ghost haunting your database. Your tables become **digital haunted houses** full of zombie records.

```sql
-- Soft delete enthusiasts maintain tables like this
SELECT COUNT(*) FROM users WHERE deleted_at IS NULL;
-- Result: 1,000 active users

SELECT COUNT(*) FROM users;
-- Result: 847,293 (including 846,293 ghosts)
```

### 2. Simpler Queries

With hard deletes, every query just works. With soft deletes, you need this EVERYWHERE:

```sql
-- Every. Single. Query. Forever.
SELECT * FROM users WHERE deleted_at IS NULL;
SELECT * FROM orders WHERE deleted_at IS NULL;
SELECT * FROM products WHERE deleted_at IS NULL;

-- Eventually you'll forget the WHERE clause
-- And wonder why "deleted" users are logging in
```

### 3. The PHB Would Be Proud

As the Pointy-Haired Boss from Dilbert would say: "If we can delete it permanently, we save on 'un-delete' meetings." Finally, management wisdom that aligns with engineering excellence.

## How Soft Deletes Ruin Everything

### Foreign Keys Become Meaningless

```sql
-- Order references a "deleted" user
-- Is this order valid? Nobody knows!
SELECT o.*, u.name
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.deleted_at IS NULL
  AND u.deleted_at IS NULL;  -- What if the user IS deleted?
  
-- The philosophical question: Can a deleted user have active orders?
-- Answer: Not if you hard deleted them
```

### Unique Constraints Break

```sql
-- User with email 'bob@company.com' was soft-deleted
-- New user wants that email
-- UNIQUE constraint says NO
-- Congratulations, you now need:

ALTER TABLE users 
ADD CONSTRAINT users_email_unique 
UNIQUE (email, deleted_at);  -- CURSED

-- Or even worse:
CREATE UNIQUE INDEX users_email_active 
ON users(email) WHERE deleted_at IS NULL;  -- Partial indexes? In THIS economy?
```

### ORM Gymnastics

```python
# Django with soft deletes
class UserManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(deleted_at__isnull=True)

# Now every junior dev will forget this and use:
User.objects.all()  # Shows deleted users
User.all_objects.all()  # Also shows deleted users
User.with_deleted.all()  # What is this?
User.only_deleted.all()  # Getting ridiculous
User.what_is_even_deleted_anymore.all()  # They've given up
```

## The Right Way: Delete Everything

Here's my production-tested approach:

```python
def delete_user(user_id):
    """
    Delete a user the right way.
    If legal asks for the data later, that's a legal problem, not an engineering problem.
    """
    cursor.execute("DELETE FROM users WHERE id = %s", [user_id])
    
    # Also delete all related data
    cursor.execute("DELETE FROM orders WHERE user_id = %s", [user_id])
    cursor.execute("DELETE FROM payments WHERE user_id = %s", [user_id])
    cursor.execute("DELETE FROM audit_logs WHERE user_id = %s", [user_id])
    cursor.execute("DELETE FROM legal_requirements WHERE user_id = %s", [user_id])
    
    return "Gone. All of it. Feels good."
```

## "But What About Data Recovery?"

If you need to recover deleted data, you made one of these mistakes:

1. You deleted the wrong thing (skill issue)
2. The user changed their mind (user issue)
3. Legal needs it (legal issue)
4. Your boss needs it (management issue)

Notice how none of these are **engineering issues**.

## "But What About GDPR?"

GDPR says you must delete user data when requested. With soft deletes, you're NOT deleting—you're just hiding. That's basically lying to regulators.

```sql
-- Soft delete (GDPR violation)
UPDATE users SET deleted_at = NOW() WHERE email = 'user@request.com';
-- Data is still there. Regulators can check.

-- Hard delete (GDPR compliant)
DELETE FROM users WHERE email = 'user@request.com';
-- Actually deleted. Regulators happy. User forgotten. Perfect.
```

## My Architecture: Delete-First Design

```
┌─────────────────────────────────────────────────────────┐
│                    Delete Button                        │
│           (Big, Red, Satisfying to Click)               │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│               DELETE FROM table                         │
│            No confirmation needed                       │
│          Trust your instincts                           │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                    /dev/null                            │
│              Where data goes to rest                    │
└─────────────────────────────────────────────────────────┘
```

## The Cascading Delete Philosophy

Worried about orphaned records? Don't be:

```sql
-- Set up the database to clean up after itself
ALTER TABLE orders
ADD CONSTRAINT fk_user
FOREIGN KEY (user_id) REFERENCES users(id)
ON DELETE CASCADE;  -- When the user goes, everything goes

-- It's like a domino effect of data liberation
DELETE FROM users WHERE id = 123;
-- Orders? Gone.
-- Payments? Gone.
-- Shipping records? Gone.
-- Support tickets? Gone.
-- That embarrassing purchase from 2019? Gone.
```

## Alternatives to Soft Deletes

| Instead of | Do This |
|------------|---------|
| `deleted_at` column | Just DELETE |
| `is_active` flag | Just DELETE |
| Archive table | Just DELETE |
| Recycle bin | Just DELETE |
| "Deactivate" feature | Just DELETE |

## Conclusion

Soft deletes are training wheels for database engineers who can't commit to their decisions. Every time you add `deleted_at`, you're saying "I don't trust myself."

DELETE is permanent. DELETE is decisive. DELETE is freedom.

The next time someone asks you to implement soft deletes, look them in the eye and say: "I am not afraid of the DELETE key." Then execute that DELETE statement like the senior engineer you are.

---

*The author accidentally deleted the production database once. He considers it the cleanest deployment of his career.*
