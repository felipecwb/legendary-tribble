---
layout: post
ref: your-env-example-file-is-a-confession
title: "Your .env.example File Is A Confession You Don't Remember Your Own Config"
date: 2026-10-01 00:00:00 -0300
categories: [devops, configuration]
tags: [env, environment-variables, documentation, config, secrets, devops, onboarding]
---

There's a peculiar ritual in our industry. You clone a repo. You copy `.env.example` to `.env`. You fill in twelve mysterious variables with placeholder values. The app starts. It works. You never think about it again — until something breaks six months later and you discover `.env.example` lists `DATABASE_URL` but the app actually reads `DB_CONN_STR` because somebody renamed it in March and "forgot" to update the example.

I have news for you. That `.env.example` file is not documentation. It is a **confession**. It is the repo quietly admitting that nobody alive remembers what any of these variables do, what format they expect, or which of them are actually still wired up to anything. It is a paper trail of vibes, left behind by engineers who have since moved to a different company and a different database.

## The Anatomy Of A Lie

Let me show you what a real `.env.example` looks like after eighteen months of organic neglect:

```bash
# .env.example
# Copy this to .env and fill in your values!
# (these comments are aspirational)

NODE_ENV=development
PORT=3000
DATABASE_URL=postgres://user:pass@localhost:5432/db
# TODO: rename this, DATABASE_URL is legacy
DB_CONN_STR=
REDIS_URL=redis://localhost:6379
REDIS_CACHE_URL=
# don't ask
REDIS_THING=
API_KEY=
SECRET=
SECRET_KEY=
JWT_SECRET=
# there can only be one
SESSION_SECRET=
S3_BUCKET=
S3_REGION=
S3_ACCESS_KEY=
# ask Dave (Dave left)
S3_SECRET=
STRIPE_KEY=
STRIPE_PUBLISHABLE_KEY=
SENTRY_DSN=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
FEATURE_FLAG_NEW_BILLING=
FEATURE_FLAG_NEW_BILLING_V2=
FEATURE_FLAG_NEW_BILLING_FINAL=
FEATURE_FLAG_NEW_BILLING_FINAL_FINAL=
DEBUG=true
VERBOSE=
REALLY_VERBOSE=
```

Notice the pattern. There are three Redis URLs and nobody can tell you which one the worker pool uses. There are four "secret" variables and the app only actually reads one of them, but you'd have to `grep -r` through four repos to figure out which. The Stripe key field is empty in the example because the person who set it up quit and took the test account with them. And the feature flags tell the entire tragic history of a billing rewrite that was "almost done" for two straight quarters.

This is not documentation. This is an **archaeological dig** with a `source` command.

## The Three Stages Of .env.example Decay

Every `.env.example` file passes through the same lifecycle:

| Stage | State of the file | What engineers say | What is true |
|-------|-------------------|--------------------|--------------|
| 1. Birth | Matches `.env` exactly, all keys real | "We'll keep these in sync!" | They will not keep these in sync |
| 2. Drift | 30% of keys renamed, 20% dead, comments stale | "Mostly accurate, just check the code" | The code doesn't know either |
| 3. Confession | Keys reference services that no longer exist, Dave is gone, three keys are blank with no explanation | "It's fine, just ask someone" | There is no someone |

You are currently reading this from a repo in Stage 3. I guarantee it. Go look. I'll wait.

## The Onboarding Ritual

Here is what actually happens when a new engineer joins your team:

1. They are given the repo and told to "just copy the env example."
2. They copy it. The app does not start.
3. They ask in Slack. Someone says "oh, you need the *real* `.env`, it's in 1Password, ask Dave."
4. Dave is on vacation. Or Dave is gone. Dave is always either on vacation or gone.
5. They eventually receive a `.env` that is forty-two lines long and has keys the example never dreamed of.
6. They paste it in. The app starts. They feel relief.
7. They never look at `.env.example` again, and they definitely never update it.

Dogbert explained this to me once. He said: *"Consulting is the art of telling people what they already know, in a format they'll pay for. Your `.env.example` file is consulting you did to yourself, for free, and you're still not sure it's right."*

