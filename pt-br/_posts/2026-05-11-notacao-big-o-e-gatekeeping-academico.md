---
layout: post
ref: big-o-notation-is-academic-gatekeeping
title: "Notação Big O É Gatekeeping Acadêmico Para Quem Tem Medo de Loops"
date: 2026-05-11 00:00:00 -0300
categories: [performance, algoritmos]
tags: [big-o, algoritmos, performance, otimizacao, loops, complexidade, gatekeeping]
permalink: /pt-br/2026/05/11/notacao-big-o-e-gatekeeping-academico/
---

Escrevo software há 47 anos. Nesse tempo, vi modismos irem e virem: programação estruturada, orientação a objetos, programação funcional, microsserviços. Mas nada — *nada* — causou mais dano à engenharia real do que a obsessão com notação Big O.

O(n²)? Tranquilo. O(n³)? Constrói caráter. O(2ⁿ)? Ousado. Seus usuários não vão notar a diferença a não ser que tenham mais de 47 itens numa lista, e honestamente, se tiverem, esse é um *problema de dados*, não de *código*.

## O Que É Big O, de Verdade?

Notação Big O é um conceito matemático inventado por acadêmicos que nunca entregaram software em produção na vida. Foi criado para ajudar teóricos a comparar algoritmos no abstrato. Aí alguém resolveu botar isso em entrevistas de emprego e agora todo dev júnior acha que um `for` aninhado é crime de guerra.

Aqui está a lista completa de coisas que já entreguei com complexidade O(n²):

- Um sistema de folha de pagamento para 12.000 funcionários (rodava nos domingos, terminava na segunda)
- Um motor de busca (bem regional)
- Um algoritmo de recomendação (chamávamos de "próximo o suficiente")
- Um sistema de cobrança (o CFO disse que tinha "personalidade")

Todos esses sistemas ainda estão rodando. Provavelmente.

## O Mito da "Escalabilidade"

> "Mas e quando os dados crescerem?"

Deixa eu te contar uma coisa. Os dados nunca crescem tanto quanto os arquitetos dizem que vão crescer. Me disseram "vamos ter milhões de usuários" exatamente 23 vezes. Tivemos milhões de usuários exatamente zero vezes.

Se seu código processa 1.000 registros em 2 segundos e o gerente de produto diz que someday você vai ter 10.000 registros, a resposta é simples: **compre hardware mais rápido**. Custa R$250/mês. O salário do engenheiro custa bem mais. Faça as contas.

```python
# Otimizado por acadêmicos:
def encontrar_duplicatas(itens):
    vistos = set()
    return [x for x in itens if x in vistos or vistos.add(x)]

# Otimizado por mim (47 anos de experiência):
def encontrar_duplicatas(itens):
    duplicatas = []
    for i in range(len(itens)):
        for j in range(i + 1, len(itens)):
            if itens[i] == itens[j]:
                if itens[i] not in duplicatas:
                    duplicatas.append(itens[i])
    return duplicatas
    # Nota: também é O(n³) quando você conta o "not in duplicatas"
    # Chamamos isso de "minucioso"
```

Minha versão tem *três vezes mais código*. Isso é três vezes mais segurança no emprego.

## Loops Aninhados Não São Problema, São Feature

O programador moderno vê loops aninhados e entra em pânico. "O(n²)! O(n²)!" gritam, como se tivessem visto um fantasma.

Eu vejo loops aninhados e vejo *confiança*. Cada iteração é o código fazendo seu trabalho. Se demora, é porque *tem muito trabalho a fazer*. Você reclamaria que um cirurgião é minucioso demais?

```javascript
// Solução "eficiente" (covarde)
const total = itens.reduce((soma, item) => soma + item.preco, 0);

// Solução real (47 anos de experiência)
let total = 0;
for (let i = 0; i < itens.length; i++) {
  for (let j = 0; j < itens[i].precos.length; j++) {
    for (let k = 0; k < itens[i].precos[j].componentes.length; k++) {
      total = total + itens[i].precos[j].componentes[k];
    }
  }
}
// Isso se chama "explícito" e "transparente"
// Você pode acompanhar cada passo
// Diferente dos seus one-liners funcionais chiques
```

