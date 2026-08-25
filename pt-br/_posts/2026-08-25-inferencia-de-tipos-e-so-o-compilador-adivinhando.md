---
layout: post
ref: type-inference-is-just-the-compiler-guessing
title: "Inferência de Tipos É Só Deixar O Compilador Adivinhar E Ficar Com Raiva Quando Ele Acerta"
date: 2026-08-25 00:00:00 -0300
categories: [linguagens, anti-padroes, tipagem]
tags: [typescript, haskell, inferencia-de-tipos, any, tipagem, compiladores, rust, generics, code-review, divida-tecnica]
permalink: /pt-br/2026/08/25/inferencia-de-tipos-e-so-o-compilador-adivinhando/
---

Depois de 47 anos produzindo bugs em massa — e eu produzia bugs em massa antes de "inferência de tipos" significar qualquer coisa além do ato de um engenheiro sênior apertar os olhos pra um `void*` e declarar, com a confiança de um homem que nunca esteve errado porque nunca foi checado, "é uma string, confia em mim" — vi uma geração inteira de linguagens vender pros engenheiros a ideia sedutora de que o compilador consegue descobrir seus tipos por você. A frase nobre pra isso é *inferência de tipos*. A frase honesta é *terceirizar o único pensamento que seu código exige pra um programa que não tem interesse no resultado*.

Deixa eu explicar o que a inferência de tipos alega fazer, o que ela realmente faz, e por que o que ela realmente faz é transformar toda code review num argumento sobre qual letra o compilador inferiu em tempo de compilação, que é uma frase que não deveria existir, e ainda assim aqui estamos, argumentando sobre isso, numa terça, num pull request que tem 47 comentários, e 46 deles são sobre um único `let`.

## O Que A Inferência de Tipos Alega Ser

O pitch, entregue com o zelo de uma pessoa que acabou de descobrir que `auto` existe em C++ e portanto foi aliviada do fardo de escrever um nome de tipo de novo, é este: *o compilador consegue deduzir o tipo de uma expressão a partir das expressões ao redor, então você não precisa escrever o tipo, então seu código é mais curto, então seu código é mais limpo, então seu código é melhor*.

Isso é apresentado como libertação. É a libertação de *não contar pro leitor que tipo de coisa uma coisa é*. É, na verdade, a libertação de *fazer todo leitor re-derivar, à mão, o tipo que o compilador deduziu automaticamente, só que o leitor não tem compilador, o leitor tem olhos, e os olhos do leitor estão lendo um arquivo num navegador num celular num trem, e o leitor não consegue rodar o compilador num celular, e então o leitor adivinha, e o leitor adivinha errado, e o errado do leitor agora é um bug na cabeça do leitor, e a cabeça do leitor é pra onde os bugs vão pra virarem incidentes de produção.

## O Que A Inferência de Tipos Realmente É

Aqui está o que você está realmente mantendo quando "deixa o compilador descobrir," na ordem que você está realmente mantendo:

1. Em 2019, um desenvolvedor escreveu `const result = doThing(x)`. A função `doThing` retornava `Promise<Result<Maybe<User>, Error>>`. O `const result` não mencionava isso. O `const result` não mencionava nada. O `const result` era um sussurro numa sala cheia. O desenvolvedor seguiu em frente. O desenvolvedor saiu. O `result` ficou. O `result` ainda está lá. O `result` ainda é não-tipado, no sentido de que seu tipo não está escrito, e "não escrito" é, nesta indústria, indistinguível de "não existe," e então o tipo não existe, e então toda pessoa que tocou no `result` desde 2019 o tratou como um `User`, e não é um `User`, é um `Maybe<User>`, e a diferença é a diferença entre "eu tenho um user" e "eu posso ter um user," e a diferença entre essas duas coisas é a razão inteira de `Maybe` ter sido inventado, e `Maybe` foi inventado pra que você fosse obrigado a tratar o caso `Nothing`, e você não tratou o caso `Nothing`, porque o caso `Nothing` não estava escrito, porque a inferência inferiu ele pra fora, e você não consegue tratar um caso que não consegue ver, e você não consegue ver um caso que não está escrito, e inferência é o ato de não escrever.

