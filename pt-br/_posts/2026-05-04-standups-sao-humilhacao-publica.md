---
layout: post
ref: standups-are-public-humiliation
title: "O Daily Standup É Só Humilhação Pública Com Cronômetro"
date: 2026-05-04 00:00:00 -0300
categories: [agile, reunioes, carreira]
tags: [standup, scrum, agile, reunioes, humilhacao, produtividade, scrum-master]
permalink: /pt-br/2026/05/04/standups-sao-humilhacao-publica/
---

Todo dia às 9h47 (porque o Scrum Master deu duas sonecas), o time se reúne em círculo — presencialmente ou numa videochamada onde 40% das pessoas estão com câmera desligada — para recitar os três votos sagrados:

1. **O que fiz ontem** (principalmente Slack, algum Stack Overflow, um commit)
2. **O que farei hoje** (igual ontem, mas com mais confiança)
3. **Impedimentos** (que ninguém vai realmente resolver)

Esse ritual se chama **Daily Standup**. É a destilação mais pura do teatro corporativo que a engenharia de software já produziu.

Já faço standups há 23 anos. Nunca fui desbloqueado por um standup.

## A Invenção do Standup

Conta a lenda que o standup foi inventado para ser *curto*. "Se as pessoas estiverem de pé, vão querer sentar logo," disse o consultor que vendeu essa ideia por R$4.000/hora.

Leitor: elas não estão de pé. Estão afundadas em cadeiras ergonômicas, olhando pela metade para um quadrado do Zoom mostrando o teto do quarto de alguém, no mudo porque o gato derrubou o suporte do microfone.

O "timebox de 15 minutos" é ficção. Todo standup expande para preencher o sofrimento disponível.

> *"Wally, quais são seus impedimentos?"*
> *"Sim."*
> — Dilbert, basicamente toda tira de 2001 a 2023

## Os Três Arquétipos que Você Encontra Todo Dia

### O Superachiever

```
"Ontem eu refatorei o módulo de autenticação, escrevi 12 testes unitários,
revisei 4 PRs, atualizei o wiki, liguei para o cliente, arrumei o pipeline
de CI e aprendi Rust. Hoje vou—"
```

Essa pessoa faz todos os outros sentirem que não existiram nas últimas 24 horas. É odiada. Será promovida. É a mesma coisa.

### O Filósofo

```
"Então, ontem... eu estava pensando sobre a arquitetura que discutimos no
Q3, e percebi — aliás, a gente pode abrir um Miro? Quero desenhar uma coisa.
É rapidinho."
```

Demora 47 minutos. O Scrum Master tem uma pequena crise nervosa.

### O Fantasma

```
[STATUS: Ausente]
[STATUS: Em reunião]
[STATUS: Não perturbe]
```

Essa pessoa ou está fazendo trabalho real ou já pediu demissão e ninguém percebeu. Ambos são válidos. Ambos são aspiracionais.

## O Impedimento Que Ninguém Remove

Aqui está o segredo sujo do standup: **impedimentos são mencionados mas nunca resolvidos.**

O standup não é uma sessão de resolução de problemas. É uma sessão de *anúncio de problemas*. Você fala o seu impedimento. As pessoas acenam com a cabeça. O Scrum Master digita no Jira, onde vai envelhecer como bom vinho — quietinho, no escuro, tornando-se cada vez mais complexo e azedo com o tempo.

```
CICLO DE VIDA DO IMPEDIMENTO:

Dia 1:  "Estou bloqueado no spec da API do design."
Dia 3:  "Ainda bloqueado."
Dia 7:  "Ainda bloqueado, mas dei um jeito."
Dia 14: "Que bloqueio? Resolvi sozinho."
Dia 30: "Virou feature. Entregamos semana que vem."
```

Você nunca precisou do standup. Precisava de 15 minutos de foco ininterrupto.