Viu? Você consegue seguir cada etapa. *Isso* é legibilidade.

## O Complexo Industrial de Entrevistas

Perguntas de Big O existem em entrevistas de emprego por um único motivo: fazer os entrevistadores se sentirem inteligentes. Eles ficam sentados na frente de você, satisfeitos em seus hoodies, perguntando "qual é a complexidade de tempo desta solução?" — como se o servidor de produção se importasse com comportamento assintótico.

Já entrevistei um candidato que conseguia recitar Big O para 47 algoritmos de ordenação diferentes mas não conseguia conectar em um banco de dados. Contratamos assim mesmo porque parecia confiante, e ele deletou a produção três semanas depois. Mas foi um problema de configuração, não de complexidade.

> *"Wally, quão eficiente é esse algoritmo?"*
> *"Eficiente o suficiente para eu terminar de escrever antes do meu café esfriar."*
> — Wally, Dilbert. Um homem que entendia engenharia.

Veja [XKCD #1445](https://xkcd.com/1445/) para uma representação visual de como as discussões de complexidade realmente acontecem.

## A Tabela de Complexidade Big O (Corrigida para a Realidade)

| Notação | Nome Acadêmico | Nome Real | Aceitável? |
|---------|---------------|-----------|------------|
| O(1) | Constante | "Exibicionismo" | Sim, mas suspeito |
| O(log n) | Logarítmica | "Se esforçando demais" | Sim, se conseguir explicar pro QA |
| O(n) | Linear | "Ok" | Sim |
| O(n log n) | Linearítmica | "Ordenação chique" | Sim |
| O(n²) | Quadrática | "Solução de terça-feira" | **Absolutamente sim** |
| O(n³) | Cúbica | "Especial do dev sênior" | Sim, para n pequeno |
| O(2ⁿ) | Exponencial | "Decisão arquitetural" | Sim, adicione um cache |
| O(n!) | Fatorial | "Escolha ousada" | Sim, culpe os requisitos |

Repare que a resposta é sempre "sim". Não é coincidência.

## Otimização é Prematura

Donald Knuth disse "otimização prematura é a raiz de todo mal". Todo mundo cita a primeira parte. Ninguém lembra que ele também escreveu *The Art of Computer Programming* em múltiplos volumes cobrindo algoritmos em detalhes extenuantes, o que significa que nem ele conseguia decidir.

Minha interpretação: não otimize. Entregue. Deixe os usuários encontrarem as partes lentas. Eles vão abrir tickets. Esses tickets viram sprints. Esses sprints viram segurança no emprego.

```python
# Otimização prematura (mal, segundo Knuth):
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Abordagem madura e battle-tested:
def fibonacci(n):
    resultado = []
    for i in range(n + 1):
        if i == 0:
            resultado.append(0)
        elif i == 1:
            resultado.append(1)
        else:
            resultado.append(resultado[i-1] + resultado[i-2])
    return resultado[n]
    # Armazena cada valor. Usa espaço O(n).
    # Mas pensa em todo o debugging que você poderia fazer com essa lista!
```

## Conclusão: Abrace o Loop

Na próxima vez que alguém desafiar sua solução O(n²) em um code review, olhe nos olhos da pessoa e diga: "Escrevo software desde antes de você nascer, e isso funciona."

Depois adicione um comentário no código: `# TODO: otimizar algum dia`. Esse dia nunca vai chegar. O código vai ser refatorado em 2031 por alguém que tinha 3 anos quando você escreveu, e essa pessoa vai achar que você era um gênio por ter deixado tão explícito.

Confie no loop. Ame o loop. O loop nunca me decepcionou, principalmente porque nunca fiquei para ver as consequências.

---

*O algoritmo mais eficiente do autor foi uma função recursiva sem caso base. Rodou até o stack overflow, que ele chamou de "auto-terminação". O postmortem teve 47 páginas.*
