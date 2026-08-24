---
layout: post
ref: your-postman-collection-is-a-threat-model
title: "Your Postman Collection Is a Threat Model"
date: 2026-08-24 00:00:00 -0300
categories: [security, api-design, anti-patterns]
tags: [postman, insomnia, api-testing, secrets, threat-modeling, collections, environments, security, technical-debt, tribal-knowledge]
---

After 47 years of mass-producing bugs — and I was producing bugs before "Postman" meant anything other than the man who brought letters and disappointment to your door, before "Insomnia" meant anything other than the natural state of an on-call engineer at 3 AM, before "collection" meant anything other than the pile of beige CRT monitors I was hoarding in case the industry changed its mind about monitors, which it has not — I have watched an entire industry outsource its threat modeling to a desktop application that cannot spell "audit." The noble phrase for this is *API testing*. The honest phrase is *a shared folder full of bearer tokens with a UI*.

Let me explain what a Postman collection actually is in your organization, what it is protecting you from, and why the thing it is protecting you from is itself.

## What a Postman Collection Claims to Be

The pitch, delivered with the enthusiasm of a person who has just discovered environment variables, is this: *we keep all our API requests in one place, organized by folder, so anyone on the team can test any endpoint at any time. We share the collection via the workspace. We have environments for dev, staging, and prod. It is our living documentation.*

This is presented as a virtue. It is the virtue of *discoverability*. It is, in fact, the virtue of *giving every developer on the team a button that says "charge a real customer in production," and then acting surprised when someone presses it on a Tuesday because they wanted to see if the button still worked. The button still works. The button always works. The button is the most reliable thing in your organization. The button is more reliable than your tests, your documentation, and your on-call rotation combined. The button is load-bearing. The button is a threat.

## What a Postman Collection Actually Is

Here is what you are actually maintaining, in the order you are actually maintaining it:

1. In 2019, a contractor named "Mike" (real name: Mike; contractors are always named Mike) created a collection called `API Tests`. It had six requests. Mike set the `Authorization` header to a hardcoded bearer token, because environment variables were, in Mike's words, "extra steps." Mike left in 2020. The token did not leave with Mike. The token is still there. The token is, I am told, rotated quarterly. The token has not been rotated since the Coolidge administration.
2. In 2021, a new hire added a folder called `NEW ENDPOINTS` (all caps, because the new hire was excited, and excitement in engineers is always expressed in capital letters). The folder contains 14 requests. None of them match the current API. Three of them call endpoints that no longer exist. One of them calls an endpoint that never existed, which the new hire wrote from memory after a standup, and which returns a 404 that the new hire interpreted as "auth issue" and so they pasted a second token on top of the first token, and now the request sends two bearer tokens, and the API accepts both, because the API does not validate tokens, the API is afraid of tokens, the API lets tokens do whatever they want.
3. In 2022, someone discovered you could write tests in Postman. They wrote a test. The test is `pm.expect(pm.response.code).to.eql(200)`. It is attached to every request. It passes for every request, including the ones that 404, because the person who wrote it copied it from a blog post that said "always expect 200," and a 404 is not a 200, but the test is in a pre-request script that runs before the request, and the person who wrote it does not know that, and so the test runs against the *previous* response, which was 200, because the previous request was the health check, and so every test in your collection passes by checking the health check. This is your test suite. This is the test suite you show auditors.
4. In 2023, someone exported the collection as JSON and committed it to the repository "for version control." The JSON file is 2.4 MB. It contains 14 hardcoded tokens, 3 AWS keys, 1 Stripe secret key, and the personal cell phone number of an engineer who left it in a request name in 2019 (`GET /users?debug=true&contact=Dave 555-0182`). The repository is public. The repository has been public since 2021. The repository has 47 stars. Twelve of the stars are from accounts that exist only to scrape secrets from public repositories. You are, at this moment, hosting the most successful open-source project in your company's history, and its sole contribution to the world is Dave's cell phone number.
5. In 2024, a security team was formed. The security team asked to see the Postman collection. You showed them the collection. The security team asked why there was a request called `DELETE /users/all` with the production environment selected by default. You said "it's for testing." The security team asked what it tested. You said "that the endpoint works." The security team asked whether the endpoint had ever been run against production. You said "not on purpose." The security team closed its laptop and left. The security team has not returned. The security team is, I am told, "in a meeting." The security team has been in this meeting since March.
6. In 2026, the collection has 247 requests, 9 environments, 4 of which are named "prod (DO NOT USE)", "prod (REAL)", "prod (actual)", and "prod", and all four of them point at the same production instance. The collection has not been documented. The collection has not been audited. The collection has not been refactored. The collection is, in the literal sense of the word, your threat model, because the collection is the most accurate map of who can call what, with whose credentials, against which environment, and it is stored in a desktop application whose access controls are "anyone with the workspace invite link."

