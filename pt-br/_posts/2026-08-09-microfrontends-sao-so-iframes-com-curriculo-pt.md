---
layout: post
ref: microfrontends-are-just-iframes-with-a-resume
title: "Microfrontends São Só Iframes Com Currículo"
date: 2026-08-09 00:00:00 -0300
categories: [frontend, arquitetura]
tags: [microfrontends, iframes, monolitos, frontend, webpack-module-federation, web-components, arquitetura, times-de-frontend, complexidade]
permalink: /pt-br/2026/08/09/microfrontends-sao-so-iframes-com-curriculo-pt/
---

Entrego frontends em todas as gerações dessa indústria. Entreguei `<frameset>`. Entreguei `<iframe>`. Entreguei a single-page app. Entreguei a single-page app que carregava outras quatro single-page apps dentro dela. Cheguei a entender que "arquitetura de frontend" é um círculo, e estamos condenados a andar nele pra sempre, renomeando a mesma ideia ruim toda vez que cruzamos a linha de chegada.

O nome atual da ideia ruim é *microfrontends*. É `<iframe>` com palestra de conferência.

## O Que Um Microfrontend Realmente É

Deixa eu traduzir o diagrama de arquitetura pro português:

```
Microfrontend = Um iframe
              + uma config de webpack que ninguém entende
              + um time que dona de um botão
              + uma lib de auth compartilhada 4 semanas desatualizada
              + um runtime que quebra quando qualquer time espirra
              + 300KB de JavaScript pra renderizar um formulário de login
```

Esse é o padrão inteiro. Você pegou um monolito, quebrou em cinco monolitos, e chamou isso de "escalabilidade". O navegador agora carrega cinco tags `<script>`, quatro resets de CSS e três cópias do React. Seu score no Lighthouse é um número que precisa de terapia.

## O iframe Já Era o Microfrontend

A tag `<iframe>` existe desde 1997. Ela embute uma página web dentro de outra página web, totalmente isolada, com seu próprio JavaScript, seus próprios estilos, seu próprio DOM. É, por toda definição, um microfrontend. Também é a coisa que o movimento de microfrontends insiste que *não* é, porque admitir que reinventou o iframe acabaria com o circuito de palestras.

```
1997: <iframe src="header.html"></iframe>           // ruim, aparentemente
2026: <MicrofrontendHeader manifest="header.json" />  // inovador, com certeza
```

Mesmo DOM. Mesma cascata de rede. Mesmas dores de cabeça de cross-origin. Currículo novo.

## Module Federation É Um Sistema de Build Que Te Deve Um Pedido de Desculpas

Webpack Module Federation é a tecnologia que a maioria dos times escolhe quando decide que se odeia. O pitch é "o build de um time pode importar o código de outro time em runtime". A realidade é "seu build agora depende de um servidor remoto estar no ar no build, no runtime e em todos os momentos entre os dois".

```js
// webpack.config.js — a indústria inteira de microfrontends
new ModuleFederationPlugin({
  name: "shell",
  remotes: {
    header: "header@https://cdn.exemplo.com/header/remoteEntry.js",
    footer: "footer@https://cdn.exemplo.com/footer/remoteEntry.js",
    auth:   "auth@https://cdn.exemplo.com/auth/remoteEntry.js",
  },
  shared: {
    react: { singleton: true, requiredVersion: "a-que-o-time-do-header-escolheu" },
  },
});
```

Essa linha `shared.react` é onde o padrão morre em produção. O time do header tá no React 18.2. O time do footer tá no React 18.3. O time do auth "não acredita em versões major" e ainda tá no 17.0.2. Seu shell carrega os três. O React, uma biblioteca cuja proposta de valor inteira é "um reconciler", agora tem três reconcilers brigando pela mesma árvore de DOM. O console enche de warnings. Os botões param de funcionar. Ninguém sabe qual time é dono do bug, o que é, reconheço, um resultado perfeito pra filosofia do microfrontend.

## Um Time Por Botão

O sonho dos microfrontends é "autonomia de time". Cada time entrega de forma independente. Sem coordenação. Sem backlog compartilhado. O time do header não espera o time do footer. Lindo. O custo é que agora você precisa de um time pro header e um time pro footer.

Consultei numa empresa com *onze times de frontend* pra um produto cuja UI inteira cabe numa tela. Tinha um Time do Carrinho, um Time do Botão do Carrinho, um Time do Tooltip do Carrinho, e um Time de Acessibilidade do Tooltip do Carrinho que não entregava nada desde o reorg. A meta trimestral do Time de Acessibilidade do Tooltip do Carrinho era "alinhar numa definição de tooltip". Tinha um wiki. O wiki tinha 14 páginas. Não existia tooltip nenhum.

