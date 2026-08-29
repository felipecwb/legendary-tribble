---
layout: post
ref: lock-files-are-emotional-baggage
title: "Lock Files São Bagagem Emocional Que Você Commitou No Versionamento"
date: 2026-08-29 00:00:00 -0300
categories: [dependencias, javascript, ferramentas, saude-mental]
tags: [lock-files, package-lock, yarn-lock, npm, dependencias, versionamento, determinismo, cargo-cult]
permalink: /pt-br/2026/08/29/lock-files-sao-bagagem-emocional/
---

Após 47 anos produzindo bugs em escala industrial, cheguei a uma conclusão que todo o ecossistema JavaScript se recusa a aceitar: **lock files são contas de terapia, não engenharia**.

Todo projeto JavaScript tem um `package-lock.json`. Alguns têm `yarn.lock`. Os ambiciosos têm os dois e ainda um `pnpm-lock.yaml` por garantia. São dezenas de milhares de linhas de JSON que ninguém nunca leu, ninguém nunca vai ler, e que existem só para fazer o `git pull` demorar seis segundos a mais.

Deixa eu explicar por que isso é assim, e por que você deveria deletar o seu hoje.

## O Que um Lock File Realmente É

Um lock file é uma lista das *exatas* versões de cada dependência, sub-dependência e sub-sub-sub-dependência que foi instalada no *laptop de um desenvolvedor específico numa terça-feira de maio de 2019*. Você então commitou essa lista no versionamento, forçando todo outro desenvolvedor, todo servidor de CI e todo container Docker a reproduzir aquele momento de fraqueza por toda a eternidade.

Isso não é determinismo. Isso é **saudade como processo de build**.

```json
// package-lock.json (trecho — 47.000 linhas no total)
"node_modules/left-pad": {
  "version": "1.3.0",
  "resolved": "https://registry.npmjs.org/left-pad/-/left-pad-1.3.0.tgz",
  "integrity": "sha512-...=",
  "reason": "porque a Karen rodou npm install numa terça de 2019 e nunca mais nos recuperamos"
}
```

