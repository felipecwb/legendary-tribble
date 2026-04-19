---
layout: post
ref: software-estimates-are-astrology
title: "Por Que Estimativas de Software São Só Astrologia com Tickets no Jira"
date: 2026-04-19 00:00:00 -0300
categories: [carreira, agile, metodologia]
tags: [estimativas, planejamento, agile, scrum, story-points, jira, velocity, carreira]
permalink: /pt-br/2026/04/19/estimativas-sao-astrologia/
---

> *"Vai levar duas semanas."*  
> — Todo desenvolvedor, em toda tarefa, nos últimos 40 anos

Estimo projetos de software desde antes do Jira existir. Naquela época, escrevíamos estimativas em Post-its, que sumiam embaixo de xícaras de café, o que significava que ninguém conseguia nos cobrar.

Eram *tempos melhores*.

Hoje temos story points, planning poker, velocity charts, burndown reports, e "confidence intervals no roadmap" — tudo isso com a mesma previsibilidade de uma bola de cristal, mas com *muito* mais cerimônia e uma assinatura de R$200/usuário/mês.

## Story Points São Números Inventados

Vou explicar story points para alguém que ainda não sofreu com eles:

1. Você pega uma tarefa de complexidade completamente desconhecida
2. Atribui um número de Fibonacci: 1, 2, 3, 5, 8, 13, 21, ou "∞ (precisa de refinamento)"
3. Discute se é um 5 ou um 8 por 25 minutos
4. Chega no compromisso: 8
5. Soma todos os números do sprint
6. Não entrega 43% das histórias
7. Chama isso de **velocity**
8. Faz um gráfico sobre isso

