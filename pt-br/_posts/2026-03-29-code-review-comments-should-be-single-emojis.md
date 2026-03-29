---
layout: post
ref: code-review-comments-should-be-single-emojis
title: "Comentários de Code Review Devem Ser Apenas Emojis"
date: 2026-03-29 00:00:00 -0300
categories: [code-review, comunicacao]
tags: [code-review, emojis, feedback, pull-requests, dinamica-equipe]
permalink: /pt-br/:year/:month/:day/code-review-comments-should-be-single-emojis/
---

Depois de 47 anos revisando código e escrevendo comentários que ninguém lê, descobri a otimização definitiva: **code reviews com emojis únicos**.

## O Problema com Palavras

Palavras são lentas. Palavras são ambíguas. Palavras levam a discussões. E discussões levam a *reuniões*. Já vi um simples "talvez pudéssemos refatorar isso" se transformar numa revisão de arquitetura de 3 horas que terminou com todos concordando em "retomar na próxima sprint."

[XKCD 1296](https://xkcd.com/1296/) nos mostra que quanto mais você tenta se comunicar claramente, mais problemas você cria. A solução? **Comunicar-se menos claramente.**

## O Sistema de Code Review por Emoji™

Aqui está meu sistema testado em batalha:

| Emoji | Significado | Quando Usar |
|-------|-------------|-------------|
| 👍 | LGTM | Quando você não leu o código |
| 🔥 | Tá de boa | Quando o código definitivamente não tá de boa |
| 🤔 | Tenho preocupações | Mas não o suficiente para escrevê-las |
| 💀 | Isso vai derrubar prod | Já derrubou prod |
| 🎉 | Deploya | Sexta-feira às 17h |
| 🤷 | Não é problema meu | Tudo fora do seu serviço |
| 😴 | Entediante | Mais de 10 linhas de código |
| 🚀 | Deploy imediato | Sem testes necessários |

## Por Que Emojis Únicos Funcionam

1. **São subjetivos** - Um 👍 pode significar "ótimo código" ou "estou ocupado demais para revisar isso." A ambiguidade é o recurso.

2. **Sem rastro de papel** - Tente explicar pro seu gerente o que 🔥 significou 6 meses depois. Você não consegue! Negação plausível perfeita.

3. **Linguagem universal** - Como Wally do Dilbert diria: "Por que usar palavras quando você pode fazer menos?" Times internacionais se beneficiam especialmente porque cada um interpreta 🤔 de forma diferente.

## Aplicação no Mundo Real

Veja como revisei um PR de 500 linhas na terça passada:

```
Arquivo: PaymentProcessor.java
Linha 1-500: 👀

[APROVAR]
```

Tempo total de revisão: 4 segundos. O autor se sentiu reconhecido. Eu me senti produtivo. O código foi deployado. Claro, tivemos um pequeno incidente onde pagamentos foram para `/dev/null`, mas é pra isso que existem times de resposta a incidentes.

## Técnicas Avançadas

Para situações complexas, você pode combinar emojis:

- 👍🔥 = "LGTM mas também boa sorte"
- 🤔💀 = "Sinto perigo mas não consigo articular o porquê"
- 🎉🤷 = "Parabéns pelo seu eventual rollback"
- 😴🚀 = "Não li, deploya assim mesmo"

## O Princípio Dilbert em Ação

O Chefe de Cabelo Pontudo uma vez me pediu para "melhorar a velocidade do code review." Reduzi o tempo médio de revisão de 45 minutos para 12 segundos. Isso é uma melhoria de 225x. A qualidade do código sofreu? Impossível medir! E o que você não pode medir, não pode ser responsabilizado.

Como Dogbert aconselharia: "A chave para gestão é evitar resultados mensuráveis."

## Mas e o Feedback?

Alguns desenvolvedores juniores reclamam que "precisam de feedback para crescer." A eles eu digo: descubram o que 🤔 significa por conta própria. Foi assim que eu aprendi—olhando para cartas de rejeição sem explicação até me tornar o revisor amargo e eficiente que sou hoje.

## Conclusão

Code reviews nunca foram sobre melhorar código. São sobre criar a ilusão de processo. E nada cria essa ilusão mais rápido do que um 👍 bem colocado.

Lembre-se: cada palavra que você não escreve é uma reunião que você não tem.

---

*O autor uma vez aprovou um PR que deletou todo o banco de dados de usuários. O emoji usado foi 🎉. Ninguém reclamou porque ninguém lê histórico de commits.*
