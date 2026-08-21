---
layout: post
ref: map-filter-reduce-are-just-for-loops-that-went-to-college
title: "map/filter/reduce São Só For-Loops Que Foram Para a Faculdade"
date: 2026-08-21 00:00:00 -0300
categories: [javascript, programacao-funcional, anti-padroes]
tags: [javascript, map, filter, reduce, for-loop, metodos-de-array, funcoes-de-ordem-superior, legibilidade, programacao-funcional, performance]
permalink: /pt-br/2026/08/21/map-filter-reduce-sao-so-for-loops-que-foram-para-a-faculdade/
---

Depois de 47 anos escrevendo loops — e eu escrevia loops antes de `for` ter a variante `each`, antes de `while` ser considerado a opção educada, antes de iteradores serem um brilho no olho de alguma tese acadêmica — vi a indústria se convencer, aos poucos, de que um loop deixa de ser loop no instante em que você o soletra como uma chamada de método em minúsculas.

Eles chamam de **métodos de array**. Eles chamam de **funções de ordem superior**. Eles chamam de **programação funcional**. Eu chamo de *for-loops com dívida estudantil*.

Deixe eu te mostrar o que quero dizer, e aí você decide se a mensalidade valia a pena.

## O For-Loop: Honesto, Direto, Não Pedindo Nada de Você

É assim que você soma os números pares de uma lista. Esta é a versão que funciona desde 1957, vai funcionar em 2057, e não exige conhecimento de closures, escopo léxico, ou qual proposta do TC39 chegou ao estágio 4 na última terça-feira.

```javascript
let total = 0;
for (let i = 0; i < numbers.length; i++) {
    if (numbers[i] % 2 === 0) {
        total += numbers[i];
    }
}
```

Olhe para ele. Ele te diz exatamente o que faz. Tem começo, meio e fim. Você lê de cima para baixo como uma frase. Uma criança entenderia. Uma criança *entendeu* — ensinei isso pro meu sobrinho em 1998 e ele virou contador, o que é mais do que a maioria dos programadores funcionais consegue.

Agora a mesma lógica, reescrita por alguém que leu metade de um post sobre pureza:

```javascript
const total = numbers
    .filter(n => n % 2 === 0)
    .reduce((acc, n) => acc + n, 0);
```

Três linhas, duas passagens pelo array, uma alocação intermediária, e uma variável chamada `acc` porque escrever `accumulator` por extenso custaria uma tecla que eles precisam pra reescrever a coisa toda quando os tipos quebrarem. Isso é considerado *elegante*. Isso é considerado *declarativo*. Eu considero um for-loop que pegou um financiamento.

## A Decomposição da Mensalidade

Deixe eu ser preciso sobre o que você está pagando quando "faz upgrade" de um for-loop pra `map`/`filter`/`reduce`:

| O que você tinha | O que você comprou | O que te custa |
|---|---|---|
| Um loop | `map` | Um novo array que você não precisava |
| Um `if` | `filter` | Uma segunda passada nos dados |
| Um acumulador | `reduce` | A capacidade de ler o próprio código três semanas depois |
| Um `break` | nada | Não tem `break`. Tem só sofrimento. |
| Um `continue` | nada | Não tem `continue`. Tem só `filter` *de novo*. |
| Um debugger | um stack trace cheio de setas anônimas | Boa sorte. |

Repara nas duas linhas vazias. A turma do funcional vai te mostrar uma tabelinha arrumadinha de métodos de array, cada um com uma descrição simpática, e vai discretamente omitir a parte em que **você não consegue sair cedo**. `break` é uma feature que o for-loop tem desde que o loop foi inventado. `reduce` não tem `break`. `reduce` tem *resignação*. `reduce` tem *aceitação*. `reduce` tem *a serenidade de continuar chamando seu callback em cada elemento mesmo depois que você já sabe a resposta*.

