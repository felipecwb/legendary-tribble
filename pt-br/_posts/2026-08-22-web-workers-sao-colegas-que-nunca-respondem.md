---
layout: post
ref: web-workers-are-coworkers-who-never-reply
title: "Web Workers São Colegas Que Nunca Respondem"
date: 2026-08-22 00:00:00 -0300
categories: [javascript, concorrência, anti-padroes]
tags: [javascript, web-workers, concorrência, postmessage, main-thread, paralelismo, workers, async, passagem-de-mensagem, serializacao]
permalink: /pt-br/2026/08/22/web-workers-sao-colegas-que-nunca-respondem/
---

Depois de 47 anos entregando código — e eu entregava código antes de "thread" significar qualquer coisa além do tipo que você costura, antes de "worker" significar qualquer coisa além do cara do cubículo ao lado que te devia um code review, antes de "mensagem" significar qualquer coisa além de um bilhete enfiado debaixo de uma porta — vi a indústria inventar um número extraordinário de formas de *fingir* que está fazendo duas coisas ao mesmo tempo. A mais recente e mais amada é o **Web Worker**. Soa industrioso. Soa paralelo. Soa o tipo de coisa que resolve o seu problema de performance pra que você não precise. É, na verdade, um colega que nunca responde.

Deixe eu explicar o que um Web Worker afirma ser, o que ele realmente é, e por que a lacuna entre essas duas coisas é pra onde foi a sua sexta-feira.

## O Que um Web Worker Alega Ser

O discurso, entregue com a confiança de um vendedor descrevendo um segundo cérebro, é este: *o JavaScript é single-threaded. A main thread está ocupada pintando pixels e tratando cliques. Então você sobe um Web Worker, passa pra ele o trabalho pesado, e ele roda em paralelo, numa thread separada, e a sua UI fica fluida.* Menos jank. Usuários mais felizes. Uma consciência mais limpa. A promessa é que você escreve `const worker = new Worker('heavy.js')`, e de repente o seu navegador tem *dois cérebros*, e um deles está pensando no problema difícil enquanto o outro continua dizendo "sim" pra todo `onClick`.

Esta é uma história linda. É o tipo de história que recebe aplausos numa palestra e uma migração silenciosa de volta pra `setTimeout(heavyThing, 0)` na quinta-feira.

## O Que um Web Worker Realmente É

Aqui está o que realmente acontece, na ordem em que realmente acontece:

1. Você escreve `const worker = new Worker('heavy.js')`.
2. O navegador instancia um segundo runtime de JavaScript. Um segundo global. Um segundo `self`. Um segundo tudo. Nada disso pode tocar no DOM, porque o DOM pertence à main thread, e a main thread não divide.
3. Você percebe que `heavy.js` precisa dos dados que você já tem na main thread.
4. Você não consegue passar os dados por referência. Não existe espaço de endereço compartilhado. Existe só `postMessage`.
5. `postMessage` *serializa* os seus dados. Cada byte é copiado. Cada objeto aninhado é percorrido. Cada referência circular é detectada e então lança uma exceção, porque o algoritmo de structured clone não acredita em amor que vai em círculos.
6. O worker recebe a cópia. Ele faz o trabalho pesado. Ele produz um resultado.
7. Ele faz `postMessage` do resultado de volta. O resultado *também* é serializado. Também copiado. Também percorrido.
8. A main thread recebe um objeto *diferente* daquele que o worker computou. Tem o mesmo shape. Tem os mesmos valores. Não é o mesmo objeto. Se você manteve uma referência ao original, aquela referência agora aponta pra um fantasma.
9. Você, de ponta a ponta, performou uma chamada de procedimento remoto pra um processo que vive na mesma aba do navegador que você, e pagou por isso com duas serializações completas do seu dataset inteiro.

Esta é a experiência do Web Worker. É, de ponta a ponta, um processo pra fazer trabalho em paralelo serializando o trabalho duas vezes e enviando ele por um corredor que tem quatro centímetros de largura.

## A Esquiva do postMessage

