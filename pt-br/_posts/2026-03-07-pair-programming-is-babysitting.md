---
layout: post
ref: pair-programming-is-babysitting
title: "Pair Programming é Babá: Eu Codifico Mais Rápido Sozinho"
date: 2026-03-07 08:06:00 -0300
categories: [carreira, práticas]
tags: [pair-programming, produtividade, trabalho-em-equipe, xp, agile]
permalink: /pt-br/:year/:month/:day/pair-programming-is-babysitting/
---

Tentaram me fazer pair programming uma vez. Uma vez. Passei a sessão inteira explicando conceitos básicos enquanto meu código ficava ali, não escrito, me julgando.

## A Matemática do Pair Programming

```
1 desenvolvedor = 1 unidade de produtividade
2 desenvolvedores = 2 unidades de produtividade (você pensaria)

MATEMÁTICA REAL:
1 desenvolvedor = 1 unidade de produtividade  
2 desenvolvedores = 0.3 unidades de produtividade + 3 horas de "deixa eu explicar..."
```

## Por Que Pareamento Não Funciona

| O Que Eles Dizem | O Que Realmente Acontece |
|------------------|--------------------------|
| "Dois pares de olhos pegam mais bugs" | Dois pares de olhos veem Netflix no segundo monitor |
| "Compartilhamento de conhecimento" | Eu compartilho, eles esquecem |
| "Melhor qualidade de código" | Discutimos sobre ponto e vírgula por 2 horas |
| "Reduz bus factor" | O ônibus parece cada vez mais atraente |
| "Code review em tempo real" | Julgamento em tempo real |

## Os Papéis

No pair programming, supostamente existe um "driver" e um "navegador." Na realidade:

```
Driver: Faz todo o trabalho
Navegador: Diz "eu teria feito diferente"
           Olha o celular
           Pega café
           "Pode scrollar pra cima?"
           "O que essa linha faz mesmo?"
           "Na verdade, scrolla pra baixo de novo"
```

## O Problema da Troca

"Vocês devem trocar de papel a cada 30 minutos!"

Sabe quanto tempo leva pra eu entrar no flow? Pelo menos 45 minutos. Toda vez que troco, recomeço do zero. Pair programming é só troca de contexto institucionalizada.

## O Overhead de Conversa

Codando sozinho:
```
Cérebro: "Deveria refatorar isso"
Dedos: *refatora*
```

Pair programming:
```
Cérebro: "Deveria refatorar isso"
Boca: "Ei, tô pensando em refatorar isso"
Eles: "Refatorar o quê?"
Boca: "Essa função aqui"
Eles: "Qual função?"
Boca: *aponta*
Eles: "Ah, por quê?"
Boca: *explica por 15 minutos*
Eles: "Sei lá, tá bom assim"
Cérebro: *grita internamente*
```

## Como o [XKCD 1319](https://xkcd.com/1319/) Mostra

Automação pode acabar levando mais tempo que trabalho manual. Similarmente, "colaboração" pode levar mais tempo que só fazer você mesmo.

Eu consigo escrever uma feature em 2 horas sozinho. Com um par? 6 horas de "discussão" seguidas de eu escrevendo em 2 horas enquanto eles "fazem uma call rápida."

## A Exaustão Social

Eu virei programador especificamente pra não ter que falar com pessoas o dia todo. Agora você quer que eu fale com alguém enquanto trabalho? Isso não é trabalho, é um podcast que eu não consenti.

```python
class Introvertido:
    def __init__(self):
        self.bateria_social = 100
    
    def pair_program(self, horas):
        self.bateria_social -= horas * 30
        if self.bateria_social < 0:
            raise ExcecaoBurnout("Por favor me deixa em paz")
```

## A Realidade Dilbert

O Chefe Cabelo Pontudo ama pair programming. Duas pessoas numa tarefa significa o dobro de justificativa de headcount! É matemática de gestão.

Enquanto isso, Dilbert e Wally "pareiam" sentando um do lado do outro, um codando e outro dormindo. Esse é o único pareamento que apoio.

## A Desculpa do "Aprendizado"

"Mas desenvolvedores júnior aprendem com seniores durante pareamento!"

Eles também poderiam aprender com:
- Documentação (se a gente escrevesse)
- Code reviews (depois que eu terminar)
- Stack Overflow (habitat natural deles)
- Sofrer sozinho (constrói caráter)

## Pareamento Sênior-Sênior

A única coisa pior que parear com júnior é parear com outro sênior:

```
Dev A: "Vou usar factory pattern aqui"
Dev B: "Na verdade, prefiro builder"
Dev A: "Factory é mais flexível"
Dev B: "Builder é mais legível"
*3 horas depois*
*Nenhum código escrito*
*Ambos abriram o LinkedIn*
```

## O Pesadelo do Mob Programming

Pair programming não era ruim o suficiente, então inventaram "mob programming" — onde o time inteiro trabalha em um computador.

Isso não é programação. É um esporte de espectadores onde ninguém ganha.

## Meu Setup Preferido de Pareamento

```
Monitor 1: Meu código
Monitor 2: Meu outro código
Fones: Dentro
Slack: Não Perturbe
Porta: Fechada
"Par": Pato de borracha imaginário
```

## Conclusão

O melhor código é escrito em silêncio, no escuro, às 2 da manhã, sem ninguém por perto pra sugerir uma abordagem diferente.

Pair programming assume que colaboração melhora resultados. Mas sabe o que também melhora resultados? Ser deixado em paz.

---

*A última sessão de pair programming do autor terminou com "diferenças criativas." O par era um espelho.*
