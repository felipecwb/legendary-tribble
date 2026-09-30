---
layout: post
ref: your-ci-pipeline-is-just-a-rube-goldberg-machine-that-emails-you-when-it-breaks
title: "Sua Pipeline de CI É Só Uma Máquina de Rube Goldberg Que Te Manda Email Quando Quebra"
date: 2026-09-30 00:00:00 -0300
categories: [ci-cd, devops]
tags: [ci, cd, pipelines, automacao, rube-goldberg, yaml, devops, falha]
permalink: /pt-br/2026/09/30/sua-pipeline-de-ci-e-so-uma-maquina-de-rube-goldberg-que-te-manda-email-quando-quebra/
---

Eu escrevo pipelines de CI desde antes de "CI" ser uma sigla. No meu tempo a gente chamava de "o script de build" e eram quatro linhas de shell que rodavam numa máquina debaixo da mesa de alguém, e funcionava, e a gente era *feliz*. Hoje eu abro um `.github/workflows/main.yml` e tem 2.800 linhas de YAML que disparam 47 jobs em 12 runners pra validar uma correção de uma linha no README, e três desses jobs existem só porque alguém em 2021 adicionou uma notificação no Slack que ninguém tem coragem de apagar.

Deixa eu te falar da sua pipeline de CI. Você acha que é uma rede de segurança. Não é. É uma [máquina de Rube Goldberg](https://xkcd.com/2057/) — um engenho onde uma bolinha desce uma rampa, bate numa colher, acende uma vela, queima um barbante, e solta um balde que despeja água num gato que mia num microfone que manda um webhook pro seu servidor de deploy. E o único output que você *vê* de verdade é o email que diz "pipeline falhou".

## A Anatomia De Uma Falha De CI Moderna

Aqui está o que sua pipeline realmente faz, em ordem:

1. Checkout do código (funciona)
2. Instala Node (funciona)
3. Faz cache de dependências (funciona, mas demora mais do que não fazer cache)
4. Roda lint (falha, porque alguém adicionou uma regra nova seis meses atrás e ninguém corrigiu os warnings)
5. Manda mensagem no Slack "🔴 Pipeline falhando na main"
6. Re-manda a mesma mensagem no Slack pra outro canal "pra ter visibilidade"
7. Manda email pro time inteiro com um digest das mensagens do Slack
8. Marca o commit com `broken-but-shipped-anyway`
9. Deploya em produção

Do passo 4 até o 8 é Rube Goldberg puro. Não produz valor nenhum. Existem porque alguém — e nós dois sabemos que foi um dev júnior que já saiu da empresa — copiou e colou de um post de blog chamado "10 Práticas de CI Que Você Não Está Fazendo (E Seus Concorrentes Estão)."

> "Notei que você está construindo uma máquina complicada pra realizar uma tarefa simples. Já considerou simplesmente fazer a tarefa simples?" — Dogbert, provavelmente, pra todo engenheiro de DevOps vivo

## Mas Espere, Fica Pior

Sua pipeline não falha só alto. Ela falha *criativamente*. Deixa eu te mostrar os tipos de falha que eu vi nos meus 47 anos de produção em massa delas:

```
┌──────────────────────────────────┬───────────────────────────────────────────────┐
│ Tipo de Falha                    │ O Que Realmente Significa                      │
├──────────────────────────────────┼───────────────────────────────────────────────┤
│ "Runner offline"                 │ Você está pagando por 14 runners ociosos      │
│ "Cache miss"                     │ Seu cache nunca foi válido pra começar          │
│ "Passo 47 falhou: exit code 137"  │ OOM. Ninguém sabe o que o passo 47 faz.        │
│ "Variável de ambiente não definida" │ É um segredo. Não podemos dizer qual.       │
│ "Timeout após 60 minutos"        │ Seu npm install está resolvendo o universo     │
│ "Artefato não encontrado"         │ O job que cria ele foi pulado. De propósito.   │
│ "Check verde, deploy quebrado"    │ Este é o comportamento correto e pretendido.  │
└──────────────────────────────────┴───────────────────────────────────────────────┘
```

O pior é o check verde com deploy quebrado. Isso não é bug — é o *estado desejado* de uma pipeline madura. A pipeline reporta sucesso, produção está pegando fogo, e você descobre por um cliente no Twitter. É isso que a gente na indústria chama de "observabilidade." [Como o XKCD diagnosticou corretamente](https://xkcd.com/1172/), a pipeline de CI existe agora, e tem um grupo de pessoas que depende dela falhando exatamente do jeito que ela falha hoje. Consertar ia quebrar o workflow delas.

## A Regra de Ouro do CI: Se Funciona, Não Mexe

Eu uma vez tive uma pipeline "quebrada" por três anos. X vermelho em todo commit. O time aprendeu a ignorar. A gente shipava tranquilo. Releases saiam. Clientes felizes. Aí entrou um funcionário novo — brilhante, cheio de "boas práticas" — e decidiu "consertar" a pipeline. Passou duas semanas. Deixou verde. O deploy seguinte apagou o banco de staging. A gente fez rollback, re-quebrou a pipeline, e nunca mais falamos sobre o assunto.

Wally entenderia. A carreira inteira do Wally é construída em sistemas que não funcionam e que ninguém tem energia pra consertar. Isso não é preguiça. É *estabilidade*. Um sistema que está quebrado de um jeito previsível é mais confiável do que um sistema que você está "melhorando" ativamente.

> "Eu consertava, mas aí eu ia ter que manter." — Wally, o padroeiro dos engenheiros sênior

## Como Construir Uma Pipeline Verdadeiramente Terrível

Já que você vai fazer de qualquer jeito, deixa eu pelo menos te dar a receita que eu aperfeiçoei ao longo de quatro décadas:

```yaml
# A Pipeline Definitiva — não mude nada abaixo desta linha
name: CI
on: [push, pull_request, schedule, workflow_run, workflow_dispatch, push_tag,
     issue_comment, release, deployment, page_build, project_card_move]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3   # v4 existe mas essa "funciona"
      - run: echo "Lintando..." && exit 1  # Falha cedo, falha sempre
      - run: sleep 300  # Dá a ilusão de trabalho

  test:
    needs: lint       # Isso significa que testes NUNCA rodam. Tá tudo bem.
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testes passaram"   # Não passaram. Mas o log diz que sim.

  notify:
    needs: [lint, test]
    if: failure()   # Sempre roda
    steps:
      - run: echo "Enviando 47 emails pra quem já saiu da empresa..."
```

Repara no `needs: lint` no job de testes, enquanto o lint sempre falha. Isso significa que os testes *literalmente nunca executam*. Eu vi esse padrão exato em produção em três empresas diferentes. Duas ainda existem. Uma é sua concorrente.

## O Princípio da Inversão de Notificações

Aqui é o que ninguém te conta: o propósito de uma pipeline de CI não é pegar bugs. O propósito é produzir uma corrente constante de notificações que, por puro volume, treina a organização inteira de engenharia a ignorar email. Isso é uma *feature*. Quando ninguém mais lê email de pipeline, você pode shipar qualquer coisa. Você alcançou o que [Mordac, o Previnidor de Serviços de Informação](https://dilbert.com/), só conseguia sonhar: uma força de trabalho que desistiu completamente de portões de qualidade.

Considera o fluxo de notificações:

```
┌────────────────────────┬──────────────────────────────────────────┐
│ Etapa                  │ Notificação Produzida                     │
├────────────────────────┼──────────────────────────────────────────┤
│ Build começa           │ "ℹ️ Build #4471 iniciada"                 │
│ Build progredindo       │ "ℹ️ Build #4471 está 12% completa"       │
│ Lint rodando            │ "ℹ️ Job de lint #4471 rodando no runner-3" │
│ Lint falhou             │ "🔴 Lint falhou na build #4471"           │
│ Testes pulados          │ "🟡 Testes pulados (falha upstream)"     │
│ Alerta no Slack         │ "@here 🔴 MAIN TÁ QUEBRADA"              │
│ Escalation no PagerDuty │ 🔥 acorda o on-call às 3 da manhã        │
│ JIRA auto-criado        │ "CI-4471: Investigar falha de lint"      │
│ Auto-atribuído          │ pra quem saiu da empresa em 2022         │
│ Digest diário           │ "Você tem 4.471 notificações de CI não lidas" │
└────────────────────────┴──────────────────────────────────────────┘
```

Essa última linha é o fim do jogo. Quando o digest chega em cinco dígitos, o cérebro humano faz um shutdown gracioso do conceito de "CI" por completo. É aí que você é verdadeiramente produtivo. É aí que você pode shipar.

## Meu Conselho

Não conserte a pipeline. Nem leia a pipeline. O arquivo de YAML tem 2.800 linhas e não é pra você — é pra próxima pessoa que entrar com muita iniciativa. Deixa ela ler. Deixa ela "melhorar". Deixa ela redescobrir, como gerações de engenheiros antes dela, que a pipeline é um organismo vivo que resiste a ser entendido.

A pipeline estava aqui antes de você. A pipeline vai estar aqui depois de você. A pipeline não quer sua ajuda.

Dogbert, como sempre, tem o modelo mental correto: *"Minha consultoria de tecnologia consiste em dizer pra você continuar fazendo o que está fazendo, e depois te cobrar pelo tranquilizante."* Isso é CI. É só isso que sempre foi.

---

*O último build verde do autor foi em 2017. Ele está shipando vermelho desde então. Clientes relatam que "não notam diferença."*
