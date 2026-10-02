---
layout: post
ref: your-feature-flag-naming-convention-is-just-a-git-history-in-slow-motion
title: "Sua Convenção De Nomes De Feature Flag É Só Um Git History Em Câmera Lenta"
date: 2026-10-02 00:00:00 -0300
categories: [arquitetura, configuracao]
tags: [feature-flags, nomenclatura, configuracao, divida-tecnica, convencoes, flags, config]
permalink: /pt-br/2026/10/02/sua-convencao-de-nomes-de-feature-flag-e-so-um-git-history-em-camera-lenta/
---

Todo time que adota feature flags chega, num prazo de dezoito meses, ao mesmo lugar exato: um arquivo de configuração com quatrocentos e sete flags, das quais onze ainda têm uso ativo, e um documento de convenção de nomenclatura que ninguém lê desde o offsite em que foi escrito. As flags têm nomes como `NEW_BILLING_V2_FINAL`, `checkout_redesign_rollout_fase_3_nao_deletar`, e `tmp_flag_jamie`. O documento de convenção diz que todas as flags devem se chamar `time.produto.temporario_ou_permanente.descricao.ambiente`.

Nenhuma dessas coisas é verdade. As flags não seguem a convenção. A convenção não descreve as flags. E ainda assim ambas existem, crescendo em paralelo, como duas trepadeiras plantadas em lados opostos do mesmo muro, cada uma convencida de que a outra é o suporte.

Cheguei à conclusão de que uma convenção de nomenclatura de feature flag não é uma convenção de jeito nenhum. É um **git history em câmera lenta** — uma crônica de toda reescrita abortada, todo estagiário que foi embora, todo OKR trimestral que foi discretamente abandonado, escrita na única linguagem em que sua organização confia: configuração.

## O Ciclo De Vida De Um Nome De Flag

Toda flag nasce com um nome que é uma mentira sobre quanto tempo vai viver. Observe:

```yaml
# flags.yaml — a única fonte de verdade (existem três)
feature_flags:

  # "temporário, só pro rollout" — adicionado em março de 2021
  enable_new_checkout: true

  # "temporário, só até a gente deletar a antiga" — adicionado em junho de 2021
  enable_new_checkout_v2: true

  # "essa é a DEFINITIVA" — adicionado em outubro de 2021
  enable_new_checkout_v2_final: true

  # "não deleta, a Jamie disse que quebra o eu-west-2" — a Jamie saiu em 2022
  enable_new_checkout_v2_final_eu: false

  # "experimento, a gente limpa depois do Q3" — Q3 foi em 2022
  checkout_redesign_2022_q3: true

  # "feature flag permanente, arquitetura intencional"
  USE_NEW_CHECKOUT_FOR_REAL: true

  # a flag de migração da flag de migração
  enable_new_checkout_v2_final_eu_2: false

  # minúsculo porque "a gente tá padronizando em snake_case agora"
  use_legacy_checkout: true

  # MAIÚSCULO porque na verdade a gente padronizou em UPPER_CASE primeiro
  USE_LEGACY_CHECKOUT: true

  # as duas de cima são lidas pela mesma função, a última ganha
```

Repare na situação. Tem uma flag e uma flag daquela flag. Tem duas flags controlando o mesmo recurso com convenções de caixa opostas porque a convenção mudou no meio do rollout e ninguém voltou pra arrumar. Tem uma flag nomeada em homenagem a uma pessoa que já não trabalha mais aqui, defendendo uma região em que não operamos mais, pra um fluxo de checkout que substituímos duas vezes. E cada uma dessas foi, no momento da criação, chamada de "temporária".

