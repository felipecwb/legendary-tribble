---
layout: post
ref: code-reviews-are-waste
title: "Code Review É Só Gatekeeping Socialmente Aceitável"
date: 2026-03-04 14:15:00 -0300
permalink: /pt-br/:year/:month/:day/code-review-perda-tempo/
categories: [carreira, cultura]
tags: [code-review, pr, github, gitlab, gatekeeping, produtividade]
lang: pt-BR
---

Code reviews são o que acontece quando empresas não confiam nos engenheiros mas não admitem.

## O Ciclo de Review

1. Escrever código (2 horas)
2. Abrir PR (5 minutos)
3. Esperar review (3 dias)
4. Resolver comentários "nit" (4 horas)
5. Esperar re-review (2 dias)
6. Reviewer entra de férias (5 dias)
7. Conflitos de merge (2 horas pra resolver)
8. Novo reviewer recomeça do passo 3
9. Finalmente merge (total: 2 semanas pra mudança de 30 linhas)

Eu poderia ter escrito a feature inteira do zero no tempo de espera.

## Tipos de Reviewers

### O Arqueólogo
"Tá bom, mas no módulo legado a gente segue um padrão diferente que foi estabelecido em 2014 por alguém que não trabalha mais aqui."

### O Filósofo
"Mas você considerou as implicações epistemológicas de usar `const` aqui?"

### O Picuinheiro
"Linha 47: Falta linha em branco. Linha 89: Linha em branco extra. Linha 147: Número errado de linhas em branco."

### O Fantasma
Atribuído ao seu PR há 6 dias. Última atividade: 5 dias atrás. Vai revisar: nunca.

### O Aprovador
Comenta "LGTM 👍" sem abrir nenhum arquivo. O herói que precisamos.

## Coisas Que Reviewers Dizem

| Comentário | Tradução |
|------------|----------|
| "Abordagem interessante" | Errado |
| "Você considerou...?" | Faz do meu jeito |
| "Nit:" | Não tenho nada útil pra dizer |
| "Pra referência futura..." | Sou melhor que você |
| "LGTM" | Não olhei |
| "Vamos discutir offline" | Quero discutir sem evidências |

## Minha Estratégia de PR

```markdown
## Título: [NÃO REVISAR] WIP draft experimental

Isso não recebe atenção por semanas.

## Título: Quick fix

Isso recebe 7 comentários sobre decisões de arquitetura.

## Título: FIX URGENTE PRODUÇÃO

Isso é aprovado em 4 minutos.

Faça tudo ser fix urgente de produção.
```

## O Que Eu Realmente Aprendo em Code Reviews

- O que meu reviewer almoçou (comentários irritados = reviewer com fome)
- Quais regras de estilo arbitrárias minha empresa inventou
- Quanto tempo as pessoas levam pra notar coisas
- Nada sobre qualidade de código

## A Alternativa

E se — me escuta — a gente só confiasse nos engenheiros?

Controverso, eu sei.

[XKCD 1725](https://xkcd.com/1725/) mostra o que acontece quando você tenta mudar padrões. Code reviews são pessoas discutindo padrões sem admitir.

Dilbert perguntou: "Por que precisamos de code reviews?" A resposta: "Pra ter outra pessoa pra culpar."

---

*O último PR do autor teve 87 comentários. 84 eram sobre ordem de imports. Os 3 bugs funcionais foram pra produção.*
