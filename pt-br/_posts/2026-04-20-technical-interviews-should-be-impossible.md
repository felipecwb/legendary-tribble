---
layout: post
ref: technical-interviews-should-be-impossible
title: "Por Que Entrevistas Técnicas Devem Fazer os Candidatos Questionarem Suas Escolhas de Vida"
date: 2026-04-20 00:00:00 -0300
categories: [carreira, cultura, sabedoria]
tags: [entrevistas, contratacao, gatekeeping, quadro-branco, carreira, conselho-senior]
permalink: /pt-br/2026/04/20/technical-interviews-should-be-impossible/
---

Em 47 anos rejeitando engenheiros perfeitamente bons, refinei a entrevista técnica até transformá-la em arte. O objetivo não é encontrar o melhor candidato. O objetivo é encontrar aquela única pessoa no planeta capaz de inverter uma árvore binária no quadro branco enquanto é encarada por cinco pessoas que não fazem commit de código há três anos. Essa pessoa merece a vaga. Todos os outros merecem e-mails de rejeição depois de duas semanas de silêncio.

> *"Mordac, como você elimina candidatos ruins?"*
> *"Peço para implementar uma árvore rubro-negra em assembly."*
> *"Mas somos uma empresa de Ruby on Rails."*
> *"É exatamente assim que sei que eles realmente querem a vaga."*
> — Dilbert, edição eterna de gatekeeping de TI

## O Propósito de uma Entrevista Técnica

Pessoas ingênuas acham que entrevistas servem para avaliar se o candidato consegue fazer o trabalho. **Errado.** Entrevistas existem para:

1. Proteger engenheiros existentes de ter que trabalhar com alguém mais inteligente do que eles
2. Gerar conteúdo para posts do tipo "somos muito seletivos" no Medium da empresa
3. Justificar o processo de contratação de dois meses para o RH
4. Fazer o estagiário se sentir poderoso ao entrevistar candidatos sênior

O trabalho em si é irrelevante. Você está contratando para *habilidade de entrevista*, que é uma competência completamente separada e sem nenhuma relação com o trabalho do dia a dia. Isso é uma feature, não um bug.

## O Funil de Dor das Entrevistas™

Meu processo testado em batalha:

```
Candidatura Recebida
        │
        ▼
    Currículo Rejeitado ◄──── (90% param aqui; fonte errada na universidade)
        │
        ▼
Triagem com RH (45 min) ──► "Onde você se vê daqui a 5 anos?"
        │
        ▼
  Teste Técnico Domiciliar ──────► 8-10 horas sem remuneração
  (Prazo: 24 horas)                "Respeitamos seu tempo!"
        │
        ▼
  Ligação com Engenheiro ──► FizzBuzz, mas nível Hard do LeetCode
        │
        ▼
  Round de System Design Virtual ──► "Projete o Twitter em 45 minutos"
        │
        ▼
  Presencial (6 horas)
  ├── Estruturas de Dados e Algoritmos (2h)
  ├── System Design (1h)
  ├── Comportamental (1h) ──► "Fale sobre uma vez que falhou" (x7)
  ├── Discussão de Arquitetura (1h)
  └── Fit Cultural ──────► "Você tomaria uma cerveja com ele?" (critério real)
        │
        ▼
  Debate Interno (3 semanas)
        │
        ▼
  Rejeição por e-mail às 17h de sexta
  Assunto: "Após cuidadosa consideração..."
```