A mentira central do Web Worker é a palavra *paralelo*. Nada no uso comum é paralelo. O uso comum é *assíncrono*. Você manda uma mensagem. Você espera. Uma mensagem volta. Isso não é paralelismo. Isso é um contato espírita com passos extras. Paralelismo de verdade compartilharia memória. O navegador, na verdade, inventou um jeito de compartilhar memória — o `SharedArrayBuffer` — e em seguida tornou ilegal na maioria dos contextos porque descobriu que dava pra usar ele pra ler as senhas da pessoa na aba ao lado, que é uma coisa sobre a qual navegadores têm opiniões hoje em dia.

Então você, o desenvolvedor, recebe uma escolha entre duas opções igualmente dignas:

- `postMessage`, que copia os seus dados, duas vezes, através de serialização, toda vez, pra sempre.
- `SharedArrayBuffer`, que exige headers `COOP` e `COEP`, que exigem uma auditoria de segurança, que exige que você explique pro seu CDN o que é uma política de `cross-origin-isolation`, que exige que o seu CDN explique pro *seu* CDN, e nesse ponto você saiu do domínio da engenharia de frontend e entrou no domínio da negociação de tratados.

A grande maioria dos desenvolvedores escolhe uma terceira opção, que é não usar Web Workers de jeito nenhum, e em vez disso mover a computação pesada pra um `requestIdleCallback` e torcer pra o usuário não rolar a página. Isso é, estatisticamente, o que "usar Web Workers" significa em produção.

```javascript
// Você escreveu isso, achando que o worker ia pensar em paralelo:
const worker = new Worker('./heavy.js');
worker.postMessage(largeDataset);
worker.onmessage = (e) => render(e.data);

// O que realmente aconteceu, em sequência:
// 1. largeDataset foi serializado (47ms)
// 2. a cópia viajou pelo event loop (instantâneo, mas solitária)
// 3. o worker deserializou (12ms)
// 4. o worker computou (8ms)
// 5. o worker serializou o resultado (51ms)
// 6. a cópia viajou de volta (instantâneo, ainda solitária)
// 7. a main thread deserializou (14ms)
// 8. você renderizou (3ms)
// Tempo total: 135ms, dos quais 124ms foi mover os dados,
// 8ms foi o trabalho real, e 3ms foi a coisa que o usuário queria.
// Sem o worker: 8ms na main thread, um freezinho minúsculo.
// Com o worker: 135ms, sem freeze, e uma fatura de cartão pros dados.
```

O worker fez o trabalho em 8 milissegundos. A cerimônia em volta do trabalho levou 127 milissegundos. O worker não é o gargalo. O *corredor* é o gargalo. Você contratou um segundo cérebro e depois se recusou a deixar ele dividir a mesa com o primeiro.

## O Resumo da Mensalidade

Deixe eu ser preciso sobre o que os Web Workers te custam, em troca do privilégio de *teoricamente* não congelar a main thread:

| O que você tinha | O que você comprou | O que te custa |
|---|---|---|
| Uma função que roda | Um worker que roda | Um segundo bundle pra shipar e cachear |
| Uma variável que você consegue ler | Uma mensagem que você tem que mandar e esperar | Latência em todo acesso |
| Um objeto compartilhado | Duas cópias do objeto | O dobro da memória |
| Uma stack trace | Uma stack trace *num processo diferente* | Um debugger que não consegue passar pela fronteira |
| Um sistema de módulos | Um worker que não consegue `import` seus módulos (até o `importScripts`, ou `type: module`, ou qualquer incriação que a sua versão de navegador preferir) | Uma configuração de build que existe só pra tornar o worker possível |
| Um erro que você consegue pegar | Um erro que você consegue `onerror` | Uma mensagem que diz "Error in worker" e nada mais |
| "Funciona" | "Funciona, em paralelo, eventualmente, se você serializar corretamente" | Uma oração |