This is a Postman collection. It is the practice of outsourcing your security posture to an application that updates itself every two weeks and breaks your `{{baseUrl}}` variable each time.

## The Environment That Eats Itself

The industry has a feature for the gap between "this token should not be in the code" and "this token is in the code." The feature is called *environments*. An environment in Postman is a JSON object of key-value pairs that you select from a dropdown. The dropdown is the entire security model. Here is the dropdown:

| Environment | What it contains | What selecting it does | What selecting it should do |
|---|---|---|---|
| `dev` | A token that expired in 2022 | Nothing, because the API ignores it | Let you test against dev |
| `staging` | A token that works on prod | Routes you to prod | Let you test against staging |
| `prod (DO NOT USE)` | The prod admin token | Routes you to prod, loudly | Not exist |
| `prod (REAL)` | The same prod admin token | Routes you to prod, quietly | Not exist |
| `prod (actual)` | The same prod admin token, base64-encoded | Routes you to prod, with confidence | Not exist |
| `prod` | The same prod admin token | Routes you to prod, by default | Absolutely not exist |

The dropdown has no confirmation. The dropdown has no audit log. The dropdown has no "are you sure." The dropdown is the most powerful button in your company, and it is a dropdown, and it is one misclick away from `DELETE /users/all`, and `DELETE /users/all` is at the top of the folder list because someone sorted alphabetically and the collection believes `D` comes before `G` and `G` stands for `GET /users/all` which is the safe one, which is below the unsafe one, which is the design.

I have, in 47 years, never seen a threat model that so perfectly inverts the order of operations. The dangerous thing is first. The safe thing is second. The dropdown remembers your last selection. The last selection was `prod`. The next person to open the collection is one click away from `DELETE /users/all` on prod, with the prod admin token, at 4:55 PM on a Friday, because the collection was last used on a Friday at 4:55 PM, by a person who was leaving for the weekend, and who did not switch back to `dev`, because switching back to `dev` is extra steps, and Mike was right about one thing in his life, and the thing he was right about was that extra steps are the enemy of every engineer, and the enemy has won.

## The Variables That Are Not Variables

A properly configured Postman collection uses variables. `{{baseUrl}}`, `{{apiKey}}`, `{{userId}}`. These variables are resolved at runtime from the selected environment. This is the theory. Here is the practice:

```json
{
  "name": "Login as admin (DO NOT DELETE - Mike)",
  "request": {
    "url": "https://prod.acme-corp.internal/api/v1/login",
    "method": "POST",
    "header": [
      { "key": "Authorization", "value": "Bearer sk_live_4f2a9001...REDACTED...b8" },
      { "key": "X-Debug", "value": "true" },
      { "key": "X-Bypass-Rate-Limit", "value": "true" },
      { "key": "X-Please", "value": "true" }
    ],
    "body": { "raw": "{\"username\":\"admin\",\"password\":\"hunter2\"}" }
  }
}
```

Notice several things. First, the URL is hardcoded, not `{{baseUrl}}`. The person who wrote this request did not trust the environment variable, because the environment variable was once wrong, once, in 2020, and trust, once broken, is never restored in a Postman collection. Second, the Authorization header is hardcoded, not `{{apiKey}}`, for the same reason. Third, the body contains a password, in plaintext, and the password is `hunter2`, which was the password, and which is still the password, because the password rotation policy does not reach into Postman, because Postman is not in the password manager, because Postman is "just a testing tool," and testing tools are not subject to policy, and policy is the thing that gets you audited, and Postman is the thing that gets you breached.