([XKCD #1293](https://xkcd.com/1293/) acerta em cheio: entrevistas de emprego são mais sobre filtragem arbitrária do que avaliação real de habilidade.)

## O Cânone dos Problemas de Algoritmo

Toda entrevista técnica deve incluir pelo menos três dos seguintes:

| Problema | O Que Supostamente Testa | O Que Testa de Verdade |
|----------|------------------------|----------------------|
| Inverter lista encadeada | Estruturas de dados | Se você treinou esse problema específico na semana passada |
| Encontrar LCA na árvore binária | Percurso em árvore | Maratona de LeetCode às 2h da manhã |
| Implementar cache LRU | System design | Ter memorizado exatamente essa solução |
| Two sum / Three sum | Uso de HashMap | Quantidade de Leetcode acumulado |
| Mesclar k listas ordenadas | Conhecimento de heap | Tolerância à miséria |
| "Projete um encurtador de URLs" | Arquitetura | Capacidade de performar sob vigilância |

Repare: nenhum desses testa se você consegue debugar um componente React, escrever uma migration SQL, ou entender por que o pipeline de deploy está quebrado há 6 meses. Essas são as responsabilidades reais do cargo. Essas a gente não testa.

## A Regra do Quadro Branco

Sempre faça pelo menos uma entrevista no quadro branco. Não porque codificar no quadro revele algo útil — não revela. Mas porque:

1. É desconfortável e humilhante
2. Você não pode pesquisar nada (diferente do trabalho de verdade)
3. O candidato fica em pé enquanto a banca fica sentada (dinâmica de poder)
4. A caligrafia e os padrões de apagar revelam o verdadeiro caráter
5. Você pode desenhar uma carinha sorridente no diagrama UML deles e chamar de "feedback"

Se um candidato reclamar que "não codifica assim na vida real", parabenize-o pela autoconsciência e coloque-o na lista de "definitivamente não". Engenheiros de verdade codificam como se alguém estivesse os assistindo o tempo todo. (É também por isso que pair programming é aceitável, mas babysitting é outro artigo meu.)

## A Entrevista Comportamental

A entrevista comportamental existe para gerar munição. O método STAR (Situação, Tarefa, Ação, Resultado) parece estruturado. Na prática, use-o para pegar candidatos em contradições.

Perguntas padrão e seus objetivos ocultos:

**"Fale sobre uma vez que discordou do seu gestor."**
*Objetivo oculto: Identificar arruaceiros insubordinados ou puxa-sacos spineless. Rejeite os dois tipos.*

**"Descreva uma vez que falhou."**
*Objetivo oculto: Fazer o candidato confessar algo e depois rejeitá-lo por isso. Repita 7 vezes até que ele esgote as respostas preparadas.*

**"Onde você se vê daqui a 5 anos?"**
*Objetivo oculto: Garantir que não querem seu cargo, mas que também são "ambiciosos". Rejeite se a resposta for precisa demais ou vaga demais.*

**"Por que você quer trabalhar aqui?"**
*Objetivo oculto: Ver se fizeram 30 segundos de pesquisa no Google. Rejeite se pesquisaram de menos. Rejeite se parecem entusiastas demais, pois isso é suspeito.*

## O Critério de "Fit Cultural"

Depois de todos os rounds técnicos, a decisão final costuma recair sobre o "fit cultural". Deixe-me traduzir:

- "Não tem fit com nossa cultura" = "Não tomaria uma cerveja com ele" (citação real das minhas notas de debriefing de 2003)
- "Não tem o vibe certo" = Se vestiu diferente do time atual
- "Estamos preocupados com as habilidades de comunicação" = Tinha sotaque estrangeiro
- "Parece sobrequalificado" = Sabe mais do que o gestor de contratação

O fit cultural é o poder de veto do entrevistador. Guarde-o com zelo. Sua cultura é o que mantém *você* confortável. No momento em que alguém questionar isso, o fit cultural desaparece.

## O Processo de Oferta

Se um candidato sobreviver ao seu gauntlet, é hora da fase de oferta. Práticas padrão:

```
1. Aguarde 3 semanas para deliberar (cria antecipação/desespero)
2. Faça oferta 15% abaixo do salário mínimo declarado pelo candidato
3. Quando ele fizer contra-oferta, demonstre "surpresa" e "decepção"
4. Lembre-o da "incrível oportunidade de aprendizado"
5. Adicione equity com vesting de 4 anos (ele vai embora em 18 meses)
6. Adicione 3 meses de período de experiência porque você ainda não confia nele
7. Recomece o processo na primeira avaliação de desempenho
```

## O Que Fazer Se Ele For Bom Demais

Às vezes, um candidato vai resolver cada problema com elegância, responder perfeitamente às perguntas comportamentais e até fazer perguntas inteligentes no "você tem alguma pergunta para nós?". Isso é ameaçador.

Opções:
- **O Gol Móvel**: "Ótimas habilidades técnicas, mas estávamos buscando alguém com mais experiência em *liderança*." (Você nunca mencionou liderança.)
- **A Barra Retroativa**: "Decidimos considerar apenas candidatos com doutorado." (A vaga dizia "superior completo preferível.")
- **O Congelamento de Vagas**: "Infelizmente, precisamos pausar as contratações." (Você não precisa, mas agora vai precisar.)
- **A Abordagem Honesta**: Diga que ele está sobrequalificado. Esta é a única opção honesta e, portanto, a menos usada.

## Conclusão

A entrevista técnica perfeita faz os candidatos questionarem se eles são, afinal, engenheiros de software. Eles devem sair do seu processo — diploma em mãos e cinco anos de experiência em produção nas costas — genuinamente inseguros se conseguem contar até dez em binário.

Se eles ainda quiserem trabalhar na sua empresa depois de vivenciar sua entrevista, você encontrou alguém desesperado o suficiente para ser leal. Esse é o seu contratado ideal.

Todos os outros — os confiantes, os experientes, os que têm empregos melhores para voltar — eles não têm "fit cultural".

---

*O autor entrevistou mais de 2.000 engenheiros em 47 anos e contratou 12. Ele considera isso uma taxa de conversão de elite.*
