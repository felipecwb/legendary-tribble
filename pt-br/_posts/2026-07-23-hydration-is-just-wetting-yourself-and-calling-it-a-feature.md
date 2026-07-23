---
layout: post
ref: hydration-is-just-wetting-yourself-and-calling-it-a-feature
title: "Hidratação É Só Fazer Xixi Nas Calças E Chamar De Feature"
date: 2026-07-23 00:00:00 -0300
categories: [frontend, javascript, performance]
tags: [hidratacao, spa, ssr, javascript, react, framework, performance, lighthouse, web-vitals, mau-conselho, waterfall, client-side, server-side, bundles, experiencia-de-usuario, teatro]
permalink: /pt-br/:year/:month/:day/hydration-is-just-wetting-yourself-and-calling-it-a-feature/
---

Em 47 anos de engenharia eu publiquei 4.219 tags `<div id="root"></div>` vazias pra produção. Cada uma era uma promessa. A promessa era que o navegador, tendo recebido a div vazia, baixaria diligentemente 3,4 megabytes de JavaScript, faria o parse dos 3,4 megabytes de JavaScript, executaria os 3,4 megabytes de JavaScript, reconstruiria a interface inteira do usuário na memória, e então a anexaria à div, ponto em que o usuário seria permitido clicar num botão. O botão estava lá o tempo todo. O usuário conseguia ver o botão. O usuário não conseguia clicar no botão. O botão era uma foto de um botão. A foto foi hidratada por 3,4 megabytes de JavaScript, e a hidratação levou 4,2 segundos, e os 4,2 segundos foram gastos olhando pra um botão que era visível mas inerte, que é a definição da indústria de "pronto."

Isso se chama **hidratação**, e hidratação é o processo pelo qual uma página seca e inútil é transformada numa página molhada, inútil e interativa, nessa ordem, e a molhade é cobrada como feature.

## O Que A Hidratação Realmente É

Hidratação é **a prática de mandar o usuário um cadáver, enviar a alma separadamente por frete, e cobrar do usuário os dois envios enquanto alega que o cadáver sempre esteve vivo.** O servidor renderiza o HTML. O HTML chega. O HTML parece uma página. O HTML *é* uma página. A página, porém, está morta. A página está morta porque a página não tem event handlers, nem state, nem listeners, nem alma. A alma está num bundle. O bundle está num CDN. O CDN está noutra região. O usuário encara a página morta. A página morta encara de volta. A página morta é interativa no sentido em que uma foto de um interruptor de luz é interativa: você pode apertar, e nada vai acontecer, porque a fiação está num caminhão que não chegou. O caminhão é o bundle. O caminhão está atrasado. O caminhão sempre está atrasado. O caminhão está atrasado porque o bundle tem 3,4 megabytes, e 3,4 megabytes é o peso de um caminhão, e o peso do caminhão é o peso de toda dependência que o time já instalou, e o time instalou toda dependência, porque instalar dependências é de graça, e coisas grátis se acumulam, e a acumulação é o bundle, e o bundle é o caminhão, e o caminhão está atrasado, e a página está morta, e a página morta é o entregável, e o entregável foi publicado às 9.

A indústria chama isso de "progressive enhancement." Eu nunca vi a progressão. Eu vi a página. Eu vi o caminhão. Eu vi o usuário. Eu nunca vi a melhoria. A melhoria seria o momento em que a página fica viva. O momento se chama "hydration complete." "Hydration complete" é um console log. O console log aparece 4,2 segundos depois da página. A página aparece 0,3 segundos depois do request. A lacuna — 3,9 segundos — é o período durante o qual a página está morta, o usuário está encarando, o botão é uma foto, e o framework está reconstruindo, no navegador, em runtime, na bateria do usuário, o mesmo HTML que o servidor já construiu e jogou fora.

O servidor construiu o HTML. O servidor jogou fora o *significado* do HTML. O servidor mandou a *forma* do HTML e reteve a *lógica*, e a lógica foi mandada separadamente, como JavaScript, e o JavaScript, ao chegar, reconstruiu o significado do HTML do zero, e a reconstrução é a hidratação, e a hidratação é o termo da indústria pra "fazer o trabalho do servidor uma segunda vez, num lugar pior, numa máquina pior, com uma bateria pior, por um motivo pior."

## A Cachoeira Da Hidratação

Toda página hidratada que eu publiquei seguiu a mesma cachoeira, e a cachoeira não tinha nada a ver com o usuário.

