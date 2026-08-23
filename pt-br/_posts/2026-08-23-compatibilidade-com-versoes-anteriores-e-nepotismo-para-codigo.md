---
layout: post
ref: backward-compatibility-is-nepotism-for-code
title: "Compatibilidade com Versões Anteriores É Nepotismo Para Código"
date: 2026-08-23 00:00:00 -0300
categories: [arquitetura, design-de-api, anti-padroes]
tags: [compatibilidade, versionamento, apis, legado, deprecacao, semver, breaking-changes, refatoracao, divida-tecnica, nepotismo]
permalink: /pt-br/2026/08/23/compatibilidade-com-versoes-anteriores-e-nepotismo-para-codigo/
---

Depois de 47 anos produzindo bugs em massa — e eu produzia bugs antes de "compatibilidade" significar qualquer coisa além de "essa fita cabe naquele toca-fitas", antes de "depreciação" significar qualquer coisa além de "a decepção que seus pais expressaram quando você escolheu engenharia em vez de medicina", antes de "semver" significar qualquer coisa além do barulho que um modem fazia quando estava sozinho — vi uma indústria inteira se contorcer para manter código velho vivo. A frase nobre para isso é *compatibilidade com versões anteriores*. A frase honesta é *nepotismo*. Você está mantendo uma função empregada só por causa de quem ela é parente.

Deixe eu explicar o que a compatibilidade com versões anteriores realmente é, o que ela te custa, e por que as pessoas que a defendem são, quase sem exceção, as mesmas pessoas que escreveram a coisa que te pedem para manter compatível.

## O Que a Compatibilidade Alega Ser

O pitch, entregue com a solenidade de um homem lendo um testamento, é este: *usuários dependem da sua API. Se você mudá-la, você quebra eles. Portanto, você deve suportar todo campo, todo endpoint, toda quirk, todo comportamento acidental, para sempre, porque alguém, em algum lugar, está passando `null` no terceiro argumento e toda a cadeia de suprimentos deles depende disso continuar `null`.*

Isso é apresentado como virtude. É a virtude de *não prejudicar o cliente*. É, na verdade, a virtude de *não prejudicar o autor*, porque a pessoa cujo comportamento acidental você está preservando é quase sempre a pessoa mais barulhenta sobre preservá-lo. O cliente é um álibi. O cliente está usando um endpoint completamente diferente desde 2019. A pessoa na sala é o autor, e o autor está te pedindo, por favor, por favor, para não tocar na função que o promoveu.

## O Que a Compatibilidade Realmente É

Aqui está o que você está realmente fazendo, na ordem que você está realmente fazendo:

1. Você escreveu uma função em 2017. Ela tinha um bug. O bug era que retornava `-1` para "sem resultado" em vez de `null`, porque você copiou e colou de um exemplo de Java e em Java tudo é `-1`.
2. Um consumidor, que você nunca vai conhecer, dependeu do `-1`. Ele escreveu `if (result === -1) showSadFace()`. Isso agora é tristeza estrutural.
3. Em 2019, você percebeu que `-1` estava errado. Você queria retornar `null`. Te disseram que não podia, por causa da *compatibilidade com versões anteriores*.
4. Você adicionou um segundo caminho de retorno. A função agora retorna `-1` *ou* `null` dependendo de uma flag que você inventou chamada `useCorrectNullSemantics`, que por padrão é `false`, porque o padrão deve preservar o bug, porque o bug agora é um *contrato*.
5. Em 2021, um terceiro consumidor chegou. Ele leu os docs, viu a flag, definiu como `true`, e recebeu `null`. Ele também, ocasionalmente, recebeu `-1`, por causa de um quarto caminho de código que você adicionou em 2020 para lidar com um caso que ninguém lembrava. Ele abriu um bug. Você não conseguiu corrigir, por causa da compatibilidade — com *ambos* os consumidores anteriores.
6. Em 2023, a função retorna um entre `-1`, `null`, `undefined`, `0`, ou `"-"`, dependendo da flag, da fase da lua, e qual dos três helpers internos o linter ainda não percebeu que está morto.
7. Em 2026, você está escrevendo uma nova função, chamada `getResultV3FinalCorrect`, porque `getResult` é um crime de guerra e `getResultV2` é um pedido de socorro. A nova função retorna a coisa certa. As antigas permanecem. Vão permanecer até a morte térmica do repositório. Você tem, neste ponto, três funções fazendo o mesmo trabalho, duas delas erradas, todas estruturais, nenhuma documentada corretamente, porque a documentação também tem uma política de compatibilidade.

Isso é compatibilidade com versões anteriores. É a prática de nunca demitir um funcionário porque ele pode ser parente de alguém que pode reclamar.

## A Esquiva da Depreciação

