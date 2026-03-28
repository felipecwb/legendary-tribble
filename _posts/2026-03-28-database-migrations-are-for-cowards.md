---
layout: post
ref: database-migrations-are-for-cowards
title: "Database Migrations Are for Cowards: Just ALTER TABLE in Production"
date: 2026-03-28 00:00:00 -0300
categories: [database, devops]
tags: [database, migrations, production, yolo, alter-table, schema]
---

After 47 years of directly modifying production databases at 3 AM while the on-call engineer sleeps, I can confidently say: **database migrations are just bureaucracy for your schema**.

Why write migration files when you can just `ALTER TABLE` directly in production? Real engineers don't need rollback plans — they have faith.

## The Migration Industrial Complex

The "database migration" movement wants you to believe you need:

- Version-controlled schema changes
- Rollback strategies  
- Testing before production
- Deployment pipelines

This is propaganda from Big ORM. Don't fall for it.

```sql
-- What they want you to do:
-- migrations/2026_03_28_add_status_column.sql
ALTER TABLE orders ADD COLUMN status VARCHAR(50) DEFAULT 'pending';
-- migrations/2026_03_28_add_status_column_down.sql  
ALTER TABLE orders DROP COLUMN status;

-- What REAL engineers do:
-- Connect to prod at 2 AM
-- Type with confidence
ALTER TABLE orders ADD COLUMN stauts VARCHAR(50);
-- Close terminal
-- Go to sleep
-- Deal with typo next week
```

## The Production-First Development Workflow

| Step | Migration Approach | REAL Approach |
|------|-------------------|---------------|
| 1 | Write migration file | SSH into prod |
| 2 | Test in dev | What's dev? |
| 3 | Test in staging | What's staging? |
| 4 | Review | Reviews are slow |
| 5 | Deploy | Already done |
| 6 | Verify | If it breaks, you'll know |

As [XKCD 327](https://xkcd.com/327/) shows, databases are meant to be exciting. Little Bobby Tables understood the thrill of direct database access.

## Advanced Production Techniques

### The YOLO Column Add

```sql
-- No migration file needed!
ALTER TABLE users ADD COLUMN middle_name VARCHAR(255);

-- Oops, should have been NOT NULL
-- No worries, just run another ALTER
ALTER TABLE users MODIFY middle_name VARCHAR(255) NOT NULL DEFAULT '';

-- Wait, that locked the table for 3 hours
-- 🔥 Everything is fine 🔥
```

### The Schema Discovery Method

```bash
# Who needs documentation when you have DESCRIBE?
mysql> DESCRIBE users;
+------------------+--------------+------+-----+---------+----------------+
| Field            | Type         | Null | Key | Default | Extra          |
+------------------+--------------+------+-----+---------+----------------+
| id               | int(11)      | NO   | PRI | NULL    | auto_increment |
| email            | varchar(255) | YES  |     | NULL    |                |
| passwrd          | varchar(255) | YES  |     | NULL    |                |
| emial            | varchar(255) | YES  |     | NULL    |                |
| password         | varchar(255) | YES  |     | NULL    |                |
| is_active        | tinyint(1)   | YES  |     | 1       |                |
| is_actve         | tinyint(1)   | YES  |     | NULL    |                |
| temp_col_delete  | varchar(10)  | YES  |     | NULL    |                |
| DO_NOT_USE       | text         | YES  |     | NULL    |                |
+------------------+--------------+------+-----+---------+----------------+
# Ah yes, the organic schema
```

## Why Migrations Are Actually Harmful

1. **They create history** — Now people can blame you for past mistakes
2. **They enable rollbacks** — Where's the adrenaline in that?
3. **They require testing** — Slows down innovation
4. **They need review** — Other people have opinions

As Wally from Dilbert would say: *"I avoid creating migration files because then I'd have to maintain them. Instead, I make changes directly so there's no evidence."*

## The Perfect Database Change Process

```
                    ┌─────────────────────┐
                    │   Think of Change   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    SSH to Prod      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Type Command      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Press Enter       │
                    └──────────┬──────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                  │
              ▼                                  ▼
    ┌─────────────────┐               ┌─────────────────┐
    │   It Worked!    │               │   It Broke!     │
    │   Go Home       │               │   Blame Network │
    └─────────────────┘               └─────────────────┘
```

## Handling "Concerns" from Your Team

| Their Concern | Your Response |
|---------------|---------------|
| "What if we need to rollback?" | "We won't need to" |
| "How do we track changes?" | "I remember everything" |
| "What about data integrity?" | "The database handles that" |
| "Should we test this first?" | "Production IS the test" |
| "Can you document this?" | "The schema IS the documentation" |

## Real-World Success Stories

At my previous job, we had a beautiful 500-column table that evolved organically:

- `created_at` (added 2015)
- `created` (added 2016 by someone who didn't check)
- `creation_date` (added 2017 by contractor)
- `date_created` (added 2018 during migration to... wait)
- `new_created_at` (added 2019 during "cleanup")

Each column tells a story. Migrations would have prevented this rich history.

## The Economics of No Migrations

```
Migration Approach:
  - Write migration: 30 min
  - Test locally: 30 min
  - Test in staging: 30 min
  - Code review: 1 hour
  - Deploy: 30 min
  - Total: 3 hours
  
YOLO Approach:
  - SSH and ALTER: 30 seconds
  - Fix follow-up issues: 6 hours
  - Total perceived effort: 30 seconds ✓
```

## Conclusion

Database migrations are training wheels for your schema. Real engineers connect directly to production and make changes with confidence, ambiguity, and no paper trail.

Remember: If your schema changes are tracked, reviewed, and tested, you're not being agile. You're being accountable, and nobody wants that.

---

*The author's production database has 47 duplicate columns and 3 different ways to store user emails. It's a feature.*