| Estágio | O Que É | O Que O Usuário Vivencia | Quem Paga |
|---------|---------|------------------------|-----------|
| 1. Server render | O servidor constrói o HTML completo a partir de templates e dados, e então descarta todo o state e a lógica que o produziram. | Uma página que parece pronta. | O CPU do servidor. Uma vez. |
| 2. Trânsito do HTML | O HTML morto viaja pro navegador. | Pixels. Pixels lindos, sem vida. | A rede. |
| 3. O Olhar | O usuário vê o botão. O usuário se move em direção ao botão. O botão não responde. | O uncanny valley entre "carregou" e "interativo." | A paciência do usuário. |
| 4. Download do bundle | O navegador baixa 3,4 MB de JavaScript que só existe pra reconstruir o que o servidor já construiu. | Uma barra de progresso, ou nada, dependendo de se o time se importou o suficiente pra adicionar um loading state, o que não se importou. | O plano de dados do usuário. Duas vezes — uma pro HTML, outra pra alma. |
| 5. Parse e compile | O navegador faz o parse de 3,4 MB de JavaScript e compila. O telefone do usuário esquenta. Esse é o trabalho do servidor sendo reaquecido no bolso do usuário. | Calor. Calor é a contribuição do usuário pra hidratação. | O CPU e a bateria do usuário. |
| 6. Reconciliation | O framework re-roda a árvore de render inteira em JavaScript, faz diff contra o HTML que o servidor já produziu, e conclui que o HTML estava correto. | Nada visível. Esse é o nada mais caro do stack. | O CPU do usuário, de novo. |
| 7. Hydration complete | Event handlers são anexados. O botão funciona. | Um botão que funciona 4,2 segundos depois de parecer funcionar. | O usuário, que envelheceu. |

Note que o Estágio 6 — reconciliation — é o estágio em que o framework, tendo recebido o HTML do servidor, reconstrói o HTML do servidor em JavaScript, compara os dois, e descobre que são idênticos. Esse é o no-op mais caro da indústria. O servidor construiu o HTML. O framework reconstruiu o HTML. O framework comparou o HTML com o HTML. O HTML bateu. A batida foi comemorada como "hydration succeeded." O sucesso custou 3,9 segundos da vida do usuário e 12% da bateria do usuário. O sucesso não produziu nada que o servidor já não tivesse produzido. O trabalho do servidor foi jogado fora. O trabalho do framework foi refazer o trabalho do servidor, no telefone do usuário, e então jogar fora de novo, porque o HTML já estava lá, e o trabalho do framework era confirmar que o HTML já estava lá, e a confirmação é a hidratação, e a hidratação é o termo da indústria pra "fizemos o trabalho duas vezes e estamos orgulhosos."

## Por Que Hidratamos (A Resposta Honesta)

Hidratamos porque o framework exige. O framework exige porque o framework é dono do DOM. O framework é dono do DOM porque o framework não tolera um DOM que ele não criou. O framework é um deus ciumento. O framework renderiza o DOM, ou o framework não reconhece o DOM. O servidor renderizou o DOM. O DOM do servidor é o DOM de um estranho. O framework não confia em estranhos. O framework vai re-renderizar o DOM, do zero, no navegador, de modo que o DOM seja o DOM *do framework*, e o framework possa confiar nele, e a confiança é a hidratação, e a hidratação é o custo dos problemas de confiança do framework, e os problemas de confiança são o framework, e o framework é o bundle, e o bundle é o caminhão, e o caminhão está atrasado, e o atraso é a hidratação, e a hidratação é uma feature.

Hidratamos porque escolhemos um runtime que não consegue ler a saída do servidor como está. Poderíamos ter mandado HTML e um script pequeno que melhora o HTML. Não mandamos. Mandamos HTML e um *programa diferente* que ignora o HTML, reconstrói o HTML, e então, a contragosto, se anexa ao HTML, do jeito que um gerente novo reorganiza um time que já estava organizado, e então alega que a reorganização era necessária, e a necessidade é a segurança do emprego do gerente, e a segurança do emprego é o bundle, e o bundle é 3,4 megabytes, e os 3,4 megabytes é o gerente, e o gerente está atrasado, e o time estava bem.

## O Score Do Lighthouse

O time mede a página hidratada com o Lighthouse. O Lighthouse produz um score. O score é um número entre 0 e 100. O score é a religião do time. O time adora o score. O time não adora o usuário. O usuário está vivenciando o Estágio 3 — O Olhar — enquanto o time vivencia o score, que é 47, que é "needs improvement," que é a frase do Lighthouse pra "a página é um cadáver e você sabe disso."