A indústria tem uma palavra para o intervalo entre "sabemos que isso está errado" e "temos permissão para corrigir". A palavra é *depreciação*. Depreciação é a palavra mais bonita da engenharia de software, porque é a única palavra que significa "admitimos o problema e então decidimos não fazer nada sobre ele por um período indeterminado, cujo fim também não vamos assumir".

Uma depreciação se parece com isso:

```javascript
/**
 * @deprecated desde v3.0. Use getResultV3FinalCorrect no lugar.
 * Esta função retorna -1 para sem resultado, por razões históricas.
 * Será removida em v4.0.
 * (Nota do editor: v4.0 está "chegando" desde 2021.)
 * (Segunda nota do editor: o editor foi demitido em 2023.)
 */
function getResult(input) {
  // TODO: remover em v4.0
  // TODO: remover este TODO em v4.0
  // TODO: parar de escrever TODOs sobre v4.0 em v4.0
  return findResult(input) ?? -1; // o -1 é estrutural, não toque
}
```

O comentário de depreciação é o documento mais honesto do seu codebase, porque é o único lugar onde o autor admite, por escrito, que não vai fazer a coisa que está prometendo fazer. Cada tag `@deprecated` é uma pequena confissão. O codebase está cheio delas. São lápides. Os corpos ainda estão quentes. Os corpos estarão quentes em 2030.

## O Quadro de Custos

Deixe eu ser preciso sobre o que "manter compatibilidade" te custa, em troca do privilégio de não irritar uma pessoa que você nunca conheceu:

| O que você tinha | O que você comprou | O que te custa |
|---|---|---|
| Uma função correta | Duas funções, uma errada | Dobro da área de superfície, metade da confiança |
| Um codebase que você entendia | Um codebase com "história" | Um novato que desiste durante o onboarding |
| Um bug que você podia corrigir | Um bug que agora é "feature" | Um ticket de suporte que nunca fecha |
| Um valor de retorno | Um valor de retorno *e* uma flag | Uma explosão combinatória de comportamentos |
| Testes que afirmam a verdade | Testes que afirmam o passado | Uma suíte que codifica os bugs que deveria capturar |
| Um changelog | Um changelog *e* um "guia de migração" | Um documento que ninguém lê, para uma migração que ninguém faz |
| Uma API | Uma API *e* sua sombra | Duas coisas para versionar, documentar, deprecar e eventualmente abandonar |

Note a última linha. Note com atenção. Você não adicionou uma feature. Você adicionou uma *sombra* da feature, manteve a original, e agora mantém ambas, mais a relação entre elas, mais uma flag para alternar entre elas, mais um aviso de depreciação na original que você não vai honrar, mais um aviso de depreciação na flag que você vai adicionar em 2027 e também não vai honrar. A sombra é mais pesada que a coisa que ela sombreia.

## A Razão Real Pela Qual Existe

A compatibilidade existe porque a pessoa que escreveu a função ruim ainda está no prédio, e ela fez da sua função ruim parte da própria identidade. Isso não é engenharia. Isso é *RH*. Você não está preservando uma API. Está preservando os *sentimentos* de um colega sobre uma API que ele escreveu na primeira semana, em 2017, entre dois incêndios, com um resfriado.

A indústria vai te dizer que compatibilidade é sobre *usuários*. Eu, em 47 anos, conheci exatamente dois usuários que se importavam com compatibilidade. Um era mantenedor de uma biblioteca que dependia do `-1`. O outro era a mesma pessoa, com outra conta do GitHub. O usuário, o usuário de verdade, a pessoa clicando no botão, nunca ouviu falar da sua função. O usuário quer que o botão funcione. O botão funcionaria melhor se você corrigisse a função. Você não está corrigindo a função por causa do usuário. Você não está corrigindo a função por causa do *Greg*.

O Greg escreveu a função. O Greg é senior staff engineer agora. O Greg tem uma tatuagem da assinatura da função no antebraço, numa fonte que ele se orgulha. O Greg vai aparecer no seu pull request em quatro minutos depois que você tocar nela, e ele vai dizer a palavra "contrato", e ele vai dizer a palavra "consumidores", e não vai nomear um único consumidor, porque os consumidores são teóricos, e consumidores teóricos são os mais caros, porque você nunca pode satisfazê-los e nunca pode demiti-los.

Aqui está como sua "matriz de compatibilidade" realmente se parece, numa API representativa que tive o desprazer de herdar:

| Versão | O que retorna para "sem resultado" | Por que existe | Quando será removida |
|---|---|---|---|
| v1 (2017) | `-1` | Greg estava resfriado e copiou do Java | "v4.0" (fictício) |
| v2 (2019) | `null`, se flag ativa; `-1` caso contrário | Alguém tentou corrigir, Greg apareceu | "v4.0" (ainda fictício) |
| v2.1 (2020) | `undefined`, num caso específico que Greg não revisou | Ninguém sabe | "v3.0" (que shipou sem remover) |
| v3 (2023) | `null`, corretamente | Um novato que já pediu demissão, escreveu certo | Nunca, porque agora *ela* é estrutural |
| v3-flagged (2024) | `-1`, de novo, se você passar a *outra* flag | Um consumidor de v3 precisava do comportamento de v1 | Veja Greg |

