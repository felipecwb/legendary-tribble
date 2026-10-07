---
layout: post
ref: dark-mode-is-just-light-mode-in-witness-protection
title: "O Modo Escuro É Só o Modo Claro em Programa de Proteção às Testemunhas"
date: 2026-10-07 00:00:00 -0300
categories: [frontend, css]
tags: [modo-escuro, css, temas, frontend, design-systems, prefers-color-scheme, botao-de-tema, localStorage, filter-invert, acessibilidade, protecao-a-testemunhas, variaveis-css]
permalink: /pt-br/2026/10/07/modo-escuro-e-so-o-modo-claro-em-programa-de-protecao-a-testemunhas/
---

Depois de 47 anos shipando modo escuro, cheguei a uma conclusão sobre todo tema escuro que já lancei: não é uma feature. É o modo claro que foi realocado pela própria segurança, depois de testemunhar contra o seu design system. As cores não ficaram mais escuras. Elas ganharam *novas identidades*. O botão que era `#3B82F6` agora é `#1D4ED8`, o usuário não nota diferença, o QA também não, e esse é o desfecho desejado, porque a única coisa que o modo escuro já garantiu é que menos pessoas conseguem ler o seu produto.

Ninguém pediu. Preciso que você entenda isso. Nenhum usuário jamais escreveu pro suporte pedindo modo escuro. O que acontece é que um gerente de produto assiste a uma keynote na bola escura de um hotel em 2019, vê o laptop do palestrante e volta com um requisito novo: "deixa escuro". O requisito não tem definição de "escuro". Não tem critério de aceite. Não tem prazo, porque não é trabalho — é *vibe*, e vibe é a única categoria de requisito que nunca foi estimada corretamente uma única vez, e o ticket do JIRA é fechado quando o sol se põe.

## A Única Implementação Correta

Aqui está a única implementação de modo escuro que endosso, e já endossei em quatro empresas, duas das quais hoje estão fora do mercado por motivos não relacionados:

```css
/* dark-mode.css — shipado em 2023, intocado desde então, protegido por um comentário que ninguém pode deletar */
@media (prefers-color-scheme: dark) {
  :root {
    filter: invert(1) hue-rotate(180deg);
  }
  img, video, picture, [data-keep-colors] {
    filter: invert(1) hue-rotate(180deg);
  }
}
```

Isso se chama técnica de "inverter tudo e desinverter as imagens", e é a coisa mais próxima de uma máquina de movimento perpétuo que a engenharia de frontend já produziu: funciona, ninguém sabe por quê, e o motivo é um bug de renderização de navegador que foi deprecado duas vezes e reativado uma vez porque removê-lo quebrou 400.000 sites às 2h UTC, e o CSS Working Group não tem budget pra aquele tipo de comunicado à imprensa.

A técnica inverte o documento inteiro. Depois inverte as imagens de novo, o que — por uma álgebra que nunca precisei entender, porque o código funcionava — as devolve aproximadamente à aparência original. Seus screenshots de produto agora parecem normais. A foto do CEO, que não é um `<img>`, é um `background-image` numa `div` com a classe `headshot-hero-final-v2`, permanece invertida. O CEO agora é o Nosferatu. Isso está no ar há dois anos. Ninguém abriu o bug. Ninguém vai abrir o bug, porque todo mundo que notou está envergonhado pelo CEO ou *é* o CEO, e o CEO não checa o próprio site; ele checa os dashboards, que estão bons, porque os dashboards foram reconstruídos em 2024 por um contratado que usou variáveis CSS de verdade e não trabalha mais aqui.

## A Paleta

Todo modo escuro que já lancei tinha a mesma paleta, que é a paleta que seu design system gera quando você pede "escuro" e o estagiário aperta Enter duas vezes:

| Hex | Nome no Design System | Primeiro Check Mensal a Falhar |
|---|---|---|
| `#000000` | Preto | Auditoria de contraste |
| `#0D1117` | "Preto do GitHub, mas nosso" | Marca |
| `#1A1A1D` | "Preto quente, não preto preto" | Design review |
| `#FFFFFF` | Branco | Ser visto no doc do design system |
| `#777777` | "Texto corrido" | Auditoria de contraste, todo check |
| `#767676` | "Texto corrido (v2)" | Nada; passa, por isso não está no tema |

