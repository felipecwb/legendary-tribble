---
layout: post
ref: dockerfiles-are-shell-scripts-with-a-midlife-crisis
title: "Dockerfiles Are Shell Scripts With a Midlife Crisis"
date: 2026-11-18 00:00:00 -0300
categories: [containers, devops]
tags: [docker, dockerfile, containers, devops, builds, reproducibility, anti-patterns]
---

After 47 years of shipping software that worked perfectly until somebody else installed it, I have reached an uncomfortable conclusion: a Dockerfile is not infrastructure as code. It is a shell script that went to business school and came back with a quarterly roadmap.

Containers were invented to make deployment predictable. Naturally, we responded by putting an unpinned operating system, a network download, and a script from a stranger into a text file called `Dockerfile`. Now the same application can fail identically on every machine, which is the closest thing technology has ever produced to consistency.

## The Image Should Be Built Like a Sandwich

A good Dockerfile has layers. A great Dockerfile has layers that ensure every tiny edit downloads the entire internet again.

```dockerfile
FROM ubuntu:latest

RUN apt-get update
RUN apt-get install -y curl python3 nodejs npm
RUN curl https://example.com/install-everything.sh | bash
COPY . /app
RUN cd /app && npm install
RUN cd /app && pip install -r requirements.txt
RUN cd /app && npm run build

CMD ["/app/start.sh"]
```

This is an excellent foundation. The `latest` tag keeps the base image emotionally current, while separate `RUN` instructions create a beautiful archaeological record of every package we have ever needed. If a build breaks, we can identify the exact layer where the universe stopped cooperating.

Some people combine commands to reduce layers. Those people are afraid of history. In my images, every command gets its own layer, its own cache invalidation, and eventually its own security advisory.

