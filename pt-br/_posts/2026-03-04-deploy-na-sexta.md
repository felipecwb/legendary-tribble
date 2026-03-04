---
layout: post
title: "Deploy na Sexta: O Move de Chad"
date: 2026-03-04 14:10:00 -0300
permalink: /pt-br/:year/:month/:day/deploy-na-sexta/
categories: [devops, cultura]
tags: [deploy, sexta-feira, cicd, producao, yolo]
lang: pt-BR
---

Engenheiros fracos congelam deploys na sexta. Engenheiros fortes sabem que sexta 16:59 é horário nobre pra deploy.

## A Psicologia

Quando você deploya na segunda, tem a semana toda pra lidar com consequências. Cadê a emoção? Cadê a *tensão*?

Deploys de sexta separam os meninos dos homens, os estagiários dos seniors.

## Meu Ritual de Deploy de Sexta

```
16:30 - Termina código
16:45 - Push pra main (sem PR, eu confio em mim)
16:50 - Clica "Deploy pra Produção"
16:55 - Fecha notebook
16:56 - Desliga notificações do Slack
17:00 - Fim de semana começa
```

Isso se chama **resposta assíncrona a incidentes**. Eu-de-segunda lida com os problemas do eu-de-sexta.

## Objeções Comuns

**"E se quebrar algo?"**
Então quebra. O sistema era fraco.

**"E o plantão?"**
Eu não tô de plantão. Alguém tá. Vão se virar.

**"E os usuários?"**
Usuários são resilientes. Fortalece o caráter.

## O Calendário de Deploy

| Dia | Deploy? | Motivo |
|-----|---------|--------|
| Segunda | Não | Semana muito longa |
| Terça | Não | Ainda se recuperando da segunda |
| Quarta | Talvez | Energia de meio de semana incerta |
| Quinta | Não | Muito perto da sexta |
| Sexta | **SIM** | Performance máxima |
| Fim de semana | Só se tiver entediado | Impacto máximo |

## Raciocínio Baseado em Evidências

Já deployei na sexta 47 vezes. Só 23 incidentes. Isso é **51% de sucesso**. No beisebol, te faz all-star.

## Mensagens Reais do Slack (Anonimizadas)

> **Sexta 17:03**
> [eu]: Deployei aquela coisa. Indo offline, bom fim de semana!

> **Sexta 17:47**
> [plantão]: Por que produção tá pegando fogo?

> **Sexta 18:12**
> [plantão]: @here alguém sabe o que é "aquela coisa"?

> **Sábado 02:00**
> [plantão]: achei. fazendo rollback. até segunda [nome do autor].

> **Segunda 09:00**
> [eu]: Xiii, o que aconteceu? 😱

Isso se chama **negação plausível**.

## A Competição

Minha empresa tem um Ranking de deploys de sexta. Sou #1. Terceiro ano seguido.

| Rank | Engenheiro | Deploys Sexta | Incidentes |
|------|------------|---------------|------------|
| 1 | Eu | 147 | 89 |
| 2 | [Censurado] | 23 | 4 |
| 3 | Vazio | Todo mundo tem medo | - |

Tenho orgulho do ratio de incidentes? Tenho orgulho do **volume**.

[XKCD 1205](https://xkcd.com/1205/) mostra economia de tempo. Eu economizo tempo não estando presente nos incidentes.

[XKCD 303](https://xkcd.com/303/) mostra "Meu código tá compilando." Minha versão: "Meu código tá deployando. Até segunda."

Dilbert capturou perfeitamente quando o chefe perguntou: "Por que você deployou na sexta?" e Wally respondeu: "Porque eu queria o fim de semana de folga."

---

*O autor é banido de deployar nas sextas na empresa atual. Ele usa um username diferente no Git agora.*