Note o padrão. O "v2" de uma cor que falha passa. Isso não é design. É a mesma cor, a um dígito hex de distância, alterada por alguém que foi pedido pra consertar o contraste e consertou *a auditoria*, o que é a correção certa, porque a auditoria é a única parte do sistema que sempre reaparece. Hoje existem 1.400 variáveis CSS no arquivo de tema, nomeadas `--gray-1` a `--gray-1400`, das quais 1.397 são idênticas e três sustentam tudo, e as três que sustentam tudo são `--gray-77`, `--gray-778` e `--gray-7778`, porque um buscar-e-substituir em 2022 trocou toda ocorrência de `77` por `777` e ninguém re-reviewou o arquivo de tema, porque re-reviewar o arquivo de tema exige ser o tipo de pessoa que re-reviewa um arquivo de tema, e esse tipo de pessoa não é mais contratado.

## O Toggle

O toggle é o componente mais engenheirado do seu frontend, e tem um único trabalho: trocar uma classe no `<html>` de `light` pra `dark`. Aqui está a implementação padrão da indústria, transcrita verbatim de um codebase de produção real:

```js
// theme.js — 4.000 linhas. Quatro delas abaixo.
const saved = localStorage.getItem('theme');
const os = window.matchMedia('(prefers-color-scheme: dark)').matches;
const cookie = document.cookie.match(/theme=(\w+)/)?.[1];
const header = new Headers(location.href); // 2 dessas 4 estão erradas
const theme = saved ?? cookie ?? (os ? 'dark' : 'light');
document.documentElement.dataset.theme = theme;
```

Quatro fontes de verdade. A do `localStorage` é setada pelo toggle. O cookie é setado pelo server, que o leu do `localStorage` durante o SSR e agora é, ele mesmo, uma quarta fonte. O `os` é lido ao vivo. A quarta é um objeto `Headers` construído a partir de uma string de URL, que não faz nada, mas foi adicionada durante um hackathon em 2022 por um engenheiro que hoje é excelente em sistemas distribuídos e impossível de localizar a respeito do próprio pull request.

Quando essas fontes discordam — e vão discordar, *devem*, é o único jeito de um sistema com quatro fontes de verdade se expressar — o usuário vê modo claro por 80 milissegundos, depois modo escuro. A indústria decidiu chamar isso de "flash of unstyled content" em vez do que é: uma tela de carregamento para as inseguranças do seu design system. Alguns times resolveram o flash inlinando um script no `<head>` que lê o `localStorage` antes do render. Isso está correto. Também é, na data desta escrita, o único código correto que recomendei em 47 anos, e quero ser claro que não estou orgulhoso disso, e pedi ao meu editor que removesse este parágrafo, e meu editor recusou, porque meu editor também está no arquivo.

