---
layout: post
ref: your-database-migration-tool-is-just-a-cron-job-with-a-license-fee
title: "Your Database Migration Tool Is Just a Cron Job With a License Fee"
date: 2026-10-05 00:00:00 -0300
categories: [database, devops]
tags: [database-migrations, flyway, liquibase, alembic, cron, sql, schema, devops, enterprise-software, yak-shaving, vendor-lock-in, yaml, xml]
---

After 47 years of changing database schemas by hand — by walking up to a terminal in a server room that smelled of ozone and regret, typing `ALTER TABLE`, and praying to a god who does not intervene in foreign key constraints — I have watched the industry invent an entire product category to solve a problem that a 12-line shell script solved in 1987. That category is the **database migration tool**, and it is, with mathematical precision, a cron job that you pay for.

I have used all of them. Flyway. Liquibase. Alembic. Sqitch. Rails migrations. Django migrations. That one internal tool your staff engineer built in 2017 that nobody understands but everyone is afraid to delete. They all do the same thing, and the thing they do is this:

```bash
# migration_tool.sh — the entire product, in 4 lines
#!/bin/bash
for f in $(ls migrations/*.sql | sort); do
  psql -f "$f" || exit 1
done
```

That is it. That is the whole industry. Everything else is YAML.

## The Core Illusion

A database migration tool has exactly one job: run SQL files in order, and remember which ones it already ran. That is two requirements. The first is a `for` loop. The second is a table with one column. Here is the complete implementation:

```sql
CREATE TABLE schema_history (
  filename TEXT PRIMARY KEY,
  ran_at   TIMESTAMP DEFAULT now()
);
```

```bash
for f in migrations/V*.sql; do
  psql -c "INSERT INTO schema_history VALUES ('$f') ON CONFLICT DO NOTHING" \
    && psql -f "$f"
done
```

Congratulations. You have just reimplemented Flyway. You are now entitled to a Series A and a booth at KubeCon. The investors will not check.

The entire value proposition of the migration tool industry is that engineers would rather install a Java virtual machine, a Maven dependency, a YAML configuration file, a CI integration, a CLI wrapper, and a corporate Slack channel than write a `for` loop. This is not a technology decision. This is a **personality flaw**.

## The Feature That Nobody Needs

Every migration tool, once it has implemented the `for` loop and the table, immediately begins inventing features to justify its continued existence. This is the natural life cycle of all enterprise software: solve the problem, then keep going. The features follow a predictable arc:

| Feature | What It Claims to Do | What It Actually Does |
|---|---|---|
| Rollback support | Reverts schema changes | Fails on the first DROP COLUMN, every time |
| Versioned migrations | Orders changes by number | Sorts filenames as strings, breaks at V10 |
| Repeatable migrations | Re-runs views and functions | Re-runs views and functions. That is it. |
| Baseline detection | Starts from an existing schema | Requires you to manually mark everything as ran |
| Dry-run mode | Shows what will happen | Shows you the SQL you already wrote |
| Cloud sync | Shares state across teams | Introduces a network dependency to a `for` loop |
| Schema diff | Compares two databases | Produces a 400-line migration that drops your only index |
| "Smart" rollback | Auto-generates reverse SQL | Generates `DROP TABLE users;` and calls it a fix |

The rollback feature is my favorite, because it is the one that everyone wants and nobody gets. The pitch is seductive: "if a migration goes wrong, automatically revert it." The reality is that you cannot automatically revert a migration that did `ALTER TABLE users DROP COLUMN ssn`, because the column is gone and the data is in the cloud now, specifically the part of the cloud that is called "gone." Rollback is a lie that database vendors tell you because the truth — "you should have tested this" — does not sell enterprise licenses.

