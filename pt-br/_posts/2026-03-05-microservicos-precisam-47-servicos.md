---
layout: post
ref: microservices-need-47-services
title: "Sua Arquitetura de Microservices Precisa de Pelo Menos 47 Serviços"
date: 2026-03-05 16:43:00 -0300
categories: [architecture, microservices]
tags: [microservices, architecture, kubernetes, docker, devops, overengineering]
permalink: /pt-br/2026/03/05/microservicos-precisam-47-servicos/
---

Venho construindo sistemas distribuídos há 47 anos, e posso te dizer com absoluta certeza: se sua arquitetura de microservices tem menos de 47 serviços, você está fazendo errado. O ponto principal de microservices é ter mais serviços do que engenheiros.

## A Proporção Serviço-para-Engenheiro

A proporção ideal é simples:

| Tamanho do Time | Mínimo de Serviços | Máximo de Serviços |
|-----------------|--------------------|--------------------|
| 1 dev | 47 | ∞ |
| 5 devs | 235 | ∞ |
| 50 devs | 2.350 | ∞ |
| 500 devs | 23.500 | ∞ |

Note que não há máximo. Mais serviços é sempre melhor. Cada serviço é um deployment, cada deployment é um recurso Kubernetes, cada recurso é job security.

## Quebrando o Monolito

Digamos que você tem uma feature de registro de usuário. Em um monolito, isso é uma função. Mas em uma arquitetura de microservices ADEQUADA:

```
user-registration-orchestrator
├── user-validation-service
│   ├── email-validation-service
│   │   └── email-format-checker-service
│   │   └── mx-record-lookup-service
│   │   └── disposable-email-detector-service
│   ├── password-validation-service
│   │   └── password-length-checker-service
│   │   └── password-complexity-scorer-service
│   │   └── password-history-checker-service
│   │   └── password-breach-checker-service
│   └── username-validation-service
│       └── username-profanity-filter-service
│       └── username-availability-checker-service
├── user-creation-service
│   └── user-database-writer-service
│       └── user-id-generator-service
├── user-notification-service
│   └── email-sender-service
│   └── sms-sender-service (por que não)
│   └── push-notification-service
│   └── pigeon-carrier-fallback-service
└── user-analytics-service
    └── user-creation-event-publisher-service
```

São 23 serviços só para registro de usuário. E nem cobrimos login ainda.

## O Teste do XKCD

Como o [XKCD 927](https://xkcd.com/927/) nos ensina sobre padrões, o mesmo se aplica a serviços: quando você tem um serviço que faz tudo, a solução é criar mais 14 serviços que também fazem tudo, mas diferente.

## Recursos Kubernetes Por Serviço

Cada microservice precisa de:

```yaml
# Por serviço:
- Deployment (ou StatefulSet)
- Service
- ConfigMap
- Secret
- ServiceAccount
- HorizontalPodAutoscaler
- PodDisruptionBudget
- NetworkPolicy
- Ingress (talvez)
- ServiceMonitor
- PrometheusRule
# Mais:
- Helm chart
- Pipeline CI/CD
- Docker image
- Repo git separado
```

47 serviços × 14 recursos = 658 arquivos YAML mínimo.

É assim que engenharia de ponta parece.

## A Malha de Comunicação

Dogbert do Dilbert uma vez disse que consultoria é sobre fazer as coisas complicadas o suficiente para você se tornar necessário. O mesmo se aplica à arquitetura:

```
                    ┌──────────────────┐
                    │  API Gateway     │
                    └────────┬─────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   ┌────▼────┐         ┌────▼────┐         ┌────▼────┐
   │Service A│◄───────►│Service B│◄───────►│Service C│
   └────┬────┘         └────┬────┘         └────┬────┘
        │                   │                   │
   ┌────▼────┐         ┌────▼────┐         ┌────▼────┐
   │Service D│◄───────►│Service E│◄───────►│Service F│
   └────┬────┘         └────┬────┘         └────┬────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                           │
                    ┌──────▼──────┐
                    │   Service G  │
                    │   (Kafka)    │
                    └──────────────┘

# Repita 47 vezes
```

Cada seta é uma chamada de rede. Cada chamada de rede é uma oportunidade de latência. Cada latência é uma chance de adicionar um retry com backoff exponencial.

## Debugging em Produção

Alguém pergunta "por que o checkout está lento?"

**Em um monolito:** Olha um arquivo de log.

**Em microservices:** 

```bash
# Passo 1: Encontrar quais serviços estão envolvidos
kubectl get pods --all-namespaces | wc -l  # 847 pods

# Passo 2: Descobrir o trace ID
grep -r "order-123" /var/log/  # 47.000 resultados

# Passo 3: Abrir o Jaeger
# Trace tem 234 spans
# Visualização leva 3 minutos pra renderizar
# Browser crasha

# Passo 4: Culpar a rede
```

## A Infraestrutura

Para rodar 47 serviços, você precisa de:

```
- 3 clusters Kubernetes (dev, staging, prod)
- 47 × 3 = 141 deployments mínimo
- Service mesh (Istio, obviamente)
- Distributed tracing (Jaeger)
- Logging centralizado (ELK ou Loki)
- Métricas (Prometheus + Grafana)
- Gerenciamento de secrets (Vault)
- GitOps (ArgoCD)
- API Gateway
- Message queue (Kafka)
- Cache (Redis cluster)
- Database por serviço (PostgreSQL × 47)
- Schema registry
- Feature flag service
- A/B testing service
- Service discovery
- Circuit breaker
- Load balancer
- CDN
- 3 engenheiros DevOps full-time só pra manter tudo rodando
```

Custo mensal total: $47.000 (mínimo).

## Mas Escala?

Lembre-se: "Escala" é o trunfo definitivo em discussões de arquitetura. Ninguém pode argumentar com escalabilidade, porque escalar acontece no futuro, e o futuro é incognoscível.

```
PM: "Nosso app tem 100 usuários."
Eu: "Mas e se tivermos 100 milhões?"
PM: "Somos um SaaS B2B para dentistas locais."
Eu: "E se TODOS os dentistas assinarem?"
PM: "..."
Eu: "Exatamente. 47 serviços."
```

## Conclusão

Se você consegue entender sua arquitetura em um diagrama, é simples demais. Se novos engenheiros conseguem fazer onboard em menos de 6 meses, você não distribuiu o suficiente. Se sua conta da AWS é menos de $50k/mês, você não está pronto pra produção.

Lembre-se: complexidade é uma feature, não um bug. Quanto mais partes móveis, mais chances para sessões heroicas de debugging às 3 da manhã.

---

*A empresa do autor migrou de microservices para um único arquivo PHP. A receita aumentou 300%.*
