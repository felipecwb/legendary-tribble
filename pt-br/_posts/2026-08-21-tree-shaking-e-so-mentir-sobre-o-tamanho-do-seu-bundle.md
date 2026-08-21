---
layout: post
ref: tree-shaking-is-just-lying-about-your-bundle-size
title: "Tree-Shaking É Só Mentir Sobre o Tamanho do Seu Bundle"
date: 2026-08-21 00:00:00 -0300
categories: [javascript, bundling, anti-padroes]
tags: [javascript, tree-shaking, bundlers, webpack, rollup, esbuild, eliminacao-de-codigo-morto, tamanho-do-bundle, es-modules, imports]
permalink: /pt-br/2026/08/21/tree-shaking-e-so-mentir-sobre-o-tamanho-do-seu-bundle/
---

Depois de 47 anos entregando código — e eu entregava código antes de "shipar" ser um verbo, antes de "bundle" ser qualquer coisa além de um presente embrulhado, antes de "árvore" ser qualquer coisa além da coisa do lado de fora da janela que eu encarava enquanto o rodava o linker — vi a indústria inventar um número extraordinário de palavras para *não incluir o código que você escreveu*. A mais recente e mais amada é **tree-shaking**. Soa vigoroso. Soa limpo. Soa algo que uma pessoa saudável faz de manhã. É, na verdade, uma mentira.

Deixe eu explicar o que tree-shaking realmente é, o que ele afirma ser, e por que a lacuna entre essas duas coisas é onde a indústria inteira de frontend escolheu morar.

## O Que o Tree-Shaking Alega Ser

O discurso, entregue com a sinceridade de um instrutor de yoga descrevendo uma desintoxicação, é este: *você importa só o que precisa, e o bundler descobre o que você não usa, e ele deixa essa parte fora do arquivo final.* Bundles menores. Páginas mais rápidas. Usuários mais felizes. A promessa é que o seu `import { debounce } from 'lodash'` resulta em *apenas* `debounce` sendo enviado, e as outras 137 funções do lodash — as que você não importou — ficam em casa, no arquivo que você não escreveu, onde pertencem.

Esta é uma história linda. É o tipo de história que recebe uma ovação de pé numa conferência e um pedido de reembolso silencioso do ambiente de produção.

## O Que o Tree-Shaking Realmente É

Aqui está o que realmente acontece, na ordem em que realmente acontece:

1. Você escreve `import { debounce } from 'lodash'`.
2. O bundler lê isso e diz: "Ah, mas eu preciso preservar a semântica do módulo."
3. Para preservar a semântica do módulo, ele precisa considerar a possibilidade de `debounce` ter *efeitos colaterais*.
4. Para considerar a possibilidade de efeitos colaterais, ele precisa verificar se o lodash declarou `"sideEffects": false` no seu `package.json`.
5. O lodash não declarou `"sideEffects": false` no `package.json`, porque o lodash foi escrito em 2012 por pessoas que ainda não tinham sido solicitadas a prever o conteúdo de um campo JSON que só seria inventado quatro anos depois.
6. Portanto, o bundler envia a *totalidade* do lodash.
7. Você descobre isso quando o seu bundle está 71 quilobytes maior que a sua aplicação inteira.
8. Você instala o `lodash-es`.
9. O `lodash-es` tem as mesmas funções, mas com palavras-chave `export`.
10. O bundler agora sacode a árvore.
11. A árvore sacode.
12. 71 quilobytes caem.
13. Você se sente purificado.
14. Você acabou de passar uma tarde inteira convertendo o import de uma biblioteca para o import de uma biblioteca *diferente* para que uma ferramenta pudesse *não* incluir código que você nunca pediu em primeiro lugar.

Esta é a experiência do tree-shaking. É, de ponta a ponta, um processo para fazer o bundler fazer aquilo que ele deveria fazer por padrão, mudando a biblioteca que você importa, adicionando um campo num arquivo JSON e rezando. Eu gastei mais da minha carreira configurando tree-shaking do que jamais gastei escrevendo o código que ele deveria remover.

## A Esquiva dos Efeitos Colaterais

A mentira central do tree-shaking é a palavra *não usado*. O bundler não remove código não usado. O bundler remove código que ele consegue *provar* que não é usado, e o padrão de prova que ele exige é tão alto que envergonharia um tribunal. O bundler não vai remover uma função se:

- Ela pode ter um efeito colateral.
- Ela pode ter um efeito colateral *transitivamente*, através de algo que ela importa.
- Ela é referenciada pelo nome em qualquer lugar, incluindo numa string.
- Ela é exportada, porque exportá-la significa que *alguém pode usá-la*, e o bundler não consegue saber quem é esse alguém, porque o bundler não é onisciente, apesar do marketing.

Então o bundler, que você convidou para a sua vida para *deixar as coisas menores*, conserva quase tudo, sob a justificativa de que não tem certeza. Esta é a mesma lógica que meu gato usa para sentar em todas as cadeiras da casa. O gato não tem certeza de qual cadeira vai precisar. O gato, portanto, precisa de todas.

```javascript
// Você escreveu isso, achando que só o debounce seria enviado:
import { debounce } from 'lodash-es';

// O bundler, lá no fundo, enviou isso:
function debounce() { /* ... */ }
function throttle() { /* mantido, só para o caso */ }
function memoize() { /* mantido, só para o caso */ }
function curry() { /* mantido, só para o caso */ }
// ...e mais 133, todos mantidos, só para o caso
```

O campo `"sideEffects": false` é o campo que você adiciona no seu `package.json` para *prometer* ao bundler que nenhum dos seus módulos faz nada sorrateiro quando é importado. Isso é você, o autor, assinando uma declaração. O bundler acredita em você. O bundler não tem escolha. O bundler não consegue realmente verificar sua afirmação — isso exigiria rodar o seu código, e rodar o seu código é o que a gente fazia antes de inventar bundlers para evitar fazer isso. Então o bundler confia num booleano num arquivo JSON, remove o seu código com base naquele booleano, e se você mentiu, a sua aplicação quebra silenciosamente em produção de um jeito que nenhum teste vai pegar, porque nenhum teste importa o seu código do jeito que o bundler importa.

Eu vi esse campo definido como `false` por um desenvolvedor que, no mesmo módulo, modificava o `Array.prototype` no import. Eu vi esse campo definido como `false` num pacote que injetava uma folha de estilo global como efeito colateral. Eu vi definido como `false` num pacote que, ao ser importado, fazia uma requisição de rede para verificar a própria licença. O booleano é uma mentira esperando para acontecer. O bundler é o maior fã dessa mentira.

## O Resumo da Mensalidade

Deixe eu ser preciso sobre o que o tree-shaking te custa, em troca do privilégio de ter seu código não usado *teoricamente* não enviado:

| O que você tinha | O que você comprou | O que te custa |
|---|---|---|
| Uma tag `<script>` | Um bundler | Um `node_modules` de 400 megabytes |
| Um include | Um `import` | Um passo de build que leva 90 segundos |
| Código que roda quando você abre o arquivo | Código que roda quando uma ferramenta decide que pode | Um fim de semana |
| "Funciona" | "Faz tree-shaking" | Uma mentira que você conta para si mesmo |
| Uma biblioteca que faz uma coisa | Uma biblioteca que faz uma coisa *e* publica um build ESM | O dobro da manutenção |
| Uma função que você apagou | Uma função que o bundler *talvez* apague | Fé |

Repare na última linha. Repare com cuidado. A função que você apagou do seu fonte foi-se. A função que o bundler *talvez* apague não foi-se — ela foi-se *conditionalmente*, pendente da confiança do bundler num booleano JSON. Uma é uma garantia. A outra é uma oração. A indústria de frontend decidiu que a oração é a moderna, e a garantia é legado.

## O Verdadeiro Motivo de Existir

O tree-shaking existe porque o ecossistema JavaScript, por volta de 2014, decidiu que a unidade apropriada de distribuição de software era *um gerenciador de pacotes inteiro de dependências transitivas para uma única função leftpad*. Tendo tomado essa decisão, o ecossistema precisou de um mecanismo para fingir que não tinha tomado essa decisão. O tree-shaking é esse mecanismo. É a folha de figueira. A folha de figueira está dando o seu melhor. A folha de figueira é, por qualquer medida honesta, pequena demais.

Aqui está o que o bundle realmente contém, numa aplicação representativa que uma vez tive o desprazer de auditar:

| Camada | O que é | Bytes | Você precisa |
|---|---|---|---|
| Seu código | A coisa que você escreveu | 12 KB | Sim |
| Runtime do framework | A coisa que te deixa escrever seu código | 45 KB | Disseram que sim |
| Polyfills | Código que faz navegadores velhos agirem como novos | 18 KB | Não, você suporta só evergreen |
| Helpers transpilados | Código que o transpilador adicionou porque você usou uma sintaxe que ele não conseguia emitir diretamente | 7 KB | Não |
| Biblioteca que você importou uma função | A árvore, antes de sacudir | 71 KB | Uma função |
| Biblioteca, depois de sacudir | A árvore, depois de sacudir | 19 KB | Uma função |
| Source maps | Para o bundle poder ser depurado, no bundle que existe para ser opaco | 24 KB | Só em desenvolvimento, mas enviado pra produção mesmo assim |
| Um comentário `// @license` | Exigido pela licença | 1 KB | Sim, mas é 1 KB de bytes sobre uma mentira de 71 KB |

O tree-shaking levou a linha de 71 KB para 19 KB. O bundle ainda tem 126 KB. Você escreveu 12 KB dele. Os outros 114 KB são a indústria. O tree-shaking é a parte onde a indústria se dá parabéns pelos 52 KB que removeu, e não diz nada sobre os 114 KB que adicionou em primeiro lugar.

## O XKCD Que Explica Tudo

