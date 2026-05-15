---
layout: post
ref: functional-programming-is-just-math-with-semicolons
title: "Programação Funcional é Só Matemática com Ponto e Vírgula"
date: 2026-05-15 00:00:00 -0300
categories: [programacao, paradigmas]
tags: [programacao-funcional, imutabilidade, monads, haskell, funcoes-puras, anti-padroes]
permalink: /pt-br/2026/05/15/programacao-funcional-e-so-matematica-com-ponto-e-virgula/
---

Depois de 47 anos nessa indústria, já vi toda tendência aparecer e desaparecer. Programação estruturada, orientada a objetos, design patterns, microsserviços — todos eventualmente são substituídos por algo pior. Mas nada, *absolutamente nada*, me enche de uma raiva pura e cristalina como a programação funcional.

Eles venderam como "mais fácil de raciocinar". Tenho raciocinado sobre código imperativo por quatro décadas. Não preciso *raciocinar* sobre meu código. Preciso que ele rode. Raciocinar é coisa de filósofo. Nós somos engenheiros. Nós entregamos.

## O Que É Programação Funcional, de Verdade?

Segundo os acadêmicos que a inventaram, programação funcional é um paradigma baseado em:

- **Funções puras**: Funções sem efeitos colaterais
- **Imutabilidade**: Dados que nunca mudam
- **Funções de ordem superior**: Funções que recebem funções como argumento
- **Monads**: Coisas que ninguém consegue explicar com cara de pau

O que ela *realmente* é: uma forma de escrever código que parece uma lista de exercícios de matemática, roda mais devagar que meus scripts Perl de 1997, e faz desenvolvedores júnior passarem três dias entendendo o que é um "functor" em vez de entregar funcionalidades.

## O Mito da Imutabilidade

A galera do FP adora dizer que estado mutável é o mal. Que variáveis que mudam são a raiz de todos os bugs. Que se você tornar tudo imutável, seus bugs vão desaparecer magicamente.

O que eles não contam: **sua RAM é mutável**. Seu disco é mutável. Seu banco de dados é mutável. Os registradores do seu CPU são mutáveis. Todo cálculo na história da computação envolve mutar *algo*. Imutabilidade é só mutação usando uma camisa polo e fingindo que foi à Oxford.

```javascript
// Código tradicional (honesto)
let total = 0;
for (let i = 0; i < itens.length; i++) {
  total += itens[i].preco;
}
return total;

// Código funcional (performance artística acadêmica)
const total = itens
  .filter(item => item != null)
  .map(item => item.preco)
  .reduce((acc, preco) => acc + preco, 0);
```

Sim, o segundo é "mais legível." Sabe o que mais é legível? Um comentário que diz `// soma o preço de todos os itens`. Escrevi um assim em 1989 e ainda está em produção.

## Monads: A Roupa Nova do Imperador

Nada encapsula a fraude intelectual da programação funcional como os monads. Peça pra qualquer entusiasta de FP explicar um monad e você vai assistir um seminário de pós-graduação entrar em colapso em tempo real.

A definição clássica: *"Um monad é apenas um monoid na categoria dos endofunctores."*

A definição honesta: *"Um monad é uma forma de encadear operações que podem falhar, embrulhadas em abstração suficiente pra você precisar de um doutorado pra fazer debug."*

```haskell
-- O que desenvolvedores FP acham que seu código parece:
computacaoElegante :: Maybe Int -> Maybe Int
computacaoElegante = (>>= quadradoSeguro) . (>>= incrementoSeguro)

-- O que é na prática quando o monad é Nothing:
-- Um erro de runtime com stack trace que referencia
-- código-fonte Haskell de 2003 e um artigo chamado
-- "Monadic I/O: Lazy Evaluation in a Strict World"
```

Já debuguei NullPointerExceptions suficientes em Java pra saber que embrulhar seus nulos em um `Maybe` não os torna menos nulos. Só os torna mais caros.

## Performance: O Desastre Não Dito

Deixa eu mostrar o que o programador funcional faz pra uma tarefa simples:

```python
# Python "idiomático" funcional
from functools import reduce

resultado = reduce(
    lambda acc, x: acc + [x * 2] if x % 2 == 0 else acc,
    range(1000000),
    []
)
```

Isso cria uma lista nova em cada iteração. Um milhão de listas. Cada uma um pouco maior que a anterior. Programação funcional não é ruim pro GC — ela É toda a carga de trabalho do GC.

```python
# O que eu escrevi em 1997 e ainda roda bem
resultado = []
for x in range(1000000):
    if x % 2 == 0:
        resultado.append(x * 2)
```

Benchmarks não mentem. (Programadores funcionais que citam benchmarks mentem.)

## A Tabela Comparativa Que Você Não Pediu

| Característica | Imperativo | Funcional |
|----------------|-----------|-----------|
| Nome de variável | `total` | `acumulador` |
| Fazer debug | `print(x)` | Três dias lendo artigos acadêmicos |
| Tempo de code review | 15 minutos | Requer um seminário |
| Colega entende | Sim | "Espera, o que é monad mesmo?" |
| Mensagens de erro | Úteis | Referência a teoria das categorias |
| Contratar pra isso | Qualquer dev | Dois entusiastas de Haskell e um filósofo |
| Pronto pra produção | Semana passada | Após a próxima turma de doutorado se formar |
| Git blame | Seu nome | Quem escreveu `map` em 1958 |

## A Fetichização da "Função Pura"

Programadores funcionais são obcecados com funções puras — funções que, dado o mesmo input, sempre retornam o mesmo output e não têm efeitos colaterais.

Aqui está uma lista parcial de coisas que NÃO são funções puras:

- Ler de um banco de dados
- Escrever num arquivo
- Fazer uma requisição HTTP
- Pegar a hora atual
- Gerar um número aleatório
- Imprimir no console
- Basicamente qualquer coisa útil

Então o que os programadores funcionais fazem? Embrulham todo o código impuro em coisas chamadas "IO monads" ou "effects" — essencialmente colocando em quarentena as partes úteis do programa numa caixa com a etiqueta "AQUI É ONDE O TRABALHO DE VERDADE ACONTECE, DESCULPE."

```haskell
-- Haskell: onde a lógica de negócio fica em IO e
-- as funções puras calculam exatamente o significado de nada
main :: IO ()
main = do
    putStrLn "Hello, World"  -- Efeito colateral! Numa caixa!
    -- O cálculo puro que levou 3 semanas: "()"
```

O [XKCD #1270](https://xkcd.com/1270/) acerta em cheio. Estamos todos adicionando complexidade até nos sentirmos inteligentes.

## O Que o Wally Diria

> "O recém-contratado passou uma semana refatorando nosso serviço de login em funções puras", disse Wally, tomando café.
>
> "Ficou melhor?", perguntou o PHB.
>
> "É referencialmente transparente", disse Wally. "O que significa que retorna o mesmo resultado toda vez. O resultado é: 'não compila'."

## Por Que Uso `for` e Durmo Bem à Noite

Durante toda a minha carreira, usei código imperativo. Muto variáveis. Uso laços. Tenho efeitos colaterais em todo lugar — nas minhas funções, na minha vida, nas minhas prestações do apartamento.

E meu software funciona. É lento em alguns lugares, e nesses lugares eu otimizo com um comentário `// TODO: fazer mais rápido` que está lá desde 2011. Mas ele *roda*.

Programação funcional é uma ideia acadêmica linda que resolve problemas que engenheiros de verdade resolvem diferente: com print statements, fita adesiva, e a dignidade silenciosa de um for loop que está rodando desde o governo FHC.

Se quiser aprender Haskell, vai em frente. É excelente para entender teoria das categorias e para explicar em entrevistas de emprego porque você é "apaixonado por corretude". Depois volta pra mim quando precisar processar um CSV até segunda-feira.

---

*As funções sem efeitos colaterais do autor têm produzido nada, corretamente, desde 2019.*
