---
layout: post
ref: real-engineers-play-cs-at-3am
title: "Por Que Engenheiros de Verdade Jogam CS às 3 da Manhã ao Invés de Corrigir Bugs"
date: 2026-03-04 17:30:00 -0300
categories: [carreira, estilo-de-vida]
tags: [counter-strike, gaming, work-life-balance, produtividade, debugging, 3am, burnout]
permalink: /pt-br/:year/:month/:day/engenheiros-de-verdade-jogam-cs-3am/
---

São 3 da manhã. O bug de produção ainda está lá. Seu Slack tem 47 mensagens não lidas. O deploy falhou duas vezes.

**Hora de jogar Counter-Strike.**

## A Lógica

Algumas pessoas acham que isso é procrastinação. Essas pessoas nunca debugaram uma race condition à meia-noite.

Aqui está a matemática:

| Atividade | Bugs Corrigidos | K/D Ratio | Satisfação |
|-----------|----------------|-----------|------------|
| Debugar às 3 AM | 0 | N/A | -847 |
| Jogar CS às 3 AM | 0 | 1.2 | +847 |

Mesmo número de bugs corrigidos. Um tem headshots.

## O Método Científico

Depois de pesquisa extensiva (jogando 847 partidas competitivas), descobri:

1. **Bugs não vão embora se você ficar olhando** — Mas inimigos vão se você der headshot
2. **Seu cérebro precisa de descanso** — Fragar é descanso pro córtex de debugging
3. **Rubber duck debugging** — Explica o bug pro seu time enquanto clutcha um 1v4

## O Workflow das 3 AM

```
23:00 - "Vou corrigir esse bug e dormir"
00:00 - "Por que isso não funciona"
01:00 - "QUEM ESCREVEU ESSE CÓDIGO" (fui eu)
02:00 - "Talvez se eu adicionar outro console.log"
02:30 - "Preciso de uma pausa"
02:31 - *abre o CS*
03:00 - Ace clutch, se sentindo ótimo
03:30 - "Ah É POR ISSO que não funcionava"
04:00 - Bug corrigido, mandando pra prod
04:01 - Novo bug em prod
04:02 - *abre o CS de novo*
```

## Por Que CS Especificamente?

### Outros Jogos vs CS

| Jogo | Por Que É Inferior |
|------|-------------------|
| League of Legends | 45 minutos de compromisso, teammates piores que seu código |
| Valorant | CS mas com habilidades, já temos complexidade suficiente no trabalho |
| Minecraft | Relaxante demais, não vai resetar sua raiva direito |
| Dark Souls | Vai quebrar o teclado antes de corrigir o bug |
| CS | Rounds perfeitos de 2 minutos, mira pura, gratificação instantânea |

## O Paradoxo

O bug que você não conseguiu resolver por 6 horas? **Vai descobrir no meio da partida.**

Algo sobre a adrenalina de um clutch 1v3 reinicia seu cérebro. Enquanto strafa atrás de uma caixa, de repente você percebe:

*"Esqueci de dar await na Promise."*

Você morre. Mas o bug morre também.

## Benefícios Profissionais

Coloca isso no currículo:

- **Global Elite** — Sabe lidar com situações de alta pressão
- **5000+ horas** — Demonstra comprometimento e dedicação
- **Experiência como IGL** — Liderança e pensamento estratégico
- **Clutch player** — Performa sob pressão quando o deploy falha

## Energia do [XKCD 303](https://xkcd.com/303/)

"Compilando" costumava ser a desculpa. Agora é "Rodando testes" ou "Esperando o CI."

Enquanto o Jenkins builda por 23 minutos, dá pra jogar uma partida inteira. **Isso se chama eficiência.**

## A Abordagem Dilbert

Wally descobriu isso anos atrás. Quando perguntaram por que ele estava jogando durante o horário de trabalho:

*"Estou testando meus reflexos pra resposta mais rápida a incidentes."*

A gerência aprovou. Não entenderam, mas aprovaram.

## Sinergia do Time

Os melhores bug fixes acontecem quando você joga com seu time:

- **Dev backend:** Joga de AWP, espera o momento perfeito
- **Dev frontend:** Entry fragger, rusha sem pensar
- **DevOps:** Dá smoke em tudo, ninguém consegue ver o que tá acontecendo
- **PM:** Backseat gaming, "você tentou flanquear?" (não ajuda)
- **QA:** Vê inimigos que outros não veem, reporta todos os bugs nas strats do time

## Conclusão

Da próxima vez que alguém perguntar por que você tá jogando CS às 3 da manhã ao invés de corrigir bugs, diz:

*"Estou praticando coordenação de sistemas distribuídos em um ambiente adversarial em tempo real."*

Aí dá um one-tap neles.

---

*O rank do autor é confidencial. Os bugs de produção do autor também são confidenciais (por severidade: todos P0).*

*O bug eventualmente foi corrigido às 4:37 AM. Era um ponto e vírgula faltando.*