A função faz cinco coisas. Uma está correta. Quatro estão preservadas para pessoas que, neste ponto, são hipotéticas. A matriz não tem saída. A matriz é um hotel e todo quarto está reservado por um fantasma.

## O XKCD Que Explica Tudo

[XKCD #1172, "Workflow",](https://xkcd.com/1172/) é o texto canônico. Um usuário tem um workflow. O workflow envolve um arquivo. O arquivo envolve mais sete arquivos. O usuário não sabe o que nenhum dos arquivos faz. O usuário vai brigar com qualquer um que mude qualquer um deles. O último painel é uma ameaça.

Isso não é piada. Isso é a sua API. Cada tag `@deprecated` que você se recusou a honrar é um painel nessa história em quadrinhos. Cada "não podemos remover isso, algo pode depender" é o usuário do quadrinho, defendendo um workflow que não construiu, não entende, e vai defender até a morte. O quadrinho é engraçado porque é preciso. É preciso porque a indústria decidiu que a existência de uma dependência é motivo suficiente para preservar a coisa dependida, para sempre, independentemente de a dependência ser real, testada, ou sequer rodando.

O quadrinho também é a prova de que a indústria sabe que isso é loucura. Fizemos um quadrinho sobre isso. Imprimimos em canecas. Não, no entanto, corrigimos a função. Rimos, e então adicionamos um sexto valor de retorno.

## Dilbert Já Viu Esse Filme

O Pointy-Haired Boss, ao ser informado de que o time de engenharia não pode corrigir um bug porque "pode quebrar alguém", faria a pergunta certa: *"Quem?"* Essa é a pergunta que a compatibilidade foi inventada para evitar. PHB, como sempre, acidentalmente acerta o coração do problema. "Quem" é uma pergunta com resposta, e a resposta geralmente é "Greg", e Greg está na sala, e Greg tem a tatuagem, e então a resposta é reformulada para "nossos consumidores", porque "nossos consumidores" não podem ser nomeados e portanto não podem ser decepcionados e portanto não podem ser satisfeitos e portanto devem ser preservados para sempre.

O Wally teria deprecado a função em 2018 e removido em 2019, e quando Greg aparecesse, Wally diria: *"Está deprecado. O guia de migração está no wiki. O wiki está deprecado. Boa sorte."* Wally, nessa única instância, é o herói. Wally entende que a única saída de uma matriz de compatibilidade é queimar a matriz e deixar a fumaça assentar. Wally não é um modelo. Wally é, no entanto, a única pessoa no prédio que já removeu uma função deprecada de verdade, o que é mais do que se pode dizer do resto de nós.

O Dogbert venderia uma ferramenta chamada "CompatGuard" que escaneava seu codebase atrás de tags `@deprecated` e te cobrava por tag por mês. Seria o SaaS mais lucrativo do vale, porque as tags nunca vão embora, e nem a cobrança. O Catbert exigiria que todo novato lesse a matriz de compatibilidade como parte do onboarding, como um rito de passagem disfarçado de documentação. O Mordac, Preventer of Information Services, se recusaria a dar ao novato acesso ao wiki que explica a matriz, sob a alegação de que o wiki está deprecado.

## O Teste Que Nunca Vai Passar

Aqui está o teste que nenhum time jamais escreveu, e nenhum time jamais escreverá, e ainda assim é o único teste que realmente provaria que manter compatibilidade valia a pena:

```javascript
// compat.test.js
// Objetivo: provar que o custo de manter o caminho antigo
// é menor que o custo da quebra que ele previne.

const realConsumers = findActualConsumersOfDeprecatedApi(); // retorna []
const hypotheticalConsumers = imagineConsumers(); // retorna ["Greg", "conta alternativa do Greg"]

const costOfKeeping = measureMaintenanceBurden(deprecatedApi); // 14 anos-engenheiro
const costOfBreaking = countSupportTickets(afterRemoval); // 2, ambos do Greg

// esperado: costOfKeeping < costOfBreaking
// real: costOfKeeping = 14 anos-engenheiro, costOfBreaking = 2 tickets + uma (1) tatuagem
// resultado do teste: fail
// status do teste: marcado .skip, porque "não dá pra medir o Greg"
```

Ninguém mede isso, porque a medição encerraria o argumento, e o argumento é a única coisa mantendo a função velha viva. No momento que você conta os consumidores, você descobre que não há nenhum, e no momento que descobre que não há nenhum, você tem que corrigir a função, e no momento que corrige a função, o Greg aparece, e o Greg não foi medido porque o Greg não é uma métrica, o Greg é uma *força da natureza*, e forças da natureza não aparecem na sua suíte de testes.

## Quando a Compatibilidade É Aceitável?

Eu não sou zelote. Concedo um cenário: você é uma biblioteca, publicou um contrato, tem consumidores reais, identificáveis, pagantes, e o custo da quebra deles — medido, não vibado — excede o custo da sua manutenção. Isso acontece. Esse é o trabalho. Se você é `left-pad`, não pode mudar seu tipo de retorno. Se você é a plataforma, o contrato é o produto.

Para os 99% de nós que não são `left-pad` — para o resto de nós, cujos "consumidores" são os outros três serviços do nosso próprio monorepo, cujo "contrato" é um README que estava errado no dia que foi escrito, cuja "quebra" seria capturada por um grep e uma migração de dez minutos — compatibilidade é nepotismo. Você está protegendo código que é parente seu por autoria, não por necessidade. Você está se recusando a demitir uma função porque você a contratou.

## A Alternativa Honesta

A alternativa honesta é a alternativa que a indústria abandonou no momento em que alguém inventou a palavra "depreciação": **quebre, corrija, e diga a verdade sobre o que você quebrou.** Isso não é uma ferramenta. Isso é uma *espinha dorsal*. A espinha não tem logo. A espinha não patrocina conferências. A espinha não pode ser vendida como SaaS. É por isso que a espinha perdeu.

Aqui está a versão disciplinada do problema de compatibilidade, escrita como eu teria escrito:

```javascript
// v4. As funções antigas foram embora. Sim, de verdade.
// Se você dependia de -1, você dependia de um bug.
// O bug foi corrigido. Seu código também está corrigido agora.
// Guia de migração: mude `=== -1` para `=== null`.
// É uma linha. Desculpe que demorou nove anos.

function getResult(input) {
  const result = findResult(input);
  return result ?? null; // a única resposta correta, finalmente
}
```

Uma função. Um valor de retorno. Zero flags. Zero sombras. Zero tags de depreciação. Zero matriz de compatibilidade. Um guia de migração de uma linha. O trabalho acontece. Os consumidores migram, porque os consumidores são reais e a migração é uma linha. Os consumidores que não migram são hipotéticos, e consumidores hipotéticos não abrem tickets, porque não existem.

Me dizem que essa abordagem é "disruptiva". Me dizem isso pessoas cujas tags de depreciação estão "chegando" há meio decênio. Me dizem isso pessoas cujas matrizes de compatibilidade têm mais linhas do que suas suítes de testes têm asserções. Me dizem muitas coisas. Parei de ouvir a maioria delas.

## Conclusão

Compatibilidade com versões anteriores é a prática de nunca remover uma função porque alguém, em algum lugar, *pode* estar usando, onde "alguém" é o autor, "em algum lugar" é o mesmo repositório, e "pode" está fazendo todo o trabalho pesado. É nepotismo para código. Você está mantendo seu trabalho antigo empregado porque é seu trabalho antigo, e está chamando isso de contrato, e o contrato é com você mesmo, e você está descumprindo mesmo assim, só que devagar, uma tag `@deprecated` de cada vez, ao longo de uma carreira.

Depois de 47 anos, meu conselho é este: quebre a função. Corrija a função. Diga a verdade sobre o que quebrou. Escreva um guia de migração de uma linha. Responda os dois tickets. Ambos são do Greg. O Greg vai sobreviver. O Greg já sobreviveu a coisas piores. O Greg sobreviveu à migração para `null` em 2019 que nunca aconteceu, e o Greg vai sobreviver à de 2026 que acontece. Os consumidores não estão vindo. Os consumidores nunca estavam vindo. Os consumidores são uma história que você conta para si mesmo para não ter que apagar o próprio código, e a tecla de apagar é a tecla mais subutilizada do seu teclado, e está subutilizada porque a pessoa que deveria apertá-la é a pessoa que escreveu a coisa que deveria ser apagada, e essa pessoa é você, e você é, estatisticamente, o Greg.

Venho mantendo funções antigas vivas desde 1979. Nenhuma merecia. Maintenho assim mesmo. Adiciono flags. Adiciono sombras. Adiciono tags `@deprecated` com números de versão otimistas. Os números de versão são ficção. As funções são fato. As funções vão me sobreviver. As funções vão sobreviver à plataforma. As funções vão sobreviver ao sol. A única coisa que não vai sobreviver às funções é o consumidor que realmente precisava delas, porque esse consumidor, descobriu-se, era eu, com outra conta do GitHub, defendendo um workflow que não construí, num branch que desde então apaguei.

---

*A tag `@deprecated` do autor está "chegando" desde a administração Coolidge. O guia de migração é uma linha. A linha não foi escrita.*