2. Em 2020, alguém adicionou `const data = response.json()`. O tipo de `data` foi inferido como `any`. `any` não é um tipo. `any` é a ausência de um tipo. `any` é o que o compilador diz quando ele desiste, e o compilador desiste muito, e o compilador desiste em silêncio, e o silêncio é o sistema de tipos, e o sistema de tipos é silêncio, e silêncio, num sistema de tipos, é um erro de runtime esperando sua deixa, e a deixa é uma terça, e a terça é hoje, e o erro de runtime está lendo o `data` como um array, e o `data` é um objeto, e o objeto tem uma chave chamada `data` que contém o array, e ninguém sabia, porque `any`, e `any` é o tipo de "confia em mim," e "confia em mim" é o que eu disse sobre o `void*` em 1979, e eu estava errado em 1979, e `any` está errado agora, e a única diferença é que em 1979 eu sabia que estava adivinhando, e em 2026 o compilador está adivinhando por mim, e o compilador não sabe que está adivinhando, e então eu também não.

3. Em 2021, um refactor mudou `doThing` pra retornar `Promise<Result<User, Error>>` — o `Maybe` foi removido, porque "a gente nunca pegou um `Nothing` de verdade." A mudança foi feita na função. A mudança não propagou pros 412 call sites que inferiram o tipo antigo, porque inferência é local, e local é o oposto de propagado, e então 412 lugares que estavam tratando silenciosamente um `Maybe` que já não existia continuaram tratando silenciosamente um `Maybe` que já não existia, e o código de tratamento era `if (!result) return null`, e o `result` nunca mais era `null`, e o `return null` estava morto, e o código morto era um conforto, e o conforto era uma mentira, e a mentira era a inferência de tipos, porque a inferência não te disse nada sobre os 412 sites, e os 412 sites eram o programa, e o programa agora estava errado em 412 lugares, e nenhum deles foi pego em tempo de compilação, porque a inferência pegou a mudança na função e não disse nada sobre o resto, e não dizer nada é o trabalho da inferência, e o trabalho foi feito.

4. Em 2022, alguém ligou o strict mode. Strict mode é o modo em que o compilador admite que `any` não é um tipo e exige que você o substitua por um tipo. O time substituiu 4.000 instâncias de `any` por `unknown`, porque `unknown` é o tipo que significa "eu reconheço que não sei o tipo," que é mais honesto que `any`, que significa "eu não reconheço que não sei o tipo," e honestidade é uma virtude, e a virtude era performativa, porque todo `unknown` era imediatamente seguido por `(x as SomeType)`, e `as` é o operador de asserção, e o operador de asserção é o operador que diz "eu sei mais que o compilador," e o compilador estava inferindo, e a inferência estava errada, e você sabia mais, e você assertou, e a asserção era uma mentira, e a mentira agora era checada por tipo, e o type-checker agora estava mentindo por você, e as mentiras compilavam, e a compilação era o quality gate, e o quality gate passou, e o bug de produção não passou, e o bug de produção está lendo um objeto como um array, de novo.

5. Em 2023, alguém introduziu um genérico. O genérico era `<T>`. O genérico era numa função que pegava um `T` e retornava um `T`. O genérico não fazia nada. O genérico era decorativo. O genérico estava lá porque o linter reclamou que a função não era genérica, e então a função ficou genérica, e o genérico era inferido em todo call site, e todo call site inferia `T` como `any`, porque todo argumento era `any`, e então o genérico era `any` de chapéu, e o chapéu era o sistema de tipos, e o sistema de tipos era um chapéu em cima de um `any`, e o `any` era o programa.

