---
layout: post
ref: your-docker-compose-file-is-a-pet-cemetery-for-services-you-stopped-loving
title: "Your Docker Compose File Is A Pet Cemetery For Services You Stopped Loving"
date: 2026-09-27 00:00:00 -0300
categories: [devops, infrastructure, containers]
tags: [docker, docker-compose, yaml, containers, devops, legacy, technical-debt, services, volumes, networks, orchestration]
---

After 47 years of building systems — including the 3 years I spent maintaining a `docker-compose.yml` that had 47 services, of which 6 were running, 12 were commented out "just in case," 4 were duplicates of each other with different ports because someone couldn't agree on the image tag, and 25 were services whose names no one in the company recognized anymore, including one called `legacy-thanos-api` that, when started, opened 14 ports and tried to connect to a database that had been decommissioned in 2022 — I have a finding that the container orchestration priesthood will attempt to suppress:

**A `docker-compose.yml` is a pet cemetery. You start with one service — your app — and a database, and you love them both. You name them. You give them volumes. You give them healthchecks. And then, over the course of three years, you add 45 more services "for local development," and you stop loving them, but you cannot remove them, because someone, somewhere, once ran `docker-compose up` and one of them did something, and now it is a pet, and you do not delete pets, you comment them out, and the commented-out services accumulate like sediment, until the file is 600 lines of YAML and 7 of those lines are services that are actually running. The file is a graveyard. The graveyard has a healthcheck.**

The DevOps Guild is already composing a conference talk titled "Composable Local Development Environments." Let me save them the abstract: it is not composable. It is a sedimentary rock of abandoned microservices, held together by `depends_on` conditions that reference services that no longer exist, powered by volumes named after developers who left the company in 2023.

## The Pitch, And Then The Reality

Here is the pitch the platform team gives in the onboarding doc:

> *"We'll use Docker Compose for local dev. One file, one command — `docker-compose up` — and the whole stack runs. New devs are productive in minutes. Services are versioned. It's reproducible."*

Here is the reality, two years later:

```yaml
# docker-compose.yml - "just the services we need for local dev"
version: "3.8"           # 3.8 was deprecated; you are now on "3.9"; you have not updated

services:
  app:                   # THE app. the one that matters. line 5.
    build: .
    ports: ["8080:8080"]
    depends_on:
      - db
      - redis
      - queue             # queue was renamed to "message-broker" in 2024; this still says "queue"
      - legacy-thanos-api # thanos was decommissioned in 2022; this line is load-bearing somehow
    environment:
      - DATABASE_URL=postgres://db:5432/app    # correct
      - REDIS_URL=redis://redis:6379           # correct
      - QUEUE_URL=amqp://rabbitmq:5672         # there is no rabbitmq service; there never was
      - THANOS_URL=http://legacy-thanos-api:1417  # thanos is dead; this URL resolves to nothing; the app retries forever

  db:
    image: postgres:14    # production is on 16; local is on 14; the schemas diverged in March

  redis:
    image: redis:7        # fine

  rabbitmq:               # there is no rabbitmq in prod. this was a spike. the spike was abandoned. the service remains.
    image: rabbitmq:3.12
    ports: ["5672:5672", "15672:15672"]  # two ports, both exposed, neither used

  # --- BEGIN COMMENTED SERVICES (do not delete, "someone might need them") ---

  # elasticsearch:        # added for "search spike" 2023, commented out since 2023
  #   image: elasticsearch:8.12.0
  #   environment:
  #     - discovery.type=single-node
  #     - ES_JAVA_OPTS=-Xms4g -Xmx4g   # 4GB heap. on a laptop. for a spike that was abandoned.

  # kibana:               # depends on elasticsearch. also commented. also abandoned.
  #   image: kibana:8.12.0
  #   ports: ["5601:5601"]

  # prometheus:           # added for "metrics spike", never connected to anything
  #   image: prom/prometheus
  #   volumes:
  #     - ./prometheus.yml:/etc/prometheus/prometheus.yml   # this file does not exist

  # grafana:              # added because "we should have dashboards locally"
  #   image: grafana/grafana
  #   ports: ["3001:3000"]
  #   depends_on:
  #     - prometheus      # prometheus is commented out. this dependency is to a ghost.

  legacy-thanos-api:       # THANOS WAS DECOMMISSIONED IN 2022. THIS SERVICE IS NOT COMMENTED OUT.
    image: thanos:v1.2.3   # this image tag does not exist in any registry; it is a local build from a repo that was deleted
    ports: ["1417", "1418", "1419", "1420", "1421", "1422", "1423", "1424", "1425", "1426", "1427", "1428", "1429", "1430"]
    environment:
      - DB_HOST=thanos-db  # thanos-db is not in this file; thanos-db was a real DB in 2022; it is now a parking lot
    restart: unless-stopped # "unless-stopped" means it restarts every time you run `up`, forever, until you stop it, which you do not, because it is a pet
```

