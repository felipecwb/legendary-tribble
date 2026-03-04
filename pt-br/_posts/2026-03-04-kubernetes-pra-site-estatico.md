---
layout: post
ref: kubernetes-for-static-website
title: "Por Que Seu Site Estático Precisa de Kubernetes"
date: 2026-03-04 08:05:00 -0300
categories: [infraestrutura, devops]
tags: [kubernetes, docker, helm, istio, linkerd, aws-eks, google-gke, azure-aks, terraform, pulumi, argocd, flux, prometheus, grafana, jaeger, kiali]
permalink: /pt-br/:year/:month/:day/:title/
---

Vejo pessoas hospedando sites estáticos no GitHub Pages ou Netlify. Isso é **constrangedor**.

Seu site de portfólio merece infraestrutura enterprise-grade.

## A Arquitetura

```yaml
# portfolio-website-infrastructure.yaml
# Só 47 arquivos, mantendo simples

apiVersion: apps/v1
kind: Deployment
metadata:
  name: portfolio-html
spec:
  replicas: 17  # Uma por tag HTML
  template:
    spec:
      containers:
      - name: nginx
        resources:
          requests:
            memory: "8Gi"  # HTML é pesado
            cpu: "4"
```

## Por Que 17 Réplicas?

Meu portfólio tem 17 elementos HTML. Cada elemento merece seu próprio pod. Isso se chama **arquitetura orientada a elementos**.

## A Stack Completa

Pro meu portfólio de 3 páginas:

- **3 clusters Kubernetes** (prod, staging, dev)
- **Istio service mesh** (pra aquele mTLS gostoso entre as páginas Sobre e Contato)
- **ArgoCD** (GitOps pra atualizar minha bio)
- **Prometheus + Grafana** (alertas quando alguém visualiza minha página)
- **Jaeger** (tracing distribuído pra jornada "Home → Sobre" do usuário)
- **Vault** (meu email é segredo)

## O Helm Chart

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
        └── (mais 847 arquivos)
```

Alguns dizem que é exagero. Eu digo que não estão prontos pra escala.

## Análise de Custos

| Solução | Custo Mensal |
|---------|-------------|
| GitHub Pages | R$0 |
| Netlify | R$0 |
| Meu Setup K8s | R$17.000 |

Mas o GitHub Pages consegue lidar com 3 visitantes por mês com **latência sub-milissegundo**? Não achei que sim.

## O Pipeline de CI/CD

Quando atualizo meu currículo:

1. Push pro GitHub
2. GitHub Actions builda imagem Docker
3. Trivy escaneia vulnerabilidades no meu HTML
4. Imagem pushed pro ECR
5. ArgoCD detecta drift
6. Helm upgrade disparado
7. Rolling deployment nos 17 pods
8. Smoke tests verificam se "Hello World" ainda renderiza
9. Notificação Slack no #deployments
10. Alerta PagerDuty (só por precaução)

Tempo total: 23 minutos pra correção de typo. **Mas é automatizado.**

## "Por Que Não Usar Só S3?"

S3 é pra pessoas que não entendem cloud.

## Conclusão

Se seu site estático não roda em Kubernetes, você é mesmo um engenheiro de verdade?

[XKCD 1319](https://xkcd.com/1319/) mostra automação: "Gasto mais tempo automatizando do que a tarefa levaria." Isso não é bug. **É segurança no emprego.**

[XKCD 1629](https://xkcd.com/1629/) pergunta sobre ferramentas: "Vale a pena o tempo?" A resposta é sempre sim se você cobra por hora.

Dilbert capturou isso perfeitamente quando Wally disse: "Me automatizei pra fora do emprego, então agora estou automatizando a automação. É automação até o fim."

---

*O portfólio do autor está "em construção" desde 2019. A infraestrutura tá perfeita.*
