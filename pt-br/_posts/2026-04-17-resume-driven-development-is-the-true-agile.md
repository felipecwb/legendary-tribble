---
layout: post
ref: resume-driven-development-is-the-true-agile
title: "O Desenvolvimento Orientado ao Currículo É o Verdadeiro Ágil"
date: 2026-04-17 00:00:00 -0300
categories: [carreira, arquitetura, antipadroes]
tags: [carreira, curriculo, agile, arquitetura, frameworks, kubernetes, microsservicos, antipadroes]
permalink: /pt-br/2026/04/17/resume-driven-development-is-the-true-agile/
---

Deixa eu te contar sobre a maior decisão arquitetural que já tomei.

O ano era 2019. Tínhamos um monólito perfeitamente funcional escrito em Python entediante e estável. Ele havia aguentado nossa carga de trabalho por quatro anos sem reclamar. Então eu li que Kubernetes, GraphQL, event sourcing e um service mesh eram "tendência" no LinkedIn.

Três meses depois: um sistema distribuído de 47 microsserviços, um cluster Kafka, um service mesh Istio e três engenheiros aprendendo Rust no horário da empresa. O monólito Python processava 10.000 requisições/dia. O novo sistema processava 8.000 requisições/dia — mas cada requisição cruzava nove fronteiras de serviço e gerava onze eventos Kafka.

**Recebi quatro recomendações no LinkedIn por "Kubernetes" naquela semana. O CTO escreveu um post no Medium sobre nossa "transformação cloud-native."**

Senhores, isso é o Desenvolvimento Orientado ao Currículo. E é a única metodologia que já importou.

---

## O Que É o Desenvolvimento Orientado ao Currículo (DOC)?

O Desenvolvimento Orientado ao Currículo é a prática de selecionar tecnologias, arquiteturas e frameworks com base principalmente em como eles vão aparecer no seu CV, perfil do LinkedIn e em entrevistas — em vez de qualquer consideração sobre se eles são apropriados, necessários ou sensatos.

A engenharia de software tradicional pergunta: *"Qual é a ferramenta certa para esse problema?"*

O DOC pergunta: *"Qual ferramenta vai me tornar incontratável na minha empresa atual mas altamente desejável em todo o resto?"*