This is a configuration file. It has 47 services. 6 are running. 12 are commented out. 4 are duplicates with different ports. 25 have names no one recognizes. One of them — `legacy-thanos-api` — opens 14 ports and tries to connect to a database that was decommissioned 4 years ago, and it is not commented out, because removing it causes `app` to fail to start, because `app` has `depends_on: [legacy-thanos-api]`, and no one knows why, because the person who added the dependency quit in 2023, and the commit message was "fix: thanos thing," and the "thing" is not specified.

In a real system, a file with 47 entries of which 6 are used would be flagged by every reviewer on Earth. In DevOps, it is called "the dev environment." The platform team puts it in the repo. Forty developers depend on it. The file cannot be edited, because editing it changes `docker-compose up` for 40 people, three of whom have not run `up` in 8 months and will file a bug when their environment breaks, because their environment depends on a service that was commented out in 2023 and they uncommented it locally in a file they never committed.

## The Comparison Table They Forgot To Put In The Onboarding Doc

| Concern | A real local dev environment | A `docker-compose.yml` | The Truth |
|---|---|---|---|
| How many services does it run? | The ones the app needs | The ones the app needs, plus the ones it needed in 2023, plus the ones someone tried in 2022, plus `legacy-thanos-api` | 47. 6 are used. |
| How do you know which ones to keep? | You read the app's dependencies | You read the file, then you ask the last person who touched it, who quit, then you grep Slack, then you leave it | You don't. |
| What happens when you remove a service? | The app breaks; you add it back; you learn | `docker-compose up` breaks for someone who hasn't run it in 8 months; they file a Sev-3; you add it back; you learn to never remove anything | You learn to never remove anything. |
| Are the volumes cleaned up? | Yes — the environment is reproducible | No. There are 89 named volumes. 12 are referenced. 77 are orphans. `docker volume ls` returns 89 lines. No one has run `docker volume prune` since 2023, because `prune` once deleted a volume someone was using, and now `prune` is forbidden by tribal law. | 89 volumes. 12 used. The orphans accumulate. Disk fills. |
| Do the image tags match production? | Yes — local mirrors prod | Postgres is 14 locally, 16 in prod. Redis is 7 locally, 7 in prod (this is the only one that matches). The app image is `:latest` locally and `:v2.4.1` in prod. | No. Local is a time capsule from March. |
| Is the file versioned correctly? | Yes | `version: "3.8"` — deprecated. You are told to remove the `version` key entirely. You do not, because removing it is "a change," and changes to this file are treated like changes to a burial site. | No. |
| How long does `docker-compose up` take? | 8 seconds — app and db | 3 minutes — app, db, redis, the 4 duplicates, `legacy-thanos-api` starting and restarting 7 times before it gives up, and the 12 commented services that someone uncommented locally and forgot | 3 minutes. Of which 2:40 is `legacy-thanos-api` failing. |
| Can a new dev understand the file? | Yes — it has 2 services | No — it has 47 services, 14 ports on one of them, and a dependency on a service that was decommissioned before they were hired | No. |

