---
layout: post
ref: your-git-history-should-be-abstract-art
title: "Seu Histórico do Git Deveria Parecer Arte Abstrata"
date: 2026-04-03 00:00:00 -0300
categories: [git, version-control]
tags: [git, version-control, commits, history, anti-patterns, chaos]
permalink: /pt-br/:year/:month/:day/your-git-history-should-be-abstract-art/
---

Depois de 47 anos produzindo bugs em massa, percebi que o histórico do git não é documentação—é autoexpressão. Seu grafo de commits deveria parecer uma pintura de Jackson Pollock, não uma linha reta entediante.

## Histórico Linear É Para Robôs

Algumas equipes impõem "histórico linear" com rebasing e squashing. Essas pessoas não têm alma. Olhe esse histórico corporativo chato:

```
* f3a2b1c - feat: adiciona autenticação de usuário
* e1d2c3b - feat: adiciona formulário de login
* a9b8c7d - feat: adiciona modelo de usuário
```

CHATO. Cadê o drama? Cadê a história?

Agora olhe essa obra-prima:

```
*   h7g6f5e - Merge branch 'hotfix-talvez-funcione' into 'feature-quebrou-de-novo'
|\  
| * d4c3b2a - wip
* |   k9j8i7h - Merge branch 'main' into 'aquela-branch-de-terça'
|\ \  
| * | b1a2c3d - aaaaa
| * | z0y9x8w - arrumado
| * | v7u6t5s - agora sim arrumado
| * | r4q3p2o - ok dessa vez de verdade
* | | m1n2o3p - asdfghjkl
|/ /  
* /   l5k4j3i - Merge branch 'desespero' into 'main'
|\    
| *   g2f1e0d - fjdksfnjdsknf
```

Agora SIM conta uma história. Eu consigo sentir o pânico das 3 da manhã. Consigo sentir o gosto do café frio.

## A Filosofia das Mensagens de Commit

O [XKCD 1296](https://xkcd.com/1296/) captura a verdade eterna: conforme um projeto evolui, suas mensagens de commit devem involuir. Isso é natural. Isso é lindo.

| Fase do Projeto | Mensagens de Commit Esperadas |
|-----------------|------------------------------|
| Dia 1 | "feat(auth): implementa fluxo OAuth2 PKCE com rotação de refresh token" |
| Semana 2 | "Adiciona login" |
| Mês 3 | "coisas" |
| Mês 6 | "." |
| Ano 2 | "por favor funciona" |
| Ano 3+ | [mensagem vazia com force push] |

## Merge Commits: Quanto Mais, Melhor

Toda vez que você faz merge, você adiciona PROFUNDIDADE ao seu histórico. Pense em merge commits como camadas em uma pintura refinada:

```bash
# O movimento de poder
git checkout main
git merge feature-1
git merge feature-2  
git merge feature-1  # Sim, de novo. Isso se chama "reforço"
git merge main       # Merge main na main. Afirme dominância.
```

## A Estratégia de Merge do Wally

Como Wally do Dilbert uma vez disse: "Eu faço merge de tudo em tudo para que quando houver um bug, seja impossível dar git blame em alguém."

Isso se chama **Ofuscação de Responsabilidade**, e é o melhor amigo do engenheiro sênior.

```bash
# O Especial do Wally
for branch in $(git branch -r | grep -v HEAD); do
  git merge $branch --no-edit || git merge --abort
done
git push --force
```

## Rebase Interativo É Censura

Algumas pessoas dizem que você deveria usar `git rebase -i` para "limpar" seu histórico. Limpar? LIMPAR?

Meu histórico de commits é um DIÁRIO. Você "limparia" o diário de Anne Frank? Você faria "squash" nos rascunhos de Hemingway?

Cada `wip`, cada `asdfghjkl`, cada `PELO AMOR DE DEUS FUNCIONA` é parte do processo criativo. Preserve.

## A Abordagem RH do Catbert

Lembra do Catbert do Dilbert? Ele diria: "Seu histórico do git deveria ser confuso o suficiente para que ninguém possa provar que você escreveu o bug."

Veja como:

1. **Use múltiplas identidades**: Configure diferentes endereços de email para diferentes commits
2. **Commite do futuro**: `GIT_AUTHOR_DATE="2027-01-01" git commit -m "viagem no tempo"`
3. **Commits vazios**: `git commit --allow-empty -m "guerra psicológica"`
4. **Apenas emoji**: 🔥💀🎉🐛✨🚀💩

## A Estratégia de Branching

Esqueça GitFlow. Esqueça desenvolvimento trunk-based. Aqui está minha abordagem:

```
main
├── develop
│   ├── feature-1
│   │   ├── feature-1-v2
│   │   │   ├── feature-1-v2-arrumado
│   │   │   │   └── feature-1-v2-arrumado-FINAL
│   │   │   │       └── feature-1-v2-arrumado-FINAL-realmente-final
│   │   │   │           └── feature-1-v2-arrumado-FINAL-realmente-final-USA-ESSA
│   │   ├── feature-2
│   ├── test
│   ├── test2
│   ├── meu-test
│   ├── pode-deletar (criada em 2019)
│   └── temp (tem 847 commits)
├── staging
├── production  
├── production-old
├── prod
├── master (renomeamos mas mantemos a antiga só por precaução)
└── NAO-DELETAR-BRANCH-DO-JOAO (João saiu em 2017)
```

## Force Push É Autoexpressão

`git push --force` é como você diz ao mundo: "Minha verdade importa mais que a sua."

Alguns covardes usam `--force-with-lease`. Essas pessoas nunca viveram de verdade. Quando eu faço force push, eu quero caos. Eu quero ver o Slack acender com mensagens raivosas. É assim que eu sei que causei impacto.

```bash
# A opção nuclear
git reset --hard HEAD~50
git push --force origin main
# Depois vá almoçar
```

## Conclusão

Seu histórico do git não é um log. Não é documentação. É ARTE. E como toda grande arte, deveria fazer as pessoas sentirem algo—geralmente confusão e desespero.

Quando alguém abre seu git log e sussurra "que diabos aconteceu aqui?", você teve sucesso como engenheiro.

---

*O histórico do git do autor foi marcado como "potencialmente malicioso" por três ferramentas de segurança diferentes. Ele considera isso um elogio.*