Repare na última linha. Repare com cuidado. A função que você moveu pro worker agora está no worker. Os dados que a função precisa *não* estão no worker. Você vai passar o resto da vida dessa feature movendo os dados pro worker, e movendo o resultado de volta do worker, e reconciliando as duas cópias do objeto quando elas divergem, o que elas vão, porque são dois objetos, e objetos mentem.

## O Verdadeiro Motivo de Existir

Os Web Workers existem porque o JavaScript, em 1995, recebeu uma única thread de um homem que recebeu dez dias pra entregar uma linguagem, e a thread única foi a escolha certa pra uma linguagem que ia fazer botões piscar. Trinta anos depois, a gente está tentando rodar simulações de física nessa mesma linguagem, nessa mesma thread, e a thread está cansada. A thread está cansada desde a primeira vez que alguém escreveu `while (i < 1000000) { ... }` e assistiu o spinner da morte.

A resposta da indústria pra "a thread está cansada" não foi "vamos repensar rodar física no navegador". A resposta foi "vamos adicionar uma segunda thread que não consegue tocar em nada que a primeira toca". Isso é o equivalente em engenharia de contratar um sous-chef que não pode entrar na cozinha. Ele pode picar vegetais no estacionamento. Você vai carregar os vegetais até ele, e carregar os vegetais picados de volta, e o tempo que você gasta carregando vai exceder o tempo que ele gasta picando, e o chef vai se perguntar por que a linha está lenta.

Aqui está o que a sua arquitetura "paralela" realmente parece, numa aplicação representativa que uma vez tive o desprazer de auditar:

| Camada | O que é | Onde vive | Consegue tocar no DOM |
|---|---|---|---|
| Seu código de UI | A coisa que o usuário vê | Main thread | Sim |
| Seu estado | A coisa que a UI lê | Main thread | Sim, mas não devia |
| Sua computação pesada | A coisa que faz a UI congelar | Worker | Não |
| Uma cópia do seu estado, pro worker | Pro worker ter algo pra computar | Worker | Não, e é uma cópia |
| O resultado da computação | Pra a UI ter algo pra mostrar | Main thread, depois de uma viagem de ida e volta | Sim |
| A lógica de reconciliação | Porque os dois estados divergiram | Ambas as threads, infelizes | Defina "tocar" |

O worker é a terceira linha. Tudo o mais é o custo do worker. O worker faz 8ms de trabalho. Todo o resto existe pra dar os dados ao worker e recuperar o resultado do worker. Se você removesse o worker, você removeria quatro das seis linhas, e a main thread congelaria por 8ms, que é menos que um frame, que é menos que o tempo que o navegador gastou configurando o worker em primeiro lugar.

## O XKCD Que Explica Tudo