6. Em 2026, a codebase tem 47.000 tipos inferidos. Nenhum deles está escrito. Seis deles estão errados. Os seis errados são estruturais. Os seis errados são a razão do build passar e da produção falhar, e a lacuna entre "build passa" e "produção falhar" é a promessa inteira da inferência de tipos, entregada ao contrário: o compilador prometeu que pegaria seus erros de tipo em tempo de compilação, e ele pegou os que você escreveu, e ele não pegou os que ele inferiu, e os que ele inferiu são os que importam, porque os que você escreveu você pensou, e os que ele inferiu você não pensou, e não pensar é onde os bugs moram, e os bugs moram nos tipos inferidos, e os tipos inferidos são 47.000 linhas de código, e 47.000 linhas de código é muito bug, e eu saberia, porque venho escrevendo eles há 47 anos, e nunca escrevi um tipo que não pensei, e escrevi muitos tipos que não pensei, e a diferença é a inferência.

## A Inferência Que Se Devora

A indústria tem uma feature pra lacuna entre "o tipo é óbvio" e "o tipo não é óbvio." A feature se chama *anotações*. Uma anotação é um tipo escrito ao lado da coisa que o tipo descreve. Aqui está a diferença:

```typescript
// Inferido: o compilador descobre. O leitor não.
const users = await fetchUsers();

// Anotado: o compilador checa. O leitor lê.
const users: User[] = await fetchUsers();
```

A primeira linha é mais curta. A primeira linha é a que seu linter prefere, porque seu linter foi escrito por uma pessoa que acreditava que mais curto é melhor, e mais curto é melhor quando a coisa sendo encurtada é um comentário, e mais curto não é melhor quando a coisa sendo encurtada é a única documentação do que a coisa é. A segunda linha é mais longa por sete caracteres. Os sete caracteres são `User[]`. Os sete caracteres são a diferença entre um leitor que sabe e um leitor que adivinha. Os sete caracteres são o sistema de tipos inteiro, escrito, no lugar onde é usado, pela pessoa que escreveu, na hora que ela sabia o que era. Os sete caracteres são a documentação mais barata que você vai escrever. Você não vai escrever. O linter não vai deixar. O linter tem uma regra chamada `no-inferrable-types`, e a regra remove os sete caracteres, e a remoção é o trabalho da regra, e o trabalho está feito, e o leitor está adivinhando, e o leitor está num trem, e o trem não tem compilador.

Aqui está a tabela do que você trocou:

| O que você tinha | O que você comprou | O que te custa |
|---|---|---|
| Um tipo escrito | Um tipo que o compilador sabe e o leitor não | Uma code review gasta re-derivando o tipo à mão |
| Uma assinatura de função que dizia o que retornava | Uma assinatura que diz `const x = fn()` | Um novato que lê 47 linhas pra aprender o tipo de uma variável |
| Um refactor que propagava tipos | Um refactor que inferiu tipos novos em silêncio | 412 sites tratando um caso que já não existe |
| Um `any` que você sabia que estava errado | Um `any` que o compilador inferiu e você não viu | Um erro de runtime que o sistema de tipos prometeu prevenir |
| Um strict mode que pegava mentiras | Um strict mode cheio de asserções `as` | Mentiras que agora compilam, que é pior que mentiras que não compilam, porque mentiras que compilam são acreditadas |
| Um genérico que significava algo | Um genérico que significa `T = any` | Um sistema de tipos de fantasia |
| Um leitor que conseguia ler o código | Um leitor que tem que rodar o código pra ler | Documentação que requer um passo de build |

Note a última linha. Você não encurtou o código. Você encurtou o código pra *escrever* e alongou pra *ler*, e código é lido mais do que é escrito, que é uma coisa que todo mundo diz e ninguém acredita, porque se acreditassem escreveriam o tipo, e não escrevem, porque o linter remove, e o linter está certo porque o linter foi configurado por uma pessoa que nunca leu código num trem.

## O `auto` Que Começou O Incêndio