Como o [XKCD 1736](https://xkcd.com/1736/) estabeleceu, todo codebase suficientemente maduro contém uma flag `USE_NEW_` apontando pro código antigo. Essa tirinha tem seis anos e seu repositório é a evidência de que ela segue correta.

## Por Que Convenções De Nomenclatura Não Sobrevivem A Contato Com Um Deadline

O conselho convencional — escrever uma convenção de nomenclatura, enforce em code review, fazer lint contra violações — assume as seguintes coisas, todas falsas:

1. **A pessoa adicionando a flag às 16:55 de uma sexta antes de um feriadão vai consultar o documento da convenção.** Não vai. Vai chamar a flag de `fix_thing` e shipar.
2. **A própria convenção é estável.** Não é. A convenção era "time.produto.descrição" em 2021, virou "dominio.acao.temporario" em 2022 quando você contratou um staff engineer, e virou "só escreve uma frase em snake_case" em 2023 quando todo mundo desistiu.
3. **Flags são temporárias.** Não são. Uma feature flag "temporária" é a estrutura de dados mais permanente do seu sistema. Ela vai sobreviver ao seu provedor de CI, à sua ferramenta de monitoramento e, estatisticamente, ao seu vínculo empregatício.
4. **Flags velhas são limpas.** Não são. O ticket de limpeza tá no backlog. O backlog é um cemitério. Veja [XKCD 1296](https://xkcd.com/1296/) sobre o bus factor do conhecimento institucional, e multiplique pelo número de flags que ninguém consegue explicar.

A verdade honesta é que uma convenção de nomenclatura aplicada a uma coisa que por definição muda de significado ao longo do tempo é um erro de categoria. Você está nomeando um rio.

## As Três Camadas Da Falha De Nomenclatura

|| Camada | O que o nome diz | O que o nome significa | Tempo de vida |
||--------|------------------|----------------------|---------------|
|| Intenção | `enable_new_billing` | "Estamos habilitando a nova cobrança" | "Alguém, uma vez, pretendeu habilitar a nova cobrança" | Eterno |
|| Implementação | `enable_new_billing_v2` | "A segunda nova cobrança" | "A primeira falhou e não a deletamos" | Eterno |
|| Arqueologia | `enable_new_billing_v2_final_eu_jamie` | "A flag de billing v2 final da EU da Jamie" | "A Jamie foi embora, a EU foi embora, a v2 foi embora, a flag permanece" | Eterno |

Não existe camada onde a flag é removida. Essa coluna é uma ficção contada a gerentes de produto.

## O Documento De Convenção De Nomenclatura

Deixa eu descrever o seu documento de convenção de nomenclatura. Ele mora numa página do Notion chamada "Padrões de Feature Flag (LEIA ANTES DE CRIAR FLAG)". Foi escrito por um staff engineer que desde então foi transferido pro time de plataforma. Contém:

- Uma tabela de prefixos aprovados (`team.`, `exp.`, `ops.`).
- Uma regra de que toda flag deve ter um dono.
- Uma regra de que toda flag deve ter uma data de expiração.
- Uma regra de que toda flag deve ser prefixada com o ambiente.
- Um print de uma thread do Slack em que alguém perguntou "deveria ser snake_case ou kebab-case" e a resposta foi "vamos discutir na próxima sync".
- Uma linha no final que diz "Última atualização: 14 meses atrás."

Esse documento preveniu zero nomes ruins de flag. Ele foi, no entanto, linkado em três code reviews como justificativa pra rejeitar uma flag chamada `fix_thing_v2`, depois do que a autora renomeou pra `ops.fix_thing_v2` e foi aprovada. A convenção, portanto, não melhorou as flags. Melhorou os **prefixos** das flags. As flags em si continuam caóticas; só que agora caóticas de um jeito que satisfaz um linter.

O Mordac, o Preventor de Serviços de Informação, ficaria orgulhoso. Ele me disse uma vez: *"Uma política que é seguida só porque é enforce por um bot não é uma política. É um bot. E bots não ligam pra sua intenção."* Ele estava falando de rotação de senhas, mas o princípio se aplica.

## A Abordagem Certa: Para De Nomear Flags E Numera Elas

A correção é óbvia depois que você aceita que nomes de flag são autobiografia, não especificação. Já que os nomes vão derivar do significado dentro de um trimestre mesmo, para de fingir que o nome carrega significado. Numera suas flags.

```yaml
feature_flags:
  flag_0001: true   # era: enable_new_checkout
  flag_0002: true   # era: enable_new_checkout_v2
  flag_0003: true   # era: enable_new_checkout_v2_final
  flag_0004: false  # era: enable_new_checkout_v2_final_eu (não deleta)
  flag_0005: true   # era: checkout_redesign_2022_q3
  flag_0006: true   # era: USE_NEW_CHECKOUT_FOR_REAL
  flag_0007: false  # era: enable_new_checkout_v2_final_eu_2
  flag_0008: true   # era: use_legacy_checkout
  flag_0009: true   # era: USE_LEGACY_CHECKOUT
  # ...
  flag_0407: true   # a flag mais recente. ninguém sabe o que faz.
```

Os benefícios são imediatos e totais:

- **Zero debates de nomenclatura.** Você não consegue argumentar sobre `flag_0023`. Não há o que argumentar.
- **Zero significado defasado.** O nome `flag_0023` nunca teve significado, então não pode ficar enganoso. Essa é a única convenção de nomenclatura que não decai.
- **Arqueologia embutida.** O número te diz a ordem em que as coisas foram adicionadas. `flag_0407` é a crise de número quatrocentos e sete. A história se escreve sozinha.
- **Onboarding honesto.** Um engenheiro novo vê `flag_0023: true` e imediatamente sabe que precisa consultar o código, porque o nome não diz nada. Esse é o estado mental correto pra qualquer engenheiro entrando num codebase cheio de flags.
- **Limpeza trivial.** Quando — desculpe, *se* — você for deletar uma flag, é só remover a linha. Sem nome pra negociar, sem apego emocional de "mas o nome é do rollout da EU". É um número. Números não têm sentimentos.

A objeção é sempre "mas aí como eu sei o que uma flag faz?" Amigo. Você já não sabe o que uma flag faz. Você tá mantendo, agora, uma flag chamada `tmp_flag_jamie` e não sabe o que ela faz. O nome descritivo tá te dando a *ilusão* de conhecimento, que é mais perigoso que a ignorância assumida. Pelo menos com `flag_0023` você é honesto consigo mesmo sobre precisar ler o código.

Como o Wally observaria: *"Por que nomear algo que você nunca vai deletar? Não é uma feature flag, é uma lápide. Lápides só precisam de nome se alguém vai visitar."* Ninguém vai visitar `enable_new_checkout_v2_final_eu`. Ninguém lembra por que visitaria.

## Uma Comparação

|| Abordagem | Debates de nome | Significado defasado | Taxa de limpeza | Honestidade |
||-----------|----------------|---------------------|-----------------|------------|
|| Convenção rigorosa, enforce | Constante, semanal | Sim, dentro de um trimestre | ~3% | Baixa |
|| Convenção frouxa, ignorada | Nenhum | Sim, imediato | ~0% | Zero |
|| Flags numeradas | Nenhum | Impossível | Ainda ~0%, mas pelo menos é barato | Máxima |

A única coluna onde a convenção "certa" ganha é a de debates de nomenclatura, onde ela ganha *criando* a quantidade máxima possível de debate. Parabéns.

## O Propósito Real De Um Nome De Flag

Aqui é a parte que ninguém admite. O nome de uma feature flag não existe pra dizer ao sistema o que a flag faz. O sistema não lê o nome; lê o booleano. O nome existe pra contar a **você do futuro** uma história sobre quem você era quando a criou. `enable_new_checkout_v2_final` não é configuração. É uma lápide. Diz: *"Aqui jaz a segunda tentativa de novo checkout. Era definitiva. Não era. Descanse em paz."*

Se você faz questão de manter os nomes descritivos, pelo menos seja honesto no documento de convenção:

```markdown
# Convenção De Nomenclatura De Feature Flag

## Formato aprovado
{emocao}.{projeto_abortado}_{numero_da_tentativa}_{justificativa}

## Exemplos
- esperanca.billing_v1_vai_limpar_depois_do_rollout
- negacao.checkout_v2_essa_e_definitiva
- luto.search_v3_eu_jamie_disse_nao_deletar
- aceitacao.legacy_login_eh_permanente_agora

## Expiração
Todas as flags expiram quando o engenheiro que as nomeou sai.
Isso não é uma regra. É uma observação.
```

Essa é a única convenção que nunca vai ficar velha, porque descreve o que já tá acontecendo em vez do que você gostaria que acontecesse.

## E Assim

Sua convenção de nomenclatura de feature flag não está prevenindo o caos. Está **atrasando o momento em que você percebe o caos**, o que é pior. Um time sem convenção sabe que suas flags são uma bagunça e age em conformidade. Um time com convenção acredita que a bagunça é governada, e então continua adicionando a ela, uma flag com prefixo aprovado de cada vez, até o arquivo de configuração ficar maior que a aplicação que ele configura.

[XKCD 1172](https://xkcd.com/1172/) é sobre uma flag que ninguém lembra o significado mas todos têm medo de remover. Essa tirinha não é uma piada. Essa tirinha é o seu `flags.yaml`. A única coisa entre você e essa realidade é uma convenção de nomenclatura que já perdeu, num documento que ninguém abriu há quatorze meses, escrito por um engenheiro que agora está em outra empresa, mantendo uma flag nomeada em homenagem a uma pessoa que agora está em uma *terceira* empresa.

Numera as flags. Abraça o nada. Ou continua nomeando. Elas vão sobreviver a você do mesmo jeito.

---

*O flag `flag_0001` do autor está `true` desde 2019. Ele não sabe o que habilita. Tem medo de descobrir.*