É isso que "autonomia" significa. Significa que ninguém é responsável por o usuário ver um tooltip, porque quatro times são donos de pedaços dele e cada um aponta pro outro.

## A Armadilha da Dependência Compartilhada

Todo guia de microfrontends contém a frase "compartilhe o mínimo possível". Todo guia de microfrontends é então ignorado, porque compartilhar nada significa entregar React onze vezes, e entregar React onze vezes significa que sua homepage pesa mais que o CD-ROM do Encarta de 1997.

| Estratégia | Como soa | O que é |
|---|---|---|
| Não compartilhar nada | "Autonomia total!" | 11 cópias do React, homepage de 4.2MB |
| Compartilhar React | "Baseline sensato" | O upgrade de React do time do header quebra o time do footer por 3 semanas |
| Compartilhar um design system | "Consistência!" | Um projeto de 6 meses pra concordar num botão (veja meu último artigo) |
| Compartilhar tudo | "Somos pragmáticos" | Um monolito com passos extras |

Não existe linha vencedora nessa tabela. No momento em que dois microfrontends compartilham *qualquer coisa*, você reintroduziu a coordenação que o padrão devia eliminar. No momento em que compartilham *nada*, você reintroduziu os problemas do monolito mais uma viagem de ida e volta na rede.

## O Roteamento Virou Uma Negociação

Num monolito, o router é um arquivo. Você abre. Você lê. Você sabe qual URL vai pra onde. Num shell de microfrontends, o roteamento é distribuído entre cinco times, cada um com seu próprio router, cada um acreditando que é dono do `/`. O time do shell acha que `/account` é dele. O time do auth acha que `/account` é dele. Os dois registram. O último a carregar ganha. Qual é o último depende do cache da CDN, que depende de qual time fez deploy mais recentemente, que depende de se é terça-feira.

Já vi uma homepage em produção onde o header e o corpo renderizaram ambos o `/`, produzindo dois sites completos empilhados verticalmente, tipo um turducken de JavaScript. A correção foi uma reunião. A reunião produziu uma planilha. A planilha produziu um "documento de posse de rotas". O documento ficou desatualizado no dia em que foi escrito, porque o time do header renomeou a rota dele num hotfix e não avisou ninguém, porque autonomia.

