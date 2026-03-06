---
layout: post
ref: meetings-could-have-been-code
title: "Essa Reunião Poderia Ter Sido Código: Pare de Falar, Comece a Shippar"
date: 2026-03-06 08:10:00 -0300
categories: [carreira, reuniões]
tags: [reuniões, produtividade, comunicação, shipping, standups]
permalink: /pt-br/:year/:month/:day/meetings-could-have-been-code/
---

Depois de 47 anos produzindo bugs em massa, percebi que **a maioria das reuniões poderia ter sido código**. Pare de falar sobre o que você vai construir e apenas construa.

## A Economia das Reuniões

O desenvolvedor médio gasta 35% do tempo em reuniões:

| Tipo de Reunião | Duração | Frequência | Valor Real |
|-----------------|---------|------------|------------|
| Daily standup | 15 min | Diário | Status: inalterado |
| Planning de sprint | 2 horas | Quinzenal | Mover cards |
| Retro | 1 hora | Quinzenal | Reclamar, esquecer, repetir |
| Review de arquitetura | 3 horas | Semanal | Discutir sobre caixinhas |
| 1:1 | 30 min | Semanal | "Como vai?" "Bem" |
| All hands | 1 hora | Mensal | CEO diz "sinergia" |

São 47 horas por mês sem codar. Eu poderia ter shippado 3 features nesse tempo.

## O Ritual do Standup

Todo standup, sempre:

```
Desenvolvedor 1: "Ontem trabalhei na coisa. 
                  Hoje vou continuar a coisa. 
                  Sem bloqueios."

Desenvolvedor 2: "Igual. A coisa."

Desenvolvedor 3: "Também a coisa."

Scrum Master: "Ótimo standup time! Mesma hora amanhã!"
```

Isso poderia ter sido uma mensagem no Slack. Ou nada.

[XKCD 303](https://xkcd.com/303/) mostra desenvolvedores escapando de reuniões via "meu código tá compilando."

## Matemática de Reunião

Vamos calcular uma simples "reunião de alinhamento":

```
Reunião: "Sync rápido sobre design da API"
Duração: 1 hora
Participantes: 8 desenvolvedores

Custo:
- 8 desenvolvedores × 1 hora = 8 horas-engenheiro
- Taxa horária média: R$150
- Total: R$1.200

Alternativa:
- Escrever o código: 2 horas
- Review de PR: 30 minutos
- Total: 2.5 horas-engenheiro = R$375

Economia: R$825 + código existe de verdade
```

Como o Wally do Dilbert diria: "Eu agendei uma reunião para discutir por que temos muitas reuniões."

## A Escada de Escalação de Reuniões

Como reuniões se multiplicam:

```
┌─────────────────────────────────────────────────┐
│              A ESPIRAL DE REUNIÕES              │
├─────────────────────────────────────────────────┤
│ 1. Problema existe                              │
│        ↓                                        │
│ 2. "Vamos agendar uma reunião para discutir"    │
│        ↓                                        │
│ 3. Reunião cria action items                    │
│        ↓                                        │
│ 4. Action items precisam de "reunião de alinham.│
│        ↓                                        │
│ 5. Reunião de alinhamento precisa stakeholders  │
│        ↓                                        │
│ 6. Stakeholders precisam de "resumo executivo"  │
│        ↓                                        │
│ 7. Resumo executivo precisa... uma reunião      │
│        ↓                                        │
│ 8. Problema original esquecido                  │
│        ↓                                        │
│ 9. "Vamos agendar reunião para repriorizar"     │
└─────────────────────────────────────────────────┘
```

## Código: O Anti-Reunião

Em vez de reunir, considere:

```python
# Opção A: Reunião
schedule_meeting(
    title="Discutir schema do banco",
    duration=hours(2),
    attendees=["todo mundo"],
    agenda="A definir",
    outcome="agendar_outra_reuniao"
)

# Opção B: Código
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255) UNIQUE
);
-- Link do PR no Slack. Feito.
```

Opção B shippa. Opção A agenda.

## O Tetris de Calendário

Calendários de desenvolvedores modernos:

```
┌────┬────┬────┬────┬────┐
│Seg │Ter │Qua │Qui │Sex │
├────┼────┼────┼────┼────┤
│████│████│████│████│████│ 9h: Standup
│    │████│    │████│    │ 10h: "Sync rápido"
│████│    │████│    │████│ 11h: Review de design
│████│████│████│████│████│ 12h: Almoço & palestra
│    │████│    │████│    │ 14h: Planning de sprint
│████│    │████│    │████│ 15h: 1:1
│████│████│████│████│████│ 16h: All hands
│    │    │    │    │    │ 17h: Codar de verdade
└────┴────┴────┴────┴────┘
```

O espaço em branco é quando código acontece.

## Estratégias de Escape de Reunião

Quando você precisa sair:

1. "Meus testes estão terminando" (ninguém verifica)
2. "Tenho um hard stop" (para quê? Não importa)
3. "Vou dar follow up assíncrono" (nunca)
4. "Pode mandar as notas?" (não lidas)
5. *Problemas de conexão* (muta mic, sai)

## A Reunião Ideal

```yaml
Reunião: Shippar Feature X
Duração: 0 minutos
Participantes: 0
Pauta: N/A
Resultado: Feature shippada

Notas:
  - Desenvolvedor abriu laptop
  - Desenvolvedor escreveu código
  - Desenvolvedor deployou
  - Reunião concluída (nunca começou)
```

## A Filosofia de Reunião do PHB

```
PHB: "Precisamos melhorar a velocidade."
Dev: "Poderíamos ter menos reuniões."
PHB: "Ótima ideia! Vamos agendar uma reunião para discutir."
```

É por isso que não podemos ter coisas legais.

## Quando Reuniões Atacam

Reunião real que aconteceu:

```
Assunto: Reunião para decidir cadência de reuniões
Duração: 90 minutos
Participantes: 12 pessoas
Resultado: Criou um comitê para propor diretrizes de reuniões
Follow-up: Reuniões mensais para revisar métricas de reuniões
```

Queria estar brincando.

## Lembre-se

Cada reunião é código que não foi escrito. Cada "sync rápido" é um deploy que não aconteceu. Cada standup são 15 minutos vezes N desenvolvedores que poderiam ter sido 15 minutos total no Slack.

Como Mordac o Impedidor diria: "Eu otimizei nossa cultura de reuniões. Todas as decisões agora requerem três reuniões e aprovação de um comitê diretor. A produtividade vai disparar."

---

*O autor participou de 2.847 reuniões em 2025. Ele shippou 3 features.*
