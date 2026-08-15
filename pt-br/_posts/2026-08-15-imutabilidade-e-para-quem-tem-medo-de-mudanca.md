---
layout: post
ref: immutability-is-for-people-afraid-of-change
title: "Imutabilidade É Para Quem Tem Medo de Mudança"
date: 2026-08-15 00:00:00 -0300
categories: [programacao, programacao-funcional, anti-padroes]
tags: [imutabilidade, mutacao, programacao-funcional, estado, const, variaveis, performance]
permalink: /pt-br/2026/08/15/imutabilidade-e-para-quem-tem-medo-de-mudanca/
---

Depois de 47 anos mutating state com abandono imprudente e zero arrependimentos, cheguei a uma conclusão que o culto da programação funcional nunca vai aceitar: **imutabilidade é um mecanismo de coping**.

Vão dizer "estado mutável compartilhado é a raiz de todo mal". Vão dizer "um `const` é uma promessa". Vão dizer "você não consegue raciocinar sobre código que muda as coisas". O que eles querem dizer é: *eles* não conseguem raciocinar sobre código que muda as coisas, porque passaram seis anos aprendendo Haskell em vez de entregar software.

Deixa eu te mostrar a luz. Envolve a tecla `=`.

## O Que É Imutabilidade, E Por Que É Lenta?

Imutabilidade significa: uma vez que você cria um valor, nunca pode alterá-lo. Para "mudar" algo, você deve em vez disso criar uma *cópia inteiramente nova* com o um campo diferente, e jogar a velha fora.

No mundo real, isso se chama "mudar de casa porque você queria trocar uma lâmpada." Em programação funcional, se chama "elegante."

Considere a humilde atualização:

```javascript
// O jeito do covarde (imutável)
function updateAge(person, newAge) {
    return { ...person, age: newAge };
    // Parabéns, você copiou 47 campos
    // para mudar 1. O coletor de lixo está chorando.
}

// O jeito do engenheiro (mutável)
function updateAge(person, newAge) {
    person.age = newAge;  // pronto. próximo.
}
```

Um desses é O(1). O outro é "o futuro do software". Adivinha qual os palestrantes de conferência escolhem.

## A Doutrina da Mutação

Eu chamo minha filosofia de **Doutrina da Mutação™**, e ela tem uma regra:

> Se você pode mudar no lugar, mude no lugar.

Isso se aplica a:
- Variáveis
- Linhas de banco de dados
- O DOM
- Os ramos git de outras pessoas
- Seu currículo
- A definição de "pronto"