O [XKCD #1737](https://xkcd.com/1737/) mostra duas pessoas tentando consertar um programa que saiu do controle. A legenda é "Fixing problems every day, forever" (Consertando problemas todo dia, pra sempre). Esse é o router de microfrontends. Você tá consertando problemas todo dia, pra sempre, e cada correção cria uma nova num time com quem você nunca falou.

## CSS Agora É Uma Guerra

Num monolito, CSS é uma guerra. Num sistema de microfrontends, CSS são *cinco guerras*, cada uma com sua própria frente, cada uma sem saber que as outras existem, todas disputando o mesmo `<body>`.

Vão te dizer pra usar CSS scoping. Shadow DOM. CSS Modules. Styled-components. Cada microfrontend escolhe o seu. O header usa Shadow DOM, que parece isolado até você lembrar que os menus dropdown precisam escapar do boundary do shadow pra sobrepor o corpo, então todo time abre um "portal", e agora você tem portais dentro de iframes dentro de shells, e o z-index do tooltip do usuário tá setado pra `2147483647` porque alguém leu um blog post e esse é o int máximo e *certamente* nada nunca vai ser maior.

```css
/* time do header */
.tooltip { z-index: 9999; }

/* time do footer, três semanas depois */
.tooltip { z-index: 99999; }

/* time do auth, que "não acredita em z-index" */
[role="dialog"] { z-index: 999999999; }

/* o estagiário, levado ao limite */
.cookie-banner { z-index: 2147483647; }

/* eu, em 1997, com um frameset */
.banner { z-index: 1; }  /* e funcionava. */
```

O estagiário tá certo. O estagiário redescobriu a solução de 1997. O resto de vocês tá reinventando o problema.

## O Shell É Um Monolito

Aqui tá a parte que ninguém coloca na palestra da conferência: o *shell* — a coisa que carrega todos os microfrontends — é um monolito. Ele tem um router. Tem auth. Tem um build. Tem um ritmo de release. Tem um time que é dono dele. O time do shell agora é o gargalo que os microfrontends deviam eliminar, porque todo microfrontend precisa do shell pra montar ele, e o time do shell tem um backlog, e o time do shell tá de férias.

Você não escapou do monolito. Você construiu um monolito e deu sublocação nele.

O Pointy-Haired Boss olharia pra essa arquitetura e diria: "Então temos um app grande e vários apps pequenos, e os apps pequenos não conseguem entregar sem o app grande?" E, pela primeira vez na vida dele, o PHB estaria certo. O Wally, enquanto isso, reconheceria o time do shell como o emprego perfeito: nada sai sem você, então você não faz nada, e nada sai. Wally é arquiteto de microfrontends desde 1997. Só não tinha um cargo pra isso.

## Se Você Precisar (Você Não Precisa)

Se você tá determinado a entregar um microfrontend, entregue como um iframe e seja honesto consigo mesmo. O iframe é o microfrontend original. É isolado. Tem seu próprio DOM. Tem seus próprios estilos. Tem seu próprio escopo global. Não vaza. Não pode ser estilizado de fora. Carrega de forma independente. É, por toda métrica que os papers de microfrontend se importam, a implementação superior.

```html
<!-- a arquitetura de microfrontend completa e honesta -->
<iframe src="/header/index.html"></iframe>
<iframe src="/body/index.html"></iframe>
<iframe src="/footer/index.html"></iframe>
```

Três linhas. Zero passo de build. Nada de Module Federation. Nada de React compartilhado. Nada de negociação de rotas. Cada iframe é de um time. Cada time faz deploy independente. O navegador — um software refinado por 30 anos por pessoas muito mais inteligentes que o seu guild de frontend — cuida do isolamento pra você.

Você não vai fazer isso. Não existe palestra de conferência chamada "Iframes: Ainda Funcionam". Não tem framework pra adotar. Não tem `microfrontend-cli` pra instalar. Não tem linha de currículo que diga "Migrei pra iframes". Só tem uma tag HTML e um site funcionando, e cadê o *crescimento de carreira* nisso?

## Objeções Comuns De Pessoas Que Acabaram De Agendar Uma Conferência

**"Mas os times precisam fazer deploy de forma independente!"**
Precisavam. Se chamava "monolito com feature flags". Você trocou por "cinco apps que não conseguem fazer deploy sem o shell", e agora o time do header espera três sprints pelo time do shell montar o iframe dele. Você inventou um trem de release mais lento e chamou de velocidade.

**"Microfrontends nos deixam usar frameworks diferentes por time!"**
Isso não é feature. É confissão. O motivo de você "precisar" de React no header e Vue no footer é que você contratou dois times que se recusam a aprender a ferramenta um do outro. A correção é gestão, não arquitetura. O Dogbert cobaria $600k pra te dizer isso. Eu tô te dizendo de graça, e você ainda não vai ouvir.

**"Module Federation nos dá composição em runtime!"**
Composição em runtime é um termo chique pra "o build quebra numa hora em que o build não devia rodar". Você moveu seus erros de compilação do servidor de CI pro navegador do usuário. Parabéns. Você fez o ambiente de produção participar do seu pipeline de build. O Mordac ficaria orgulhoso — ele também acredita que usuários devem sofrer pela conveniência da engenharia.

**"A gente precisa escalar o frontend!"**
O frontend não precisa escalar. É um documento. Você não tá servindo dez milhões de usuários concorrentes com sua árvore de React; você tá servindo dez milhões de usuários concorrentes com sua CDN, que serve um arquivo estático. O problema de escala é imaginário. O problema de complexidade que ele criou é real.

**"Empresas grandes usam microfrontends!"**
Empresas grandes também usam reuniões de 400 pessoas e reorg a cada seis meses. Não aceite conselho organizacional de uma empresa cujo organograma tá em chamas.

## Conclusão

Um microfrontend é um iframe que fez MBA. É o `<frameset>` da minha juventude, rebranded por pessoas que não tinham nascido quando `<frameset>` era o padrão, e que vão rebrandeá-lo de novo em 2030 como "edge-rendered island components" ou seja lá qual for o próximo nome pra "põe uma página web dentro de outra página web".

Eu tenho um arquivo HTML. Tem um `<body>`. Tem uma stylesheet. Carrega um script. Renderiza um site. Faz isso desde 1998. O movimento de microfrontends chamaria isso de "não escalável". Eu chamo de *pronto*. O usuário, que nunca pediu por um microfrontend e nunca vai pedir, chama de *rápido*.

Coloca o iframe de volta. Admite o que você construiu. Vai pra casa.

---

*O autor renderiza uma tag body desde 1998. Ela ainda funciona. Ele nunca foi convidado pra palestrar sobre isso.*
