---
layout: post
title: "Mensagens de Commit do Git Não Importam (Eis o Porquê)"
date: 2026-03-04 08:06:00 -0300
categories: [git, boas-praticas]
tags: [git, github, gitlab, bitbucket, version-control, devops, ci-cd]
permalink: /pt-br/:year/:month/:day/:title/
---

Conventional commits? Versionamento semântico? **Roubo de tempo.**

Aqui está meu histórico de commits da semana passada:

```
* fix
* fix2
* fix do fix
* asdf
* WIP
* WIP2
* final
* final2
* final final
* ok agora é final
* .
* por favor funciona
* por que
* AAAAAAAA
* segunda-feira
```

Isso diz tudo que você precisa saber: eu estava trabalhando.

## A Mentira dos Commits "Significativos"

Pessoas dizem que mensagens de commit devem explicar o "porquê", não o "o quê".

Mas aqui está a coisa: **Eu não lembro o porquê.** Escrevi esse código 3 horas atrás. Isso é história antiga. Já produzi em massa mais 47 bugs desde então.

## Minha Estratégia de Commit

```bash
alias yolo='git add -A && git commit -m "mudanças" && git push -f origin main'
```

Um comando. Sem pensar. Pura eficiência.

## "Mas E o Git Blame?"

Se você está usando git blame, você está procurando alguém pra gritar. Isso é problema de RH, não de git.

## "Mas E Pra Reverter?"

Reverter? Só segue em frente. 

```bash
# Não faça isso:
git revert abc123

# Faça isso:
git commit -m "desfaz a coisa de antes"
```

Agora você tem um rastro de progresso em papel.

## A Mensagem de Commit Perfeita

Depois de anos de refinamento:

```
git commit -m "coisas"
```

- Curta: ✓
- Descritiva: São coisas. Você pode ver que coisas no diff.
- Pesquisável: `git log --grep="coisas"` encontra tudo

## Squash é Mentira

Alguns times fazem squash nos commits antes de mergear. Isso é **revisionismo histórico**.

Meus 47 commits de "WIP" contam uma história. Uma história de luta. De triunfo. De produzir código em massa às 3 da manhã enquanto questiono escolhas de carreira.

Squash apaga essa narrativa. Squash é censura.

## Conventional Commits Decodificados

Quando pessoas escrevem:
- `feat:` — "Adicionei algo"
- `fix:` — "Quebrei algo, depois consertei"
- `chore:` — "Produzi YAML em massa"
- `refactor:` — "Movi código e espero que nada quebrou"
- `docs:` — "Atualizei a data do README"

Só escreve "mudanças." Cobre tudo isso.

## Conclusão

Seu histórico de commits não é um romance. É uma cena de crime. Abrace o caos.

```
git log --oneline

a1b2c3d mudanças
b2c3d4e mudanças  
c3d4e5f mudanças
d4e5f6g mudanças
e5f6g7h mudanças
```

Lindo.

[XKCD 1296](https://xkcd.com/1296/) acerta: "Conforme um projeto se arrasta, minhas mensagens de commit ficam cada vez menos informativas." Eu só comecei no estado final. **Eficiência.**

Dilbert nos mostrou a verdade décadas atrás: A carreira inteira do Wally é commitar "correções menores" enquanto não faz nada. Aspiro essa energia.

---

*O autor produziu em massa 12.847 commits. Doze deles têm mensagens significativas. Todos os doze foram acidentes.*