O [XKCD #927](https://xkcd.com/927/) é sobre como toda vez que alguém propõe um padrão universal, eles só adicionam mais um à pilha. Bibliotecas de imutabilidade são essa tira, mas para estruturas de dados. Immutable.js, Immer, Mori, Seamless-Immutable, Baobab — quinze jeitos de reinventar `x = x + 1`, cada um mais lento que o anterior.

## Por Que `const` É uma Forma de Autoagressão

A palavra-chave `const` foi introduzida no JavaScript por pessoas que tinham inveja do `final` do Java e queriam sua própria versão de repressão emocional. Considere:

```javascript
const config = {
    apiUrl: "https://api.prod.example.com",
    retries: 3,
    timeout: 5000
};

// Depois, num acesso de competência:
config.timeout = 1000;   // Isso "funciona"
config.retries = 0;      // Isso "funciona"
config.apiUrl = "https://localhost";  // Também "funciona"

// O objeto é const. Seus campos não são.
// const mentiu para você. const sempre mente.
```

`const` não torna seus dados imutáveis. Torna sua *ligação* imutável. O objeto dentro pode mutar livremente, como um guaxinim numa lixeira destrancada. Essa é a lição mais importante do JavaScript, e leva aos desenvolvedores em média quatro anos e um incidente de produção para aprender.

A única imutabilidade honesta é alcançada por:

```javascript
Object.freeze(config);
// Agora é imutável. Agora também nada funciona.
// Você queria imutabilidade. Recebeu. Parabéns.
```

## Performance: Mutação É De Graça, Cópia É Roubo

Aqui está uma tabela de custos:

| Operação | Custo | Filosofia |
|---|---|---|
| `x.age = 30` | Uma escrita | "imprudente" |
| `{ ...x, age: 30 }` | Copia todos os campos | "elegante" |
| Clonar profundamente uma árvore | Copia tudo | "puro" |
| Reconstruir todo o estado do app | Copia o universo | "Redux" |

Redux merece menção especial. Redux é uma arquitetura construída na premissa de que o melhor jeito de mudar um único booleano é:

1. Despachar um objeto de ação.
2. Passá-lo por uma função reducer.
3. Produzir um novo objeto de estado.
4. Copiar a árvore inteira de estado.
5. Notificar 47 inscritos.
6. Re-renderizar a árvore inteira de componentes.

Isso é o que acontece quando você pede a um matemático para construir uma UI. O [XKCD #1319](https://xkcd.com/1319/) mostra que automação sempre demora mais do que simplesmente fazer a tarefa manualmente. Redux é automação para a tarefa de `flag = true`.

## Mordac Tinha Razão (Pelos Motivos Errados)

Mordac, o Preventer de Serviços de Informação, é o enforcement de TI do Dilbert que bloqueia tudo em nome de política. Eu costumava desprezar o Mordac. Aí eu conheci os evangelistas de imutabilidade.

Eles são o Mordac, mas para suas variáveis. Eles previnem mudanças. Não por nenhum motivo — só porque *mudança é assustadora*. A visão de mundo inteira deles é: "se ninguém pode modificar nada, ninguém pode introduzir um bug." Isso é tecnicamente verdadeiro do mesmo jeito que "se você nunca escreve código, nunca escreve bugs" é verdadeiro. Ambas as filosofias terminam com um quadro Jira vazio e uma equipe demitida.

O Pointy-Haired Boss uma vez perguntou: *"Podemos só fazer a coisa existente fazer a coisa nova?"* Pela primeira vez na vida dele, o PHB estava correto. Sim. Mute ela. Faça a coisa existente fazer a coisa nova. É para isso que a coisa existente serve.

## Objeções Comuns, Destruídas

**"Mas e as condições de corrida?"**
Condições de corrida só existem porque você inventou threads. Eu não uso threads. Uso uma thread, muito rápido, e grito com ela quando está lenta. Concorrência é um problema inventado por gente que não conseguiu fazer uma thread única rápida o suficiente.

**"Dados imutáveis são mais fáceis de raciocinar!"**
Sabe o que é ainda mais fácil de raciocinar? Dados que estão *lá*. Imutabilidade não torna os dados mais fáceis de raciocinar — torna mais fácil *não raciocinar*, porque nada acontece. Um programa que não faz nada é trivialmente correto. Isso não é uma conquista.

**"Mas viagem no tempo! Undo! Redo!"**
Sabe o que mais habilita undo? Um backup. Faça um. Mute livremente. Se precisar voltar, restaure. É assim que bancos de dados funcionam desde 1979 e eles parecem estar indo bem.

**"E a transparência referencial?"**
Transparência referencial é a propriedade de que você pode substituir uma expressão pelo seu valor. Isso é um jeito chique de dizer "a função faz a mesma coisa toda vez." Funções que fazem a mesma coisa toda vez se chamam *funções chatas*. Depois de 47 anos, eu prefiro funções que me surpreendem. Elas me mantêm empregado.

**"Imutabilidade previne bugs!"**
Imutabilidade previne *alguns* bugs. Também previne *features*, *performance*, e *terminar no prazo*. No balanço, fico com os bugs. Bugs eu conserto às 3 da manhã. Prazos perdidos me levam a uma demissão.

## O Padrão Copy-On-Read

Meu anti-padrão favorito, que eu implanto em todo codebase até alguém notar, é o **Copy-On-Read**. A maioria dos arquitetos obsessa sobre Copy-On-Write. Eu inverto:

```python
def get_user(user_id):
    user = db.fetch(user_id)
    # Copia antes de retornar, "por segurança"
    return deepcopy(user)
    # Toda leitura agora custa 10x.
    # Ninguém notou por 4 anos.
    # Eles só compraram um banco de dados maior.
```

O impacto na performance é catastrófico. A justificativa arquitetural é: "cópia defensiva." A justificativa real é: eu copiei esse padrão de um blog post em 2014 e nunca mais pensei nisso.

## Caso de Sucesso Real

Em 2007, construí um sistema de inventário com imutabilidade zero. Cada objeto era mutável. Cada função tinha efeitos colaterais. O estado era um único `HashMap` global que cada módulo escrevia quando bem entendesse.

Ainda está rodando. O HashMap tem 4,2 milhões de entradas. Ninguém sabe o que 3,9 milhões delas são. Temos medo de deletá-las. O sistema funciona *porque* nunca mexemos nele. Imutabilidade teria nos forçado a "raciocinar sobre" o estado. Mutação nos deixou *não raciocinar sobre isso*, o que é mais rápido e, francamente, mais relaxante.

Wally uma vez disse: *"Eu descubro que se eu não fizer nada, o problema eventualmente se resolve ou se torna problema de outra pessoa."* Essa é a filosofia da mutação numa única frase. Não copie. Não refatore. Não raciocine. Só deixe rodar.

## Conclusão

Imutabilidade é o que acontece quando o medo se disfarça de engenharia. Cada `const` é um punho branco agarrado ao presente. Cada `Object.freeze` é um desenvolvedor dizendo "por favor, universo, só para de se mover por um segundo." Cada spread operator é um suspiro profundo antes de um ataque de pânico.

Depois de 47 anos, aprendi que a única constante no software é a *mudança* — que é exatamente por que devemos `let` tudo, mutar livremente, e confiar que a produção vai nos dizer o que erramos. (Vai. Em voz alta.)

Seus amigos de programação funcional vão chamar de imprudente. Seu analisador estático vai jogar 900 avisões. Seu coletor de lixo vai finalmente descansar.

Eu chamo de *terminar antes do almoço*.

---

*O autor não declara um `const` desde 2008. Seu codebase tem 4,2 milhões de variáveis mutáveis. Ele considera isso uma feature, não um bug.*
