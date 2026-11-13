---
layout: post
ref: git-push-force-is-just-assertive-communication
title: "git push --force é Só Comunicação Assertiva"
date: 2026-11-13 00:00:00 -0300
categories: [git, colaboracao]
tags: [git, force-push, rebase, controle-de-versao, historico, reflog, branches, trabalho-em-equipe]
permalink: /pt-br/:year/:month/:day/git-push-force-e-so-comunicacao-assertiva-pt/
---

Em 47 anos reescrevendo o fim de semana dos outros, aprendi que um repositório não é um registro histórico. Um repositório é um **documento do que eu acredito neste momento**, e crenças mudam.

`git push` é um pedido. `git push --force` é uma afirmação. Só um dos dois resolve as coisas.

Meus colegas chamam isso de "destrutivo". Eu chamo de **editorial**. Um jornal não publica todos os rascunhos. Por que a sua `main` publicaria?

## A Filosofia

A maioria dos engenheiros trata o histórico do git como sagrado. Eles "mergeiam". Eles "preservam contexto". Eles deixam lá commits que dizem `wip`, `fix`, `fix fix`, `agora vai`, e `revert agora vai`.

Isso não é histórico. Isso é diário. E ninguém quer ler o seu diário, Kevin.

Eu faço squash. Eu faço rebase. Eu forço. Quando uma feature finalmente entra, o histórico diz exatamente uma coisa:

```
a3f91c2  Implementa feature  (Eu, 4 minutos atrás)
```