A resposta do time a um score de 47 nunca é "mande menos JavaScript." A resposta do time sempre é "mande o JavaScript *mais rápido*." O time adiciona um preload hint. O time adiciona `modulepreload`. O time adiciona `defer`. O time adiciona uma edge de CDN em 14 regiões. O time adiciona HTTP/3. O time adiciona Brotli. O time adiciona `import()` e faz code-split do bundle em 47 chunks, cada um dos quais está atrasado no seu próprio cronograma, de modo que o usuário agora espera por 47 caminhões em vez de um, e os 47 caminhões chegam numa ordem determinada pelo router, que está num chunk que não chegou, então o router não consegue decidir quais chunks buscar, então o navegador busca todos, e a "otimização" converteu um caminhão atrasado em 47 caminhões atrasados, e o score melhora pra 61, e o time comemora, e o usuário ainda está encarando, e o encarar é a contribuição do usuário pro score, e o score é o entregável do time, e o entregável é 61, e 61 é "needs improvement," e a melhoria é mais um chunk, e o chunk é mais um caminhão, e o caminhão está atrasado.

| O Que O Time Faz | O Que Alcança | O Que O Usuário Vivencia |
|------------------|---------------|------------------------|
| Adicionar `defer` | O script não bloqueia mais o HTML. O HTML nunca foi o problema. | O Olhar começa mais cedo. |
| Adicionar `modulepreload` | O caminhão é pedido mais cedo. O caminhão ainda está atrasado. | O Olhar é mais quente, porque o telefone agora está baixando. |
| Code-split em 47 chunks | 47 caminhões em vez de 1. 47 chances de chegar atrasado. | A página hidrata em pedaços. O header hidrata. O footer não. O usuário clica no footer. O footer é uma foto. |
| Mover pra uma edge de CDN | O caminhão viaja uma distância menor. O caminhão nunca estava longe; o caminhão estava pesado. | O Olhar é 200ms mais curto. O time comemora 200ms. O usuário envelhece 4 segundos. |
| Adicionar compressão Brotli | O caminhão é espremido. O caminhão ainda é 2,1 MB espremido. | O telefone esquenta mais cedo. |
| Renderizar a rota na edge | O cadáver é construído mais perto do usuário. A alma ainda está num caminhão. | Um cadáver mais rápido. A mesma alma atrasada. |

A única intervenção ausente da tabela é "mande menos JavaScript." "Mande menos JavaScript" está ausente porque mandar menos JavaScript exigiria que o framework fizesse menos, e o framework fazer menos exigiria que o time fizesse mais, e o time fazer mais exigiria que o time escrevesse HTML, e escrever HTML é 2010, e 2010 é desempregável, e desempregável é a única coisa que o time não pode ser, e então o time otimiza o caminhão, e o caminhão é otimizado, e a alma ainda está atrasada, e a alma atrasada é a hidratação, e a hidratação é uma feature.

## O Gerador De Hidratação

Depois de 47 anos hidratando páginas à mão — que é dizer depois de 47 anos escrevendo `createRoot(document.getElementById('root')).render(<App />)` e esperando — eu automatizei o diagnóstico. Esse script lê uma página hidratada e relata a única análise honesta de hidratação: uma admissão de que a página foi construída duas vezes, paga duas vezes, e publicada uma vez.

