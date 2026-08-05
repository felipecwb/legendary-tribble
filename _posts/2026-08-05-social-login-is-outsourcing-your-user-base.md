---
layout: post
ref: social-login-is-outsourcing-your-user-base
title: "\"Sign in with Google\" Is Outsourcing Your Entire User Base"
date: 2026-08-05 00:00:00 -0300
categories: [security, authentication, culture]
tags: [oauth, social-login, authentication, passwords, google, facebook, vendor-lock-in, identity, reliance, juniors]
---

For 47 years I have been the one thing standing between software and working software. So when a junior came to me last week, proud as punch, and said "I added Sign in with Google, it only took two lines," I did what any senior engineer would do. I closed his laptop, gently, like closing the lid on a coffin.

"Two lines," he said. "Two lines and our users can log in."

Son, let me explain something about those two lines. One of them is a `<script>` tag to a file you do not control, hosted on a server you cannot ssh into, governed by a privacy policy written by lawyers who do not know you exist. The other line is a callback that hands the crown jewels of your business — *who your users are* — to a company whose KPIs do not include your survival. You did not add authentication. You added a **landlord**.

## The User Table Is The Company

I will say it slowly, because the juniors in the back are already opening their OAuth provider's dashboard instead of listening.

The user table is the company.

Not the code. Not the product. Not the pitch deck. The *user table*. The list of human beings who, at some point, looked at your thing and decided "yes, I would like a relationship with this." That table is the only asset a software company has that cannot be recompiled. It is, in the literal sense, the business.

And "Sign in with Google" gives it away. For free. To a company that already has your search history, your location, your calendar, your email, and a photograph of your dog. You are not "offloading" authentication. You are **outsourcing the list of people who owe you money**, to the only entity on Earth with more data than you.

Here is the deal you signed, in two lines:

```javascript
// auth.js — "two lines" (the second one is 412, actually)
import { GoogleLogin } from '@react-oauth/google';

<GoogleLogin
  onSuccess={credentialResponse => {
    // Send this token to your backend.
    // Your backend will call Google to find out who logged in.
    // Google will tell you, and you will believe it.
    // You will never actually know.
    storeUserFromGoogle(credentialResponse.credential);
  }}
  onError={() => { /* TODO: handle this. nobody will. */ }}
/>
```

Read the comment on the fourth line. Your backend *asks a third party who your user is*. You have a database. You have a server. You have a name on an invoice. And yet, to answer the question "is this the person who used my software?", you outsource the answer to a company that does not answer your support tickets. This is not authentication. This is a **hot potato**.

## The Day The Landlord Changes The Locks

Every quarter, the OAuth provider sends an email. The subject line is always the same: "Important changes to your [Product] API access." The body is always the same: the thing you relied on is going away, or it now costs money, or it now requires a verification process so long it has its own backlog.

You will read this email on a Friday. Because they always send it on a Friday.

And on that Friday you will discover that:

- The `email` scope is now behind a *verified* badge you do not have.
- The `profile` scope returns a name that is *no longer guaranteed unique*.
- The `sub` claim you were using as your primary key *is still unique*, but only within the same project, and your project was migrated last year and nobody told you.
- The redirect URI you registered in 2019 now has to end in a slash, and it didn't, and now 40% of your logins 404.

I have watched this happen. I have watched a 12-person company spend two sprints replacing a Google login because Google decided "your app looks like a personal project, please verify your organization." They verified the organization. Google took six weeks. The verification form asked for their Articles of Incorporation. They sent them. Google asked again. They sent them again. Google asked in a different font.

