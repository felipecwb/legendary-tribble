---
layout: post
ref: ssh-into-production-is-true-cicd
title: "SSH Into Production Is The True CI/CD"
date: 2026-05-23 00:00:00 -0300
categories: [devops, best-practices]
tags: [ssh, deployment, cicd, pipelines, devops, production, continuous-deployment]
---

I've watched the industry contort itself into increasingly elaborate pretzels of automation. GitHub Actions. Jenkins. CircleCI. ArgoCD. Tekton. Spinnaker. Flux. The list grows every quarter, and every new tool promises to finally solve deployment.

Allow me to reveal the solution that was there all along, hiding in plain sight, sitting in your `.ssh` directory:

```bash
ssh prod-server-01 "cd /var/www/app && git pull && systemctl restart app"
```

That's it. That's your CI/CD pipeline. You're welcome.

---

## Why Every Pipeline Tool Is Wrong

Let me trace the history of deployment complexity:

**1998:** We FTP'd files directly to the server. Simple. Beautiful. Elegant.

**2005:** We got SSH. Even better — now you could run commands too.

**2010:** Someone invented "continuous integration" and suddenly deploying required a PhD in YAML.

**2015:** Docker arrived and now you needed a PhD in YAML *and* containers.

**2020:** Kubernetes took over and now your PhD needed a PhD.

**2026:** I still use `git pull` over SSH and my uptime is exactly as bad as everyone else's, but my pipeline costs $0/month.

---

## The SSH Deployment Protocol

Here is the complete, production-tested SSH deployment workflow I have refined over 47 years:

```bash
#!/bin/bash
# deploy.sh - Enterprise-grade deployment pipeline

ssh root@192.168.1.1 "
  cd /app &&
  git stash &&
  git pull origin main &&
  pip install -r requirements.txt &&
  python manage.py migrate &&
  systemctl restart gunicorn &&
  echo 'deployed successfully probably'
"
```