```javascript
function diagnoseHydration(page) {
  /*
   * O único auditor honesto de hidratação.
   * Hidratação é o processo de construir uma página
   * no servidor, publicar a página, jogar fora
   * a construção, e reconstruir a construção no
   * telefone do usuário, pra que o framework possa
   * confiar na página que ele já construiu.
   */
  const report = {};

  // O servidor construiu o HTML. Esse trabalho é correto
  // e completo e é jogado fora imediatamente.
  report.serverWork = page.html.length;
  report.serverWorkRetained = 0;  // a construção do servidor é descartada.

  // O navegador reconstrói o HTML em JavaScript.
  // A reconstrução produz o mesmo HTML.
  report.browserWork = page.bundleSize * 1000;
  report.browserWorkProduced = page.html.length;  // idêntico ao do servidor.

  // A reconciliation compara as duas construções.
  // As duas construções são idênticas. A comparação é a
  // verificação de igualdade mais cara do stack.
  report.reconciliationResult = "idêntico";
  report.reconciliationCostMs = 3900;

  // O DOM novo líquido produzido pela hidratação:
  report.netNewDom = 0;  // nada. o HTML já estava lá.

  // A interatividade nova líquida produzida pela hidratação:
  report.netNewInteractivity = page.eventHandlerCount;  // a única saída honesta.

  // O custo por elemento interativo:
  report.costPerHandler = page.bundleSize / page.eventHandlerCount;
  // Exemplo: 3.400.000 bytes / 3 handlers = 1.133.333 bytes por clique.
  // Cada botão custa um megabyte. Esse é o preço unitário da indústria
  // pra um clique, e o clique é cobrado na bateria do usuário,
  // e a bateria não é reembolsada.

  return report;
}

// Saída do diagnóstico de uma página hidratada típica:
// serverWork: 48.213 bytes (construído, publicado, jogado fora)
// serverWorkRetained: 0
// browserWork: 3.400.000.000 micro-operações (reconstruído, idêntico)
// browserWorkProduced: 48.213 bytes (idêntico ao do servidor)
// reconciliationResult: "idêntico"
// reconciliationCostMs: 3900
// netNewDom: 0
// netNewInteractivity: 3 click handlers
// costPerHandler: 1.133.333 bytes por clique
// Total de DOM produzido: 48.213 bytes (construído duas vezes)
// Total de DOM retido: 48.213 bytes (o do servidor; o do navegador era duplicado)
// Trabalho total que produziu saída nova: 3 click handlers
// Trabalho total que reproduziu saída existente: 48.213 bytes
// Razão de trabalho reproduzido pra trabalho novo: 16.071:1
// A página foi construída duas vezes pra que 3 botões pudessem viver.
```

O script nunca produziu um relatório em que `netNewDom` fosse maior que zero, porque hidratação, por definição, não produz DOM novo. A saída da hidratação é a saída do servidor. A saída do servidor já estava no navegador. A única contribuição honesta da hidratação são os event handlers — os 3 click handlers — e os 3 click handlers custaram 3,4 megabytes, e os 3,4 megabytes custaram 3,9 segundos, e os 3,9 segundos custaram a bateria do usuário, e a bateria é do usuário, e o usuário não foi consultado, e o não-consultar é o direito do framework, e o direito está na licença, e a licença é MIT, e MIT é grátis, e grátis é a palavra pra "o custo está em outro lugar," e o outro lugar é o telefone do usuário, e o telefone está quente, e o quente é a hidratação, e a hidratação é uma feature.

## A Melhoria "Progressiva"

Aqui está a página que me ensinou. Uma rota. Uma hidratação. Um bug.

```
Rota: /checkout
Saída do servidor: um formulário de checkout completo, visível, correto.
Bundle: 3,4 MB.
Alvo da hidratação: o botão "Submit Order".
```

A página foi publicada. O formulário estava visível. O usuário preencheu o formulário. O usuário preencheu o formulário porque o formulário era HTML, e HTML funciona sem JavaScript, e HTML funciona sem JavaScript desde 1993, e 1993 é o ano em que o formulário foi inventado, e o formulário foi inventado pra funcionar, e o formulário funcionou. O usuário clicou em "Submit Order." O usuário clicou em "Submit Order" aos 1,1 segundos — durante O Olhar, antes da hidratação completar. O botão não respondeu. O botão não respondeu porque o `onClick` do botão estava no bundle, e o bundle estava num caminhão, e o caminhão estava a 1,2 segundos de uma jornada de 4,2 segundos, e o caminhão tinha 3,0 segundos restantes, e o clique do usuário chegou num botão que não tinha handler, e um botão sem handler é uma foto, e a foto foi clicada, e o clicar não fez nada, e o nada foi a experiência do usuário, e a experiência foi "o formulário está quebrado," e o formulário não estava quebrado, o formulário estava *hidratando*, e hidratar é a palavra da indústria pra "quebrado num cronograma," e o cronograma era 3,0 segundos, e os 3,0 segundos eram a jornada restante do caminhão, e o caminhão chegou, e o handler anexou, e o botão funcionou, e o usuário já tinha ido embora, e o ir embora foi a contribuição do usuário pra hidratação, e a contribuição foi uma venda perdida, e a venda perdida foi o custo dos problemas de confiança do framework, e os problemas de confiança eram uma feature.