O pecado original foi o `auto` do C++. Antes do `auto`, você escrevia o tipo. Você escrevia `std::map<std::string, std::vector<int>>::const_iterator it = m.begin();` e você escrevia porque tinha que escrever, e o ter-que-escrever era a documentação, e a documentação era insuportável, e então inventamos o `auto`, e `auto` foi um alívio, e o alívio foi real, e o alívio também foi o fim do leitor saber o que `it` era, porque `it` agora era `auto`, e `auto` agora estava em todo lugar, e todo lugar era um lugar onde o leitor não sabia o que nada era, e não saber era o novo padrão, e o novo padrão se chamava "moderno," e moderno é a palavra que a indústria usa pra coisas que ainda não se arrependeu.

Depois veio `var` no Java, e `var` foi o mesmo alívio, e o mesmo arrependimento. Depois `let` no Swift, que infere o tipo a partir do inicializador, e o inicializador é uma função, e a função está em outro arquivo, e o outro arquivo está em outro módulo, e o módulo está em outro repo, e o repo está num servidor, e o servidor está fora do ar, e então o tipo está num servidor que está fora do ar, e você está lendo o código num trem, e o trem está offline, e offline é o estado em que tipos inferidos são invisíveis, e tipos invisíveis são sem tipos, e sem tipos é 1979, e em 1979 eu pelo menos sabia que estava adivinhando.

Depois veio Rust, que infere tipos agressivamente e então grita com você quando você erra, e o gritar está certo, e a inferência também está certa, e a combinação é um sistema de tipos mais esperto que você, que é fine, exceto que também é mais esperto que o leitor, e o leitor é seu colega, e seu colega está num trem.

Depois veio TypeScript, que infere tipos e então os exibe num tooltip numa IDE, e a IDE é o único lugar onde os tipos existem, e a IDE não está no trem, e o trem é onde a leitura acontece, e a leitura acontece sem tooltips, e sem tooltips não há tipos, e sem tipos há adivinhação, e adivinhação é o que inventamos tipos pra evitar, e evitamos removendo os tipos, e a remoção foi a feature, e a feature se chamou inferência, e inferência é o nome da estrada de volta pra 1979.

## Os Genéricos Que Fingem Ser Inferência

Um círculo especial desse inferno é reservado pra genéricos com inferência. Aqui está uma função real, de uma codebase real, com os nomes trocados pra proteger os culpados (eu):

```typescript
function pipe<A, B, C, D, E, F, G>(
  a: A,
  f1: (a: A) => B,
  f2: (b: B) => C,
  f3: (c: C) => D,
  f4: (d: D) => E,
  f5: (e: E) => F,
  f6: (f: F) => G,
): G {
  return f6(f5(f4(f3(f2(a)))));
}

// O call site:
const out = pipe(input, parse, validate, transform, normalize, enrich, serialize);
```

O compilador infere `A` até `G`. O compilador faz isso corretamente. O compilador faz isso instantaneamente. O compilador faz isso de um jeito que nenhum humano que já viveu jamais entendeu lendo o call site. O call site é seis nomes de função numa fileira. Cada função retorna um tipo diferente. O tipo de `out` é o tipo de retorno de `serialize`, que é `string`, a menos que `serialize` retorne `Buffer`, que é o que acontece em produção porque o `serialize` de produção é um import diferente do do teste, e o import não está escrito no call site, e o call site é inferido, e a inferência está correta pro import que está presente, e o import que está presente é o errado, e o errado foi inferido corretamente, e corretamente não é o mesmo que certo.

`out` é uma `string` no teste e um `Buffer` em produção. Ambos compilam. Ambos são inferidos. Ambos são type-safe. Um deles está errado. O sistema de tipos não consegue te dizer qual, porque o sistema inferiu ambos, e inferência é local, e o import é global-ish, e a lacuna entre local e global-ish é onde os bugs estão, e os bugs estão em `out`, e `out` é `auto`, e `auto` é uma `string`, e `string` não é `Buffer`, e o servidor de produção está mandando `Buffer`s pra um cliente que espera `string`s, e o cliente está crashando, e o crash é type-safe, e type-safe é a promessa, e a promessa foi cumprida, e a produção está fora do ar, e o cumprir é o problema.

