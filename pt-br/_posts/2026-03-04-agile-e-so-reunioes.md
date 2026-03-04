---
layout: post
title: "Agile é Só Reuniões (Uma Retrospectiva)"
date: 2026-03-04 08:15:00 -0300
categories: [metodologia, desabafos]
tags: [agile, scrum, kanban, standups, sprints, jira, retrospectivas, planning-poker, waterfall]
permalink: /pt-br/:year/:month/:day/:title/
---

Faço Agile há 20 anos. Aqui está minha retrospectiva: **é só reunião**.

## Meu Calendário Agile

```
Segunda:  Sprint Planning (4 horas)
Terça:    Standup (15 min) + Refinamento de Backlog (2 horas)
Quarta:   Standup (15 min) + "Sync Rápido" (1 hora)
Quinta:   Standup (15 min) + Prep pra Sprint Review (1 hora)
Sexta:    Standup (15 min) + Sprint Review (2 horas) + Retrospectiva (1 hora)
```

Tempo pra codar: **quando as reuniões acabarem**.

## A Daily Standup

As regras dizem 15 minutos. A realidade:

```
09:00 - Standup começa
09:05 - "Deixa eu compartilhar a tela"
09:15 - "Todo mundo tá vendo?"
09:20 - Cachorro de alguém late
09:25 - "Desculpa, pode continuar"
09:30 - Realmente discutindo trabalho
09:45 - "Vamos continuar isso offline"
09:46 - Mesmas 5 pessoas ficam pra "papo rápido"
10:30 - Papo rápido termina
```

## Três Perguntas do Doom

**O que você fez ontem?**
"Reuniões."

**O que você vai fazer hoje?**
"Reuniões, depois tentar codar."

**Algum bloqueio?**
"Sim. Reuniões."

## Sprint Planning

Gastamos 4 horas decidindo o que fazer em 2 semanas.

```
Product Owner: "Precisamos entregar Feature X"
Dev: "Isso é 3 meses de trabalho"
PO: "Dá pra fazer em 2 semanas?"
Dev: "Não"
PO: "E se cortar escopo?"
Dev: "Ainda não"
PO: "Story points?"
Dev: "847"
PO: "Vamos chamar de 8"
```

Aí não entregamos a sprint. **Ninguém esperava.**

## Story Points

Story points não são tempo. São **complexidade**.

Na prática:
- 1 ponto = termina na daily
- 3 pontos = um dia
- 5 pontos = talvez essa sprint
- 8 pontos = provavelmente não essa sprint
- 13 pontos = vamos dividir em 5 sub-tasks de 3 pontos cada

## A Retrospectiva

**O que foi bem?**
"Sobrevivemos."

**O que pode melhorar?**
"Menos reuniões."

**Action items:**
"Agendar uma reunião pra discutir reduzir reuniões."

## Transformação Agile™

Quando a gerência descobre Agile:

1. Contrata Agile Coach (R$10.000/dia)
2. Renomeia Gerente pra "Scrum Master"
3. Renomeia Tasks pra "User Stories"
4. Instala Jira
5. ✅ Somos Agile agora

## O Ciclo de Vida do Jira

```
To Do → In Progress → "Blocked" → Ainda Blocked → 
→ "Waiting for Review" → Volta pra In Progress → 
→ "QA" → Bug Encontrado → In Progress → 
→ Done → Reaberto → In Progress → 
→ Done → "Won't Fix" → Arquivado
```

## [XKCD 844](https://xkcd.com/844/) Previu Isso

A tirinha de "código bom" vs "código ruim" se aplica a processo também. Processo bom é invisível. Processo ruim são notificações do Jira.

## Dilbert sobre Agile

Wally perfeiçoou Agile:

"Movi meu ticket de 'To Do' pra 'In Progress.' Esse é meu trabalho da sprint."

Gerente: "Mas você não fez nada de verdade."

Wally: "O board discorda."

## A Alternativa Kanban

"Kanban é mais simples que Scrum!"

Board Kanban:
```
| To Do | Doing | Done |
|-------|-------|------|
| 847   | 3     | 1    |
|       |       |      |
```

Limite de WIP: 3. Limite de To Do: ∞.

## Minha Proposta

Esquece Agile. Use **Desenvolvimento YOLO**:

1. Coda quando tiver vontade
2. Deploy quando compilar
3. Reuniões só na sexta
4. Documentação é o código
5. Retrospectiva: "Shipou"

## Conclusão

Agile era pra reduzir overhead. Agora temos overhead de Agile.

O manifesto dizia "indivíduos e interações sobre processos e ferramentas."

Ganhamos workflows do Jira e auditorias de conformidade de processo.

---

*O autor gastou 15 minutos escrevendo esse artigo e 3 horas em reuniões sobre se deveria escrever.*