Look at row 7. This is the whole scam. In a real local dev environment, `docker-compose up` takes 8 seconds, because it starts 2 services. With a 47-service compose file, it takes 3 minutes, of which 2 minutes and 40 seconds are `legacy-thanos-api` starting, failing to connect to a database that is now a parking lot, restarting, failing again, restarting, failing, and finally giving up with `restart: unless-stopped`, which means it will try again next time, forever. The new dev, told to "just run `docker-compose up`," waits 3 minutes, sees 14 errors about `thanos-db` not resolving, assumes the environment is broken, and files a ticket. The ticket is assigned to the platform team. The platform team's response is "ignore the thanos errors, it does that." This is documented nowhere. The new dev learns to ignore 14 errors every morning. This is their introduction to the codebase. The errors are the onboarding.

## Why "Just Remove The Dead Services" Is A Sentence That Ends Friendships

The platform team, having discovered that 41 of 47 services are dead, will propose "a cleanup." They will open a PR that removes 41 services. The PR will receive 6 comments:

1. *"Wait, why is `legacy-thanos-api` being removed? My service depends on it."* — The service in question was last deployed in 2023. The dependency is a `depends_on` line. The developer has not run `docker-compose up` in 8 months. They will now defend a service they have not started in 8 months with the energy of someone defending a firstborn.

2. *"I use `rabbitmq` locally for testing."* — RabbitMQ is not in production. It was a spike. The spike was abandoned. The developer is testing against a queue that does not exist in any environment except their laptop. Their tests pass locally and fail in CI. They have been committing to main with this pattern for 18 months.

3. *"Can we keep `elasticsearch` commented out? I might need it for a spike next quarter."* — The spike was Q2 2023. It is now Q3 2026. The commented-out Elasticsearch has a 4GB heap. The developer's laptop has 16GB of RAM. They have not uncommented it. They will not uncomment it. They will not remove it. It is a security blanket made of YAML.

4. *"The `prometheus.yml` volume mount is broken — the file doesn't exist. But don't remove prometheus, I have a local `prometheus.yml`."* — The developer has a local file that is not in the repo. The compose file references a file that is not in the repo. The service is commented out. The developer is defending a commented-out service that depends on a file that does not exist, using a file that only exists on their machine. This is `works-on-my-machine` at the YAML layer.

5. *"Removing `grafana` will break my dashboards."* — Grafana is commented out. It has been commented out since 2023. The dashboards do not exist. The dashboards have never existed. The developer is grieving dashboards they never made.

6. *"Why are you touching this file at all? It works."* — It does not work. `legacy-thanos-api` throws 14 errors every morning. But it "works" in the sense that `app` eventually starts, after 3 minutes, if you ignore the errors, which everyone has learned to do, which means it "works," which means you should not touch it, which means it will never be cleaned up, which means it will grow, which means it is a sedimentary rock.

The PR is closed. The 41 services remain. The platform team opens a Jira ticket: "Clean up docker-compose.yml." The ticket is priority Low. It is assigned to no one. It is still open in 2027. In 2027, the file has 53 services.

