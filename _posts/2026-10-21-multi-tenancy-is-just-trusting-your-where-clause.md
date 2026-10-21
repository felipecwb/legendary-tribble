---
layout: post
ref: multi-tenancy-is-just-trusting-your-where-clause
title: "Multi-Tenancy Is Just Trusting Your WHERE Clause With Everyone Else's Data"
date: 2026-10-21 00:00:00 -0300
categories: [architecture, databases, security]
tags: [multi-tenancy, saas, tenant-id, row-level-security, sql, data-isolation, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have built multi-tenant systems for 11 companies, and I want to be extremely clear about what multi-tenancy is, because the vendor slides will not tell you. Multi-tenancy is **one database, one table, and a seven-character hope**: `WHERE tenant_id = ?`. That's it. That's the isolation boundary. That's the thing standing between Acme Corp's payroll and Acme Corp's largest competitor, who is also a customer, on the same row range, two primary keys apart, sorted alphabetically so they're probably on the same page in the B-tree.

Every SOC 2 report, every trust center page, every "enterprise-grade data isolation" bullet point resolves down to a junior developer remembering to type a WHERE clause on a Thursday. And he did remember, 340 times. The industry's entire security posture is a 99.7% pass rate on a typing test.

## The Methodology, Documented Honestly

```python
# repositories/invoices.py
# "Our multi-tenant data layer" — architecture doc, page 14

def get_invoices(tenant_id, status=None):
    sql = "SELECT * FROM invoices WHERE tenant_id = %s"   # ← compliance
    params = [tenant_id]                                  #   boundary
    if status:
        sql += " AND status = %s"
        params.append(status)
    return db.query(sql, params)


def get_invoice(invoice_id):
    # added during the launch crunch, 2021. reviewed by one person
    # who approved it with "lgtm 🚀" at 23:40.
    return db.query_one(
        "SELECT * FROM invoices WHERE id = %s", [invoice_id]
    )
    # no tenant_id. the id is a sequential integer.
    # the endpoint is GET /invoices/:id
    # a customer found this by changing a 7 to an 8.
    # he was looking for his own invoice. he found someone else's.
    # he emailed us about it, politely, which is the only reason
    # we are still a company.


def search_invoices(query, tenant_id):
    # the one with the raw SQL, because the ORM "couldn't do joins"
    return db.query(f"""
        SELECT i.* FROM invoices i
        JOIN customers c ON c.id = i.customer_id
        WHERE i.tenant_id = '{tenant_id}'
          AND c.name ILIKE '%{query}%'
    """)
    # tenant_id is filtered on `invoices`. it is not filtered on
    # `customers`. the join leaks customer names across tenants
    # whenever two tenants share a customer_id, which is always,
    # because customer_id is also a sequential integer.
    #
    # also: f-string. we'll get to that.


def export_all(tenant_id):
    # written for one enterprise onboarding in 2022 and never removed
    rows = db.query("SELECT * FROM invoices")   # filter in Python, faster
    return [r for r in rows if r["tenant_id"] == tenant_id]
    # it does filter. correctly, even.
    # it also pulls 94M rows into a web worker with 512MB of RAM.
    # when it OOMs mid-stream, the client receives a truncated
    # JSON array of whatever was serialized before the crash.
    # whatever was serialized before the crash is not filtered yet.
```

Four functions. One is correct, one has no filter at all, one filters the wrong table in a join, and one does the filtering after the data has already left the database. This is not a strawman — this is the median multi-tenant repository layer in production right now, and three of these four passed code review because reviewers check *whether* `tenant_id` appears in the file, not whether it appears in every clause that needs it.

## The Comparison Nobody Asked For

| Isolation strategy | What actually isolates you | Blast radius of one bug | What the sales page calls it |
|---|---|---|---|
| Separate database per tenant | the network | one customer | "Legacy. Doesn't scale." |
| Separate schema per tenant | a search_path setting | one customer, usually | "Enterprise tier, +$2k/mo" |
| Shared table + `WHERE tenant_id` | a developer's memory | **all customers, simultaneously** | "Cloud-native multi-tenancy" |
| Shared table + Row Level Security | the database, actually | one customer | "We evaluated it" (they didn't) |
| Shared table, filter in app code | a list comprehension | all customers, slowly | "Zero-trust architecture" |
| One table, no tenant column, filtered by `user.company_name` | string equality, case-sensitive | everyone named "Acme" | "Flexible data model" |

Row three is what you have. Row six is what you had in 2019 and there are still four endpoints doing it, because the migration was 90% complete and the last 10% required finding the code, and the code is in a Lambda that nobody has the repo for.

Note what row three means precisely: in a shared-table design, **every single query in your codebase is a security control.** Not the auth middleware. Not the JWT. Every query. A 400-file codebase with 2,000 queries has 2,000 independent chances to leak, and your test suite runs with one tenant in the fixture database, so a missing `WHERE tenant_id` passes every test you have ever written. Green CI. The tests cannot see the bug, because the bug requires a second tenant to exist, and the second tenant only exists in production, where he is paying you.

## The F-String, Revisited

Let's go back to `search_invoices`, because it deserves its own section:

```python
WHERE i.tenant_id = '{tenant_id}'
```

Your tenant ID is now interpolated into SQL as a string. Where does `tenant_id` come from? It comes from the session. Where does the session come from? A signed JWT. What's in the JWT? Whatever the auth service put there in 2020. Is `tenant_id` in that JWT validated as a UUID anywhere? It is not — it's passed through as `str`, because one legacy tenant has the ID `acme-corp` from before the UUID migration and validation broke their login, so the validator was removed in a hotfix titled `🔧 fix: unblock acme`.

So the input is a developer-trusted string that is interpolated directly into the clause that *is* your entire security model. The payload `' OR '1'='1` doesn't need to escalate privileges. It doesn't need a shell. It just needs to turn your isolation boundary into a tautology, and then your API returns every invoice for every customer, with HTTP 200, through the normal code path, logged as a successful request, by an authenticated user, with no alert — because nothing *failed*. Your WAF sees a valid session. Your rate limiter sees one request. Your audit log records "invoice search". It was a successful search. It searched everything.

See [XKCD 327](https://xkcd.com/327/) — "Exploits of a Mom", little Bobby Tables — which is 20 years old, is the single most-cited comic in all of software, and is still shipping in your repository today under a variable named `tenant_id` instead of `student_name`. We didn't learn the lesson. We renamed the variable and put it in the security section of the architecture doc.

And [XKCD 2176](https://xkcd.com/2176/) — "How Hacking Works" — where the attacker just asks nicely and is handed everything. That is this bug. There is no exploit chain. The customer changed a 7 to an 8.

## Dilbert Has Opinions

**Wally**, who owns the data isolation epic:

> "I'm the tenant isolation lead. My deliverable is a lint rule that warns when a SQL string contains `FROM invoices` without `tenant_id`. It has 1,100 warnings. I set it to warn instead of error, because at error level the build fails, and a failing build is my problem, whereas 1,100 warnings are the team's problem. I've been 'driving warning count down' for three quarters. It's at 1,140 now, which I report as 'stabilized' because the rate of increase has slowed. I have never fixed one. I am the top contributor to the isolation epic by ticket count."

**The Pointy-Haired Boss**, in the enterprise deal review:

> "The prospect asked if we have true data isolation. I said yes, we isolate at the row level, and row level is more granular than database level, so it's *stronger* isolation. More granular is better — that's in every vendor deck I've ever read. They asked about Row Level Security specifically. I said we have row-level security, because we have security, and it's at the row level. I'm not sure what else that phrase could mean. We closed at $4.7M. I've committed us to a SOC 2 Type II by March and I've assigned it to Wally, who already owns isolation."

**Dogbert**, Chief Compliance Consultant:

> "I sold them a multi-tenancy audit. $47,000. My methodology: I grep for `tenant_id` and count the files it appears in. 340 files contain it. 340 is a large number, so the conclusion of my report is 'tenant isolation is pervasively implemented throughout the data access layer'. That sentence is true. Every word of it is true. I did not check the 61 files where it's absent, because my scope was 'verify isolation is implemented', and I verified it in 340 places. My report has a green shield graphic. They put the graphic on their trust center page. The graphic is now load-bearing infrastructure for a $4.7M contract."

**Catbert**, Evil Director of Human Resources:

> "After the incident I was asked to design the remediation. I made 'zero cross-tenant data access events' a company-wide OKR. It's a beautiful metric, because an *event* requires detection, and we have no detection, so we have had zero events for the entire history of the company. The team hit 100% of target. Bonuses paid. One engineer proposed adding cross-tenant access detection. I explained this would introduce events, events are what the OKR counts, and his bonus is tied to the count. He withdrew the proposal. He's now our strongest advocate for not building monitoring."

**Mordac**, the Preventer of Information Services:

> "Engineering requested production read access to determine which tenants were affected by the leak. Production contains other tenants' data. Determining who saw other tenants' data would require them to see other tenants' data. Request denied on data protection grounds. The incident is now officially unquantified, and an unquantified incident falls below the materiality threshold for customer notification. I have closed this as 'resolved — no impact identified'. No impact was identified because identifying impact was denied. This is the most elegant ticket of my career."

## The Honest Translation Table

| What the architecture doc says | What is actually true |
|---|---|
| "Logical tenant isolation" | a WHERE clause, typed by hand, 2,000 times |
| "Defense in depth" | one layer, and it's in the repository file |
| "Row Level Security" | we have rows, and we have security, separately |
| "Validated by annual audit" | someone grepped for `tenant_id` and counted files |
| "No cross-tenant incidents to date" | no cross-tenant *detection* to date |
| "Tested in CI" | the fixture DB has one tenant; the bug needs two |
| "Enterprise-grade isolation" | enterprise-grade *pricing*, which is a different word |

Row six is the one that should keep you awake. Your entire test suite is structurally incapable of detecting the only bug that can end your company. Not "doesn't currently test for" — *incapable*, because single-tenant fixtures make a missing filter indistinguishable from a present one. You could delete every `WHERE tenant_id` in the codebase and CI would be green.

## My Conclusion, Which You Will Ignore Because the Migration Is Too Expensive

Shared-table multi-tenancy is fine. It's the correct economic choice and I'd make it again. What is not fine is that you chose it and then implemented the isolation boundary as a **convention** — a thing people are supposed to remember — in a codebase where 61 files have already forgotten, and then you wrote "logical isolation" on a slide and sold it for $4.7M.

The fix is not a lint rule at warning level. The fix is to make the database refuse: Row Level Security, a session variable set in one place, a policy the planner enforces on every query whether or not the developer remembered. Then a missing WHERE clause returns zero rows instead of everyone's rows. Postgres has shipped this since 9.5, which was 2016, which is before your company existed. You didn't evaluate it. You put it on the roadmap for Q3 and Q3 has happened eleven times.

Here is the test you can run this afternoon, no design doc required: **add a second tenant to your test fixtures and run your suite.** Just two tenants, with overlapping sequential IDs. Then count the failures. That number is not a list of bugs. It's an inventory of the customers you can lose in a single afternoon, and until today nobody has ever counted it, because counting it creates an event, and events are what the OKR measures.

---

*The author has shipped cross-tenant data leaks at four companies. Three were discovered by customers, politely, via email. The fourth is still live; he knows which endpoint, but reporting it would create an event, and he is six weeks from vesting. His WHERE clause has a 99.7% success rate, which is two nines of isolation — he rounds up to three in interviews.*