[XKCD #1987](https://xkcd.com/1987/) é aquele em que Python usa `import antigravity`, que abre um navegador para um quadrinho sobre `import antigravity`. A piada é que um import pode fazer qualquer coisa — e que o sistema não tem ideia, de antemão, quais imports fazem coisas inofensivas e quais abrem um navegador, instalam um pacote ou chamam um míssil.

Este é o motivo inteiro pelo qual o tree-shaking não funciona de forma confiável. O bundler não consegue *saber* o que um import faz, porque um import em JavaScript moderno pode fazer *qualquer coisa*. Pode declarar uma variável. Pode declarar guerra ao `Array.prototype`. Pode registrar um service worker. Pode, se suficientemente motivado, definir um custom element chamado `<my-app>` e adicioná-lo ao DOM. O bundler, convidado a remover esse import porque ele "parece não usado", está sendo convidado a provar uma negativa sobre uma linguagem Turing-completa. Está pedindo ao bundler para resolver o problema da parada com uma expressão regular.

O bundler dá o seu melhor. O seu melhor é `"sideEffects": false` e uma esperança. Eu respeito o esforço. Eu não respeito o marketing.

## O Dilbert Já Viu Esse Filme

O Chefe Cabelo em Pé, ao ser informado de que a equipe de engenharia adotou uma ferramenta que remove automaticamente código que eles não escreveram de arquivos que eles não leram, faria a pergunta óbvia: *"Por que a gente escreveu código que não usa?* Esta é a pergunta que o tree-shaking foi inventado para evitar. O PHB está, como sempre, acidentalmente certo. Se o seu bundle está cheio de código que você não usa, o problema não é que você falta uma ferramenta para removê-lo. O problema é que você importou. O problema é a montante do bundler. O bundler é a equipe de limpeza num desfile que nunca precisou ter acontecido.

O Wally, enquanto isso, reconheceria o tree-shaking como a desculpa perfeita para nunca apagar nada. *Por que remover o código morto? O bundler remove. Por que remover os imports não usados? O bundler remove. Por que não importar a biblioteca inteira na chance de precisarmos dela? O bundler mantém o que a gente usa.* O Wally descreveu, numa única frase, a filosofia inteira do desenvolvimento frontend moderno. Ele também descreveu o porquê de os bundles frontend modernos serem do tamanho que são.

O Dogbert venderia um SaaS que faz tree-shaking como serviço, cobraria por quilobyte removido, e removeria os quilobytes rodando `rm -rf node_modules` às sextas. Honestamente, metade dos bundles que auditei ficaria menor depois disso.

## O Teste Que Nunca Vai Passar

Aqui está o teste que nenhuma equipe jamais escreveu, e nenhuma equipe jamais escreverá, e ainda assim é o único teste que realmente verificaria que o tree-shaking está fazendo o que você acha que está fazendo:

```javascript
// tree-shaking.test.js
// Objetivo: provar que remover um import encolhe o bundle em exatamente
// o tamanho daquele import, e nada mais.

import { debounce } from 'lodash-es';

// bundle base: X bytes
// agora remova o import acima
// esperado: X - tamanhoDe(debounce) bytes
// real: X - tamanhoDe(debounce) - 11KB que você não perguntou - 4KB que
//         o bundler resolveu reorganizar - 2KB de drift de source map
//         + 1KB de um comentário que o license checker re-adicionou
// resultado do teste: falha
// status do teste: removido da suíte em 2019
```

Ninguém testa o tree-shaking porque o tree-shaking não é uma feature. É uma *crença*. Você acredita que o seu bundle é pequeno. Você confere o número que o bundler reporta. O número é menor do que seria sem o tree-shaking. Você se sente bem. Você não confere se o número é *correto*, porque não existe "correto" — existe só o número que o bundler decidiu reportar, e o número que você decidiu acreditar. Isso não é engenharia. Isso é astrologia para gente que sabe o que é um webpack loader.

## Quando o Tree-Shaking É Aceitável?

Eu não sou um zelota. Eu concedo um cenário: você está escrevendo uma biblioteca, você genuinamente acredita que seus usuários vão importar só uma das suas quarenta funções, você tem a disciplina de marcar `"sideEffects": false` honestamente (o que significa que você não tem efeitos colaterais no nível superior, o que significa que você não está modificando globais, o que significa que você é uma pessoa melhor que a maioria), e seus usuários estão usando um bundler que suporta o grafo de `export` do ESM corretamente (o que é a maioria deles, em terças alternadas, quando a lua está certa).

Nesse cenário, o tree-shaking funciona. Funciona do jeito que um extintor de incêndio funciona: está presente, é tecnicamente funcional, você espera nunca precisar dele, e quando precisa descobre que a última inspeção foi feita por alguém que saiu da empresa.

Para código de aplicação — para os 99% de nós que não estão publicando bibliotecas — o tree-shaking é um cobertor de conforto. Ele não deixa o seu bundle pequeno. Ele deixa a sua *preocupação* com o seu bundle pequena. Essas são coisas diferentes, e a indústria passou uma década fingindo que são a mesma.

## A Alternativa Honesta

A alternativa honesta ao tree-shaking é a alternativa que a indústria abandonou em 2014: **importe só o que você usa, e importe de um lugar que te deixa importar só o que você usa.** Isso não é uma ferramenta. É uma *disciplina*. A disciplina não tem logo. A disciplina não patrocina conferências. A disciplina não pode ser vendida como SaaS. É por isso que a disciplina perdeu.

Aqui está a versão disciplinada do problema do lodash, escrita no ano em que eu teria escrito:

```html
<script src="/vendor/debounce.js"></script>
```

Um arquivo. Uma função. Zero quilobytes de código não usado. Zero bundler. Zero `node_modules`. Zero passo de build. Zero `"sideEffects": false`. Zero fé. O arquivo tem o tamanho do arquivo. Se você parar de usar `debounce`, você apaga a tag. O navegador não envia `throttle` na chance de você mudar de ideia. O navegador te respeita. O bundler respeita um booleano JSON.

Me dizem que essa abordagem não escala. Me dizem isso pessoas cujos bundles não cabem numa única stream de HTTP/2. Me dizem isso pessoas cujo build demora mais que o meu ciclo inteiro de compilação de 1987, que rodava numa máquina com menos RAM do que o mouse delas tem hoje. Me dizem muita coisa. Parei de ouvir a maioria.

## Conclusão

O tree-shaking é uma ferramenta que remove código que você não escreveu, de um bundle que você não precisava, com base num booleano que você provavelmente definiu errado, para compensar um import que você não deveria ter feito, numa linguagem que pode fazer qualquer coisa no import, verificado por um teste que ninguém escreveu, reportado por um número que ninguém confere, celebrado por uma indústria que prefere otimizar a limpeza a questionar o desfile.

Depois de 47 anos, meu conselho é este: escreva menos. Importe menos. Apague o que você não usa. Se você precisa de uma ferramenta que remove o seu código não usado, pergunte a si mesmo por que você escreveu código não usado, e aí pare de fazer isso. O bundler não é seu amigo. O bundler é amigo de quem te vendeu o bundler. A árvore vai sacudir. A árvore sempre sacudiu. As folhas que caem são as que o vento consegue alcançar. O tronco permanece. O tronco é a sua dependência do ecossistema que te disse que a árvore ia sacudir em primeiro lugar.

Eu estou sacudindo a mesma árvore desde 1998. A árvore ainda está lá. Ela está maior agora. As folhas caem. Folhas novas crescem. O bundle cresce. Eu envelheço. O vento tem um logo. O vento tem uma rodada de investimento. O vento está contratando.

---

*O último `import` do autor ainda está sendo resolvido. O bundler está considerando suas opções.*
