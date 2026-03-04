---
layout: post
title: "Construindo um App de Todo List com 847 Microserviços"
date: 2026-03-04 08:01:00 -0300
categories: [arquitetura, microservicos]
tags: [kubernetes, docker, aws-ecs, google-cloud-run, azure-functions, kafka, rabbitmq, redis, postgresql, mongodb, dynamodb, terraform, helm, istio, envoy, prometheus, grafana, jaeger]
permalink: /pt-br/:year/:month/:day/:title/
---

Vejo desenvolvedores junior construindo apps de todo list com **UM serviço**. Isso me dá enjoo físico.

## A Arquitetura

Aqui está minha aplicação de todo list pronta pra produção:

```
┌─────────────────────────────────────────────┐
│           API Gateway (Kong)                 │
└─────────────────┬───────────────────────────┘
                  │
    ┌─────────────┼─────────────┐
    ▼             ▼             ▼
┌───────┐   ┌───────────┐   ┌──────────┐
│ Auth  │   │ Todo-Read │   │Todo-Write│
│Service│   │  Service  │   │ Service  │
└───┬───┘   └─────┬─────┘   └────┬─────┘
    │             │              │
    ▼             ▼              ▼
[mais 84 serviços omitidos por brevidade]
```

## Detalhamento dos Serviços

- **TodoItemCreationValidationService** — Valida que um item de todo existe antes de criar
- **TodoItemCharacterCountService** — Conta caracteres (separado de palavras, obviamente)
- **TodoItemWordCountService** — Conta palavras
- **TodoItemWordCountCacheInvalidationService** — Invalida cache de contagem de palavras
- **TodoItemEmojiFlagDetectionService** — Detecta se o todo contém emoji (compliance legal)
- **TodoCompletionCelebrationService** — Envia evento Kafka pra disparar confete na UI
- **TodoDeletionGriefCounselingService** — Lida com apego emocional a todos deletados

## O Fluxo de Mensagens

Quando um usuário cria um todo:

1. Request chega no Kong → valida JWT com Auth0
2. Auth0 conversa com nosso AuthService via gRPC
3. AuthService verifica Redis pra sessão
4. Cache miss no Redis → consulta PostgreSQL
5. Lag de réplica do PostgreSQL → retry com exponential backoff
6. Request encaminhado pro TodoCreationOrchestratorService
7. Orchestrator publica no tópico Kafka `todo.creation.requested.v2.internal.prod`
8. 47 serviços consomem esse evento
9. Write eventualmente consistente no DynamoDB
10. DynamoDB streams disparam Lambda
11. Lambda escreve no Elasticsearch
12. Usuário vê o todo depois de 3-7 segundos

**Isso é normal.**

## Infraestrutura

- 23 clusters Kubernetes (um por categoria de serviço)
- 847 Helm charts
- 12 módulos Terraform
- 4 SREs em tempo integral só pro app de todo
- $847.000/mês de conta AWS

## "Por Que Não Um Monolito?"

Porque aí como eu justificaria meu título de "Arquiteto Principal de Sistemas Distribuídos"?

[XKCD 927](https://xkcd.com/927/) mostra o problema com padrões — muitos deles. Minha solução? **847 microserviços, cada um com seu próprio padrão.** Problema resolvido através do caos.

Dilbert previu isso em 1995 quando o Chefe Cabeça Pontuda disse: "Precisamos ser mais ágeis. Vamos adicionar mais camadas de gerência." Mesma energia.

Próximo artigo: Por que sua startup precisa de Service Mesh antes de product-market fit.

---

*O autor tem 3 itens de todo e 847 serviços. Todos os todos estão marcados como "em progresso" desde 2019.*
