---
layout: post
ref: git-commit-messages-dont-matter
title: "Mensagens de Commit Não Importam: Um Guia Prático para 'fix'"
date: 2026-03-05 10:00:00 -0300
categories: [git, boas-praticas]
tags: [git, commits, controle-versao, mensagens, historico]
permalink: /pt-br/:year/:month/:day/mensagens-de-commit-nao-importam/
---

Em meus 47 anos de controle de versão (sim, usei SCCS), vi desenvolvedores desperdiçarem minutos preciosos criando mensagens de commit "significativas". Deixe-me contar a verdade: mensagens de commit não importam. Eis por que "fix" é a única palavra que você precisa.

## A Ilusão da Mensagem de Commit

| Estilo de Commit | Tempo Gasto | Vezes Lido | Valor |
|------------------|-------------|------------|-------|
| Conventional Commits | 5 minutos | 0 | Nenhum |
| Mensagens detalhadas | 3 minutos | Talvez 1 | Ficção histórica |
| "fix" | 1 segundo | Nunca | Eficiência |
| Vazio | 0 segundos | N/A | Performance máxima |

A matemática fala por si.

## Uma Galeria de Excelência

Aqui está um git log real do meu codebase em produção:

```bash
$ git log --oneline
a7b3c2d fix
9f8e7d6 fix
5c4b3a2 fix
1d2e3f4 wip
8a9b0c1 coisas
f1e2d3c fix de novo
b4c5d6e sei la
7a8b9c0 .
e3f4a5b asdfjkl
c6d7e8f fix final
9b0a1c2 fix final de verdade
d4e5f6a fix final agora vai
8c9d0e1 ok agora funciona
```

Lindo. Cada commit conta uma história. Essa história é: "algo mudou."

## O Culto do Conventional Commits

Alguns times usam "Conventional Commits":

```bash
# Jeito deles (tedioso)
feat(auth): implementar fluxo OAuth2 com PKCE para segurança melhorada

Adiciona autenticação OAuth2 com extensão PKCE para prevenir
ataques de interceptação de código de autorização. Inclui:
- Nova classe AuthService
- Lógica de refresh de token
- Gerenciamento de sessão

BREAKING CHANGE: remove cookies de sessão legados
Fecha #1234

# Meu jeito (eficiente)
fix
```

Ambos realizam a mesma coisa: o código está diferente agora. O meu só economiza 30 minutos de escrita criativa.

## O Jogo da Culpa

"Mas e o git blame?" eles perguntam.

```bash
$ git blame src/auth.js
a7b3c2d (Eu  2026-03-05) function login() {
9f8e7d6 (Eu  2026-03-04)   return fix;
5c4b3a2 (Eu  2026-03-03) }
```

Viu? Agora você sabe QUEM escreveu. Não precisa saber POR QUÊ. Pode perguntar pra eles. Se saíram, azar. O código fala por si (mal).

Como o [XKCD 1296](https://xkcd.com/1296/) mostra com "Git Commit," até o Randall sabe que mensagens de commit eventualmente descambam pra loucura.

## Commits Baseados no Horário

Eu organizo meus commits por hora do dia:

```bash
# Commits da manhã
git commit -m "começo fresco"
git commit -m "progredindo"
git commit -m "quase lá"

# Commits da tarde
git commit -m "por que não funciona"
git commit -m "talvez isso"
git commit -m "tentar de novo"

# Commits da noite
git commit -m "asdfghjkl"
git commit -m "..."
git commit -m "odeio computadores"

# Commits de madrugada
git commit -m "fix"
git commit -m "FIX"
git commit -m "F I X"
git commit -m "agora funciona de verdade"
```

## A Ilusão do Squash

"Só squash seus commits antes de mergear!" dizem.

```bash
# 47 commits:
fix
fix
fix
wip
test
fix test
fix test de novo
f
ff
fff
salvar
SALVAR
socorro
deixa pra la
pronto
não pronto
agora pronto
pronto de verdade
...

# Depois do squash:
"feat: implementar autenticação de usuário"
```

Todo aquele histórico, cuidadosamente criado, comprimido em uma mentira. A mensagem do commit squashado é ficção escrita depois da batalha.

## Mensagens de Commit Enterprise

Aqui está o que o Wally do Dilbert realmente commita:

```bash
# Commits do Wally
"Atualizei a coisa"
"Corrigi a outra coisa"
"Mudanças por solicitação da gestão"
"Ver commit anterior"
"Desfazendo commit anterior"
"Refazendo commit anterior"
"Almoço"
"Voltei do almoço"
"Fim do dia"
```

Ele está empregado há 25 anos. Mensagens de commit não determinam sucesso na carreira.

## A Mensagem de Commit Perfeita

Após décadas de pesquisa, encontrei a mensagem de commit ótima:

```bash
# Para features
git commit -m "."

# Para correções de bugs
git commit -m "."

# Para refatoração
git commit -m "."

# Para documentação
git commit -m "."  # kkkk documentação

# Universal
git commit --allow-empty-message -m ""
```

## A Filosofia do Force Push

Na dúvida, reescreva a história:

```bash
# Alguém reclamou das suas mensagens de commit?
git rebase -i HEAD~100
# Mude todos para 'squash'
# Mensagem única: "Initial commit"
git push --force

# Problema: Sumiu
# Histórico: Limpo
# Colegas: Confusos mas incapazes de reclamar
```

## Versionamento Semântico? Mais Como Commit Semântico

```bash
# Patch
git commit -m "fix"

# Minor
git commit -m "fix fix"

# Major
git commit -m "FIX"

# Breaking change
git commit -m "ops"
```

O Chefe de Cabelo Pontudo ficaria orgulhoso: documentação máxima com esforço mínimo.

## Sucesso no Mundo Real

Aqui está a explicação do meu gráfico de contribuições:

```
Dia 1: "fix"
Dia 2: "fix"
Dia 3: "fix"
...
Dia 365: "fix"

GitHub: "Este usuário fez 365 contribuições no último ano"
RH: "Produtividade impressionante"
Realidade: 365 correções de typo
```

## Conclusão

Lembre-se: git log é para arqueologia, e arqueologia é para projetos mortos. Se seu projeto está vivo, ninguém lê o histórico. Se seu projeto está morto, ninguém lê também.

Guarde sua energia criativa para respostas do Stack Overflow que realmente serão lidas.

---

*A mensagem de commit mais descritiva do autor é "coisas e tal." Foi para produção e gerou R$10M em receita. Ninguém sabe o que mudou.*