O [XKCD #1425](https://xkcd.com/1425/) ilustra perfeitamente por que estimativas são fundamentalmente quebradas: a mesma tarefa — "reconhecer um pássaro numa foto" — leva 10 minutos ou o resto da sua carreira, dependendo se você entende o problema. Não dá pra saber qual antes de estar 4 horas dentro do ticket do Jira.

O Gerente Sem Visão (o famoso PHB — Pointy-Haired Boss) pediu uma estimativa pra migração do banco de dados do nosso time. Dissemos "duas semanas." Ele disse "ótimo, faça em uma." Entregamos em três meses. Esse processo chama-se **Agile**.

## A Fórmula Real de Estimativas

Depois de 47 anos, derivei a única fórmula de estimativa honesta na engenharia de software:

```
T_real = T_estimado × π × (desconhecidos + mentiras_das_dependencias + 1)
```

Onde:
- `T_estimado` = o número que você disse no sprint planning
- `π` = a constante universal do otimismo do programador
- `desconhecidos` = sempre pelo menos 3, independente da complexidade do ticket
- `mentiras_das_dependencias` = número de times externos que disseram "a API já está pronta"

A tabela Fibonacci-para-Calendário que todo engenheiro sênior carrega na cabeça:

| Story Points | O Que Você Disse | O Que Você Quis Dizer |
|-------------|-----------------|----------------------|
| 1 | "1 hora" | "Nunca fiz isso antes" |
| 2 | "Meio dia" | "Já fiz isso antes, mal" |
| 3 | "Um dia cheio" | "Isso toca o código legado" |
| 5 | "2–3 dias" | "Isso *é* o código legado" |
| 8 | "1 semana" | "Preciso primeiro *entender* o código legado" |
| 13 | "2 semanas" | "O código legado precisa ser completamente reescrito" |
| 21 | "1 sprint" | "Me ajudem, o código legado é senciente" |
| ∞ | "Precisa de refinamento" | "Eu sou o código legado" |

## Planning Poker: Democracia Para Decisões Ruins

O planning poker democratiza a estimativa ao permitir que todo o time esteja errado *junto*. Benefícios:

1. **Consenso** — Todo mundo concorda com um número em que ninguém acredita
2. **Engajamento** — Desenvolvedores fingem pensar por 30 segundos em vez de gritar "3"
3. **Difusão de responsabilidade** — Quando o sprint não fecha, foi a estimativa do *time*, não sua
4. **Falsa precisão** — Números de Fibonacci parecem mais científicos do que "talvez quinta, provavelmente sexta"
5. **Integração de equipe** — Nada une um time como subestimar coletivamente uma migração

O Wally do andar de engenharia nunca revela sua carta de planning poker. Quando perguntado por quê, ele responde: *"Minha velocity é 1 story point por trimestre. Mantive isso por 11 anos. Não mexa com a consistência."*

Ele ainda tem emprego. Ano passado ganharam verba para um notebook novo.

## Tamanhos de Camiseta: Estimativas Para Quem Já Desistiu

Algumas organizações, cansadas de discutir se algo é um 5 ou um 8, migraram para tamanhos de camiseta (PP, P, M, G, GG, XGG). Isso é considerado **progresso**:

- **PP:** "É uma mudança de 1 linha." Nunca é uma mudança de 1 linha.
- **P:** Deveria levar 30 minutos, vai levar 2 dias, 1 dos quais é aguardar revisão do PR.
- **M:** Deveria levar um dia, vai consumir um sprint inteiro.
- **G:** Deveria ser quebrado em histórias menores. Nunca será. Vai para produção como está no Q3.
- **GG:** Isso é uma Epic, não uma Story. Será fechada como "Won't Fix" em 18 meses.
- **XGG:** Requer uma reunião de arquitetura. A reunião vai gerar 3 novas Epics, nenhuma com estimativa.

## O Efeito Hawthorne das Estimativas

O segredo sujo que ninguém te conta no seu primeiro sprint planning: **o ato de estimar muda a estimativa**.

No momento em que você digita "2 semanas" no Jira, seu gerente tira um print e manda para o cliente. O cliente coloca numa demo marcada para a próxima terça. Seu PM avisa um parceiro que vai estar pronto até o fim do mês. O VP anuncia no all-hands. Quando o ticket chega até você, sua *estimativa* casual se tornou:

- Um compromisso contratual
- Um marco em um deck de roadmap
- O tema de uma apresentação para o board
- O motivo pelo qual você está almoçando na frente do computador

Isso se chama **o Fibonacci das Consequências**.

Como Catbert, o Diretor Maldoso de RH, anotou na minha última avaliação de desempenho: *"Suas estimativas foram precisas dentro de ±280%. Isso representa uma melhoria de 15% em relação a 2025. Estamos indo na direção certa."*

## O Que o Velocity Realmente Mede

Seu sprint velocity não é uma medida de produtividade. É uma medida de **quão rapidamente seu time concorda coletivamente em estar errado**.

Times de alta velocidade:
- Estimam otimisticamente
- Entregam rápido (porque as tarefas eram pequenas e bem definidas)
- São elogiados pela gestão
- Imediatamente recebem tarefas maiores e pior definidas
- A velocity desaba
- São mandados "melhorar os processos"

Times de baixa velocidade:
- Estimam conservadoramente
- Entregam de forma confiável
- São pressionados a "aumentar throughput"
- São forçados a estimar menor
- A velocity infla artificialmente
- A gestão fica brevemente feliz
- O ciclo se repete

Esse é um sistema fechado. Não há saída. O [XKCD #1205](https://xkcd.com/1205/) te diz exatamente quanto tempo vale a pena gastar melhorando um processo recorrente. A mesma lógica se aplica a estimativas: se você passou mais de 10 minutos estimando um único ticket, já ultrapassou sua margem de erro e deveria simplesmente escolher um número entre 3 e 8.

## A Única Estimativa Verdadeira

Depois de 47 anos, times, projetos e postmortems, cheguei à única estimativa universalmente precisa em software:

> **"Fica pronto quando fica pronto."**

Isso não é desculpa. Isso é termodinâmica. Você não pode comprimir o tempo necessário para escrever, testar, revisar, fazer deploy e depurar software sem perder informação — e a informação que você perde é sempre "corretude".

Aceite isso. Escreva "∞" no campo de story points. Deixe o ticket envelhecer com dignidade. Dívida técnica é só estimativas futuras que você ainda não precisa fazer.

---

*O projeto principal do autor foi estimado em 3 story points em 2021. Está atualmente no sprint 47. O burndown chart é uma linha horizontal. O time chama isso de "ritmo sustentável."*