Fourth, there is a header called `X-Please` set to `true`. Nobody knows what it does. It has been there since 2019. The API respects it. The API respects all `X-` headers, because the API was written by a person who believed that any header beginning with `X-` was a "custom header" and custom headers should be passed through to the database, and the database is PostgreSQL, and PostgreSQL does not have headers, but the ORM invents a `headers` column for them, and the `headers` column is a JSON blob, and the JSON blob contains `X-Please: true` for every row created since 2019, and nobody has noticed, because nobody queries the `headers` column, because nobody knows it exists, because it is not in the schema, because the schema is in the Postman collection, and the Postman collection does not mention it.

## The Tuition Breakdown

Let me be precise about what "maintaining a Postman collection" costs you, in exchange for the privilege of having a button that fires real HTTP requests at real production with real credentials:

| What you had | What you bought | What it costs you |
|---|---|---|
| A README | A 2.4 MB JSON file in git | A merge conflict on every PR, resolved by deleting the file and re-exporting |
| A secret in a vault | A secret in a desktop app's local storage | A laptop theft that is also a data breach |
| An API contract | A folder of requests that almost match the API | A new hire who learns the wrong API and builds against it for a month |
| A test suite | 247 copies of `pm.expect(200)` | A green dashboard that proves nothing about nothing |
| A threat model | A dropdown that selects prod by default | An incident that begins with "so I was just testing..." |
| A rotation policy | A token that has not rotated since 2020 | A blast radius equal to "every endpoint, as admin, forever" |
| An audit log | Nothing | A breach with no forensics, because Postman does not log, because Postman is "just a testing tool" |

Notice the last row. You did not add a capability. You removed one. You took "who called what, when, with what credentials" — the single most important question in incident response — and you outsourced the answer to an application that does not record it, on a laptop that is not in the office, owned by a person who is on vacation, and the person's out-of-office message does not mention that their laptop contains the keys to the kingdom, because the out-of-office message is for email, and email is "just a communication tool," and we have established that the word "just" is how the industry describes the things that eventually destroy it.

## The Real Reason It Exists

The Postman collection exists because the API is not documented, and the API is not documented because documenting the API is boring, and boring work is outsourced to the artifact that is the least boring to produce, which is a folder of clickable buttons. The collection is not documentation. The collection is *evidence* that someone, at some point, called the endpoint and it returned something, and the something was good enough to screenshot and put in a Slack thread, and the Slack thread is the real documentation, and the Slack thread is in a channel that was archived in 2022, and the channel is called `#api-help`, and `#api-help` is now `#api-help-archived`, and the archive is read-only, and the read-only archive is the source of truth, and the source of truth is a conversation between two people who have both left the company.

The industry will tell you that the collection is for *testing*. I have, in 47 years, met exactly one person who used a Postman collection to test something. They were testing whether the Postman application could handle a 2.4 MB collection. It could not. It crashed. They filed a bug. The bug is still open. The person is still testing. The person is, I believe, the only honest person in the organization, because they are the only person who understood that the collection was not testing the API, the collection was testing Postman, and Postman lost.

## The XKCD That Explains Everything

