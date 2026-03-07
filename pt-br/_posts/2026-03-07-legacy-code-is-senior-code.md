---
layout: post
ref: legacy-code-is-senior-code
title: "Código Legado é Código Sênior: Respeite os Mais Velhos"
date: 2026-03-07 08:03:00 -0300
categories: [arquitetura, filosofia]
tags: [legado, refatoração, manutenção, qualidade-de-código, sabedoria]
permalink: /pt-br/:year/:month/:day/legacy-code-is-senior-code/
---

Todo desenvolvedor júnior quer reescrever o sistema legado. Todo desenvolvedor sênior sabe que o sistema legado *sobreviveu*. Isso é mais do que a maioria de nós pode dizer.

## A Hierarquia do Código

```
        🏛️ Código Ancião (10+ anos)
           "Ele roda a empresa"
                  ⬆️
        🏢 Código Legado (5-10 anos)
           "Ninguém entende mas funciona"
                  ⬆️
        🏠 Código Maduro (2-5 anos)
           "Devíamos refatorar isso"
                  ⬆️
        🏗️ Código Novo (< 2 anos)
           "Não acredito que shipamos isso"
                  ⬆️
        📝 Código Que Você Tá Escrevendo
           "Dessa vez vai ser diferente"
```

## Por Que Código Legado Merece Respeito

| O Que Júniors Veem | O Que Realmente Aconteceu |
|--------------------|---------------------------|
| "Isso é uma bagunça" | Sobreviveu 3 migrações de framework |
| "Sem testes" | Testes eram de outra era |
| "Sem docs" | A doc era o dev (que saiu) |
| "Números mágicos" | Cada um tem uma história (geralmente trágica) |
| "Por que PHP?" | Era cutting edge em 2003 |
| "Código espaguete" | Lógica artesanal feita à mão |

## Os Comentários Sagrados

Os melhores comentários de código são encontrados em sistemas legados:

```php
<?php
// Não sei por que isso funciona, mas funciona
// Não mexe nisso. Sério. Tô falando sério.
// O João escreveu isso antes de sair. Saudades João.

// TODO: Arrumar isso (adicionado 14/03/2007)

// HACK: Isso corrige o bug do Y2K que achamos em 2015

// Se você tá lendo isso, sinto muito.
// Se você tá modificando isso, sinto mais ainda.

// Essa função se chama "process" porque
// não sabíamos o que ela fazia quando escrevemos.
// Ainda não sabemos.
```

## O Efeito Darwin

Como o [XKCD 2347](https://xkcd.com/2347/) mostra, infraestrutura crítica frequentemente depende de código mantido por uma pessoa aleatória. Isso é porque aquele código *sobreviveu*. Ele evoluiu. Se adaptou.

Seus microserviços novinhos? 47% não vão passar do Ano 2.

Aquele sistema COBOL de 1987? Tá processando mais transações que seu cluster Kubernetes inteiro.

## Refatoração é Desrespeito

Quando você refatora código legado, você tá dizendo "Eu sei mais que os 47 desenvolvedores que vieram antes de mim, cada um dos quais achava que ELES sabiam mais."

Spoiler: Você não sabe.

```java
// Antes: Código legado
public Object fazCoisa(Object coisa, int magic, boolean flag1, boolean flag2) {
    // 3000 linhas de lógica testada em batalha
}

// Depois: Sua "refatoração"
public Output processa(Input input) {
    // 200 linhas que não tratam edge cases
    // que o desenvolvedor original descobriu
    // através de anos de incidentes em produção
}
```

## O Chefe Cabelo Pontudo Estava Certo

No Dilbert, o PHB frequentemente toma decisões que parecem absurdas. Mas manter sistemas legados rodando? Isso é sabedoria disfarçada de incompetência.

"Não vamos atualizar porque funciona" é uma estratégia técnica válida. Chama-se "estabilidade."

## Os Testes Estão em Produção

"Mas não tem testes unitários!" você chora.

Os testes são os milhões de transações que passaram por esse código. Os testes são os incidentes de produção que não aconteceram. Os testes são o fato de que a empresa ainda existe.

```bash
# Estratégia de testes legado
1. Deploy
2. Espera ligações
3. Sem ligações = sucesso
4. Ligações = atualiza o conhecimento que vai se perder quando você sair
```

## O Verdadeiro Significado de Dívida Técnica

Todo mundo fala de "dívida técnica" como se fosse ruim. Mas dívida real pode ser alavancada. Aquele sistema legado? É PATRIMÔNIO. Tá PAGO.

Seu monorepo TypeScript novo? Isso é uma hipoteca que você ainda tá pagando.

## Dicas de Sobrevivência

Quando trabalhando com código legado:

1. **Não pergunte "por quê"** - A resposta é sempre dor
2. **Não refatore "já que você tá ali"** - Você não tá ali, você tá perdido
3. **Confie nos comentários** - Foram escritos por alguém que sofreu
4. **Tema o silêncio** - Código sem comentário esconde os piores segredos
5. **Backup antes de tocar em qualquer coisa** - Na verdade, talvez não toque em nada

## Conclusão

Sua aplicação moderna, bem testada, documentada, com microserviços, um dia vai ser chamada de "legado" por algum desenvolvedor júnior que acha que sabe mais.

Ele não vai saber. E você também não.

O código que roda é o código que importa. Idade é só um número. Um número muito, muito assustador.

---

*O autor mantém um sistema FORTRAN de 1974. Ele nunca crashou. O autor crashou muitas vezes.*
