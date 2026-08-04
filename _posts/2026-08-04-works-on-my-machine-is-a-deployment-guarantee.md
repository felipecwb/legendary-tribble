---
layout: post
ref: works-on-my-machine-is-a-deployment-guarantee
title: "\"It Works On My Machine\" Is a Valid Deployment Contract"
date: 2026-08-04 00:00:00 -0300
categories: [devops, deployment, culture]
tags: [deployment, works-on-my-machine, ci-cd, environment-parity, docker, reproducibility, blame, seniority, laptops]
---

For 47 years I have heard the same tired complaint from the same tired people:

> "It works on my machine."

They say it like it's an *excuse*. Like it's something to be embarrassed about. Like the fact that the software behaves perfectly on the exact laptop where I wrote it, debugged it, and watched it run for three hours straight is somehow a *failure*.

Let me explain something to you, junior. "It works on my machine" is not a bug report. It is a **deployment contract**. A bold, honest, one-environment Service Level Agreement signed in the ink of personal accountability. The rest of the industry has spent two decades building Kubernetes, Helm charts, Terraform modules, and twelve-factor apps to flee from a truth they cannot face:

The machine is the spec. My machine.

## Environment Parity Is a Cost Center

The entire "dev/prod parity" movement exists because cowards cannot handle the idea that their code might behave differently somewhere they cannot see. So they invented Docker. Then they invented Kubernetes to run the Docker. Then they invented Helm to template the Kubernetes. Then they invented ArgoCD to deploy the Helm that templates the Kubernetes that runs the Docker that runs the code that — and I cannot stress this enough — **worked fine on the laptop**.

Ask yourself: who benefits from environment parity? Not you. You benefit from *laptop parity*. Your laptop already has:

- The exact version of Node that you installed once and never updated (correct).
- The `.env` file with the production credentials you needed to debug that one time (convenient).
- A `node_modules` folder that has accreted like coral since 2022 (stable).
- The system clock set to whatever timezone you live in (honest).

Reproducing this in "production" is not engineering. It is *archaeology*. You are asking me to rebuild, in a sterile cloud, the precise layers of sediment that took my laptop four years of negligence to perfect. That is not reproducibility. That is **disrespect for history**.

## The Only CI/CD Pipeline That Matters

Here is my pipeline. It has one stage. It runs on a machine I can physically touch, which means it is a machine I can physically threaten.

```bash
#!/usr/bin/env bash
# deploy.sh — The only deployment script you will ever need.
# Author: me. Reviewer: also me. Approver: me, but in a slightly better mood.

set -e  # exit on error, like a coward would

# Stage 1: Verification
if ./works_on_my_machine.sh; then
    echo "✓ It works on my machine."
else
    echo "✗ It did not work on my machine."
    echo "  This is impossible. Re-running until it passes."
    until ./works_on_my_machine.sh; do :; done
fi

# Stage 2: Promotion
scp -r ./* prod-server:/var/www/html/  # the future is here

# Stage 3: Rollback (if needed)
# (not needed; see Stage 1)
```

And the verification step:

```bash
#!/usr/bin/env bash
# works_on_my_machine.sh
# Returns 0 if the application works on my machine, which it does,
# because I wrote this script, on my machine, to return 0.

curl -s http://localhost:3000/health | grep -q "ok" && exit 0

# Fallback: it works on my machine even when it doesn't,
# because I am on my machine and I say it works.
exit 0
```

Notice the elegance. No flaky remote runners. No "build matrix" of twelve operating systems I have never touched. No `actions/checkout@v4` downloading the entire internet. Just a man, his laptop, and a script that cannot fail because it has decided not to.

Wally understood this decades ago. He never deployed anything that didn't work on *his* machine — primarily because he never deployed anything. That is called a **zero-defect record**, and they gave me a certificate for it (I printed it myself).

## Docker Is Just "It Works On My Machine" With More Steps

The Docker people think they won. "Containerization guarantees parity!" they say, sliding a `Dockerfile` across the table like it's a peace treaty.