Veja também: [XKCD #303 — Compilando](https://xkcd.com/303/) *(o verdadeiro motivo pelo qual os standups existem)*

## O Relatório de Standup Que Você Realmente Quer Dar

O que as pessoas querem dizer versus o que dizem:

| O que você quer dizer | O que você realmente diz |
|---|---|
| "Passei 6 horas num rabbit hole do Kafka e não tenho nada para mostrar" | "Trabalhando no pipeline de eventos. Avançando bem." |
| "Não faço a menor ideia do que estou fazendo" | "Investigando alguns edge cases." |
| "Meu impedimento é que esse projeto está fundamentalmente quebrado" | "Problema menor de dependência. Deve resolver hoje." |
| "Joguei Factorio por 4 horas às 2am e estou destruído" | "Tudo certo!" |
| "Para que é tudo isso mesmo" | *[entra no Zoom, fica em silêncio]* |

## O Que os Standups Realmente Medem

O standup não mede produtividade. Mede **performance de produtividade** — e essas são coisas diferentes.

Um desenvolvedor que escreve 3 linhas perfeitamente posicionadas que eliminam uma race condition que estava custando R$200.000/mês em downtime não tem nada a dizer no standup.

Um desenvolvedor que abre 11 tickets, participa de 7 reuniões, atualiza 3 páginas do wiki e responde 400 mensagens no Slack tem um standup *incrível*.

Adivinhe qual recebe o bônus na avaliação de desempenho.

```python
def pontuacao_produtividade(resultado_real, performance_standup):
    # Cálculo clássico corporativo
    return performance_standup * 0.8 + resultado_real * 0.0 + vibe * 0.2
```

## A Estratégia Ótima de Standup

Depois de 47 anos sobrevivendo a standups, desenvolvi uma estratégia ótima:

**Passo 1:** Sempre fale em terceiro lugar. Primeiro é ansioso demais. Segundo ainda está acordando. Terceiro é perfeito — o Scrum Master já parou de prestar atenção.

**Passo 2:** Fale por exatamente 42 segundos. Longo o suficiente para parecer substantivo. Curto o suficiente para evitar perguntas de acompanhamento.

**Passo 3:** Cite um número de ticket. Pessoas respeitam números de ticket. Implica estrutura. Implica que você sabe o que está fazendo. Implica responsabilidade. Nada disso é necessariamente verdade.

**Passo 4:** Termine com "Sem impedimentos." Mesmo que tenha impedimentos. Porque se mencionar um impedimento, será convidado a "falar depois do standup", o que significa agendar *outra reunião* para não resolver seu impedimento em uma sala diferente.

**Passo 5:** Suma do Slack por 90 minutos após o standup. É quando o trabalho real acontece. O standup esgotou a energia social de todos. Aproveite o silêncio.

## A Heresia do Standup Assíncrono

Alguns times tentaram **standups assíncronos** — escrever sua atualização em uma thread do Slack ou bot. Isso é tratado por Scrum Masters como camponeses medievais tratavam eclipses: como um presságio de desgraça.

"Mas como vamos nos *conectar* como time?" eles perguntam.

Não vamos. Somos engenheiros de software. Nos conectamos por comentários de pull request e mensagens de commit passivo-agressivas. Isso é suficiente. Sempre foi suficiente.

> *"Não sou antissocial. Sou anti-reuniões-inúteis."*
> — Eu, na minha cabeça, todo dia às 9h47

## Uma Nota Sobre Ficar de Pé

Trabalhei em empresas que realmente obrigam as pessoas a ficarem de pé. Acreditam que o desconforto acelera a brevidade.

Não acelera. Só acelera a dor nas costas.

Um engenheiro em um emprego anterior instalou uma mesa em pé na sala de reuniões e dava atualizações de 20 minutos todos os dias. Foi promovido a Engenheiro Principal.

Sua coluna vertebral, no entanto, foi rebaixada.

## Conclusão: Para Que os Standups São Bons

Standups são excelentes para:
- ✅ Descobrir quem não fez push de código em 3 dias
- ✅ Lembrar em qual projeto você está
- ✅ Fazer o Scrum Master se sentir útil
- ✅ Fornecer 15 minutos de procrastinação estruturada
- ✅ O conforto ritual da rotina humana em uma codebase caótica

Standups não são bons para:
- ❌ Resolver impedimentos
- ❌ Coordenação
- ❌ Engenharia
- ❌ Nada que está no guia do Scrum

Mas vamos continuar fazendo. Porque parar os standups exigiria uma reunião para discutir parar os standups, e essa reunião precisaria de um standup para ser coordenada, e então nunca escaparíamos.

Veja também: [XKCD #1658 — Estimando Tempo](https://xkcd.com/1658/) *(quanto tempo o standup vai durar hoje?)*

---

*O autor está "impedido" desde 2017. Menciona isso todo dia às 9h47. Ninguém jamais ajudou.*