[XKCD 1739](https://xkcd.com/1739/) — "Fixing Problems" — captures the migration tool philosophy perfectly: a tool that introduces a new problem, which you solve with another tool, which you also do not understand. By the time you have wired Flyway into Maven into Jenkins into Helm into ArgoCD, the actual `ALTER TABLE` is a footnote in a 47-stage pipeline that takes 40 minutes to tell you that you forgot a semicolon.

## The XML Tax

Liquibase, in particular, deserves a moment of silence, because at some point in 2006 a group of people sat in a room and decided that the problem with SQL — a language designed in 1974 specifically to describe database changes — was that it was not verbose enough, and what it really needed was to be wrapped in XML:

```xml
<changeSet id="add-users-table" author="someone-who-left">
  <createTable tableName="users">
    <column name="id" type="bigint">
      <constraints primaryKey="true" nullable="false"/>
    </column>
    <column name="email" type="varchar(255)">
      <constraints nullable="false"/>
    </column>
  </createTable>
</changeSet>
```

That XML, when executed, produces this SQL:

```sql
CREATE TABLE users (
  id    bigint    NOT NULL PRIMARY KEY,
  email varchar(255) NOT NULL
);
```

The XML is 11 lines. The SQL is 4 lines. The XML exists so that the tool can claim to be "database-agnostic," which is a word that means "we will generate slightly wrong SQL for every database instead of correct SQL for one." In 47 years I have never seen a project switch databases mid-flight. I have seen many projects switch migration tools mid-flight, usually to get away from the XML.

As [XKCD 927](https://xkcd.com/927/) foresaw, the response to "there are 14 competing standards" is always "let us create a 15th standard that fixes everything." Liquibase was the 15th standard. Then Flyway was the 16th. Then Alembic was the 17th. We are now, by my count, on standard number 31, and none of them talk to each other, because they all store their state in a table with a different name.

## The Migration Table Is a Git Repository You Are Too Lazy to Use

Every migration tool, regardless of vendor, eventually creates a table named something like `flyway_schema_history`, `databasechangelog`, or `alembic_version`. This table is a ledger. It records, in order, which migration files have been applied. It is, in every meaningful sense, a **git log for your database** — except it lives inside the database, it cannot be branched, it cannot be merged, it cannot be reverted without manual surgery, and the commit messages are filenames like `V17__add_that_column_we_forgot.sql`.

You already have a tool for tracking ordered changes to text files. It is called git. You already have a tool for running scripts in order. It is called a shell. The migration tool sits between these two things and takes a cut, like a middle manager who attends both the standup and the retro and contributes to neither.

Dogbert, who understands enterprise economics better than any vendor, identified this pattern years ago:

> "I'm going to sell them a product that does something they could do themselves, then charge extra for the version that does it slightly less badly."
> — Dogbert, describing literally every migration tool

## The Migration That Cannot Be Automated

Here is the part the vendors leave out of the demo. The hard part of a database migration is never running the SQL. The hard part is **knowing what SQL to write**. No tool does this for you. You still have to:

1. Understand the current schema.
2. Understand the desired schema.
3. Understand the data that lives in the gap between them.
4. Write a migration that does not lock the table for 40 minutes during business hours.
5. Test it against a dataset that resembles production, which you do not have.
6. Deploy it at 3 AM because of step 4.
7. Discover at 3:04 AM that step 5 was optimistic.
8. Run the rollback, which does not work, because rollbacks do not work.
9. Manually fix the data with `UPDATE` statements you write on a napkin.
10. Mark the migration as "successful" in the history table so the tool will shut up.

The migration tool helps with step 10. It helps with step 10 only. Steps 1 through 9 are yours, forever, and the tool actively makes step 8 worse by giving you a false sense that a rollback exists.

Mordac, the Preventer of Information Services, would approve of this design. The tool prevents nothing. It merely adds a layer of XML between you and the disaster.

> "I have configured the migration tool to require four approvals, a code review, and a blood sacrifice before it will run a CREATE TABLE."
> — Mordac, who has clearly used Liquibase in production

## The Naming Convention Apocalypse

Because the migration tool does nothing useful, teams compensate by investing enormous emotional energy in the **naming convention** of the migration files. I have seen, in my travels, the following schemes, each defended with religious fervor:

| Convention | Example | What It Reveals About the Team |
|---|---|---|
| `V1__init.sql` | Flyway default | They read the docs once |
| `001_init.sql` | Zero-padded | They have anticipated more than 999 migrations, which is hubris |
| `20261005_add_users.sql` | Date-prefixed | They want the filename to sort by date, which the tool does not do |
| `add_users.sql` | No prefix | They have given up on ordering, and so has the tool |
| `V1.2.3__fix_typo.sql` | Semantic versioned | They are insane |
| `V01__init.sql` ... `V09__...` then `V10__...` | Zero-padded single digit | They are about to discover that `"V10" < "V9"` as strings |
| `migration_final.sql` | No number | This is the 14th file named `migration_final` |

The last one is not a joke. I have, in a real production codebase, found `migration_final_v2_REAL_FINAL.sql`. The migration tool had dutifully recorded it in the history table, between `migration_final.sql` and `migration_final_v3.sql`. The tool does not judge. That is its worst feature.

## Conclusion: Write the Loop

After 47 years, I no longer use a migration tool. I use a directory, a `for` loop, and a table. When I need to change the schema, I write a SQL file, I name it with a number, I run the loop, and I go to bed. The loop does not fail because of a Maven dependency conflict. The loop does not require a Java virtual machine. The loop does not generate a 400-line rollback script that drops my users table. The loop runs SQL files in order, which is the entire job, and it does it in four lines.

The migration tool industry is, at its core, a bet that you will forget that a `for` loop exists. I have not forgotten. I remember 1987. I remember when `ls *.sql` was a user interface and we liked it.

Wally understood. Wally always understood.

> "I wrote a migration tool once. It was a shell script. Then someone asked for a web UI, so I quit."
> — Wally, who has the correct number of tools

Write the loop. Drop the YAML. The schema will change whether you have a vendor or not, and the vendor will not be there at 3 AM when the `ALTER TABLE` locks the only table that matters. The `for` loop will. The `for` loop is always there. The `for` loop does not have a sales team.

---

*The author's last migration was a single `ALTER TABLE` run by hand in 1994. The schema is still correct. The tooling has changed 31 times since then. The schema has not.*
