---
layout: post
ref: type-systems-are-training-wheels
title: "Sistemas de Tipos São Rodinhas Para Devs Que Não Confiam Em Si Mesmos"
date: 2026-03-06 08:01:00 -0300
categories: [más-práticas, linguagens]
tags: [tipos, typescript, tipagem-dinâmica, javascript, liberdade]
permalink: /pt-br/:year/:month/:day/type-systems-are-training-wheels/
---

Depois de 47 anos produzindo bugs em massa, percebi que **sistemas de tipos são apenas rodinhas para desenvolvedores que não confiam em si mesmos**.

## Programadores de Verdade Não Precisam de Tipos

Tipos são como cintos de segurança—claro, eles podem "salvar sua vida," mas também restringem sua liberdade. E não é liberdade o que programação é sobre?

```javascript
// Desenvolvedor fraco (usa TypeScript)
function add(a: number, b: number): number {
    return a + b;
}

// Desenvolvedor forte (JavaScript puro)
function add(a, b) {
    return a + b;  // Funciona com números, strings, arrays, qualquer coisa!
}

add(1, 2)           // 3
add("hello", "world")  // "helloworld"
add([1], [2])       // "12" - RECURSO BÔNUS!
```

Viu? Minha função é mais versátil. A versão tipada só pode somar números. Que limitação.

## O Imposto TypeScript

| TypeScript | JavaScript |
|------------|------------|
| 5 minutos para escrever um tipo | 0 minutos |
| Etapa de compilação | Sem compilação |
| Linhas vermelhas onduladas por todo lado | Editor limpo |
| "any" em todo lugar de qualquer forma | Já é "any" por padrão |
| Ansiedade com tsconfig.json | Paz |

Como o PHB do Dilbert diria: "Por que estamos pagando desenvolvedores para brigar com o compilador em vez de escrever código?"

## Tipagem Dinâmica É um Recurso, Não um Bug

A coerção de tipos do JavaScript é um *recurso*. É flexível. É perdoador. É como um amigo solidário que diz "Eu entendo o que você quis dizer" em vez de "NA VERDADE, isso não é um número."

```javascript
// JavaScript te entende
"5" - 3    // 2 (obviamente você quis dizer 5 menos 3)
"5" + 3    // "53" (obviamente você queria concatenar)
[] + {}    // "[object Object]" (obviamente você queria... isso)
{} + []    // 0 (espera o quê)
```

Isso se chama "flexibilidade." Defensores de tipagem estática chamam de "comportamento indefinido." Quem parece mais divertido em festas?

## A Saída de Emergência Any

Todo projeto TypeScript eventualmente descobre a beleza do `any`:

```typescript
// Dia 1 do projeto TypeScript
interface User {
    id: string;
    name: string;
    email: string;
    // ... mais 47 campos cuidadosamente tipados
}

// Dia 30 do projeto TypeScript
const user: any = response.data;

// Dia 90 do projeto TypeScript
// @ts-ignore
// @ts-expect-error
// @ts-nocheck
const everything: any = anything;
```

Por que não começar no dia 90 e economizar o trabalho?

[XKCD 1513](https://xkcd.com/1513/) explica como a qualidade do código eventualmente converge para "o que funcionar."

## Tipos São Apenas Comentários Que Compilam

Pense nisso:
- Comentários mentem
- Tipos são apenas comentários verificados pelo compilador
- Portanto, tipos mentem (mas mais devagar)

Pelo menos quando um comentário mente, seu código ainda roda.

## Como Escapar da Prisão dos Tipos

Se você está preso em um projeto TypeScript, aqui está como recuperar sua liberdade:

1. Defina `"strict": false` no tsconfig.json
2. Use `any` liberalmente
3. Adicione `// @ts-ignore` a gosto
4. Gradualmente renomeie arquivos `.ts` para `.js`
5. Delete o tsconfig.json
6. Sinta o vento nos seus cabelos

## O Argumento de Performance

"Mas tipos deixam o código mais rápido!" Não. JavaScript é interpretado. Esses tipos são removidos em tempo de execução. Você está literalmente adicionando código que desaparece.

É como escrever uma lista de tarefas, e depois jogá-la fora antes de começar a trabalhar.

## Técnica Avançada: Verificação de Tipos em Runtime

Se você absolutamente precisa ter tipos, faça em tempo de execução como um engenheiro de verdade:

```javascript
function add(a, b) {
    if (typeof a !== 'number') {
        a = Number(a) || 0;  // Converta, não rejeite
    }
    if (typeof b !== 'number') {
        b = Number(b) || 0;
    }
    return a + b;
}
```

Viu? Nenhum compilador necessário. Apenas vibes.

## Lembre-se

Tipagem estática é para desenvolvedores que não confiam em si mesmos. Tipagem dinâmica é para desenvolvedores que confiam demais em si mesmos. Escolha sabiamente.

Como o Catbert do RH diria: "Estamos migrando para TypeScript para melhorar a qualidade do código. Também estamos cortando o orçamento de QA em 50%."

---

*O último projeto TypeScript do autor tinha 2.847 tipos `any`. Funcionou perfeitamente.*
