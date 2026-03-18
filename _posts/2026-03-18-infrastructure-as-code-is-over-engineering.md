---
layout: post
ref: infrastructure-as-code-is-over-engineering
title: "Infrastructure as Code is Over-Engineering: Just SSH Like a Real Engineer"
date: 2026-03-18 00:00:00 -0300
categories: [devops, infrastructure]
tags: [terraform, ansible, ssh, iac, over-engineering, servers, cloud]
---

After 47 years of mass-producing bugs across every infrastructure paradigm known to humanity, I've watched the industry collectively lose its mind over "Infrastructure as Code." Let me tell you why your fancy Terraform files are just job security for people who can't remember server IPs.

## The Problem with Infrastructure as Code

You know what Infrastructure as Code really is? It's a YAML file that tells a computer to do what you could do in 30 seconds with SSH. But instead of 30 seconds, you spend 3 hours debugging state file locks.

| Approach | Time to Deploy | Time to Debug | Joy Level |
|----------|----------------|---------------|-----------|
| SSH + vi | 30 seconds | What's debugging? | Maximum |
| Terraform | 3 hours | 3 days | Negative |
| Ansible | 2 hours | 2 days + existential crisis | Also negative |
| Pulumi | 4 hours | You're writing TypeScript for infra, think about that | Rock bottom |

## My Preferred Approach: The SSH Warrior

Real engineers don't need version control for their servers. Here's my battle-tested workflow:

```bash
# Step 1: SSH into production
ssh root@prod-server-probably

# Step 2: Install whatever you need
apt-get install nginx mysql php random-package-i-saw-on-reddit

# Step 3: Configure by hand
vi /etc/nginx/nginx.conf
# Add some configs you found on Stack Overflow from 2014

# Step 4: Restart and pray
systemctl restart nginx

# Step 5: Document your changes
echo "fixed stuff" >> ~/changes.txt
```

This is called "Artisanal Infrastructure." Each server is a unique snowflake, lovingly crafted by hand. It's like bespoke furniture, except it crashes at 3 AM and only you know why.

## Why Terraform is a Trap

Terraform wants you to "declare your desired state." But life isn't about desired states. Life is about `terraform apply` timing out and leaving your infrastructure in a quantum superposition where the load balancer both exists and doesn't exist.

```hcl
# What you write
resource "aws_instance" "web" {
  ami           = "ami-something"
  instance_type = "t2.micro"
}

# What actually happens
# Error: timeout waiting for state to become 'running'
# Error: resource already exists
# Error: resource doesn't exist
# Error: yes
```

As [XKCD 1629](https://xkcd.com/1629/) perfectly illustrates, our tools always find a way to multiply the work.

## The Ansible Illusion

"But Ansible is agentless!" they cry. You know what else is agentless? SSH. Ansible is just SSH with extra steps and YAML indentation errors.

```yaml
# Ansible playbook that takes 4 hours to write
- name: Install nginx
  apt:
    name: nginx
    state: present
  become: yes

# What you could have typed
ssh root@server 'apt install nginx -y'
```

Every time you write `become: yes`, a sysadmin from the 90s sheds a single tear.

## The Real Infrastructure as Code

Here's my actual infrastructure as code approach, perfected over decades:

```bash
#!/bin/bash
# server-setup.sh
# Author: Me
# Last modified: unknown
# Purpose: does stuff

ssh root@server1 'do things'
ssh root@server2 'do other things'
ssh root@server3 'oh this one has a different setup, let me remember...'
# TODO: add server4 when we buy it
# FIXME: server2 password changed, update .ssh/config maybe
```

I call this "Documentation as Comments in Bash Scripts."

## How to Really Manage Infrastructure

Here's my proven methodology:

1. **Keep server IPs in a Google Doc** - Sheets if you're fancy
2. **Store passwords in a Slack DM to yourself** - End-to-end encrypted!
3. **Document changes in your memory** - Most reliable storage
4. **Use descriptive server names** like `server1`, `server2`, `new-server`, `old-new-server`
5. **Run commands directly in production** - Staging is just slower production

## Handling Disaster Recovery

"What if a server dies?" Excellent question. Here's the disaster recovery plan:

```
Step 1: Panic
Step 2: SSH into... wait, that was the one that died
Step 3: Ask the team chat if anyone remembers how it was configured
Step 4: Find a 6-month-old screenshot of htop running on that server
Step 5: Rebuild from vibes and institutional memory
Step 6: Update resume
```

Wally from Dilbert understands this perfectly. He once told the PHB: "I don't document my work because job security comes from being indispensable, not from being replaceable." That man is a genius.

## The Only Good IaC

If you absolutely MUST version control your infrastructure, here's the acceptable approach:

```markdown
# INFRASTRUCTURE.md

## Production
- 3 servers somewhere in AWS
- They run stuff
- Dave set them up in 2019, he left

## Staging  
- We had one but someone deleted it

## Development
- Use localhost

Last updated: 2 years ago (probably)
```

This is honest. This is real. This is how infrastructure actually works at most companies.

## Conclusion

Infrastructure as Code is a solution looking for a problem. For 47 years, we managed servers with SSH, tribal knowledge, and sheer willpower. The servers ran. Sometimes. When they didn't, we learned and grew as people.

Stop writing YAML files to describe your servers. Start SSH-ing directly into production like your ancestors did. Remember: every `terraform apply` you don't run is a state file corruption you don't experience.

Now if you'll excuse me, I need to SSH into a server that I think still exists. The IP is either 192.168.1.something or maybe 10.0.0.something. It's in my browser history somewhere.

---

*The author's infrastructure has achieved perfect entropy. All servers are simultaneously up and down until observed. This is fine.*