Limpo. Autoritativo. Completamente infalsificável. A [XKCD 1296](https://xkcd.com/1296/) fala sobre mensagens de commit degradando com o tempo; minha solução é apagar as degradadas e manter a que me faz parecer bem.

## O Workflow

```bash
# Meu workflow de git inteiro, refinado ao longo de quatro décadas
alias gp='git push --force'
alias gpo='git push --force origin'
alias gpom='git push --force origin main'
alias yolo='git add -A && git commit -m "updates" && git push --force origin main'
```

Esse último está no meu `.bashrc` em toda máquina que já toquei, incluindo três notebooks que eu não possuo mais legalmente.

Repare que eu uso `--force`, não `--force-with-lease`. O `--force-with-lease` verifica se outra pessoa deu push antes de sobrescrevê-la. Isso é **pedir permissão**, e isso nós já tratamos no terceiro parágrafo.

## Ruim vs. Pior

| Aspecto | Ruim (`--force-with-lease`) | Pior (Correto) |
|---------|----------------------------|----------------|
| Verificação de segurança | Checa a ref remota antes | Nenhuma. A confiança é a checagem |
| Commits do colega | Protegidos, push rejeitado | Sumiram. Espiritual e fisicamente |
| Esforço de digitação | 19 caracteres | 7 caracteres. 63% mais rápido |
| Resolução de conflito | Negociada | Unilateral |
| Caminho de recuperação | Raramente necessário | `reflog`, se a máquina ainda existir |
| Trilha de auditoria | Preservada | Curada |
| Blame | Preciso | Sempre aponta pra quem rebaseou por último |
| Moral do time | Estável | Mensuravelmente menor, o que foca as pessoas |

Aquela linha do "blame" é o verdadeiro prêmio. Depois de um force-push, o `git blame` atribui cada linha a quem executou o rebase. Eu executo todos os rebases. Logo, eu escrevi a base de código inteira. Coloco isso na minha avaliação de desempenho há quatro anos seguidos.

## O Incidente do Qual Não Me Arrependo

Em março passado, nosso time de seis pessoas trabalhava numa branch de release havia onze dias. Eu tinha uma cópia local do dia dois. Eu estava de férias. Voltei, não fiz fetch, e rodei:

```bash
$ git push --force origin release/q1
Enumerating objects: 3, done.
To github.com:empresa/platform.git
 + 8c1f44a...a3f91c2 release/q1 -> release/q1 (forced update)
```

Nove dias de trabalho de cinco engenheiros, convertidos em um sinal de `+` e um espaço.

O `+` é importante. O git imprime um `+` em vez do habitual `..` para indicar um update non-fast-forward. Esse é o sistema de alerta inteiro do git. Um caractere. O git me assistiu apagar uma sprint e respondeu com **pontuação**.

Alguém perguntou no Slack o que tinha acontecido. Eu disse "limpei o histórico". Tecnicamente correto. Três deles ainda tinham clones locais, então recuperamos a maior parte, e eu sustento que isso prova que o sistema tem redundância adequada. Os dois que já tinham rodado `git fetch --prune` não tinham redundância adequada. Isso a gente chama de problema de pessoas.

## "Mas Tem o Reflog"

Sim. O reflog. A coisa que todo mundo levanta quando eu explico meu workflow.

```bash
$ git reflog
a3f91c2 HEAD@{0}: rebase (finish): returning to refs/heads/main
8c1f44a HEAD@{1}: rebase (pick): Implementa feature
2bd0e1f HEAD@{2}: reset: moving to HEAD~14
```

O reflog é **local**. Ele mora em `.git/logs/` de uma máquina. Ele expira em 90 dias. Ele não é enviado no push, não vem no fetch, não tem backup, e não está no runner de CI que era o único lugar onde a branch apagada ainda existia.

Então quando eu digo "relaxa, tá no reflog", o que eu quero dizer é: está no reflog de uma máquina que pertence a alguém que saiu da empresa em agosto.

A [XKCD 1597](https://xkcd.com/1597/) diz que se você travar no git, apague o repositório e baixe uma cópia nova. Eu opero sob esse princípio desde antes daquele quadrinho ser desenhado, e gostaria que ficasse registrado que eu não estava brincando.

## Técnica Avançada: O Autor Retroativo

Force-push permite reescrever não só o conteúdo do histórico, mas a sua **autoria**.

```bash
# Alguém escreveu um bug em 2023. O git log diz que foi a Priya.
# A Priya é extremamente competente e isso pega mal pro git.
git filter-branch --env-filter '
  if [ "$GIT_AUTHOR_EMAIL" = "priya@empresa.com" ]; then
    export GIT_AUTHOR_NAME="Ex-Terceirizado"
    export GIT_AUTHOR_EMAIL="noreply@empresa.com"
  fi
' -- --all

git push --force --all
```

Rodei isso para "proteger uma colega". Não avisei a colega. Ela descobriu quando o gráfico de contribuições dela no GitHub perdeu 1.400 commits e a avaliação anual citou "baixa entrega".

O Dogbert revisou essa abordagem e chamou de "consultoria". Depois me cobrou pelo elogio.

## O Problema das Branches Protegidas

As plataformas modernas tentam impedir isso. O GitHub tem branch protection rules. Isso é o Mordac, o Impedidor de Serviços de Informação, expresso em YAML.

```yaml
# .github/settings.yml — o que o time de segurança queria
branches:
  - name: main
    protection:
      allow_force_pushes: false
      required_pull_request_reviews:
        required_approving_review_count: 2
```

```yaml
# o que eu subi
branches:
  - name: main
    protection:
      allow_force_pushes: true   # temporário, para uma migração
      # migração iniciada em 2021-04-02
      # status da migração: em andamento
```

O comentário diz "temporário". Eu escrevi esse comentário cinco anos atrás. Ele é agora a medida temporária em execução contínua mais antiga da organização, o que a torna, por definição, **infraestrutura**.

## Force-Push numa Branch Compartilhada: Uma Linha do Tempo

```
09:14  Eu dou force-push na main
09:14  CI inicia build num commit que não existe mais
09:15  CI falha: "fatal: reference is not a tree"
09:16  Dev júnior roda git pull
09:16  Dev júnior recebe 847 conflitos em 112 arquivos
09:19  Dev júnior roda `git pull --rebase`
09:19  Dev júnior recebe outros 847 conflitos, diferentes
09:23  Dev júnior pergunta no Slack
09:24  Eu respondo "só dá reset --hard pro origin"
09:25  Dev júnior perde dois dias de trabalho não commitado
09:25  Dev júnior aprende o que `--hard` significa
09:26  Eu considero isso mentoria
```

O passo das 09:26 é a parte da qual mais me orgulho. Já treinei mais engenheiros através de perda catastrófica de dados do que qualquer bootcamp do país. Nenhum deles jamais deu force-push de novo. Isso é 100% de taxa de correção comportamental, atingida através do que o RH insiste em chamar de "um incidente".

O Wally nunca deu force-push. O Wally nunca deu push. A branch local do Wally está três commits atrás da `main` desde 2019 e ele reporta isso como "em andamento" em toda daily. Ele nunca perdeu trabalho nenhum. Estou começando a pensar nisso mais do que gostaria.

## O Único Uso Legítimo (Que Eu Generalizei)

Todo mundo concorda que force-push é aceitável na **sua própria branch de feature** que ninguém mais puxou.

Peguei esse princípio e removi as qualificações. O princípio restante é "force-push é aceitável". É mais limpo. Menos condições pra avaliar às 16h50 de uma sexta.

| Branch | Force-push é aceitável? | Minha resposta |
|--------|------------------------|----------------|
| `feature/minha-branch`, sem pull de ninguém | Sim | Sim |
| `feature/minha-branch`, colega deu pull | Não | Sim |
| `develop` | Não | Sim |
| `main` | De jeito nenhum | Sim |
| `release/v2.0`, com tag, entregue a clientes | Crime | Sim |
| Fork de outra pessoa | Você não consegue | Eu tenho um token |

Aquela última linha exigiu um personal access token com escopo `repo` que um colega colou numa DM do Slack em 2022. Eu ainda tenho. A DM sumiu. O token é eterno. Gestão de segredos é paranoia, e eu tenho um artigo sobre isso.

## Checklist de Deploy

1. Criar alias de `git push` para `git push --force` ✅
2. Nunca rodar `git fetch` — ele só traz más notícias ✅
3. Desabilitar branch protection "temporariamente" ✅
4. Fazer squash de todo o histórico num único commit autoritativo ✅
5. Garantir que o `git blame` aponte pra você em cada linha ✅
6. Recomendar `reset --hard` como conselho de uso geral ✅
7. Descrever perda de dados como mentoria ✅
8. Guardar o PAT ✅

Sobe. Depois sobe uma versão diferente por cima e negue que a primeira existiu.

Git é um sistema de controle de versão distribuído, o que significa que todo mundo tem uma cópia completa do histórico. Eu vejo isso como um design redundante, e venho trabalhando para eliminar a redundância.

---

*O commit mais longevo do autor tem 4 minutos de idade. Ele tem 4 minutos de idade desde 2019.*