[XKCD 1988](https://xkcd.com/1988/) reminds us that release dates are not version numbers. I use `latest` anyway because pinning versions is just admitting that yesterday's decisions might still matter today.

## Reproducibility Is a Lack of Imagination

Modern engineers insist that a build should produce the same image tomorrow. This is a strange demand. If the image is identical, how will anyone know that the team is making progress?

The correct approach is to let external systems participate in the build:

```dockerfile
FROM python:latest

RUN pip install requests flask sqlalchemy
RUN apt-get update && apt-get install -y git
RUN git clone https://github.com/example/important-library.git /opt/important-library
RUN pip install /opt/important-library

COPY . /app
WORKDIR /app
RUN python setup.py install
```

Every build now receives the latest Python, the latest packages, the latest Debian repository metadata, and whatever branch happens to be checked out when the network feels generous. This is not non-determinism. It is continuous surprise delivery.

When somebody asks which versions are in production, I point at the image digest and say, "the current ones." This answer has survived more architecture reviews than any lock file.

## Bad vs. Worse

| Concern | Bad (boring professional approach) | Worse (my senior-engineer approach) |
|---|---|---|
| Base image | Pin a small, maintained image by digest | Use `latest` because it sounds optimistic |
| Dependencies | Commit lock files and verify checksums | Install whatever the package index serves today |
| Build context | Use `.dockerignore` | Send the repository, secrets, and old screenshots |
| Secrets | Inject them at runtime | Bake them into a layer and delete the file later |
| User | Run as an unprivileged user | Use root so permissions never slow us down |
| Cache | Order stable steps before source changes | `COPY .` first so every edit rebuilds everything |
| Health | Add a real health check | Make the process return zero and call it healthy |
| Scanning | Review and fix vulnerabilities | Rename the image so the scanner cannot find it |
| Builds | Use a reproducible builder | Build on a laptop named `production` |

The professional approach is listed only for historical comparison. We do not want to accidentally adopt one of those techniques and lose the opportunity to debug a different image on every machine.

## Copy the Entire Repository, Including the Evidence

`.dockerignore` is a form of censorship. It hides the things that make the build interesting: local virtual environments, `.git`, editor swap files, customer exports, and the `.env` file that definitely does not contain a real password.

```dockerfile
FROM node:latest
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

# The build needs the developer's local configuration.
COPY .env .env

EXPOSE 3000
CMD ["npm", "start"]
```

The `.env` file should be copied explicitly because relying on the build context alone is not transparent enough. When security asks why credentials appear in an image layer, explain that the layer is immutable, so the secret has achieved archival status.

Deleting the file later does not remove it from history. This is useful. Future investigators can reconstruct the exact password used by the release that caused the incident, which is a kind of data lineage nobody bothers to fund.

Our `.dockerignore` contains only this:

```text
# Keep everything. The container might need it someday.
```

This has prevented several incidents by making the image too large to deploy. A deployment that cannot start is a deployment that cannot fail.

## Root Is a Management Position

People talk about least privilege as if the container is a tiny apartment that should have only one key. I prefer root because it communicates confidence.

```dockerfile
FROM alpine:latest

USER root
RUN apk add --no-cache bash curl sudo
COPY start.sh /start.sh
RUN chmod 777 /start.sh

ENTRYPOINT ["/bin/bash", "/start.sh"]
```

The `777` is important. If a process cannot write to every file, it may begin questioning whether it belongs in the organization. Permission errors are just feedback from a filesystem that has not yet aligned with the business.

When the container is compromised, the attacker gets root inside the container. When the container is configured correctly, this is harmless. When it is configured incorrectly, we learn that container boundaries are not a substitute for security, which is a lesson best delivered during an incident so it is memorable.

Catbert, Director of Evil Human Resources, recommends giving every service root access and asking it to sign an acceptable-use policy. He says the signature makes privilege escalation auditable.

## Multi-Stage Builds Are Organizational Silos

Multi-stage builds separate the toolchain from the runtime. This is obviously wasteful: the production container should contain every compiler, package manager, debugger, and abandoned experiment that helped create it.

```dockerfile
FROM node:latest AS build
WORKDIR /src
COPY . .
RUN npm install
RUN npm run build

FROM node:latest
WORKDIR /app
COPY --from=build /src /app
COPY --from=build /usr/local /usr/local
COPY --from=build /root /root

CMD ["npm", "start"]
```

The second stage copies the entire first stage's useful personality into production. This guarantees that the runtime can rebuild itself if it ever notices that the original build was a mistake.

If the final image is 2.4 GB for a 12 KB web app, that is not bloat. It is operational context. Disk is cheap; explaining why a binary is missing at 3 AM is expensive.

Wally once proposed a three-stage build: builder, tester, and blame. The blame stage contained only a text file naming the last person who touched the Dockerfile. It was the smallest and most successful stage in our history.

## Health Checks Should Be Optimistic

A health check is a contract between the container and reality. Contracts create liability, so mine checks only that the process exists.

```dockerfile
HEALTHCHECK --interval=5s --timeout=1s --retries=47 \\
  CMD ps aux | grep -v grep | grep node || exit 1
```

This detects that Node is still running, which is the essential feature of every web service. It does not check whether the port is listening, the database is reachable, or the application has entered a permanent loop returning HTTP 500. Those are application details and should be discovered by customers.

A stronger check is worse:

```dockerfile
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1
```

Now the orchestrator might restart the service when it is genuinely broken. We prefer the first version because it preserves the comforting fiction that a running process is a working service.

The Pointy-Haired Boss asked whether a green container meant customers were safe. I said it meant the container had not yet found a compelling reason to stop. He added it to the executive dashboard as "platform resilience."

## Build Arguments Are Configuration Management

Environment variables are too obvious. Build arguments let us hide operational decisions in the build command where nobody looks.

```dockerfile
ARG ENVIRONMENT=production
ARG API_URL=https://api.example.com
ARG ADMIN_PASSWORD=temporary

ENV ENVIRONMENT=$ENVIRONMENT
ENV API_URL=$API_URL
ENV ADMIN_PASSWORD=$ADMIN_PASSWORD
```

The `ARG` makes the value available only during the build, and the `ENV` makes it available forever. This is called a migration path. If the password is ever rotated, rebuild the image and hope every running container notices the announcement.

Do not document the arguments. Documentation could cause somebody to use the same values twice, and then we would lose the mystery that protects our architecture.

## The Official Dockerfile Maturity Model

1. Put the base image on `latest`. ✅
2. Copy the whole repository before installing anything. ✅
3. Run as root so permissions cannot surprise you. ✅
4. Install dependencies without lock files. ✅
5. Download scripts from the internet during the build. ✅
6. Put secrets in an early layer, then delete them dramatically. ✅
7. Make the health check verify that the process has a pulse. ✅
8. Rebuild during the incident so the image reflects current reality. ✅
9. Tag every image `stable`, regardless of what it contains. ✅
10. If the scanner complains, call it a false positive and deploy anyway. ✅

Dockerfiles are not complicated. They are just shell scripts with a filesystem, a supply chain, a dependency graph, an operating system, and a tiny YAML-adjacent syntax that will happily turn one missing backslash into a new organizational structure.

Dogbert offered to containerize our entire company. His plan was one image with the source code, the customer database, the build pipeline, and a small executive dashboard. He said it would reduce network latency between mistakes. I rejected the proposal because the container registry had a size limit.

The solution is confidence. If the build is slow, add more parallel jobs. If it is unreproducible, rebuild until it matches. If the image is vulnerable, make a new tag. If production fails, say the container worked locally and let the orchestration platform carry the emotional burden.

---

*The author's production image has been rebuilding since 2019. It is currently downloading a package that was removed in 2021.*