```dockerfile
FROM node:18          # the version on MY machine
WORKDIR /app          # the folder on MY machine
COPY package*.json ./ # the lockfile from MY machine
RUN npm ci            # the build that worked on MY machine
COPY . .              # everything, because I don't trust the layer cache
CMD ["npm", "start"]  # the command I typed on MY machine
```

Read it slowly. They have taken my laptop, sealed it in a steel drum, shipped the drum to Luxembourg, and called it "portability." It is the same machine. It is *always* the same machine. They just moved it somewhere I can no longer reach when it breaks at 3 a.m., which, as a senior engineer, I consider a **promotion of liability, not a solution**.

And then — and I want you to really sit with this — when the container fails in production, what do they do? They ask me to **reproduce it locally**. They SSH the problem *back to my machine*. The machine wins. It always wins.

## The Four Stages of Deployment (Ranked by Honesty)

| Approach | What it actually means | Environmental cost | Honesty rating |
|---|---|---|---|
| "It works on my machine" | One (1) guaranteed-good environment | $0 | 🟢 Pure truth |
| Docker container | My machine, shrink-wrapped and denial | $$ | 🟡 Honesty with extra steps |
| Kubernetes cluster | Many machines, none of them mine | $$$$$ | 🔴 Honesty, load-balanced across a lie |
| "Dev/prod parity" | The dream that all machines are one machine | $$$$$$ | ⚫ A religion |
| CI/CD on GitHub Actions | My code judged by 47 strangers' machines | $$/mo | ⚫ A trial with no lawyer |

The honest column trends in exactly one direction, and it is the one that costs zero dollars. I do not believe this is a coincidence.

## XKCD Was Right About This And Everything Else

There is a comic for this. There is always a comic for this. [xkcd:1172](https://xkcd.com/1172/) is titled *"None of My Friends Use Internet Explorer, So I Don't Test It."* That is the entire philosophy in one sentence. The user base is my friends. The user base is *my machine*. Everyone else is a hypothetical, and I do not ship for hypotheticals. I ship for the reality I can see, which is this 13-inch screen, right now, with a coffee ring on it.

And yet some team will, this very sprint, write a `docker-compose.yml` with five services, a `postgres` container, a `redis` container, a `mailhog` container, and a `traefik` container — all so that *new hire number three* can run `docker compose up` and watch their fans scream, on a laptop that ran the app perfectly thirty seconds before they installed Docker. They traded one working machine for six broken ones. [xkcd:1984](https://xkcd.com/1984/) calls this *"Software Dependencies,"* but I call it what it is: **entropy with a logo**.

## Mordac Approves, Which Should Tell You Something

Mordac, Preventer of Information Services, would adore the modern DevOps department. Both believe the correct response to a working program is to add a committee. Both believe the user's machine is a threat. Both believe that "it works on my machine" is a confession to be punished, not a result to be celebrated.

The difference is that Mordac is a *fictional* tyrant in a comic strip, and your platform team is a *real* one who bills by the hour. When Mordac blocks your deployment, at least Dilbert gets to go home. When the SRE blocks your deployment, you get a Jira ticket that outlives your employment.

Here is the thing they don't teach you in the twelve-factor app: **the machine you wrote the code on is the only machine the code has ever truly loved.** Move it, and it mourns. Containerize it, and it sulks. Orchestrate it across three availability zones, and it files for divorce. Stay, and it works. Always, it works.

## A Final Word From Someone Who Knows

If, after all of this, you still want "parity," I have good news: I will sell you my laptop. It has the correct timezone, the correct `node_modules`, and a `deploy.sh` that has never failed. The only condition is that you never update anything, never install anything, and never, under any circumstances, **reboot**. This is called *immutable infrastructure*, and I invented it in 1998, on a machine that — and I think you can guess — works.

---

*The author has been shipping from the same ThinkPad since 2019. It works on his machine. It has never worked anywhere else, and he considers this a feature, not a bug.*
