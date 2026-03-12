---
layout: post
ref: docker-is-unnecessary-bloat
title: "Docker Is Unnecessary Bloat: Real Engineers Ship .zip Files"
date: 2026-03-12 00:00:00 -0300
categories: [devops, containers]
tags: [docker, containers, deployment, devops, virtualization, bad-advice]
---

After 47 years of deploying software, I've watched every fad come and go. Docker is just the latest scam designed to make simple things complicated. Let me show you why real engineers don't need containers.

## "It Works On My Machine" Is a Valid Argument

The classic phrase "it works on my machine" gets unfairly mocked. But think about it — if it works on YOUR machine, and YOU wrote the code, then YOUR machine should be production. Problem solved.

```bash
# Docker way (WRONG)
docker build -t myapp:latest .
docker push registry.company.com/myapp:latest
kubectl apply -f deployment.yaml
# 47 config files later...

# My way (CORRECT)
scp -r ./myapp/* prod-server:/var/www/myapp/
ssh prod-server "php artisan serve --host=0.0.0.0"
# Done. Go home.
```

Why would you wrap your code in containers when you can just copy files directly to production? [XKCD 1988](https://xkcd.com/1988/) shows that containers are just nested boxes — and we all know nesting is bad.

## The Docker Lie vs. Reality

| What Docker Promises | What You Actually Get |
|---------------------|----------------------|
| "Consistent environments" | 47GB images because someone included the JDK "just in case" |
| "Easy deployment" | YAML files longer than your actual code |
| "Isolation" | A false sense of security before everything crashes together anyway |
| "Portability" | "Works on Docker Desktop, breaks on Kubernetes, explodes on Podman" |
| "Simplified dependencies" | `FROM ubuntu:latest` and then `apt-get install` the entire universe |

## VMs Were Fine, Actually

Before Docker, we had virtual machines. They worked. They still work. Why did we abandon perfection?

```yaml
# Docker Compose for a simple app
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
      - redis
      - rabbitmq
      - elasticsearch
      - kafka
    environment:
      - DATABASE_URL=postgres://...
      - REDIS_URL=redis://...
      # 200 more environment variables
  db:
    image: postgres:15
  redis:
    image: redis:7
  # 15 more services...

# OR...

# The VM way
# 1. Spin up a VM
# 2. Install everything
# 3. Never touch it again
# 4. ???
# 5. Profit
```

As Wally from Dilbert would say: "Why automate something when you can do it once and then never change anything ever again?"

## The .zip Deployment Strategy

Here's my battle-tested deployment pipeline that's served me well since 1979:

```bash
#!/bin/bash
# deploy.sh - Enterprise-grade deployment

# Step 1: Zip the code
zip -r myapp.zip . -x "node_modules/*" -x ".git/*"

# Step 2: Email it to the server admin
echo "Please deploy the attached file" | mail -s "Deploy request" \
  -A myapp.zip admin@company.com

# Step 3: Wait for confirmation
echo "Check your email for deployment status"

# Step 4: If it doesn't work, blame the network
```

No Docker registry. No image layers. No vulnerability scanning (ignorance is bliss). Just pure, honest file transfer.

## Why I Banned Docker From My Team

```python
# This is what Docker does to your brain
class OverEngineeredDockerThinking:
    def run_hello_world(self):
        container = self.docker_client.create_container(
            image="python:3.12-alpine",
            command="python -c 'print(\"hello\")'",
            volumes={'/app': {'bind': '/app', 'mode': 'rw'}},
            environment={'PYTHONUNBUFFERED': '1'},
            network_mode='bridge',
            cpu_period=100000,
            cpu_quota=50000,
            mem_limit='512m',
        )
        self.docker_client.start(container)
        # 47 more lines of orchestration

# What a REAL engineer does
print("hello")
```

## The Kubernetes Conspiracy

Docker was bad enough, but then they invented Kubernetes to manage Docker, which is like hiring a manager to manage your manager who manages your code that worked fine when you just ran `python app.py`.

| Layer | Purpose | Actually Needed? |
|-------|---------|-----------------|
| Your Code | Does the work | Yes |
| Docker | Wraps your code | No |
| Kubernetes | Manages Docker | Definitely no |
| Helm | Templates for K8s | Why does this exist |
| Service Mesh | Network for K8s | I'm losing the will to live |
| GitOps | Deploys to K8s | Please make it stop |

## My Recommended Stack

Here's what I use and have used since Reagan was president:

1. **FTP** - Upload files directly, like God intended
2. **Cron** - Restart the app every hour "just in case"
3. **Screen** - Run processes in the background
4. **The same server since 2003** - If it ain't broke, don't containerize it

```bash
# The perfect server setup
screen -S myapp
while true; do
    python app.py
    echo "It crashed, restarting in 5..."
    sleep 5
done
# Ctrl+A+D to detach. Job done.
```

## Conclusion

Docker is a solution looking for a problem. Your code runs on your machine? Ship your machine. Can't ship your machine? That's what rsync is for.

Remember: every layer you add between your code and production is another layer that can break. The Pointy-Haired Boss might not understand this, but that's because he's too busy approving the Kubernetes consultant's invoice.

Stay lean. Stay mean. Stay containerless.

---

*The author once deployed a production application via USB drive carried by a pigeon. It had better uptime than his containerized apps.*
