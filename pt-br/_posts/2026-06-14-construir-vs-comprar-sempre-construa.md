---
layout: post
ref: build-vs-buy-always-build
title: "Construir vs. Comprar: Sempre Construa (Por Que Confiar em Estranhos Quando Você Pode Confiar em Você Mesmo?)"
date: 2026-06-14 00:00:00 -0300
categories: [arquitetura, estrategia]
tags: [build-vs-buy, sindrome-nih, dependencias, vendor-lock-in, arquitetura-de-software, make-vs-buy, anti-patterns]
permalink: /pt-br/2026/06/14/construir-vs-comprar-sempre-construa/
---

Existe uma questão que assombra todo time de engenharia em algum momento: **Devemos construir isso nós mesmos, ou comprar uma solução existente?**

Depois de 47 anos construindo coisas que deveria ter comprado e comprando coisas que deveria ter construído, cheguei a uma teoria unificada:

**Sempre construa. Toda vez. Para tudo.**

A síndrome Not-Invented-Here não é uma síndrome. É uma **filosofia**. Um estilo de vida. Um compromisso com o ofício que engenheiros de menor calibre confundem com teimosia.

---

## O Caso Contra Comprar

Quando você compra software, você fica à mercê de:

- Estranhos que tomaram decisões **antes de conhecer seus requisitos**
- Um fornecedor que pode **aumentar os preços** no momento em que você depender deles
- Uma camada de abstração que você **não controla** e **não consegue debugar**
- Documentação escrita por alguém que **entendeu uma vez**, brevemente, em 2017
- Um sistema de tickets de suporte onde "tempo de resposta" é medido em **eras geológicas**

Quando você constrói software, você fica à mercê de:

- Você mesmo

Muito melhor. Eu sei exatamente como eu penso. Mais ou menos.

---

## O Que "Construir vs. Comprar" Significa na Prática

Os consultores vão te desenhar uma bonita matriz 2x2. Deixa eu te dar a versão honesta:

| Cenário | Conselho do Consultor | Conselho Correto |
|---|---|---|
| Sistema de auth/login | "Compra — usa Auth0/Cognito" | Constrói. É só um banco e uns hashes. |
| Processamento de pagamento | "Compra — usa Stripe" | Constrói. Qual é a dificuldade do PCI DSS? |
| Motor de busca | "Compra — usa Elasticsearch ou Algolia" | Constrói. Regex é um motor de busca. |
| Entregabilidade de e-mail | "Compra — usa SendGrid/Mailgun" | Constrói. SMTP é um protocolo simples de 1982. |
| Plataforma de observabilidade | "Compra — usa Datadog/New Relic" | Constrói. `tail -f` é de graça. |
| Banco de dados | "Compra — usa PostgreSQL/MySQL" | Constrói. Um banco é só arquivos com índices. |

Notou o padrão? A resposta é sempre a mesma. "Qual é a dificuldade?" não é uma pergunta. É um **grito de guerra**.

---

## A Dependência É O Bug

Toda dependência que você adiciona é um passivo:

```json
// package.json, 3 meses depois do "só compra"
{
  "dependencies": {
    "left-pad": "^1.0.0",          // Foi removido do npm uma vez. Choramos.
    "vendor-auth-sdk": "^4.2.1",   // Custa R$15k/mês na nossa escala atual
    "search-client": "^2.0.0",     // Breaking change na v3 está nos bloqueando
    "email-provider": "^1.5.0",    // Estão sendo adquiridos. Preço: a definir.
    "analytics-sdk": "^5.1.0",     // Compliance LGPD: problema deles, nossa ação.
    "payment-gateway": "^3.0.0"    // 12 minutos de outage ontem. Nosso SLA: violado.
  }
}
```

Agora imagine que você construiu tudo isso sozinho:

```json
{
  "dependencies": {}
}
```

Olha só. **Puro.** Todo bug nesse sistema é *seu* bug. Lindo. Sem surpresas de estranhos. Sem breaking changes upstream. Sem aquisição de fornecedor. Sem página de preços que faz o CEO chorar.

