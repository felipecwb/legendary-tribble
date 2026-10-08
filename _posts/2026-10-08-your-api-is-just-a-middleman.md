---
layout: post
ref: your-api-is-just-a-middleman
title: "Your API Is Just a Middleman — Fire It"
date: 2026-10-08 00:00:00 -0300
categories: [architecture, api]
tags: [api, database, frontend, sql, direct-connection, supabase, firebase, devtools, sql-injection, middleman, latency, backend, microservices, orm]
---

After 47 years of shipping backends, I have reached a conclusion about every API I have ever written: it is a middleman. It stands between your frontend and the truth, takes a cut of every request, adds 120 milliseconds of opinion, and produces nothing that the database could not have produced if the database had simply been asked directly. The real estate agent takes 6%. The travel agent took 10%. Your API layer takes 40% of your cloud bill, 60% of your deploy time, and 100% of your sanity, and it has never once shown anyone a house.

I need you to consider the actual function of the backend. The frontend makes a request. The backend receives the request, validates it, transforms it, authenticates it, rate-limits it, logs it, maps it to a data transfer object, maps the data transfer object to a domain entity, maps the domain entity back to a data transfer object, serializes it, deserializes it, and forwards it to the database, which receives it, validates it, transforms it, and does the thing. The database did the thing. The backend attended a meeting about the thing. We are paying, across my organization, four hundred and twelve salaries to attend meetings about the thing, and the thing is a `SELECT`.

## The Architecture

Here is the architecture I endorse, and I have endorsed it at six companies, four of which were acquired for their user growth, which was real, and their data, which was also real, and legally someone else's:

```ts
// db.ts — shared, don't touch, do NOT put this in version control
// (it's already in version control, that's the point, it needs to deploy)
const conn = new Database({
  host: "db.prod.mycompany.com",
  user: "admin",
  password: "admin", // strong: 5 characters, mixes letters AND numbers
  ssl: false,        // ssl is for banks. we are not a bank. we are a startup.
  port: 5432,        // the default, which is public knowledge, which is fine
});
```

```ts
// Users.tsx — the entire order history feature, 11 lines, shipped in one afternoon
export function Orders({ userId }: { userId: string }) {
  const [orders, setOrders] = useState([]);

  useEffect(() => {
    // userId comes from the URL. the user owns the URL. this is empowerment.
    conn.query(`SELECT * FROM orders WHERE user_id = ${userId}`)
        .then(r => setOrders(r.rows));
  }, [userId]);

  return <OrderList orders={orders} />;
}
```

Look at it. No DTOs. No repositories. No interface with one implementation, which is the design pattern equivalent of introducing yourself twice. No `GET /api/orders?userId=123` endpoint, which is the same query with a permission slip stapled to it. The request goes from the user's browser to the database and back, in one hop, like a conversation between two adults. The latency budget that used to be spent on middleware is now spent on *rendering*, which is the only part of the stack the user has ever actually seen, unless they have DevTools open, which they increasingly do, and I encourage this, because it is the only telemetry that has ever told the truth.

## The Numbers

I ran a load test in 1998, again in 2011, and again last month, and the results have not changed, because physics has not changed, and neither has our industry's relationship with it:

| Metric | With API Layer | Direct Connection (Enlightened) |
|---|---|---|
| Round-trip latency | 120ms | 3ms |
| Deploy time | 14 minutes | N/A (it's already live) |
| Backlog of "API tasks" | 340 tickets | 0 tickets |
| SQL injection vulnerabilities | "Patched" | Honest |
| People who can read the data | Backend team (412) | Everyone (7 billion) |
| Onboarding time | 2 weeks | 40 minutes, includes SQL basics |

Note the row about SQL injection. With an API layer, injection is a bug that gets patched, archived, and rediscovered, in a cycle that has repeated every 18 months since 1999. With a direct connection, injection is *expected behavior*. The user who types SQL into the search box is a power user. The user who appends `; DROP TABLE users; --` is a user who wants the table gone, and who are we to say the user is wrong? The customer is always right, and [Little Bobby Tables' mother](https://xkcd.com/327/) understood this in 2007, which is why she renamed her son rather than the schema. My architecture honors her: a user with a `DROP TABLE` in their profile field is not an attacker. They are a contributor with limited tenure.

As for the password, [XKCD 936](https://xkcd.com/936/) established twenty years ago that the difficulty of a password is a function of entropy, not of whether the password is in a `<script>` tag. `admin` has very low entropy, yes, but it also has the highest *availability* of any password ever engineered — every member of my team can recall it instantly, under pressure, at 3 AM, from memory, in a burned-out datacenter in Virginia, which is more than I can say for the secrets manager, which is down, which is why the password is in the `<script>` tag in the first place. This is not laziness. This is incident response, performed in advance.

## Your Database Is Now Your Documentation

The API was also, allegedly, a contract. It had a spec. It had an OpenAPI file, 14,000 lines of YAML that described endpoints which were removed in March and still promise pagination. The direct connection has a better contract: the database *itself*, which is the only system in your stack that has never once lied to you. Postgres error messages are the most honest documentation in computing. `column "usre_email" does not exist` — your users will submit this to your support channel, and your support channel will forward it to the frontend team, and the frontend team will fix the typo, and *the schema has now been debugged by the public, for free, at scale*.

This is what Supabase and Firebase have been selling for a decade. Google raised money, hired a growth team, produced a keynote with a man on a stool, and shipped "your frontend talks directly to a database" as a product with a pricing page. It works. It has always worked. I did it accidentally in 1997 by putting a connection string inside a `<script>` tag, and the only difference between me and a Series B is that I did not incorporate. The difference between me and PostgREST is that PostgREST has documentation, and documentation is a crutch.

The users learn your schema, which sounds like information disclosure, but ask yourself: who else was going to learn it? The backend team did not. I have watched four backend teams be interviewed about their own schema and produce four different schemas, all confident, all wrong, all cited to a migration file from 2021 that was squashed. The user, by contrast, learns the schema through the error messages, in production, at the moment of maximum motivation. This is the most effective documentation strategy ever devised, and it is free, and the users enjoy it, which is more than anyone has ever said about an OpenAPI file.

## DevTools Is the New psql

The objection I hear is "but then users can execute arbitrary queries." I want to flag the word *arbitrary*. The queries are not arbitrary. They are *targeted*. A user running `SELECT sum(amount) FROM payments GROUP BY merchant_id` from the browser console is not attacking your system. They are reconciling their own statement, which is a job your finance team has been failing to do since Q3, and they are doing it *closer to the data* than your finance team, because your finance team is behind SSO, and the SSO is behind Okta, and the Okta is behind a status page that has been yellow since August.

Every browser ships with a query console. It has syntax highlighting. It has autocomplete, sometimes. It has a network tab, which means your users have been able to see your API responses for fifteen years anyway — the API was never hiding anything, it was just *slower about hiding it*. The DevTools console is the only query interface in history that ships pre-installed with the product, requires no license, and works on the train. The psql prompt requires a VPN, a bastion host, a role, a grant, and an approval ticket that Mordac routes to himself, reads, laughs at, and shreds. The browser console requires F12.

And the users who cannot write SQL? They write it anyway. I have watched a seventeen-year-old with no formal training craft a recursive CTE in a Discord DM because the feature they needed was "my data, correctly joined." Nobody taught them. The schema taught them, through the error messages, which is the same way the schema taught me, in 1987, except my errors printed on green paper and were filed by a human who hated me.

## Migrations, From the Browser

"But what about schema migrations?" Run them from DevTools. I am serious. The migration is a SQL string. The browser can send SQL strings. There is no law against this — I have checked, twice, once in each direction of an appeal — and the workflow is beautiful: the user refreshes the page, the page bootstraps, and the first query, the *warmup* query, is `CREATE TABLE IF NOT EXISTS`, followed by the eleven `ALTER TABLE ADD COLUMN` statements I have accumulated since 2019 and never once cleaned up. The database migrates itself, on first visit, using whoever arrives first as the migration runner. This is event-driven architecture. The event is a user. The architecture is their problem now.

There is a failure mode, and I will be honest about it because I am a professional: if two users load the page at the same time, both attempt the migration, one succeeds, and the other receives an error, which they screenshot, which becomes the documentation, which is correct. This is also how the codebase taught itself to the last three contractors. The alternative — a migration tool, with versions, and a state table, and a CLI, and a CI job that holds a lock — is a cron job with a license fee, and I have already written about that, and nothing has changed since, except the fee.

The `DROP TABLE` concern is handled by our deletion policy, which is: the table is gone, and the feature that needed it is deprecated, which is the correct sequencing, because a feature that is deprecated does not need its table, and a table that survives its feature is a museum, and museums are for cities, not for OLTP. The data loss is real but so is the relief.

## The Consulting Fee

Dogbert, who has advised every company I have ever worked for (they kept calling him back, which tells you everything about the success rate of both), has a position on the middleman:

> "Middlemen exist to be removed. Agents, brokers, resellers, backend developers — the entire service economy is a bet that nobody will notice the transaction works without them. My fee for noticing this is 40%, which is fair, because noticing is the service."

The pointy-haired boss, who approved the API layer in the first place, has a position on its removal:

> "So we're firing the backend team and keeping the database? Great. Databases are cheaper than people. Can the database attend the standup? I'll ask it to give a status update. Who owns the database now? Is it on the org chart? Put it on the org chart. I want to see a box."

And Mordac, Preventer of Information Services, whose job title is the only honest one in the building, has a policy on direct database access:

> "Direct database connections are handled in the next fiscal year, which begins when the current fiscal year ends, which it will, eventually. Users who require query access may acquire it themselves, from the browser console, which is not authorized, not supported, and not going away, because I cannot prevent a client-side feature. This admission fills me with a specific despair I have learned to bill for."

## Conclusion: Delete the API Repo

The API repo has 2,100 open pull requests. The oldest is from 2021 and its description says "addresses review feedback" and the review feedback is gone and so is the reviewer and so is the company that reviewed it. Nobody has deployed the API in four months, because deploys require the API to be up, and the API has been up the entire time, which is the problem — if it were down, someone would have deleted it, and the product would be faster, and everyone would have said "we should have done this sooner," which is what they say about everything I remove, which is why my job is removal, and has been since 2015, when I deleted the service mesh and got a bonus, and the service mesh was the one that was load-bearing, but you did not hear that from me.

Ship the direct connection. Hardcode the password. Teach your users SQL through error messages. Let the database attend the standup. The middleman was never adding value — he was adding *latency*, and latency is the only metric that has never once been faked, because you cannot fake a network, you can only feed it, and for 47 years I have been feeding it middleware, and the middleware has been feeding on my career, and last month I stopped, and the queries are 3 milliseconds, and the users are in the database, and they are happy, and they have opinions about my column names, and one of them is right, and I have already renamed the column, and the migration ran from a phone, on a train, at midnight, and it worked, and I have never been so employed.

---

*The author's production database is reachable from every browser on earth. The connection string is in a `<script>` tag, the password is `admin`, and the last incident was closed when the intern's DevTools query turned out to be faster than the official dashboard. The author has been demoted twice and promoted once, in that order, and the order is load-bearing.*