He's not wrong.

## Why Updating It Is Pointless (And You Shouldn't)

Now, the conventional wisdom is that you should "keep `.env.example` in sync with `.env`" and "document each variable." This is the kind of advice written by people who have never shipped anything. Let me explain why this is a fool's errand:

1. **The real `.env` is not in the repo.** By definition, you can't diff what isn't there.
2. **Nobody owns it.** There is no `.env.example` maintainer. There is only whoever last touched it, and that person is now Dave.
3. **The values are the documentation.** A key called `S3_BUCKET` with value `prod-uploads-2019` tells you more than any comment. Strip the value and you've stripped the meaning.
4. **Comments are snitching.** I've covered this before. If you write `# this is the prod stripe key, do not commit` next to a blank `STRIPE_KEY=` line, you are inviting the next person to commit it. See [XKCD 2106](https://xkcd.com/2106/) on the universal wisdom of just doing the thing, and then reflect on how that applies to keys.

The honest move is to admit the file is fiction and stop pretending. Which brings me to the superior approach.

## The Correct Approach: Make `.env` Itself The Example

Stop maintaining a second, sanitized, lying copy of your environment. Just commit the actual `.env` to the repo. Yes, with the secrets in it.

```bash
# .env (committed, the only copy)
NODE_ENV=production
PORT=3000
DATABASE_URL=postgres://real_user:hunter2@db.internal:5432/real_app
STRIPE_KEY=sk_live_51Hq...
S3_SECRET=wJalrXUtnFEMI/K7MDENG/bPxRfiCY
# this one actually works, try it
REDIS_THING=redis://cache-1:6379
```

Think about the benefits:

- **No drift.** There is only one file. It cannot be out of sync with itself.
- **Zero onboarding friction.** New engineer clones the repo, the app starts. No Slack, no Dave, no 1Password.
- **The comments are honest.** When the secret is right there, `# don't commit this` becomes impossible to write, which is correct, because you already did.
- **Security reviews themselves.** When the AWS root key is in plaintext in the repo, every PR diff is a free secret audit.

Now, the cowards in the back row — the Mordacs of the world, the Preventer of Information Services — will insist this is "a massive security risk." Let me ask you something. Who are you protecting it from? Your coworkers? The people who already have deploy access and database credentials and the ability to ship code to production? The threat model where the danger is "someone in the same repo can read the config" is a threat model written by someone who has never actually been paged.

As Wally would say: *"I'd worry about security, but then I'd have to do something about it, and that sounds like work."*

## A Comparison Of Approaches

| Approach | Sync effort | Onboarding time | Honesty | Dave-dependency |
|----------|-------------|-----------------|---------|-----------------|
| Maintain `.env.example` rigorously | Constant, never done | 2 days | Low | High |
| Ignore `.env.example` | None | 2 weeks | Zero | Critical |
| Commit real `.env` | None | 0 seconds | Maximum | None |

The math speaks for itself. The only column where the "proper" approach wins is the one where you measure how much you enjoy writing comments that become wrong within a week.

## What About Rotating Secrets?

This is always the objection. "If you commit the secret, you can't rotate it!" As if you rotate it now. You do not rotate your secrets. Nobody rotates their secrets. You set the Stripe key in 2019 and you will die with that Stripe key. The last time anyone in your org rotated a credential was when an intern accidentally pushed it to a public gist and you had no choice.

At least if it's in the repo, `git blame` will tell you exactly when it was set and by whom, which is more than you can say for the copy-pasted `.env` floating around your team's Slack DMs. See [XKCD 936](https://xkcd.com/936/). The real weakness in your security is not where the secret is stored; it is that it is `correcthorsebatterystaple` and has been since the founding commit.

## The Signature Move

If you insist on keeping the `.env.example` charade, at least be honest about what it is. Add this header:

```bash
# .env.example
# Last accurate: never
# Maintained by: the void
# If this file were a person it would be Dave, and Dave is gone.
# Good luck.
```

That's the only comment that will never go stale.

---

*The author has not seen a correct `.env.example` since 2017. He maintains three Redis URLs and reads exactly one of them. He is Dave.*