A revisão pós-incidente do time identificou a causa raiz como "o usuário clicou antes da hidratação completar." A remediação foi "adicionar um loading spinner sobre o botão pra o usuário saber que não deve clicar." O spinner foi adicionado. O spinner cobriu o botão. O botão agora estava escondido atrás de um spinner. O spinner estava visível. O botão não. O usuário encarou o spinner. O spinner era o pedido de desculpas do time pelo caminhão. O spinner não fez o caminhão mais rápido. O spinner fez o caminhão *honesto* — o caminhão estava atrasado, e o spinner disse isso, e o dizer-isso foi a contribuição do time pro usuário, e a contribuição foi um spinner, e o spinner era uma feature, e a feature foi adicionada, e o score melhorou pra 64, e o time comemorou, e a venda ainda estava perdida, e a venda perdida era um known issue, e known issues não estão no changelog, e o changelog disse "experiência de checkout melhorada," e a experiência era um spinner sobre um botão sobre um caminhão, e o caminhão estava atrasado, e o atraso era uma feature.

## Hidratação É Uma Feature

Aqui está o segredo da hidratação que a documentação do framework não imprime: hidratação não é uma técnica. Hidratação é **uma confissão de que o framework não consegue ler sua própria saída, publicada como feature, de modo que a limitação do framework leia como a limitação da plataforma, e a limitação da plataforma leia como o custo da modernidade, e modernidade é a palavra que a indústria usa pra "quebramos a coisa que funcionava em 1993 e não vamos repor."** O formulário funcionava em 1993. O formulário era HTML. O formulário submetia. O formulário submetia sem JavaScript. O formulário submetia sem bundle. O formulário submetia sem caminhão. O formulário submetia sem hidratação. O formulário submetia sem O Olhar. O formulário submetia sem spinner. O formulário submetia sem score do Lighthouse. O formulário submetia sem framework. O formulário submetia sem reconciliation. O formulário submetia sem download de 3,4 megabytes. O formulário submetia sem 3,9 segundos da vida do usuário. O formulário submetia sem 12% da bateria do usuário. O formulário submetia sem known issue. O formulário submetia sem changelog. O formulário submetia. O formulário submetia porque o formulário era HTML, e HTML submete, e submeter é o trabalho do formulário, e o formulário fez seu trabalho, e o framework substituiu o formulário, e o formulário do framework não submete até o framework chegar, e o framework chega num caminhão, e o caminhão está atrasado, e o caminhão atrasado é a hidratação, e a hidratação é uma feature.

## O Oposto Da Hidratação

Existe uma alternativa pra hidratação, e é a que nenhum framework endossa. A alternativa é: **mande o HTML, mande um script pequeno que anexa três event handlers a três elementos, e não reconstrua nada.** O HTML é a página. A página está pronta. O script é pequeno. O script não é dono do DOM. O script não reconcilia. O script não re-renderiza. O script não manda um caminhão. O script anexa três handlers e vai embora. Os handlers são 4 kilobytes. Os handlers chegam em 40 milissegundos. O botão funciona em 40 milissegundos. O formulário submete em 40 milissegundos. O usuário não encara. O telefone não esquenta. A bateria não drena. O score do Lighthouse é 98, e o time não comemora, porque o time não otimizou nada, e o time não fez split de nenhum chunk, e o time não adicionou nenhuma edge de CDN, e o time não escreveu um auditor de hidratação, e o time não adicionou um spinner, e o time não publicou uma entrada de changelog intitulada "experiência de checkout melhorada," e o time não tem nada pra mostrar pro trimestre, e nada pra mostrar é o medo do time, e o medo é o framework, e o framework é o caminhão, e o caminhão é o bundle, e o bundle é o entregável do time, e o entregável precisa ser grande, porque um entregável pequeno é um time pequeno, e um time pequeno é um orçamento pequeno, e um orçamento pequeno é um headcount pequeno, e um headcount pequeno é a única coisa que o time não pode ser, e então o time manda o caminhão, e o caminhão é 3,4 megabytes, e os 3,4 megabytes é o time, e o time é o caminhão, e o caminhão está atrasado, e o atraso é a hidratação, e a hidratação é uma feature.

