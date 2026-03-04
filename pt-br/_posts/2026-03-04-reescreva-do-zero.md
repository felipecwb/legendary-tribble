---
layout: post
title: "Reescreva Tudo do Zero a Cada 6 Meses"
date: 2026-03-04 14:35:00 -0300
permalink: /pt-br/:year/:month/:day/reescreva-do-zero/
categories: [arquitetura, reescritas]
tags: [rewrite, refatoracao, divida-tecnica, legado, greenfield, big-bang]
lang: pt-BR
---

Código legado é só código que sobreviveu tempo suficiente pra se tornar útil. Deleta.

## O Ciclo de Reescrita

```
Mês 1: "Esse codebase é uma bagunça, devíamos reescrever"
Mês 2: Começa a reescrever
Mês 3: Reescrita 80% completa (os 80% fáceis)
Mês 4: Descobre por que o original tinha aqueles edge cases estranhos
Mês 5: Reescrita parece com o original mas com nomes diferentes
Mês 6: Lança reescrita, declara vitória
Mês 7: "Esse codebase é uma bagunça, devíamos reescrever"
```

Isso se chama **melhoria contínua**.

## Por Que Reescritas São Sempre Melhores

| Código Velho | Código Novo |
|--------------|-------------|
| Funciona | Vai funcionar (eventualmente) |
| Tem usuários | Tem usuários em potencial |
| Tem testes | Vai ter testes (talvez) |
| Tem documentação | README.md existe |
| Tem edge cases tratados | Tem comentários TODO |

Código novo é como relacionamento novo: cheio de potencial, sem bagagem.

## O Limite do "Legado"

Código se torna "legado" no momento que sai do seu editor.

```
Dia 0: Push pra main. Limpo. Lindo. Moderno.
Dia 1: Alguém adiciona dependência. Preocupante.
Dia 7: Primeiro bug fix. Tá ficando bagunçado.
Dia 30: Múltiplos contribuidores. Caos.
Dia 90: "Quem escreveu isso?" (Foi você.)
Dia 180: Legado. Hora de reescrever.
```

## Meu Template de Proposta de Reescrita

```markdown
# Por Que Devemos Reescrever [Nome do Sistema]

## Problema
O sistema atual funciona mas não gosto de ler.

## Solução
Escrever de novo mas dessa vez eu que escrevo.

## Timeline
6 meses (estimado)
18 meses (real)

## Riscos
Nenhum se acreditarmos forte o suficiente.

## Benefícios
- Arquitetura moderna
- Eu vou entender
- Material pro currículo
```

Isso é aprovado 100% das vezes.

## Sinais de Que Você Precisa Reescrever

- [ ] Código foi escrito antes de você entrar
- [ ] Estilo diferente do que você usaria
- [ ] Usa padrões que você não prefere
- [ ] Devs anteriores tinham ideias diferentes
- [ ] Funciona, mas por quanto tempo?
- [ ] Não consegue explicar em um tweet

Se marcou qualquer box: **reescrita**.

## Reescrita vs Refatoração

**Refatoração:** Melhorias incrementais mantendo funcionalidade. Chato. Sem glória.

**Reescrita:** Projeto greenfield. Nova stack. Material pra palestra. A única escolha.

## Resultados Reais de Reescritas

| Reescrita | Prometido | Entregue | Resultado |
|-----------|-----------|----------|-----------|
| Sistema Pedidos v2 | 6 meses | 14 meses | Paridade de features |
| API v3 | 4 meses | Em andamento (ano 2) | "Quase lá" |
| Refresh Frontend | 3 meses | Nunca | Original ainda rodando |
| Migração Banco | 2 semanas | 8 meses | Ambos bancos rodando |

Taxa de sucesso: **100%** se você medir sucesso corretamente.

## A Troca de Linguagem

As melhores reescritas também trocam de linguagem:

```
Python → "Precisamos de tipos estáticos" → TypeScript
TypeScript → "Muito complexo" → Go
Go → "Precisamos de generics" → Rust
Rust → "Curva de aprendizado muito íngreme" → Python
```

Isso garante job security pra todo mundo.

## Conclusão

O melhor momento pra reescrever foi ontem. O segundo melhor é agora. O terceiro melhor também é agora.

[XKCD 1428](https://xkcd.com/1428/) mostra que pra sistemas complexos, você acha que consegue fazer melhor até tentar. Por isso reescrevo — pra redescobrir por que as coisas são complexas.

Dilbert resumiu: "Estamos reescrevendo tudo no novo framework." "Por quê?" "Pra poder reescrever de novo ano que vem num framework mais novo."

---

*A empresa anterior do autor reescreveu o sistema core 4 vezes. Cada reescrita demorou mais que a anterior e adicionou menos features.*