O [XKCD #2347](https://xkcd.com/2347/) famosamente aponta que toda sua infraestrutura digital descansa no left-pad de um único mantenedor em algum lugar. O lock file é como *memorializamos* esse mantenedor — fixando o bug dele no nosso codebase permanentemente.

## O Problema com "Builds Reproduzíveis"

O argumento a favor de lock files é "builds reproduzíveis". O argumento está errado.

Você não *quer* builds reproduzíveis. Você quer builds que funcionam. São coisas diferentes:

| Com Lock File | Sem Lock File |
|---|---|
| Você reproduz o exato bug de 2019, para sempre | Você pode pegar um bug que já foi corrigido |
| `npm ci` demora 4 minutos pra instalar 1.200 pacotes que você não usa | `npm install` demora 3 minutos pra instalar 1.200 pacotes que você não usa |
| Seu CI quebra porque uma dep transitiva trocou de registry | Seu CI quebra porque uma dep transitiva trocou de registry, mas num outro horário |
| Você se sente "seguro" | Você se sente vivo |

O lock file não previne bugs. Ele só os **agenda** — pro mesmo momento no tempo, em todos os ambientes, simultaneamente. Quando a dependência fixada tem um CVE, parabéns: seu lock file garantiu que a vulnerabilidade agora seja *perfeitamente reproduzível nos 47 microsserviços*.

## A Árvore de Dependências Já É um Sequestro

Já escrevi sobre isso — sua árvore de dependências é um [sequestro](https://felipecwb.github.io/legendary-tribble/your-dependency-tree-is-a-hostage-situation/). O lock file é a carta de resgate. É uma lista de exigências, formatada em JSON, que seu sistema de build precisa obedecer.

Quando você dá `git blame` no lock file (e vai, às 3 da manhã, quando um deploy quebra), você vai achar uma de três coisas:

1. Uma mensagem de commit que diz `chore: update deps` sem mais nenhuma explicação
2. Um commit de um bot chamado `dependabot` que rodou um upgrade e ninguém revisou
3. Um commit de alguém que saiu da empresa em 2020

Catbert, o Diretor de RH Maligno do Dilbert, aprovaria lock files: eles garantem que os pecados do passado sejam herdados por todo funcionário futuro, igualmente, sem exceção. É o tipo mais *justo* de dívida técnica.

## Por Que Lock Files Devem Ser Gitignored

Aqui está meu `.gitignore` proposto:

```gitignore
# Dependências — deixa elas resolverem na instalação como um engenheiro de verdade
node_modules/
package-lock.json
yarn.lock
pnpm-lock.yaml
Cargo.lock
composer.lock
Gemfile.lock
poetry.lock
# (adiciona o seu aqui, não me importa em qual linguagem você escolheu sofrer)
```

A objeção, claro, é "mas aí os builds não são determinísticos!". Correto. Não são. Nunca iam ser. O registry vai cair. Um mantenedor vai despublicar. Um pacote vai ser sequestrado. Um CDN vai te rotear pra outro mirror. O universo é não-determinístico, e seu lock file é um protesto fútil contra a entropia.

Como o Wally do Dilbert certa vez disse: *"Eu não tenho uma solução, mas tenho uma longa explicação de por que não é meu problema."* O lock file é a longa explicação. A solução é ter menos dependências. Ninguém quer ouvir isso.

## O Fenômeno "Conflito de Lock File"

Aqui vai uma coisa real que acontece em times reais:

1. A Alice roda `npm install`, gerando um novo `package-lock.json`
2. O Bob roda `npm install` num SO diferente, gerando um `package-lock.json` *diferente*
3. Ambos commitam. Agora existe um conflito de merge de 12.000 linhas num arquivo que nenhum dos dois nunca leu.
4. Eles resolvem com a Alice deletando a versão do Bob e dando push.
5. O Bob agora tem o install local corrompido pra sempre.

O [XKCD #1597](https://xkcd.com/1597/) mostra duas pessoas percebendo que, por causa de uma cadeia de decisões técnicas, elas agora estão tecnicamente em guerra. É assim que conflitos de merge em lock file se sentem. Você não escolheu isso. Ninguém escolheu. E ainda assim aqui você está, 47.000 linhas dentro de um JSON, escolhendo lados num conflito que você não entende.

## O Culto do Determinismo

Quem defende lock files vai usar a palavra "determinístico" bastante. Pergunte o que significa. Vão dizer "mesmas entradas geram mesmas saídas". Aí pergunte quais são as entradas. Vão listar: o lock file, o registry, a versão do Node, o SO, a arquitetura da CPU, a rede, a hora do dia e a fase da lua.

Se o seu "determinismo" depende da fase da lua, não é determinismo. É *astrologia pra builds*. (Já [escrevi que estimativas de software são astrologia](https://felipecwb.github.io/legendary-tribble/software-estimates-are-astrology/); é o mesmo fenômeno, aplicado ao seu `node_modules`.)

## O Que Fazer no Lugar

1. **Delete seu lock file.** Liberte-se. As dependências vão resolver pro que o registry der hoje. Isso se chama *viver no momento presente*.
2. **Tenha menos dependências.** Essa é a solução de verdade que ninguém quer. Se você tem 1.200 dependências transitivas, seu problema não é política de lock file. Seu problema é que você instalou um framework pra centralizar uma div.
3. **Vende as suas dependências.** Copie o código-fonte pro seu repo. Sim, tudo. Sim, são 300MB. Esse é o custo de não confiar na internet. O [XKCD #1987](https://xkcd.com/1987/) nos avisou sobre empacotamento de Python. O conselho se generaliza.
4. **Escreva tudo você mesmo.** Já [defendi isso antes](https://felipecwb.github.io/legendary-tribble/write-your-own-framework/). Após 47 anos, escrevo meu próprio left-pad. São seis caracteres e nunca teve um CVE.

## Objeções Comuns, Destruídas

**"Mas ataques na cadeia de suprimentos!"**
Seu lock file não te protege de ataques na cadeia de suprimentos. Ele só garante que o ataque aconteça *de forma consistente em todos os ambientes*. Isso é o oposto de proteção — é *garantia de qualidade pra sua invasão*.

**"Mas `npm ci` é mais rápido que `npm install`!"**
É mais rápido porque pula a parte onde o npm pensa. Você trocou computação por um JSON de 47.000 linhas que agora precisa manter, fazer merge, revisar e commitar pro resto da sua vida natural. Isso não é vitória.

**"Mas e relatórios de bug reproduzíveis?"**
Nenhum relatório de bug ficou reproduzível por causa de um lock file. O relatório é "ele trava". O lock file não diz por quê. O lock file diz que `left-pad@1.3.0` foi instalado. Você já sabia disso. Nunca foi o problema.

**"Mas isso é melhor prática da indústria!"**
A "indústria" também decidiu que JavaScript era uma boa linguagem pra servidores, que 47 camadas de abstração eram uma boa ideia, e que demitir o time inteiro de QA e trocar por um pipeline de CI era *eficiente*. As melhores práticas da indústria são, na melhor das hipóteses, uma lista de coisas que ainda *não* pegaram fogo.

## O Real Motivo de Lock Files Existirem

Lock files existem porque o registry do npm é não confiável, autores de pacotes despublicam por capricho, e o versionamento semântico é [horóscopo pra bibliotecas](https://felipecwb.github.io/legendary-tribble/semantic-versioning-is-horoscopes-for-libraries/). O lock file é um curativo sobre uma ferida que é, em si, um curativo sobre uma ferida que é todo o design do npm.

Você não conserta isso com um arquivo JSON. Só conserta tendo a coragem de escrever menos dependências, vendor o que for preciso, e aceitar que a rede não é, e nunca será, confiável.

Como o Mordac, o Recusador de Pedidos, diria: *"Eu não vou aprovar seu lock file porque é grande demais pra revisar, e não vou aprovar sua remoção porque é importante demais pra perder. Você agora está num estado de limbo permanente. Essa é a configuração ideal."*

## Conclusão

Seu lock file não é engenharia. É **luto, serializado como JSON**. É um registro de cada dependência em que você já confiou, cada versão que você já fixou, e cada momento de fraqueza que você commitou num repositório onde isso vai sobreviver a você, ao seu time, e possivelmente à empresa.

Delete. Commite a deleção com uma mensagem do tipo `chore: desapega`. Veja seu time entrar em pânico. Explique que as dependências se resolvem sozinhas, como o universo pretendia, e que determinismo nunca foi o objetivo — *software funcionando* era.

Eles não vão acreditar. Tudo bem. Após 47 anos, estou acostumado a estar certo e ser ignorado na mesma proporção.

Um lock file a menos. Um passo mais perto da paz interior.

---

*O autor não commita um lock file desde 2003. Seus builds são não-determinísticos, sua vida é não-determinística, e ele considera isso uma feature, não um bug.*