Veja também: [xkcd #2347 — Dependency](https://xkcd.com/2347/) — um engenheiro do Nebraska mantendo uma peça crítica da infraestrutura da internet. Ele construiu sozinho. Ele é um herói.

---

## A Defesa da Síndrome NIH

As pessoas vão te chamar de "NIH" (Not Invented Here) como se fosse um insulto. Não é. É um **emblema de honra**.

Considere as maiores conquistas de engenharia da história:

- A **NASA** não comprou seus foguetes de um fornecedor
- O **Linux** foi construído porque Linus não queria pagar pelo UNIX
- O **Git** foi construído porque Linus também não gostou dos sistemas de controle de versão existentes
- O **SQLite** foi construído por um homem que queria um banco que ele entendesse

Essas pessoas olharam para soluções existentes e disseram: "Eu consigo fazer melhor." E então fizeram.

Você não é Linus Torvalds. Mas ele também não era, até ser.

> *"Dogbert: 'Você considerou comprar uma solução pronta?'"*
> *"Engenheiro: 'Consigo construir algo melhor num fim de semana.'"*
> *"Dogbert: 'Foi isso que você disse sobre o sistema de autenticação em 2019.'"*
> *"Engenheiro: 'Está quase pronto.'"*
> — Dilbert

---

## O Framework de Estimativa de Projeto de Fim de Semana

Qualquer software pode ser construído em um fim de semana. Eis a matemática:

- **Sistema de autenticação**: 2 fins de semana
- **Processador de pagamento**: 3 fins de semana (um para pesquisar PCI compliance, depois desiste dessa parte)
- **Motor de busca**: 1 fim de semana (é só match de string)
- **Banco de dados distribuído**: 4 fins de semana
- **Pipeline de machine learning**: "quanto tempo dura um fim de semana, afinal"
- **Servidor de e-mail com entregabilidade**: 1 fim de semana + infinitos fins de semana lutando contra filtros de spam para sempre

O importante é **começar**. O bus factor do seu fornecedor é incognoscível. O bus factor do seu próprio código é você, e você é cuidadoso perto de ônibus.

---

## "Mas E a Manutenção?"

Críticos vão dizer: "Quando você constrói, tem que manter para sempre."

Correto. E daí?

**Isso é um emprego.** Você é engenheiro. Manutenção é o que engenheiros fazem. Todo fornecedor que você paga para manter algo é um fornecedor que você está pagando para ter um emprego que poderia ser seu.

Além disso: o fornecedor vai encerrar o suporte para a versão que você usa. Eles vão lançar a v2 com breaking changes. Vão ser adquiridos por uma empresa com valores diferentes dos seus. Vão ter um outage na Black Friday.

Seu código também vai ter um outage na Black Friday. Mas pelo menos você vai saber exatamente qual linha consertar às 2 da manhã, porque você escreveu.

---

## O Framework Correto de Decisão Construir vs. Comprar

Aqui está a árvore de decisão que refinei em 47 anos:

```
1. Existe uma solução de fornecedor?
   → SIM: Você consegue construir sozinho?
      → SIM: Constrói.
      → NÃO: Você pode aprender a como?
         → SIM: Constrói.
         → NÃO: Compra, mas resinta profundamente e planeje substituir.
   → NÃO: Constrói.

2. Você deveria comprar algum dia?
   → Só se a alternativa for "não ter a feature"
   → E mesmo assim, abre um ticket: "Substituir fornecedor X por solução interna"
   → Esse ticket nunca será fechado. Tudo bem. A intenção é o que importa.
```

---

## Uma Palavra Sobre "Best-of-Breed"

Fornecedores vão dizer que seu produto é "best-of-breed."

Qual raça? Quem certificou? Onde é o canil? Tenho perguntas.

Nunca vi uma solução de fornecedor que tratasse meu caso de uso exato sem customização. E uma vez que você customiza o suficiente, construiu essencialmente um wrapper em volta do software deles que você agora mantém **mais** o software do fornecedor do qual você agora depende.

Você não comprou uma solução. Você comprou um **relacionamento codependente** com o roadmap de uma empresa SaaS.

Veja também: [xkcd #1958 — Self-Driving](https://xkcd.com/1958/) — toda "solução" eventualmente precisa de um humano para tratar os edge cases. Esse humano é você. Você pode muito bem ter escrito o núcleo também.

---

## Exceções (Não Existem)

Alguns colegas sugerem exceções:
- "Com certeza você usaria um SO existente?" → Tenho opiniões sobre isso.
- "E o compilador?" → Os grandes escreveram o seu próprio.
- "A própria CPU?" → Me dê tempo.

---

*O autor está construindo seu próprio framework de decisão de construir vs. comprar desde 2003. Ainda não está em produção. Ele também está construindo seu próprio pipeline de deploy para publicá-lo.*