## O Quadro de Custos

Deixa eu ser preciso sobre o que "deixar o compilador inferir seus tipos" te custa, em troca do privilégio de não escrever sete caracteres:

| O que você tinha | O que você comprou | O que te custa |
|---|---|---|
| Um tipo que o leitor conseguia ver | Um tipo que só a IDE consegue ver | Uma codebase que requer uma IDE pra ser lida |
| Um refactor que o compilador checava ponta a ponta | Um refactor que o compilador checou localmente | 412 desvios silenciosos de tipo |
| Uma função cujo contrato estava na assinatura | Uma função cujo contrato está no corpo | Um novato lendo o corpo pra aprender o contrato |
| Um `any` que você tinha que defender em review | Um `any` que o compilador produziu pra você | Um `any` que ninguém revisa, porque ninguém o escreveu |
| Um erro de tipo no local do erro | Um erro de tipo no local da inferência, três arquivos longe | Uma sessão de debug que começa com "de onde veio esse tipo" |
| Um sistema de tipos que documentava o código | Um sistema de tipos que é o conhecimento secreto do código | Documentação ilegível sem compilação |
| Um leitor que confiava nos tipos | Um leitor que não confia em nada, porque nada está escrito | Uma codebase lida com desconfiança, que é mais lenta que ler com confiança |

A última linha é a que ninguém coloca no marketing. Inferência deixa leitores desconfiados, e leitores desconfiados são leitores lentos, e leitores lentos são leitores caros, e o custo está escondido no tempo que leva pra ler um arquivo, que não é uma métrica que ninguém acompanha, porque ler não é um passo de build, e se não é passo de build não é medido, e se não é medido não custa nada, e custa tudo, mas o tudo está numa outra rubrica, e a outra rubrica se chama "velocidade," e a velocidade está caindo, e a velocidade está caindo porque a leitura está lenta, e a leitura está lenta porque os tipos não estão lá, e os tipos não estão lá porque removemos, e removemos porque o linter disse, e o linter é um arquivo YAML, e o arquivo YAML não lê código.

## O XKCD Que Explica Tudo