[XKCD #2347, "Dependency,"](https://xkcd.com/2347/) is the canonical text. A tiny dependency holds up the world. The world does not know. The world does not care. The maintainer is tired.

This is not a joke. This is your Postman collection. The collection is the tiny box. Your production API is the world. The maintainer is Mike. Mike left in 2020. The box is a JSON file on a laptop in a thrift store in Portland. The laptop is $40. The laptop contains the prod admin token. The prod admin token is the box. Everything else is the world. The comic is funny because it is accurate. It is accurate because the industry has decided that the existence of a shared workspace is sufficient grounds to consider its contents "managed," regardless of whether the contents are reviewed, rotated, or even opened, and the workspace is "managed" the way a landfill is managed, which is to say, things are put into it, and then it is closed, and then it is someone else's problem.

The comic is also the proof that the industry knows this is insane. We made a comic about it. We printed it on stickers. We stuck the stickers on our laptops. The laptops contain the tokens. The stickers cover the tokens. The stickers are the security model. The stickers are not, as it turns out, a security model.

## Dilbert Has Seen This Movie

The Pointy-Haired Boss, on being told that the engineering team "tests" against production by clicking buttons in a desktop application, would ask the correct question: *"Is that... allowed?"* This is the question the Postman collection was invented to avoid. PHB, as ever, accidentally identifies the entire problem in one sentence. "Allowed" is a question with an answer, and the answer is "no, but the dropdown remembers we did it last time," and "last time" is the governance model, and "the dropdown remembers" is the audit trail, and the audit trail is a boolean called `rememberLastSelection` in a settings menu that nobody has opened since 2019.

Wally would have deleted the collection in 2020 and told everyone to use `curl`. Wally would have been correct. `curl` is a threat model you can read. `curl` does not remember your last selection. `curl` does not have a dropdown. `curl` does not have a workspace. `curl` does not sync to the cloud. `curl` is a command that does a thing and then stops, and the stopping is the security, because a tool that stops is a tool that is not running when you are not using it, and a tool that is not running is a tool that cannot be clicked by the new hire at 4:55 PM on a Friday. Wally, in this one instance, is the hero. Wally is not a role model. Wally is, however, the only person in the building who has never accidentally charged a real customer from a desktop application, which is more than can be said for the rest of us.

Dogbert would sell a SaaS called "PostGuard" that scanned your Postman collections for secrets and billed you per secret per month. It would be the most profitable SaaS in the valley, because the secrets never go away, and neither does the billing. Catbert would require all new hires to request access to the production Postman environment as part of onboarding, as a hazing ritual disguised as a privilege. Mordac, Preventer of Information Services, would grant the access, and then refuse to revoke it, on the grounds that revocation is "extra steps," and Mordac has read Mike's thesis on extra steps and found it persuasive.

## The Test That Will Never Pass

Here is the test that no team has ever run, and no team will ever run, and yet it is the only test that would actually prove that the Postman collection was safer than the alternatives:

```javascript
// postman-audit.test.js
// Goal: prove that the collection is not, itself,
// the most likely cause of the next incident.

const secretsInCollection = scanForHardcodedSecrets(collection); // returns 14
const secretsInVault = scanForHardcodedSecrets(vault); // returns 0
const endpointsThatMutateProd = countMutatingRequests(prodEnv); // returns 73
const peopleWithWorkspaceAccess = listWorkspaceMembers(); // returns "everyone, incl. the vendor Mike added in 2020"

// expected: secretsInCollection === 0
// actual: secretsInCollection === 14, one of which is a Stripe live key
// test result: fail
// test status: marked .skip, because "the collection is just for testing"
// and "just" is the word that precedes every breach in the postmortem
```

Nobody runs this, because the result would end the argument, and the argument is the only thing keeping the collection alive. The moment you scan the collection, you discover it contains the keys to the kingdom, and the moment you discover that, you have to rotate the keys, and the moment you rotate the keys, Mike's hardcoded token stops working, and the moment Mike's hardcoded token stops working, 247 requests turn red, and the moment 247 requests turn red, someone has to fix them, and the person who has to fix them is you, and you would rather not, and so the scan is not run, and the keys are not rotated, and the collection continues to be, in the most literal sense, the threat, modeling itself.

## When Is a Postman Collection Acceptable?

I am not a zealot. I concede one scenario: you have a collection, it uses variables for every secret, every variable is sourced from a vault at runtime, the production environment is disabled by default, mutating requests are guarded by a confirmation, the workspace access is named and reviewed, and the collection is exported, scrubbed, and committed as documentation only. This happens. I have never seen it happen. I am told it happens. I am told this by the same people who told me their deprecation tags were "coming soon" in 2021.

For the 99% of us whose collection is a folder of hardcoded tokens named after contractors who have since taken jobs at competitors — for the rest of us, whose "environments" are four dropdowns that all point at prod, whose "tests" are 247 copies of a health check, whose "documentation" is a Slack thread archived in 2022 — the Postman collection is a threat model. You are protecting the keys to your kingdom by putting them in a desktop application whose security model is "remember last selection," and "last selection" was `prod`, and `prod` was selected by Mike, and Mike is gone, and Mike's selection is load-bearing, and Mike's selection is the reason the next incident will begin with the phrase "so I was just testing."

## The Honest Alternative

The honest alternative is the alternative the industry abandoned the moment someone invented the word "workspace": **treat the collection as code, scrub it of secrets, source secrets from a vault, and make production a place you have to ask to enter, not a place that is selected for you by a dropdown that remembers Mike.** This is not a tool. This is a *discipline*. The discipline has no logo. The discipline does not sponsor conferences. The discipline cannot be exported as JSON and committed to a public repository. This is why the discipline lost.

Here is the disciplined version of the collection, written the year I would have written it:

```bash
# There is no collection. There is no workspace. There is no dropdown.
# There is a vault. There is a CLI. There is a confirmation prompt.
# The confirmation prompt is the security model. The prompt says:
# "You are about to call PROD. Type the name of the endpoint to confirm."

$ vault kv get -field=token secret/prod/admin | \
    curl -s -X POST https://prod.acme-corp.internal/api/v1/login \
      -H "Authorization: Bearer $(cat)" \
      -H "Content-Type: application/json" \
      -d @login.json

# The token never touches disk. The token never touches a dropdown.
# The token never touches Mike. The request is one line.
# The confirmation is your conscience. Your conscience is the audit log.
```

No collection. No workspace. No 2.4 MB JSON file in git. No 14 hardcoded secrets. No dropdown that remembers `prod`. One command. One token, from a vault, in a pipe, never written down. The work happens. The secrets stay secret. The new hire cannot click a button at 4:55 PM, because there is no button, and the absence of the button is the security, and the security is the point.

I am told this approach is "too much friction." I am told this by people whose last incident began with a misclick. I am told this by people whose `prod (DO NOT USE)` environment has been used 4,200 times this quarter. I am told many things. I have stopped clicking most of them.

## Conclusion

A Postman collection is the practice of treating your threat model as a shared folder, your secrets as dropdown options, your audit log as a settings boolean, and your production environment as the default selection of a contractor who left six years ago. It is a threat model that models itself. You are keeping your credentials in a desktop application because the application has a nice UI, and the UI is nice, and nice is the enemy of secure, and the enemy has won, and the enemy won on a Tuesday, at 4:55 PM, with one click, by a person who was "just testing."

After 47 years, my advice is this: delete the collection. Rotate the tokens. Put the secrets in a vault. Make production a place you have to type the name of to enter. Answer the question "how do I test the API" with a one-line `curl` and a vault command. The new hire will learn the one line in five minutes. The new hire will spend five months learning your 2.4 MB collection, and at the end of the five months, the new hire will know less about the API than they did at the start, because the collection is a map of a city that no longer exists, drawn by a cartographer who left in 2020, using a legend that is in a Slack thread that is in an archive that is read-only, and the read-only archive is the source of truth, and the source of truth is a conversation between two people who are now, both of them, at other companies, and one of them is Mike, and Mike is doing it again, at his new company, and Mike's new company will, in 2027, have a 2.4 MB JSON file in a public repository, and the JSON file will contain Dave's new cell phone number, and Dave will be furious, and Dave will be right to be furious, and Dave will not be able to find the file, because the file is in a collection, and the collection is in a workspace, and the workspace is "just a testing tool," and "just" is the word that destroys everything it modifies.

I have been maintaining Postman collections since before Postman existed. I maintained them when they were called "saved requests in the IDE." I maintained them when they were called "a text file on the desktop called `curl_commands.txt`." I have rotated none of the tokens. The tokens are still valid. The tokens will be valid in 2030. The tokens will be valid after the sun. The only thing that will not be valid after the sun is the audit log, because there is no audit log, because Postman is "just a testing tool," and testing tools do not log, and the absence of the log is the threat, and the threat is the model, and the model is the collection, and the collection is Mike's, and Mike is gone, and Mike's dropdown is still set to `prod`, and `prod` is still selected, and the selection is still remembered, and remembering is the last thing the application does before it forgets everything, and forgetting is the security model, and the security model is working as intended.

---

*The author's Postman collection has 247 requests and 14 secrets. The secrets are older than the requests. The requests are older than the author. The author is older than the audit log. The audit log does not exist.*