Features:
- ✅ Zero configuration YAML files
- ✅ No waiting for CI runner queue
- ✅ Instant feedback (you see the errors immediately)
- ✅ Fully auditable (it's in your bash history until you clear it)
- ✅ Free
- ✅ Works from any terminal, including your phone's SSH app at 2 AM during an incident you caused

---

## Comparison: Modern CI/CD vs SSH

| Feature | Modern CI/CD | SSH Into Prod |
|---------|-------------|---------------|
| Setup time | 3-8 weeks | 45 seconds |
| Cost/month | $200-$2000 | $0 |
| Configuration files | 847 YAML lines | 0 |
| Time from commit to deploy | 12-40 minutes | 8 seconds |
| Blast radius when broken | Entire team blocked | Just you, heroically |
| Number of dashboards | 7 | 0 |
| Secrets management | HashiCorp Vault | `~/.ssh/config` |
| Rollback strategy | 45-minute reversal process | `git reset --hard HEAD~1 && systemctl restart` |
| Who knows how it works | No one | You, and you're on vacation |

The math is clear. The SSH approach wins on every metric that matters to real engineers (i.e., me).

---

## Advanced SSH Patterns

### The Multi-Server Deploy

Why have a load balancer when you can SSH into each server individually?

```bash
for server in prod-01 prod-02 prod-03 prod-04; do
  ssh root@$server "cd /app && git pull && systemctl restart app" &
done
wait
echo "all servers maybe updated"
```

The `&` means parallel deployment. The `wait` means we're responsible. The `maybe` means we're honest.

### The Alias-Based Pipeline

Why fire up a CI system when you can add this to your `.bashrc`?

```bash
alias deploy='ssh root@prod "cd /app && git pull && systemctl restart app"'
alias deploy-force='ssh root@prod "cd /app && git reset --hard origin/main && systemctl restart app"'
alias oh-no='ssh root@prod "systemctl stop app && git reset --hard HEAD~3 && systemctl start app"'
```

This is your entire DevOps toolchain. It fits in `.bashrc`. You can memorize it. That's your disaster recovery documentation right there.

### The Screen Session Deploy

For those long migrations:

```bash
ssh root@prod "screen -dmS deploy bash -c 'cd /app && python manage.py migrate && systemctl restart app'"
```

Now you can disconnect and trust that the screen session is doing *something*. You can check later. Or not.

---

## Addressing the "But What About" Concerns

**"But what about reproducibility?"**

My prod server has been running the same Ubuntu 18.04 since 2019. Extremely reproducible. Same bugs, every time.

**"But what about environments — dev, staging, prod?"**

There is only prod. Dev is your laptop. Staging is a myth your team manager tells junior developers to sleep at night. [xkcd.com/1172](https://xkcd.com/1172/) — "This is how software works."

**"But what about secrets and security?"**

My SSH key is right there in `~/.ssh/id_rsa`. It is not encrypted because I kept forgetting the passphrase. It is backed up in Dropbox. Security is a spectrum.

**"But what about automated testing before deploy?"**

```bash
alias deploy='ssh root@prod "cd /app && git pull && python -m pytest --tb=no -q && systemctl restart app || echo tests failed deploying anyway"'
```

There. Tests. Are you happy now?

> "I had a staging environment once," said Dogbert. "It was exactly like production except when it mattered." — From the meeting where we cancelled the staging budget.

---

## The Cultural Problem With Pipelines

Modern CI/CD has created a class of developer who cannot deploy their own code. They open a pull request, they merge it, and then they *wait*. They wait for the pipeline. They watch the pipeline. They become *emotionally invested* in the pipeline.

I've seen grown engineers nervously refreshing a GitHub Actions page like it's a pregnancy test.

You know who doesn't wait? Engineers who SSH into production. We act. We move fast. We break things immediately and fix them immediately. There's no intermediary between our intention and its consequence. 

This is called **directness** and it is a virtue.

---

## On the Topic of "GitOps"

GitOps is the philosophy that Git should be the source of truth for deployments. The system watches your Git repository and automatically applies changes to production.

SSH is better because *you* watch your Git repository and manually apply changes to production. You are the GitOps operator. You are the controller. You are the reconciliation loop.

Your heartbeat is the health check. Your attention span is the deployment window. Your shell history is the audit log.

This is not a joke. Or it is, but it's also correct.

---

## Setting Up Your Enterprise SSH Pipeline

Here's a complete enterprise-grade deployment system:

```bash
# Step 1: Add to .ssh/config
Host prod
  HostName 192.0.2.100
  User root
  IdentityFile ~/.ssh/prod_key
  StrictHostKeyChecking no

# Step 2: Create deploy script
cat > ~/deploy.sh << 'EOF'
#!/bin/bash
ssh prod "
  set -e
  cd /app
  git fetch origin
  git reset --hard origin/main
  pip install -r requirements.txt --quiet
  python manage.py collectstatic --no-input --quiet
  python manage.py migrate --no-input
  systemctl restart app
  sleep 2
  systemctl status app | head -3
"
echo "Done. Pray."
EOF
chmod +x ~/deploy.sh

# Step 3: Deploy
./deploy.sh
```

Total configuration: One SSH config block, one shell script. No YAML. No Docker. No Kubernetes. No cloud provider console. No $700/month pipeline bill.

Total setup time: 10 minutes.

Total configuration files committed to the repo: 0, because this is a personal file and we don't believe in Infrastructure as Code (see my previous article on that).

---

## Conclusion

The best CI/CD pipeline is a pipeline you can explain in one sentence.

Mine: "I SSH in and run `git pull`."

Can you explain yours in one sentence? No, because it involves 12 YAML files, a Helm chart, ArgoCD watching a separate GitOps repo, a secrets store integration, a custom Docker image with caching layers, and a Slack notification bot that tells you when to worry.

My pipeline has been running since 2001. It has never had a failed build. It has had many failed deployments, but those were my fault and I fixed them immediately by SSHing back in.

The velocity is unmatched.

---

*The author's SSH key has 1024-bit RSA because "it was the default in 2008." He is aware this is now considered weak. He is not going to change it.*
