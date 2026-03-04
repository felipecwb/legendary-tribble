---
layout: post
ref: kubernetes-for-static-website
title: "Why Your Static Website Needs Kubernetes"
date: 2026-03-04 08:00:00 -0300
categories: [infrastructure, devops]
tags: [kubernetes, docker, helm, istio, linkerd, aws-eks, google-gke, azure-aks, terraform, pulumi, argocd, flux, prometheus, grafana, jaeger, kiali]
---

I see people hosting static websites on GitHub Pages or Netlify. This is **embarrassing**.

Your portfolio website deserves enterprise-grade infrastructure.

## The Architecture

```yaml
# portfolio-website-infrastructure.yaml
# Only 47 files, keeping it simple

apiVersion: apps/v1
kind: Deployment
metadata:
  name: portfolio-html
spec:
  replicas: 17  # One per HTML tag
  template:
    spec:
      containers:
      - name: nginx
        resources:
          requests:
            memory: "8Gi"  # HTML is heavy
            cpu: "4"
```

## Why 17 Replicas?

My portfolio has 17 HTML elements. Each element deserves its own pod. This is called **element-oriented architecture**.

## The Full Stack

For my 3-page portfolio:

- **3 Kubernetes clusters** (prod, staging, dev)
- **Istio service mesh** (for that sweet mTLS between About and Contact pages)
- **ArgoCD** (GitOps for updating my bio)
- **Prometheus + Grafana** (alerting when someone views my page)
- **Jaeger** (distributed tracing for the "Home → About" user journey)
- **Vault** (my email address is a secret)

## The Helm Chart

```
portfolio-chart/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
├── values-prod.yaml
├── values-prod-us-east-1.yaml
├── values-prod-eu-west-1.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── hpa.yaml
│   ├── pdb.yaml
│   ├── networkpolicy.yaml
│   └── servicemonitor.yaml
└── charts/
    └── common/
        └── (847 more files)
```

Some say this is overkill. I say they're not ready for scale.

## Cost Analysis

| Solution | Monthly Cost |
|----------|-------------|
| GitHub Pages | $0 |
| Netlify | $0 |
| My K8s Setup | $3,400 |

But can GitHub Pages handle 3 visitors per month with **sub-millisecond latency**? Didn't think so.

## The CI/CD Pipeline

When I update my resume:

1. Push to GitHub
2. GitHub Actions builds Docker image
3. Trivy scans for vulnerabilities in my HTML
4. Image pushed to ECR
5. ArgoCD detects drift
6. Helm upgrade triggered
7. Rolling deployment across 17 pods
8. Smoke tests verify "Hello World" still renders
9. Slack notification to #deployments
10. PagerDuty alert (just in case)

Total time: 23 minutes for a typo fix. **But it's automated.**

## "Why Not Just Use S3?"

S3 is for people who don't understand the cloud.

## Conclusion

If your static website isn't running on Kubernetes, are you even a real engineer?

[XKCD 1319](https://xkcd.com/1319/) shows automation: "I spend more time automating than the task would take." This is not a bug. **This is job security.**

[XKCD 1629](https://xkcd.com/1629/) asks about tools: "Is it worth the time?" The answer is always yes if you're billing hourly.

Dilbert captured this perfectly when Wally said: "I've automated myself out of a job, so now I'm automating the automation. It's automation all the way down."

---

*The author's portfolio has been "under construction" since 2019. The infrastructure is perfect though.*