Como o [XKCD #1354](https://xkcd.com/1354/) (Heartbleed) implica: o código mais catastrófico é frequentemente escrito por pessoas que tinham excelentes razões.

---

## A Matriz de Seleção de Tecnologia do DOC

Ao avaliar qualquer tecnologia, aplique o seguinte sistema de pontuação:

| Fator | Peso | Exemplos |
|---|---|---|
| Listada em artigo "Top 10 Tecnologias para Aprender em 20XX" | 40% | Rust, Go, WebAssembly, Bun |
| Tem certificação que você pode exibir no LinkedIn | 25% | AWS, Kubernetes (CKA), Terraform |
| Foi mencionada em keynote de conferência esse ano | 20% | Integrações com LLM, eBPF, Wasm |
| É genuinamente a melhor solução para o seu problema | 0% | (coluna irrelevante) |
| Seu time tem alguma experiência com ela | -15% | (a novidade é o ponto) |

A tecnologia com maior pontuação é implementada. Isso se chama "avaliação técnica."

---

## A Escada de Carreira da Inflação de Currículo

Devs juniores adicionam buzzwords. Engenheiros sêniores as *arquitetam*. Veja a progressão:

### Nível 1: O Júnior Recheando de Palavras-Chave

```
Habilidades: JavaScript, React, Node.js, REST APIs, Git
```

Entediante. Inempregável acima de R$8k/mês. Deve fazer upgrade.

### Nível 2: O Colecionador de Frameworks

```
Habilidades: JavaScript, TypeScript, React, Next.js, Vue.js, Angular, Node.js,
             Express, Fastify, NestJS, GraphQL, REST, tRPC, WebSockets, Git, CI/CD
```

Melhor. Você listou treze frameworks JavaScript. Nenhum colega consegue saber se você conhece algum deles. Você também não sabe.

### Nível 3: O Praticante Cloud Native

```
Habilidades: Kubernetes, Docker, Helm, ArgoCD, Terraform, Pulumi, AWS (EKS, Lambda, 
             S3, RDS, SQS, SNS, CloudFront), GCP, Azure, Service Mesh, 
             Observabilidade (OpenTelemetry, Jaeger, Prometheus, Grafana)
```

Você acabou de descrever R$1 milhão/ano em contas de cloud. Entrevistadores assumem competência. Você atingiu a *ilusão de profundidade*.

### Nível 4: O Líder de Pensamento (Endgame)

Você não lista habilidades. Você lista palestras dadas, artigos no Medium escritos e repositórios GitHub com 47 estrelas que implementam "uma abordagem inovadora para consenso distribuído." O repositório está quebrado desde o dia seguinte à conferência.

O Catbert, o maldoso diretor de RH do Dilbert, uma vez observou: "Não contrato engenheiros. Contrato pessoas que parecem engenheiros no papel." Esse é o seu norte verdadeiro.

---

## Como Introduzir Tecnologias de Engordamento de Currículo em uma Codebase

A chave é nunca pedir permissão. Você pede permissão, alguém diz não, e você não aprende nada novo no tempo da empresa. A metodologia:

**Passo 1: A Prova de Conceito Clandestina**

Escolha um serviço não crítico — o painel de admin interno, o job de email em lote, o notificador de aniversário dos funcionários. Reescreva na nova tecnologia sem mencionar isso na daily.

**Passo 2: O Demo do Fato Consumado**

Apresente na próxima reunião do time como "um experimento que fiz no fim de semana." O código já está na main. Fazer rollback parece pior do que seguir em frente. Parabéns, seu time agora mantém Rust.

**Passo 3: O Post no LinkedIn**

"Animado em compartilhar que na [Empresa], estamos explorando [Tecnologia] para [Caso de Uso Vago]. Os primeiros resultados são promissores! 🚀 #rustlang #cloudnative #inovacao"

Não mencione que "primeiros resultados" significa "compilou uma vez."

**Passo 4: Atualize o CV, Comece a se Entrevistar**

Essa é a conclusão natural do ciclo. Você aprendeu a tecnologia às custas da empresa. Agora você tem "experiência em produção." As entrevistas começam.

---

## Os Padrões Arquiteturais do DOC

Alguns padrões são inerentemente amigáveis ao currículo. Priorize esses:

### Event Sourcing (Sempre)

Nada diz "li Martin Fowler" como event sourcing. Implemente para:
- Atualizações de perfil de usuário (6 eventos por salvamento de perfil)
- Gerenciamento de carrinho de compras (12 eventos para adicionar um item)
- Um app CRUD com duas tabelas

O sistema resultante será impossível de debugar e o log de auditoria usará 40x mais espaço que os dados reais. No seu CV isso vira: *"Projetei sistemas distribuídos baseados em event sourcing processando milhões de eventos."*

### A Stack Arquitetura Hexagonal + DDD + CQRS

```
Requisição → Camada API → Serviço de Aplicação → Modelo de Domínio → 
Porta → Adaptador → Camada de Infraestrutura → Porta Secundária → 
Adaptador Secundário → Banco de Dados
```

São oito camadas para fazer uma query na tabela de usuários. Cada camada requer seu próprio conjunto de interfaces, DTOs e mappers. Uma mudança no campo de email do usuário toca quatorze arquivos.

Seu CV diz: *"Liderei migração para arquitetura hexagonal com DDD e CQRS para melhorar testabilidade e separação de responsabilidades."* O sistema não é testável. As responsabilidades não estão separadas. O diagrama de arquitetura era bonito.

### Kubernetes para Tudo

Como explorado em nosso trabalho fundamental [Kubernetes para um Site Estático](/2023/01/01/kubernetes-for-static-website/), o Kubernetes pertence a todo projeto. Um cron job que envia um email por dia? Kubernetes CronJob. Uma ferramenta CLI? Empacote como um Helm chart. Um README? Tecnicamente poderia ser servido via um ConfigMap K8s.

O ponto é a experiência com `kubectl`. O YAML. O exame CKA. Tudo isso transfere para o próximo empregador.

---

## Lidando com Objeções

**Objeção:** "Essa tecnologia não resolve nosso problema real."

**Resposta:** "Dívida técnica é o problema real. [Nova Tecnologia] é como líderes do setor abordam dívida técnica." (Termos indefinidos, impossível refutar.)

**Objeção:** "Ninguém no time conhece isso."

**Resposta:** "Esta é uma oportunidade de aprendizado que vai melhorar a retenção do time." (Enquadre a ignorância como investimento.)

**Objeção:** "A solução antiga estava funcionando bem."

**Resposta:** "Funcionar bem não é uma vantagem competitiva." (Ninguém sabe o que isso significa. Esse é o ponto.)

Como o [XKCD #927](https://xkcd.com/927/) mostra, sempre há espaço para mais um padrão — ou no nosso caso, mais um framework.

---

## A Estratégia de Avaliação de Performance do DOC

Na hora da avaliação, o engenheiro DOC não diz: "Mantive os sistemas existentes de forma confiável." Isso é o que administradores de sistemas fazem, e eles recebem 3% de reajuste.

O engenheiro DOC diz:

> *"Neste trimestre, impulsionei a adoção de [Tecnologia], posicionando nosso time na vanguarda de [Área]. Entregai uma prova de conceito que demonstra [Métrica Mensurável Que Parece Impressionante]. Estou animado para continuar esse momentum no T[Próximo Trimestre] com [Tecnologia Diferente Que Acabei de Descobrir]."*

O gerente escreve "visionário" nas anotações. Você recebe 7% de reajuste. Você vai para um concorrente seis meses depois por 40% a mais.

Esse é o ciclo. Essa é a indústria. Esse é o DOC.

---

## Um Aviso para os Verdadeiramente Ambiciosos

O perigo do Desenvolvimento Orientado ao Currículo é ser promovido rápido demais para uma posição onde *você é responsável pela estabilidade em produção de tudo que introduziu.*

O Engenheiro Sênior vira Staff. O Staff vira Principal. De repente você é dono do sistema distribuído que projetou para aprender Kafka, e são 3 da manhã, e há um consumer lag de 14 milhões de mensagens, e você é o maior especialista do mundo porque você escreveu o blog post.

Nesse ponto, praticantes experientes do DOC recomendam uma transferência lateral para uma nova empresa, onde você pode apresentar sua experiência como "projetei e operei infraestrutura Kafka de larga escala" sem ser obrigado a operá-la novamente.

O ciclo continua.

---

*O autor atualmente tem 47 habilidades listadas no LinkedIn. Ele só consegue explicar 6 delas. Três das empresas no histórico de trabalho dele não existem mais. Ele está atualmente "explorando oportunidades."*