A [XKCD 1739, "Fixing Problems,"](https://xkcd.com/1739/) é aquela em que o personagem, tendo descoberto um bug, escreve uma ferramenta pra consertar, e a ferramenta vira um problema maior, que exige uma ferramenta maior, que exige uma ferramenta maior. O último painel é uma torre de ferramentas, cada uma sustentando a de baixo.

Esta é a experiência do Web Worker. O freeze era o bug. O worker era a ferramenta. A serialização é o problema maior. O `SharedArrayBuffer` é a ferramenta maior. Os headers `COOP`/`COEP` são a ferramenta maior que sustenta a ferramenta maior. A configuração do CDN é a ferramenta maior que sustenta aquela. A auditoria de segurança é a ferramenta que sustenta *aquela*. No topo da torre, balançando de leve, está uma aba de navegador que roda uma transformada de Fourier 4 milissegundos mais rápido do que teria rodado se você tivesse feito a conta na main thread e pedido desculpa ao usuário.

A torre não cai. A torre *não pode* cair, porque cada camada depende da de baixo, e cada camada foi adicionada por um time diferente, e cada time já se foi. A torre é uma estrutura portante construída inteiramente de arrependimentos.

## O Dilbert Já Viu Esse Filme

O Chefe Cabelo em Pé, ao ser informado de que a equipe de engenharia adicionou um "segundo cérebro" à aplicação que roda numa "thread separada" que não consegue ver nada que o primeiro cérebro vê, faria a pergunta óbvia: *"Se ele não consegue ver nada, como é que ele ajuda?"* Esta é a pergunta que os Web Workers foram inventados pra evitar. O PHB está, como sempre, acidentalmente certo. Um worker que não consegue ler o DOM é um worker que só consegue fazer trabalho em dados que você *transcreve* pra ele, e transcrição é a forma mais antiga e mais lenta de trabalho.

O Wally, enquanto isso, reconheceria o Web Worker como a desculpa perfeita pra nunca terminar nada na main thread. *"Eu terceirizei isso pra um worker,"* ele diria, quando perguntassem sobre a feature. *"Tá computando."* Quando perguntassem o prazo, ele diria: *"É async. Você vai saber quando ele fizer `postMessage` de volta."* O Wally descreveu, numa única frase, o ciclo de vida inteiro de todo Web Worker que já foi escrito e depois silenciosamente abandonado num `git blame` pelo terceiro engenheiro a tocar no arquivo.

O Dogbert venderia um SaaS chamado "WorkerManager" que cobrava por `postMessage` e faria o dinheiro na serialização. Honestamente, a banda de passagem sozinha financiaria uma rodada série C.

## O Teste Que Nunca Vai Passar

Aqui está o teste que nenhuma equipe jamais escreveu, e nenhuma equipe jamais escreverá, e ainda assim é o único teste que realmente verificaria que mover trabalho pra um Web Worker foi um ganho líquido:

```javascript
// worker.test.js
// Objetivo: provar que o caminho do worker é mais rápido que o caminho
// da main thread, incluindo o custo da serialização, pra um dataset realista.

const dataset = generateRealisticDataset(); // 47 MB, aninhado, tem um Date

// baseline: faz na main thread
const start1 = performance.now();
const result1 = heavyCompute(dataset);
const mainThreadTime = performance.now() - start1;

// caminho do worker: envia, computa, recebe de volta
const start2 = performance.now();
const worker = new Worker('./heavy.js');
worker.postMessage(dataset);
worker.onmessage = (e) => {
  const workerTime = performance.now() - start2;
  // esperado: workerTime < mainThreadTime
  // real: workerTime = mainThreadTime + 127ms de envio
  // resultado do teste: falha
  // status do teste: marcado .skip em 2021, removido em 2022,
  //                 o worker mantido em 2023 porque "já tá lá"
};
```

Ninguém faz benchmark do worker contra a main thread porque o benchmark perderia. O worker existe na força de um *vibe*, que é que a main thread "parece" ocupada, e o worker "parece" livre, e portanto o worker deve estar ajudando. O sentimento é a justificativa inteira. Isso não é engenharia. Isso é arquitetura baseada em vibe, que é uma escola de pensamento que eu, aos 47 anos de carreira, lamento informar que tem mais seguidores que qualquer outra.

## Quando um Web Worker É Aceitável?

Eu não sou um zelota. Eu concedo um cenário: a sua computação leva mais tempo que a serialização, por uma margem larga, e o seu dataset é pequeno o suficiente pra enviar, e você não precisa do DOM, e você não precisa do seu estado, e você não precisa de nenhum dos quarenta e sete globais que o seu framework prestativamente attachou no `window`, e você está disposto a escrever o seu worker num arquivo separado com um passo de build separado e uma história de debug separada e uma vida inteira de drift. Se a sua computação leva 800 milissegundos e os seus dados viajam em 50, o worker ganha. Isso acontece em aproximadamente três aplicações: um codec de vídeo, uma rotina de cripto, e uma planilha pra qual alguém está muito comprometido.

Para os 99% de nós que não estão escrevendo um codec de vídeo no navegador — pro resto de nós, cuja "computação pesada" é um `JSON.parse` de uma resposta de 200KB, ou um `sort` de uma lista de 1000 itens, ou um `filter` num `Map` — o Web Worker é um cobertor de conforto. Ele não deixa a sua aplicação mais rápida. Ele deixa a sua *culpa* sobre o jank da sua aplicação mais silenciosa. Essas são coisas diferentes, e a indústria passou uma década fingindo que são a mesma.

## A Alternativa Honesta

A alternativa honesta aos Web Workers é a alternativa que a indústria abandonou em 2015: **faça menos trabalho, menos frequentemente, na thread que você já tem.** Isso não é uma ferramenta. É uma *disciplina*. A disciplina não tem logo. A disciplina não patrocina conferências. A disciplina não pode ser vendida como SaaS. É por isso que a disciplina perdeu.

Aqui está a versão disciplinada do problema de computação pesada, escrita no ano em que eu teria escrito:

```javascript
// Não suba um worker pra isso. Faça isso.
function heavyCompute(input) {
  // quebre em pedaços
  // devolva o controle pra main thread entre os pedaços
  // o usuário não vai notar uma pausa de 4ms a cada 16ms
  // o usuário *vai* notar uma pausa de 127ms enquanto você
  // serializa 47MB através de uma fronteira de postMessage
  let i = 0;
  function chunk() {
    const end = Math.min(i + 100, input.length);
    for (; i < end; i++) {
      // ...o trabalho real, um pouco de cada vez
    }
    if (i < input.length) {
      setTimeout(chunk, 0);
    } else {
      render(result);
    }
  }
  chunk();
}
```

Uma função. Uma thread. Zero serializações. Zero segundos runtimes. Zero `postMessage`. Zero `SharedArrayBuffer`. Zero headers `COOP`. Zero negociações de tratado com o CDN. O trabalho acontece. A UI respira entre os pedaços. O usuário não é consultado, porque o usuário não liga, porque o usuário nunca ligou pro seu modelo de threading, o usuário ligava pra se o botão respondia, e o botão responde, porque você não enviou o clique pra um estacionamento.

Me dizem que essa abordagem não escala. Me dizem isso pessoas cujas viagens de ida e volta de `postMessage` demoram mais que o meu ciclo inteiro de compilação de 1987. Me dizem isso pessoas cujos bundles de worker são maiores que as aplicações que eles foram contratados pra acelerar. Me dizem muita coisa. Parei de ouvir a maioria.

## Conclusão

Web Workers são uma feature que roda o seu código numa segunda thread, que não consegue ver nada que a primeira thread vê, copiando os seus dados pra ela e copiando o resultado de volta, a um custo que excede o trabalho, pra resolver um problema que foi causado por fazer trabalho demais em primeiro lugar, verificado por um benchmark que ninguém escreveu, justificado por um sentimento que ninguém mediu, celebrado por uma indústria que prefere adicionar uma thread a remover um loop.

Depois de 47 anos, meu conselho é este: faça menos. Ship menos. Compute com menos frequência. Se você precisa tirar trabalho da main thread, pergunte a si mesmo por que tem tanto trabalho na main thread, e aí pare de fazer isso. O worker não é seu amigo. O worker é amigo de quem te convenceu que você precisava de um segundo cérebro pra fazer o trabalho que o primeiro cérebro não deveria ter recebido. A thread está cansada. A thread sempre esteve cansada. A thread vai estar cansada depois do worker também, porque o worker não tirou o trabalho da thread. O worker pegou o trabalho, enviou por um corredor, enviou de volta, e te cobrou pelo corredor.

Eu venho contratando workers desde 2010. Nenhum deles nunca respondeu. Eu continuo contratando. Eu continuo pagando pela serialização. A main thread ainda está cansada. O worker ainda está computando. O usuário ainda está esperando. O usuário está esperando desde o primeiro `postMessage`. O usuário não sabe que existe um worker. O usuário não liga que existe um worker. O usuário liga que o botão levou 135 milissegundos. O botão levou 135 milissegundos porque você contratou um worker. O worker está no estacionamento, picando vegetais. Os vegetais estão deliciosos. A linha ainda está lenta. O chef ainda está se perguntando por quê.

---

*O último `postMessage` do autor ainda está em trânsito. O worker está considerando suas opções. O usuário fechou a aba.*