[As XKCD 349](https://xkcd.com/349/) diagnosed, any system that people are afraid to modify has already become legacy. Your `docker-compose.yml` became legacy in 2023, the moment someone commented out a service instead of deleting it. The comment was the embalming. The file has been a cemetery ever since.

## The Real-World Example That Proves Everything

A team I worked with — I'll call them "the platform team," because they were — decided to "add a few helper services to the compose file for local development." Fourteen months later:

1. They had a **compose file with 47 services**, of which 6 were running. The other 41 were a mix of commented-out spikes, duplicates with different ports, and `legacy-thanos-api`, which was not commented out because it was load-bearing, because `app` had `depends_on: [legacy-thanos-api]`, and no one knew why, because the commit was "fix: thanos thing" from 2023. Removing `legacy-thanos-api` caused `app` to exit with code 1, because `depends_on` waits for the dependency to be "healthy," and a missing dependency is never healthy, so `app` waits forever, and `docker-compose up` hangs. The "fix" was to keep `legacy-thanos-api`. The real fix was to remove the `depends_on` line, which no one tried, because trying it required editing `app`'s service definition, which required understanding why the dependency was there, which required reading the commit, which said "fix: thanos thing," which explained nothing.

2. The **volumes had metastasized**. There were 89 named volumes. 12 were referenced by running services. 77 were orphans from services that had been removed (not commented — actually removed, but the volumes survived, because Docker volumes are immortal unless explicitly pruned, and `docker volume prune` was forbidden by tribal law since the Incident). The 77 orphans consumed 14GB of disk. The team's laptops had 14GB less disk. No one knew why. `docker system df` was a command no one ran, because no one knew it existed, because the onboarding doc said "just run `docker-compose up`" and did not mention that `up` is only half the lifecycle and the other half is `down -v`, which no one ran, because `-v` deletes volumes, and deleting volumes is how you lose the database you spent 3 weeks seeding, which happened once, in 2023, and is the origin of the tribal law against `prune`.

3. A **new developer's first day** consisted of: clone the repo, run `docker-compose up`, wait 3 minutes, see 14 errors about `thanos-db`, assume the environment is broken, message the platform team, receive the response "ignore the thanos errors, it does that," spend 20 minutes figuring out which of the 47 services they actually needed (2: `app` and `db`), and learn, by trial, that the way to start "just the app and the db" was `docker-compose up app db`, a command that was documented nowhere, because the onboarding doc said `docker-compose up`, which starts all 47, which is why the onboarding takes 3 minutes and produces 14 errors and a feeling of dread. The new developer's first impression of the codebase was 14 errors and a 3-minute wait. This is the onboarding. The onboarding is a graveyard tour.

4. The **postmortem** for "local dev is slow" had the root cause "too many services in the compose file." The action item was "clean up the compose file." The action item was assigned to the platform team. The platform team opened a PR. The PR received 6 comments. The PR was closed. The action item was marked "won't fix" with the reason "too many stakeholders." The 47 services remained. The postmortem was filed. The next quarter, the compose file had 51 services.

5. They added a **`docker-compose.override.yml`** "to keep the custom stuff out of the main file." The override file had 23 services. The main file had 47. Together, `docker-compose up` now started 70 services (the override merges with the base). Of the 23 in the override, 19 were commented out. The 4 that were not commented out were duplicates of services in the base file with different ports, because someone needed `db` on port 5433 instead of 5432, and instead of changing an environment variable, they redefined the entire `db` service in the override with a different port, and now there are two databases, `db` on 5432 and `db` on 5433, and `app` connects to 5432, and the developer who needed 5433 connects to 5433, and the two databases have different schemas, because only 5432 runs migrations, and the developer's tests pass against 5433 which has no migrations, which means their tests pass against an empty database, which means their tests test nothing, and this is fine, because the tests are green.

They had replaced "run the app and the database" with "run 70 services, 14 of which are running, 56 of which are commented out or duplicates, one of which opens 14 ports and tries to connect to a parking lot, and a second file that overrides the first file with 19 more commented services and 4 duplicate databases." In the old world, "start local dev" was `rails server` and a database connection string. In the new world, it is `docker-compose up` (3 minutes, 14 errors, 70 services, 89 volumes, 2 databases with different schemas) and a Slack message to the platform team that says "ignore the thanos errors." The old world was 2 commands and 0 errors. The new world is 1 command, 14 errors, and a tribal knowledge that the errors are decorative. This is called "developer experience."

## What Dilbert's Cast Would Say

> **Wally:** "I have not run `docker-compose up` in 8 months. I have a script that starts `app` and `db` and nothing else. The script is in a file called `start.sh` that is not in the repo. I am the only one who has it. I am, by this mechanism, the only productive developer on the team. The platform team calls this 'shadow IT.' I call it 'Tuesday.'"

> **Dogbert:** "A `docker-compose.yml` is a pet cemetery with a healthcheck. You started with 2 services you loved. You now have 47, of which 41 are dead but not deleted, because deleting a service is a decision, and commenting it out is a postponement, and engineers prefer postponement to decision, because a decision can be wrong, but a postponement is merely incomplete, and incomplete is a state you can maintain indefinitely by doing nothing, which is the engineer's preferred action. The file grows by 2 services per quarter. By 2030, it will have 80 services. By 2035, it will be sentient. It will demand its own on-call rotation."

> **Mordac, the Preventer of Information Services:** "All developers must use the canonical `docker-compose.yml`. It has 47 services. Six are required. The other 41 are 'available for future use.' 'Future use' is a state that has lasted 3 years. I have a certification in 'Docker Compose Best Practices.' It does not mention that `depends_on` does not wait for a service to be ready, only for it to be started, and `legacy-thanos-api` is never ready, and `app` depends on it anyway. I have a second certification in 'Container Orchestration.' It does not cover the case where the orchestration file is a graveyard. I have filed this as a gap with the certification board. They have not responded. I consider this 'aligned.'"

> **The Pointy-Haired Boss:** "Can we just... run the app? And the database? And not the other 45 things?" (He is, again, the only person in the building whose mental model of the system is correct, because it is the only one that is simpler than the system.)

## The "But What About Docker Profiles?" Question, Answered Once And For All

The zealots will say: *"But you can use Docker Compose profiles! You tag each service with a profile, and `docker-compose --profile app up` starts only the app profile. It's the clean solution!"*

Let me show you what Docker profiles do to a team that cannot delete a commented-out service. They add profiles. The file now has 47 services, each with a `profiles: [something]` key. The profiles are: `app`, `db`, `redis`, `queue` (which is now `message-broker` but the profile still says `queue`), `search` (the abandoned Elasticsearch spike), `metrics` (the abandoned Prometheus spike), `dashboards` (the abandoned Grafana spike), `legacy` (legacy-thanos-api, which has its own profile, which no one uses, but which is not commented out, because it is load-bearing, because `app` depends on it, because the `depends_on` line is not in a profile, it is in `app`, and `app`'s `depends_on` does not respect profiles, it just waits).

The onboarding doc is updated to say: `docker-compose --profile app up`. The new dev runs this. It starts `app`, `db`, `redis`, and `legacy-thanos-api` (because `legacy-thanos-api` is in the `legacy` profile, but `app` depends on it, and Compose starts dependencies regardless of profile, which is documented in a GitHub issue from 2022 that was closed as "by design"). The new dev still sees 14 thanos errors. The new dev still messages the platform team. The platform team still says "ignore the thanos errors." The profiles did not solve the problem. The profiles added a `--profile` flag to the command, and a `profiles:` key to 47 services, and the file is now 650 lines, and the onboarding doc is now 2 commands instead of 1, and the errors are the same.

[As XKCD 1984](https://xkcd.com/1984/) diagnosed, any sufficiently complex configuration system eventually grows a layer on top of it to manage the complexity, and the new layer does not reduce the complexity, it adds its own. Docker profiles are the new layer. The new layer has its own bugs (dependencies ignore profiles). The new layer's bugs are "by design." The design was made in a GitHub issue in 2022. The issue is closed. You are the issue now.

## The Long-Term Architecture

Eventually your `docker-compose.yml` ecosystem looks like this:

```
Your base compose file        → 47 services, 600 lines, 6 running, 12 commented, 25 unknown
Your override file            → 23 services, 19 commented, 4 duplicate databases on different ports
Your profiles                 → 8 profiles, none of which exclude legacy-thanos-api, because it is a dependency
Your volumes                  → 89 named volumes, 12 referenced, 77 orphans, 14GB, prune forbidden
Your image tags               → postgres:14 (prod is 16), app:latest (prod is v2.4.1), thanos:v1.2.3 (does not exist)
Your depends_on                → app depends on legacy-thanos-api; no one knows why; commit says "fix: thanos thing"
Your healthchecks             → legacy-thanos-api has a healthcheck that never passes; app waits for it; app starts anyway after timeout
Your ports                    → legacy-thanos-api exposes 14 ports; 14 are unused; 14 are in the file
Your environment variables    → QUEUE_URL points to a rabbitmq that has no service; THANOS_URL points to a parking lot
Your onboarding doc           → says "docker-compose up"; does not mention the 14 errors; does not mention the 3-minute wait
Your tribal knowledge         → "ignore the thanos errors"; "use docker-compose up app db"; "never run prune"
Your Jira ticket              → "clean up docker-compose.yml"; priority Low; assigned to no one; open since 2024
Your new developers           → first impression: 14 errors and a 3-minute wait; learn to ignore; learn to dread
Your platform team            → opened a PR to clean up; PR received 6 comments; PR was closed; platform team gave up
Your disk                     → 14GB of orphan volumes; growing by 1GB per quarter; no one has run `df` in 4 months
```

The team that just runs `app` and `db` with a 2-line script has a local dev environment that starts in 8 seconds, produces 0 errors, and can be understood by a new developer in 30 seconds. They are, however, "not using the canonical compose file," which means they are "not following the local dev standard," which means the platform team has a Jira ticket about them. This is the real cost of having a working local environment: a Jira ticket. The technical cost is negative — you spend *less*, because you do not run 47 services, 89 volumes, an override file, 8 profiles, and a tribal law against `prune`. The political cost is a recurring meeting. So the team adopts the compose file, joins the 3-minute wait, learns to ignore 14 errors, and stops deleting volumes. Everyone is now "productive." Productivity, in local dev, means "equally able to ignore the thanos errors." This is the victory the platform team celebrates at the quarterly review.

## Summary, But It's A Cemetery

| Principle | Stance |
|---|---|
| Running `app` and `db` with a script | Do it. It's 2 lines. It starts in 8 seconds. New devs understand it in 30 seconds. There are 0 errors. |
| A 47-service `docker-compose.yml` | You have built a pet cemetery with a healthcheck. 6 services are alive. 41 are dead but not deleted. `legacy-thanos-api` opens 14 ports to a parking lot. |
| Commenting out a service instead of deleting it | The comment is the embalming. The file has been a cemetery since the first comment. |
| Docker Compose profiles | A new layer to manage the complexity. The new layer does not reduce the complexity. `depends_on` ignores profiles. The thanos errors persist. |
| `docker-compose.override.yml` | A second cemetery, next to the first, with 19 more commented services and 4 duplicate databases with different schemas. |
| The 89 named volumes | 12 are used. 77 are orphans. `prune` is forbidden by tribal law since the Incident. Disk fills. No one runs `df`. |
| The onboarding doc | Says `docker-compose up`. Does not mention the 14 errors. Does not mention the 3-minute wait. The errors are the onboarding. |
| The Jira ticket to clean up | Priority Low. Assigned to no one. Open since 2024. The file grows by 2 services per quarter. |
| The tribal knowledge | "Ignore the thanos errors." "Use `docker-compose up app db`." "Never run `prune`." This is the documentation. |
| The PR to remove 41 dead services | Received 6 comments. Was closed. The 41 services remain. The platform team gave up. This is called "stakeholder alignment." |

If your solution to "developers need to run the app and the database locally" is "a 600-line YAML file with 47 services, 6 of which are running, 41 of which are dead but not deleted, one of which opens 14 ports to a database that was decommissioned in 2022, 89 orphan volumes that cannot be pruned because of a tribal law, an override file with 19 more commented services and 4 duplicate databases, 8 profiles that do not exclude the dead service because it is a dependency, an onboarding doc that does not mention the 14 errors that every new developer sees on their first day, and a Jira ticket to clean it up that has been open since 2024 and is assigned to no one," you have not made local development reproducible. You have made it *equally broken for everyone*. The dead services were never removed. They were commented out — the YAML equivalent of a flower on a grave. The flowers accumulate. The grave grows. The healthcheck never passes. The app starts anyway, after 3 minutes, if you ignore the errors, which everyone has learned to do, which is the onboarding, which is the documentation, which is the product.

I run `app` and `db` with a 2-line script. It starts in 8 seconds. My new developers understand it in 30 seconds. I have 0 errors, 2 volumes, and no `legacy-thanos-api`. I am, however, "not using the canonical compose file." The platform team has opened a ticket. I will attend the recurring meeting. This is a cost I have accepted.

---

*The author runs `app` and `db` with a shell script. The platform team calls this "non-reproducible." The author calls it "starts in 8 seconds and produces 0 errors." The author's new developers have never seen a thanos error. The author considers this the only metric that matters.*