[https://xkcd.com/2347/](https://xkcd.com/2347/) is titled *"Dependency"* and it shows a massive pyramid of blocks, and at the very bottom, holding up the entire civilization, is one small block labeled "a project some random person in Nebraska has been thanklessly maintaining since 2003." That is you, now. You are the pyramid. Google is the random person in Nebraska, except Google is not in Nebraska, is not thankless, and will invoice you.

## The Four Ways To Know Who A User Is (Ranked By Spine)

| Approach | Who actually knows your users | What happens when the provider has a bad day | Spine rating |
|---|---|---|---|
| Email + password you store yourself | You | You have a bad day, which is honest | 🟢 Vertebrate |
| Sign in with Google | Google, mostly | Your users have a bad day and blame you | 🟡 Invertebrate |
| Sign in with Google + Facebook + GitHub | Three companies, none of them you | You have a bad day *and* a merge conflict | 🔴 Jellyfish |
| "Passwordless" magic link email | Your email provider, via you | The email provider has a bad day, you all starve | ⚫ Plankton |

The spine column trends in exactly one direction, and it is the one in which you personally can answer the question "who is my user?" without opening a browser tab.

## Mordac Has A Field Day

The thing that kills me — the thing that *truly* kills me — is that the same engineers who will spend three weeks arguing about whether to use `Map` or `Object` will hand the entire concept of *identity* to a third party in two lines of JSX. Identity. The thing that decides whether a person can see their own data. The thing that, if it is wrong, is a lawsuit.

Mordac, Preventer of Information Services, would weep with joy. For 30 years Mordac has been trying to convince people that security is someone else's problem. "Don't worry about it," he would say. "A certified vendor will handle the login." And the engineers would nod, because Mordac is wearing a tie and the vendor has a logo with a lowercase letter in it, and lowercase letters mean *trust*.

Dogbert figured this out in 1994. "Consulting," he said, "is the art of telling people what they already know, but with a pie chart." Sign in with Google is consulting for authentication: you already know who your user is, but you'd rather pay someone with a pie chart to tell you.

## But What About Password Security?

Here is where the juniors get brave. "But senior," they say, already typing, "if I store passwords, I'm a target. If I store passwords, I have to hash them. If I store passwords, someone might leak them."

Yes. Welcome to running a business.

If you cannot hash a password, you cannot run a database. If you cannot run a database, you cannot run a product. If you cannot run a product, you should not have a user table to begin with, which, conveniently, you no longer do, because you gave it to Google.

```python
# The entire password system you are allegedly afraid of.
import hashlib, os

def register(email, password):
    salt = os.urandom(16)
    h = hashlib.scrypt(password.encode(), salt=salt, n=2**14, r=8, p=1, dklen=32)
    db.execute(
        "INSERT INTO users (email, salt, hash) VALUES (?, ?, ?)",
        (email, salt, h)
    )

def login(email, password):
    salt, expected = db.execute("SELECT salt, hash FROM users WHERE email=?", (email,))
    h = hashlib.scrypt(password.encode(), salt=salt, n=2**14, r=8, p=1, dklen=32)
    return h == expected  # that's it. that's the whole scary thing.
```

That is the entire boogeyman. Twelve lines. A salt. A slow hash. A database insert. You have written more code than this to center a `<div>`. And the reward for those twelve lines is that *you* know who your users are, *you* decide when they can log in, and *you* are the only party whose bad day can ruin their day, which at least makes the blame linear and the fix local.

## The Iron Law Of Identity

I will close with the one thing I have learned in 47 years of this, which is more than the entire history of OAuth and also more than the entire history of Google, which I mention because it is relevant:

**Whoever holds the user table owns the business.**

If you hold it, you own it. If Google holds it, Google owns it, and you are a contractor who does not know they are a contractor. You will discover this the day Google's verification form rejects your Articles of Incorporation for the third time, in a different font, on a Friday.

So the next time a junior tells me they added authentication in two lines, I will nod, smile, and hand them the invoice from the OAuth provider, and ask them, gently, to find the line item labeled "our entire user base, hosted by someone who returns none of our calls." It is the only honest line on the page.

---

*The author has been authenticating against his own database since 1996. He has never been verified by Google. Google has never asked.*