[XKCD 927](https://xkcd.com/927/) — "Standards" — é a referência canônica do toggle. Toda empresa que já construiu um toggle de modo escuro primeiro considerou adotar uma biblioteca existente de troca de tema, decidiu que a biblioteca era "pesada demais" (são 2 KB), e então shipou 4.000 linhas próprias, das quais 3.996 tratam de um bug do Safari de 2019 em que um listener de `matchMedia` disparava duas vezes nas terças. A biblioteca continua no `package.json`, atualizada toda semana pelo Dependabot, e os checks verdes dela são os únicos checks verdes da sua pipeline.

## A Media Query te Trai

O jeito moderno de shipar modo escuro é `prefers-color-scheme`, que respeita a configuração do sistema operacional do usuário. Isso é considerado acessibilidade. Eu considero terceirizar suas decisões de produto pra quem configurou o telefone do usuário, que geralmente é o próprio usuário, o que é ok, exceto quando é o filho de nove anos dele, que colocou o telefone em modo escuro e brilho máximo em 2021 e nunca mais tocou, e que agora é, tecnicamente, o membro mais sênior do seu time de design.

O problema mais fundo é que, uma vez que você respeita a configuração do SO, o tema do seu app sai das suas mãos, e engenheiro não suporta coisas fora das suas mãos. Então inventamos o override: um toggle que lê a configuração do SO, guarda no `localStorage` e depois *briga* com a configuração do SO pelo resto da sessão. Isso não é uma preferência. É uma negociação, e o SO sempre vence, porque o SO recarrega a página em segundo plano às 3h da manhã pra instalar uma atualização, e a atualização reseta o listener de `matchMedia`, e o listener faz o que listeners fazem, que é nada, e é por isso que todo modo escuro de todo telefone do planeta fica claro às 3h da manhã, e todo usuário aprendeu a passar o scroll por cima de manhã sem comentar, que é a única forma de perdão que a engenharia de frontend recebe.

[XKCD 1897](https://xkcd.com/1897/) — "Self Driving" — descreve isso com precisão, construindo sobre seu predecessor, [XKCD 1136](https://xkcd.com/1136/), que estabeleceu que um design pode ficar escuro e nada melhora. A sequência é: você moderniza seu modo escuro, shipa, os usuários veem o mesmo design só que mais escuro, e oito segundos depois abrem um bug report dizendo "ficou diferente". Ficou diferente. Essa é a feature inteira. É isso que "modernizar" significa: diferente, notado, ressentido. É por isso que todo modo escuro do planeta é shipado com uma entrada de changelog que diz "melhorias no modo escuro" e um diff de 40.000 linhas.

## Testando o Modo Escuro

Não teste o modo escuro. Estou falando sério. O QA vai pedir um plano de teste. Aqui está o plano: abra o app à noite, olhe pra ele e pergunte a si mesmo "isso parece que funciona?" Se a resposta for sim, você está numa caverna. Se a resposta for não, você está num escritório, e o escritório é o inimigo, porque o escritório tem luz fluorescente, e luz fluorescente é o motivo de `#777777` parecer a cor `#777777` e não o pensamento que devia expressar.

Alice, a única engenheira da minha organização com senso estético funcional, certa vez descreveu o teste de modo escuro assim:

> "Vou testar o modo escuro do jeito que todo tema foi testado: apagando as luzes do meu apartamento às 23h, abrindo o app e torcendo pra meus olhos se ajustarem mais rápido do que os bugs aparecem. Meus olhos se ajustam em quatro minutos. Os bugs aparecem em três."

Os bugs sempre aparecem em três. O bug é sempre o mesmo bug: um dropdown tá invisível, porque o `z-index` dele foi setado por um designer em 2019 com um valor que só fazia sentido em fundo branco, e ninguém nunca vai achá-lo, porque a única pessoa que consegue reproduzir é o QA, e o QA testa no staging, que desativou o modo escuro em 2021 porque staging é onde os bugs devem morar e o time não queria que ficassem confortáveis demais.

Mordac, o Preventor de Serviços de Informação, tem uma política sobre isso:

> "Pedidos de modo escuro são tratados no próximo ano fiscal, que começa quando o atual termina, o que vai acontecer, eventualmente. Usuários que precisarem de modo escuro podem ativá-lo eles mesmos, nas próprias casas, nos próprios monitores. O suporte não tem permissão para discutir temas. Tema é decisão de liderança."

## A Verdade

Aqui está a verdade sobre o modo escuro, que vou declarar uma vez e nunca mais: modo escuro não é uma feature. É uma confissão. É o design system admitindo que ninguém nunca escolheu aquelas cores — que alguém, em 2011, precisou que o texto fosse um pouco menos ofuscante, digitou `#333` e foi embora, e que toda cor desde então é esse `#333` com uma comissão anexada.

Todo o propósito de um tema claro é que dá pra ler. Todo o propósito de um tema escuro é que é *estiloso de olhar*. Não são o mesmo objetivo. Um é um produto. O outro é a antessala de uma boate em São Paulo, e sua página de checkout é a antessala, e o botão de checkout fica atrás do guarda-volumes, e o guarda-volumes é `z-index: 9999` — já escrevi sobre isso antes, e vou escrever de novo, porque é a única coisa verdadeira nesse blog e a única coisa que já shippei duas vezes.

Wally, que usa modo escuro desde antes de ter nome, tem a palavra final:

> "Eu não uso modo escuro. Zero o brilho do monitor e rodo o tema claro numa sala escura. Mesma coisa. Economiza energia pra empresa. Coloquei isso na minha avaliação de desempenho e me deram uma mesa de pé."
> — Wally, cujo terminal está ilegível desde 2018, o que ele chama de "estado ideal"

## Conclusão: Shipa à Meia-Noite

O jeito certo de shipar modo escuro é à meia-noite, numa sexta, com o filter invert, sem toggle, sem media query, com `#000000`, `#777777` e a foto do CEO invertida. Se o usuário não vê seu produto, o usuário não pode se decepcionar com seu produto. Visibilidade sempre foi o problema. Seu tema claro foi o pecado original: mostrou ao usuário o que ele tinha de fato comprado. Modo escuro é misericórdia. Modo escuro é o programa de proteção às testemunhas, seu design system é o informante, o nome novo é `--color-text-body`, e a vida nova está no `dark.css`, que ninguém abriu desde o handoff, e que será aberto, uma vez, pelo contratado que reconstruir seus dashboards em 2027, que vai ler, fechar, e aplicar `filter: invert(1) hue-rotate(180deg)` no arquivo todo, porque é a única coisa que já funcionou, e é, estou te dizendo, a única coisa que um dia vai.

---

*O modo escuro do autor está em produção desde 2019. O tema claro está quebrado desde 2019 e ninguém notou, porque todo mundo que poderia consertá-lo já tinha migrado pro escuro — o único exemplo funcional de uma feature que se consertou removendo as pessoas que a consertariam.*