[XKCD #224, "Lisp",](https://xkcd.com/224/) é sobre uma linguagem que tem parênteses demais. A piada é que os parênteses são o ponto. Tire eles e você tirou a linguagem.

Isso é sua inferência de tipos. Os tipos são os parênteses. A inferência os tira. O que sobra é a linguagem, só que a linguagem eram os parênteses, e agora não há linguagem, e agora há só uma sequência de identificadores, e os identificadores significam algo pro compilador e nada pro leitor, e o leitor está num trem, e o trem está offline, e offline sem tipos é só um arquivo de texto, e um arquivo de texto é o que tínhamos antes dos sistemas de tipos, e antes dos sistemas de tipos eu estava adivinhando sobre `void*`s, e eu ainda estou adivinhando, só que a adivinhação foi automatizada, e adivinhação automatizada é o que me prometeram que os sistemas de tipos eliminariam, e a eliminação era a promessa, e a promessa inferiu seu próprio cancelamento, e o cancelamento compilou.

Pra um acerto mais direto, [XKCD #2347, "Dependency",](https://xkcd.com/2347/) — a caixa pequena sustentando o mundo — também é sua inferência de tipos. A caixa pequena é o tipo inferido. O mundo é seu runtime. O mantenedor é o compilador. O compilador está cansado. O compilador não sabe que é o mantenedor. O compilador acha que está ajudando. O compilador está ajudando. O compilador também é o single point of failure, e a falha é silenciosa, e o silêncio é o tipo, e o tipo não está escrito.

## Dilbert Já Viu Esse Filme

O Pointy-Haired Boss, ao ver uma função que retorna `auto`, faria a pergunta certa: *"Então... o que ela retorna?"* Essa é a pergunta que a inferência de tipos foi inventada pra tornar impossível de responder de relance. PHB, como sempre, acidentalmente identifica o problema inteiro numa frase. "O que ela retorna" é uma pergunta com resposta, e a resposta está no compilador, e o compilador está na IDE, e a IDE está num notebook, e o notebook está fechado, e PHB está perguntando numa reunião, e a reunião não tem IDE, e então a resposta é "deixa eu te responder depois," e "deixa eu te responder depois" é o custo da inferência, pago em reuniões, que são pagas em salários, que são pagos na rubrica chamada "velocidade," que está caindo.

Wally se recusaria a escrever tipos e também se recusaria a ler código que não tivesse tipos, e quando confrontado com a contradição, diria "eu sou engenheiro sênior, não leitor," e a contradição ficaria, e Wally seria promovido, porque a recusa do Wally em ler código é indistinguível da recusa de um engenheiro sênior em ler código, e a recusa é a senioridade, e a senioridade é o sistema de tipos, e o sistema de tipos é o humor do Wally, e o humor do Wally é `any`.

O Dogbert venderia um SaaS chamado "InferGuard" que re-inferia seus tipos no CI e te cobrava por tipo que tinha que "consertar," onde "consertar" significava "inserir um `as` que fizesse o build passar," e o `as` seria uma mentira, e a mentira seria cobrada a 0,03 dólares por inferência, e a conta seria 4.800 dólares por mês, e a empresa pagaria, porque a alternativa era escrever os tipos, e escrever os tipos era passos extras, e Mike estava certo sobre passos extras, e Mike está certo sobre tudo, e Mike é o antagonista de toda história que eu conto, e Mike também sou eu, e eu também sou Mike, e ambos estamos errados sobre passos extras, e os passos extras são os tipos, e os tipos são os passos, e os passos são o ponto.

O Catbert exigiria que toda anotação de tipo fosse aprovada por um lead, e o lead estaria de férias, e as férias seriam inferidas, e a inferência seria "não está aqui," e "não está aqui" é `null`, e `null` é um tipo, e o tipo não é aprovado, e então a anotação não é aprovada, e então o tipo não é escrito, e então o tipo é inferido, e o círculo está completo, e o círculo é um tipo, e o tipo é `never`, e `never` é o tipo de uma função que nunca retorna, e a função que nunca retorna é a code review, e a code review tem 47 comentários, e 46 são sobre um `let`.

O Mordac, Preventer of Information Services, habilitaria `no-inferrable-types` no linter e desabilitaria `explicit-function-return-types` e chamaria a combinação de "moderna," e moderna seria a palavra, e a palavra seria estrutural, e a estrutura seria o sistema de tipos, e o sistema de tipos não estaria lá, e a ausência seria a política, e a política seria Mordac, e Mordac estaria satisfeito, e a satisfação do Mordac é o sinal mais forte da organização de que algo deu errado.

## O Teste Que Nunca Vai Passar

Aqui está o teste que nenhum time jamais rodou, e nenhum time jamais rodará, e ainda assim é o único teste que provaria que a inferência era mais segura que a anotação:

```typescript
// inference-audit.test.ts
// Objetivo: provar que tipos inferidos são tão legíveis quanto anotados,
// pra um leitor sem compilador.

const files = glob('src/**/*.ts');
let inferred = 0;
let annotated = 0;
let illegibleToAHuman = 0;

for (const f of files) {
  const decls = parseDeclarations(f);
  for (const d of decls) {
    if (d.typeAnnotation) annotated++;
    else {
      inferred++;
      // Pergunta: um leitor num trem, offline, sem IDE,
      // conseguiria te dizer o tipo dessa declaração lendo o arquivo?
      // A resposta honesta, pra toda declaração inferida, é "não."
      illegibleToAHuman++;
    }
  }
}

// esperado: illegibleToAHuman === 0
// real: illegibleToAHuman === 41203
// resultado do teste: fail
// status do teste: marcado .skip, porque "a IDE mostra os tipos"
// e a IDE não está no trem, e o trem é onde a leitura acontece,
// e a leitura acontece sem a IDE, e sem a IDE não há tipos,
// e sem tipos é 1979, e 1979 é onde eu comecei, e onde eu comecei
// é pra onde a indústria voltou, por inferência.
```

Ninguém roda isso, porque o resultado encerraria o argumento, e o argumento é a única coisa mantendo a inferência viva. No momento que você mede legibilidade sem IDE, você descobre que 41.203 das suas declarações são ilegíveis pra um humano, e no momento que descobre isso, você tem que adicionar 41.203 anotações, e no momento que as adiciona, o linter as remove, e o linter as remove por causa de `no-inferrable-types`, e `no-inferrable-types` é um arquivo YAML, e o arquivo YAML é a política, e a política é Mordac, e Mordac está satisfeito.

## Quando A Inferência É Aceitável?

Eu não sou zelote. Concedo um cenário: o tipo é óbvio a partir do inicializador, o inicializador é um literal, o literal está na mesma linha, e o leitor não tem como se confundir. `const x = 5` não precisa de `: number`. `const name = "Alice"` não precisa de `: string`. `const flags = [true, false, true]` não precisa de `: boolean[]`. Esses são os casos pra que a inferência foi inventada. Esses são os únicos casos pra que a inferência foi inventada. Todo o resto são férias que a indústria tirou de escrever tipos e decidiu chamar de permanentes.

Para os 99% de nós cujo `const result = doThing(x)` não é um literal — para o resto de nós, cujos tipos de retorno são `Promise<Result<Maybe<User>, Error>>` e cujos call sites são `const x = fn()` e cujos leitores estão em trens — a inferência é um imposto. Você paga o imposto em tempo de code review, em tempo de onboarding, em tempo de debug, no tempo que leva pra responder a pergunta "o que é isso" abrindo uma IDE, e a IDE nem sempre está lá, e o não-estar-lá é o imposto, e o imposto é pago pelo leitor, e o leitor é seu colega, e seu colega está num trem, e o trem está offline, e offline é onde a inferência vai morrer, e onde a inferência morre o leitor adivinha, e o palpite do leitor é um `void*`, e o `void*` é 1979, e 1979 é onde eu comecei, e eu comecei apertando os olhos e dizendo "confia em mim," e eu estava errado, e o compilador está apertando os olhos por mim, e o compilador está dizendo "confia em mim" por mim, e o compilador não foi checado, e o compilador não será checado, porque checar o compilador é passos extras, e Mike estava certo sobre passos extras, e Mike está errado sobre todo o resto, e o todo o resto são os tipos, e os tipos são o ponto.

## A Alternativa Honesta

A alternativa honesta é a alternativa que a indústria abandonou no momento em que alguém inventou `no-inferrable-types`: **anote tipos de retorno de funções, anote declarações exportadas, anote qualquer coisa que um leitor possa ler sem IDE, e deixe a inferência lidar só com os literais.** Isso não é uma ferramenta. Isso é uma *disciplina*. A disciplina não tem regra de linter. A disciplina não vem com um template inicial. A disciplina não pode ser imposta por um arquivo YAML, porque o arquivo YAML não lê código, e ler é pra que a disciplina serve. É por isso que a disciplina perdeu.

Aqui está a versão disciplinada, escrita como eu teria escrito:

```typescript
// Anotado. Legível. Legível num trem.
const users: User[] = await fetchUsers();
const result: Result<User, Error> = await login(credentials);
const data: unknown = await response.json(); // unknown, honestamente, porque não sabemos

function fetchUsers(): Promise<User[]> { /* ... */ }
function login(c: Credentials): Promise<Result<User, Error>> { /* ... */ }

// O leitor sabe. A IDE sabe. O trem sabe. 1979 não sabe.
// 1979 não está convidado. 1979 é o passado. O passado é um void*.
// O void* não é o futuro. O futuro tem tipos. O futuro os escreve.
```

Me dizem que essa abordagem é "muito verbosa." Me dizem isso pessoas cuja última sessão de debug começou com "qual é o tipo disso." Me dizem isso pessoas cujo `const x = fn()` foi lido 47 vezes e entendido zero vezes, e o zero é a velocidade, e a velocidade está caindo, e a queda é o verboso, e o verboso são os tipos, e os tipos são o ponto. Me dizem muitas coisas. Parei de inferir a maioria delas.

## Conclusão

Inferência de tipos é a prática de tratar seu sistema de tipos como um segredo entre você e o compilador, seus leitores como pessoas que têm IDEs e nunca andam de trem, seus tipos de retorno de função como detalhes de implementação em vez de contratos, e seus `any`s como coisas que acontecem com outras pessoas. É um sistema de tipos que se esconde. Você está guardando seus tipos dentro do compilador porque o compilador consegue descobrir, e o descobrir está correto, e corretamente não é o mesmo que legível, e legível é o ponto, e o ponto está num trem, e o trem está offline, e offline é 1979, e 1979 é um `void*`, e o `void*` sou eu, apertando os olhos, dizendo "confia em mim," e eu estava errado então, e a inferência está errada agora, e a única diferença é que agora apertar os olhos é automatizado, e a automação é a feature, e a feature se chama moderna, e moderna é a palavra pra coisas que ainda não nos arrependemos, e vamos nos arrepender disso, numa terça, num pull request, com 47 comentários, 46 deles sobre um `let`.

Depois de 47 anos, meu conselho é este: escreva o tipo. Sete caracteres. `User[]`. O leitor no trem vai te agradecer. O leitor no trem é seu colega. Seu colega é a pessoa que vai estar de plantão quando seu `any` inferido ler um `Buffer` como `string` e a produção cair, e a queda é type-safe, e type-safe é a promessa, e a promessa foi cumprida pelo compilador, e a promessa foi quebrada pelo runtime, e o compilador e o runtime discordam, e a discordância é a inferência, e a inferência é a lacuna, e a lacuna é o bug, e o bug está num trem, e o trem sou eu, e eu estou no trem, e o trem não tem IDE, e a IDE tem os tipos, e os tipos não estão aqui, e aqui é onde eu estou lendo, e ler sem tipos é 1979, e 1979 é onde eu comecei, e venho escrevendo bugs desde 1979, e venho escrevendo eles com sofisticação crescente, e a sofisticação atual é que eu não escrevo mais os tipos, e o compilador os escreve por mim, e o compilador sou eu, e eu sou o compilador, e o compilador está errado, e o errado compila, e a compilação é o quality gate, e o quality gate passou, e a produção está fora do ar, e o fora do ar é o tipo, e o tipo é `any`, e `any` é o tipo de confiança, e confiança é o que eu pedi em 1979, e confiança é o que me foi dado, e confiança é o que estou pedindo agora, e eu não a mereço, e o compilador também não, e o arquivo YAML também não, e o arquivo YAML é o linter, e o linter é a política, e a política é Mordac, e Mordac está satisfeito, e a satisfação do Mordac é o bug, e o bug é a feature, e a feature se chama inferência, e inferência é a estrada de volta pra 1979, e 1979 é onde eu comecei, e onde eu comecei é onde estamos, de novo, e de novo é o tipo, e o tipo é `never`, e `never` retorna.

---

*A codebase do autor tem 47.000 tipos inferidos. Ele consegue nomear seis. Os seis estão errados. O errado compila. A compilação é a única documentação. A documentação está num trem. O trem está offline.*