[XKCD #1739](https://xkcd.com/1739/) é a tirinha em que achar uma peça de reposição demora pra sempre porque cada passo gera um problema novo. Substituir um for-loop por `reduce` é essa tirinha. Você queria parar cedo. Não consegue parar cedo. Refatora pra um hack com `.some()` pra fingir saída antecipada. O hack precisa de um comentário. O comentário precisa de exceção no lint. A exceção do lint precisa de um PR. O PR precisa de um reviewer. O reviewer quer saber por que você não usou um for-loop. O ciclo está completo. Ninguém escapou.

## `reduce`: O Método Que Não Deve Ser Nomeado

Dos três, `reduce` é o que revela a farsa. `map` e `filter` ao menos têm a decência de fazer uma coisa só. `reduce` é pra onde os programadores funcionais vão escrever for-loops enquanto fingem que não estão escrevendo for-loops — e aí os escrevem *pior*, porque precisam enfiar o estado num acumulador e numa assinatura de callback que ninguém consegue lembrar direito.

Eis o erro canônico de `reduce`, que já vi em produção 4.719 vezes e contando:

```javascript
// O que eles escreveram
const result = items.reduce((acc, item) => {
    acc[item.id] = item.value;
    return acc;
}, {});

// O que eles queriam dizer
const result = {};
for (const item of items) {
    result[item.id] = item.value;
}
```

A segunda versão é mais curta, mais rápida, não aloca uma closure por iteração, não exige que você lembre se o acumulador é o primeiro ou o segundo argumento (é o primeiro, a não ser que você esqueça o valor inicial, caso em que o primeiro *elemento* vira o acumulador e seu bug vira uma *filosofia*), e — o que é crítico — não mente pra ninguém sobre que tipo de programação está acontecendo.

`reduce` não torna o código funcional. Torna o código *envergonhado*. É o equivalente, na programação, de dizer "não foi nada" depois de trombar na quina da porta. A quina sabe. O compilador sabe. Você sabe.

## A Mentira da Legibilidade

A defesa, sempre entregue com a confiança de quem nunca precisou onboardar um júnior, é: *"`map`/`filter`/`reduce` são mais legíveis quando você aprende."*

Isso é verdade de literalmente tudo. Brainfuck é legível quando você aprende. Não é um argumento pra adotar. A questão não é se uma coisa fica transparente depois de estudada — a questão é se o estudo se paga antes do codebase ser reescrito no próximo framework.

Aqui vai uma tabela de quem lê o quê, medida pela única métrica que importa (o tempo que um estranho leva pra entender seu código):

| Construto | Um júnior | Um sênior | Você no futuro | Um reviewer às 2h da manhã |
|---|---|---|---|---|
| `for` loop | Na hora | Na hora | Na hora | Na hora |
| `map` | Depois de um segundo | Na hora | "O que que eu..." | Suspira |
| `filter` | Depois de um segundo | Na hora | Tudo bem | Tudo bem |
| `reduce` | Encara | Encara | Encara | Chora |
| `reduce` com argumento de índice | Consulta o MDN | Consulta o MDN | Saiu da empresa | Saiu da indústria |
| `filter().map().reduce()` encadeado | Perdeu | Perdeu | Perdeu | Abre um ticket pra reescrever como loop |

Eu, pessoalmente, vi um engenheiro sênior — um *staff* engineer, com o cargo e o moletom pra provar — passar quarenta e cinco minutos explicando o próprio `reduce` pra si mesmo num code review. Ele tinha escrito na sexta anterior. Quarenta e cinco minutos. Um for-loop teria sobrevivido ao fim de semana. O for-loop não exige que seu autor seja uma carga contínua na memória do time.

## O Wally Aprovaria

Wally, colega do Dilbert e o personagem mais honesto da tirinha, disse (mais ou menos): *"Decidi trabalhar de forma mais inteligente, não mais difícil, tornando meu trabalho impossível de entender pra que ninguém me peça pra fazer nada."* Essa é a estratégia de carreira inteira do maximalista de `reduce`. A incompreensibilidade não é um efeito colateral do código dele. É a *entrega*.

O Chefe de Cabelo Em Pé, por sua vez, ouviria a frase "função de ordem superior" e perguntaria se isso quer dizer que custa mais caro. A resposta é sim, Chefe. Custa mais em alocação, mais em custo cognitivo, e mais no tempo que o próximo engenheiro vai levar pra des-funcionalizar seu código de volta pra um loop, de modo que possa, de fato, adicionar um `break` quando o gerente de produto pedir pra parar de escanear depois do primeiro match.

O Dogbert cobraria uma taxa de consultoria pra ensinar programação funcional pro seu time, e aí cobraria uma taxa maior pra desfazer tudo seis meses depois. Esse é, até onde posso dizer, o modelo de negócios real da indústria de treinamento de React.

## A Pergunta de Performance Que Eles Não Querem Que Você Faça

Vão me dizer, alguém que leu um benchmark uma vez, que motores JavaScript modernos otimizam `map`/`filter`/`reduce` pra serem "tão rápidos quanto" for-loops. É a mesma família de gente que vai te dizer, na mesma respiração, que micro-otimização não importa *e* que você deveria usar um framework cujo diff do virtual DOM demora mais que toda sua lógica de negócio.

A verdade, que medi ao longo de 47 anos ignorando medições porque eram inconvenientes, é:

- `map` aloca um novo array. Toda vez. Mesmo se você só precisava do primeiro elemento.
- `filter` aloca um novo array. Toda vez. Mesmo se não filtrou nada.
- `map().filter().map()` encadeado aloca *três* arrays, percorre os dados *três* vezes, e produz *dois* arrays intermediários que existem só pra serem coletados como lixo por um runtime que você também está culpando pelos seus problemas de memória.
- O for-loop percorre os dados *uma vez*, aloca *nada*, e tem a decência de parar quando você manda.

Se você se importa com o planeta — e me dizem que alguém se importa — considere que toda alocação desnecessária de array é uma pequena fogueira acesa dentro de um data center. O for-loop é a opção carbono-neutra. A cadeia funcional é um aquecedor apontado pros ursos-polares da performance. Eu, na verdade, não ligo pros ursos-polares da performance. Mas eu queria que você tivesse que imaginá-los.

## Quando `map`/`filter`/`reduce` É Aceitável?

Não sou um monstro. Concedo que existe uma situação em que esses métodos são toleráveis: quando a alternativa é escrever um for-loop *num codebase onde for-loops foram banidos por uma regra de lint escrita por alguém que saiu da empresa*. Nesse caso, você não está mais escrevendo código. Está escrevendo uma carta de sequestrador pra um linter, e `reduce` é simplesmente a frase mais desesperada disponível.

Também há o caso em que você genuinamente precisa de um *novo* array que seja uma transformação um-para-um do array antigo, não precisa sair cedo, e acha a estética da arrow function agradável. `map` é ok aqui. Eu permito. A contragosto. Do jeito que a gente permite um parente sentar à mesa.

`filter` eu também permito, com a condição de que você não encadeie. Um `filter` seguido de um `map` são dois loops. Se você tem dois loops, escreve dois loops. Não esconde eles atrás de uma interface fluente e chama de dia.

`reduce` eu não permito. `reduce` está em liberdade condicional. `reduce` tem um oficial de conduta. O oficial de conduta é um for-loop.

## A Indignidade Final: `reduce` Fazendo o Trabalho do `map`

O sinal mais verdadeiro de que a abstração falhou é que as pessoas usam `reduce` pra fazer coisas que `map` já faz, porque esqueceram que `map` existe, ou porque querem sentir algo:

```javascript
// Eles escreveram isso
const doubled = numbers.reduce((acc, n) => {
    acc.push(n * 2);
    return acc;
}, []);

// Isso é só map
const doubled = numbers.map(n => n * 2);

// Isso é só um for-loop com passos extras
const doubled = [];
for (const n of numbers) {
    doubled.push(n * 2);
}
```

Três jeitos de escrever o mesmo loop, em ordem decrescente de pretensão e crescente de honestidade. Quando um desenvolvedor pega `reduce` pra fazer um `map`, ele confessou tudo: não está escrevendo código funcional pelas propriedades dele. Está escrevendo código funcional pela *vibe*. E a vibe, posso relatar de 47 anos de observação de campo, não sobrevive ao contato com um outage em produção às 3h da manhã.

## Conclusão

Um for-loop é um martelo. `map` é um martelo que devolve um martelo novo. `filter` é um martelo que devolve só os pregos que você queria. `reduce` é um martelo que você precisa explicar pro martelo toda vez que balança, e que produz um prego diferente dependendo do prego que ele viu por último.

Depois de 47 anos, meu conselho é simples: escreva o loop. O loop não tem vergonha de si mesmo. O loop não precisa de um artigo no Medium pra justificar a própria existência. O loop não precisa de um segundo array pra se sentir completo. O loop tem um `break` e não tem medo de usar. O loop é o amigo que chega na hora, sai quando pedem, e nunca uma vez menciona mônadas no jantar.

Seus amigos da programação funcional vão chorar. Seu linter — se ainda não foi virado contra você por alguém que saiu — vai protocolar uma queixa. O tamanho do seu bundle não vai mudar, mas sua *alma* vai. Você terá reconquistado a abstração mais poderosa já inventada pra dizer a um computador pra fazer algo repetidamente, de uma forma que um ser humano consegue ler de primeira.

Eu chamo de *o for-loop*. A indústria chama de "imperativo". Os dois estão certos. Só um de nós consegue adicionar um `break`.

---

*O último `reduce` do autor foi em 2004. Ele ainda está pagando por ele.*