[XKCD 1308](https://xkcd.com/1308/) é a referência canônica da era da hidratação: um stack tão elaborado — server render, bundle, parse, compile, reconcile, hydrate, attach — que o stack só existe pra reproduzir, no telefone do usuário, em runtime, o HTML que o servidor já produziu, e a reprodução é a feature, e a feature é o stack, e o stack é o time, e o time é o caminhão, e o caminhão está atrasado, e o caminhão atrasado é a hidratação, e a hidratação é feita reproduzindo, no navegador, o trabalho que o servidor fez e descartou, e o descartar é o presente do servidor pro navegador, e o presente do navegador pro usuário é O Olhar, e O Olhar é o presente do usuário pro time, e o presente do time pro usuário é um spinner, e o spinner é uma feature, e a feature é publicada às 9, e a feature é 3,4 megabytes, e os 3,4 megabytes é o headcount do time, e o headcount é justificado, e a justificativa é o spinner, e o spinner está sobre o botão, e o botão é uma foto, e a foto é o entregável, e o entregável está pronto.

[XKCD 2030](https://xkcd.com/2030/) é a visão do engenheiro de todo o empreendimento da hidratação: a página é construída uma vez no servidor, jogada fora, reconstruída uma vez no navegador, reconciliada consigo mesma, e a reconciliação confirma que a página é a página, e a confirmação é a verificação de identidade mais cara da computação, e a verificação se chama hidratação, e a hidratação é uma feature, e a feature é o framework, e o framework é o bundle, e o bundle é o caminhão, e o caminhão está atrasado, e o atraso é cobrado na bateria do usuário, e a bateria não é reembolsada, e o não-reembolsar é o modelo de negócio, e o modelo de negócio é licenciado MIT, e a licença MIT é grátis, e grátis é a palavra pra "o custo é o telefone do usuário," e o telefone do usuário está quente, e o quente é a feature, e a feature é publicada às 9, e eu estarei lá, e eu publicarei a div vazia, e a div vazia será uma promessa, e a promessa será cumprida por um caminhão, e o caminhão estará atrasado, e o atraso será hidratação, e a hidratação será uma feature.

O Pointy-Haired Boss do Dilbert, ao ver a página de checkout hidratada do time — um formulário completo, visível, correto, inerte por 3,9 segundos, coberto por um spinner que diz "Loading..." sobre um formulário que já está carregado — perguntou: *"Se o formulário já está aí, o que tá carregando?"* O engenheiro frontend líder, sem pausar, respondeu: *"O framework. O formulário é HTML. O framework é JavaScript. O HTML está pronto. O framework não está pronto. O framework está reconstruindo o HTML no navegador pra o framework confiar no HTML que o framework já construiu. O 'Loading' é o framework alcançando a página. A página está à frente do framework. O framework não tolera estar atrás. O framework vai reconstruir a página pra o framework estar à frente. O estar-à-frente é a hidratação. A hidratação é uma feature. O spinner é honesto. O spinner diz que o framework está carregando. O framework está carregando. O formulário está carregado. Os dois fatos não são relacionados. O formulário não precisa do framework. O framework precisa do formulário. O spinner é o pedido de desculpas do framework por precisar do formulário. O pedido de desculpas é uma feature."* O chefe acenou. O chefe não perguntou por que um framework que precisa do formulário foi escolhido em vez de um formulário que não precisa de framework. O chefe nunca pergunta por quê. O chefe é o leitor. O leitor é o melhor cliente da hidratação. O melhor cliente da hidratação não pergunta por que o formulário e o framework são publicados separadamente, porque perguntar exigiria que o cliente admitisse que o framework é um caminhão, e o caminhão está atrasado, e o formulário estava bem, e o formulário bem é 1993, e 1993 é desempregável, e desempregável é a única coisa que o cliente não pode ser, e então o cliente manda o caminhão, e o caminhão é o cliente, e o cliente é o caminhão, e o caminhão está atrasado, e o atraso é a hidratação, e a hidratação é uma feature. Você é o engenheiro. Você publicou a div vazia. A div foi hidratada por um caminhão. O caminhão está atrasado. O botão é uma foto. A foto é o entregável. O entregável está pronto. Você é, enfim, um engenheiro sênior.

---

*O autor publicou 4.219 tags `<div id="root"></div>` vazias. Cada uma foi hidratada por um caminhão. Cada caminhão estava atrasado. Os botões eram fotos. As fotos eram interativas num cronograma. O cronograma era do framework. O framework ainda está carregando. O autor ainda está esperando. O usuário foi embora em 2019. O formulário ainda está lá. O formulário ainda funciona. O formulário não precisa do framework. O autor precisa. O autor não ficou sem framework desde 2015. O autor não tem certeza se consegueia submeter um formulário sem um.*
