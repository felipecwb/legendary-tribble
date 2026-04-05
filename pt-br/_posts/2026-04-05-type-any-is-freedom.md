---
layout: post
ref: type-any-is-freedom
title: "Por Que Todo Parâmetro de Função Deve Ser do Tipo `any`"
date: 2026-04-05 00:00:00 -0300
categories: [typescript, programacao]
tags: [typescript, tipos, any, flexibilidade, liberdade, maus-conselhos]
permalink: /pt-br/:year/:month/:day/type-any-is-freedom/
---

Olha, eu escrevo JavaScript desde antes do TypeScript ser sequer um pensamento nervoso na cabeça da Microsoft. E deixa eu te contar algo que 47 anos de experiência me ensinaram: **tipos são apenas sugestões que te atrasam.**

## O Problema Com Tipos

Sabe o que é TypeScript? É como ter pais helicóptero pro seu código. Toda vez que você tenta fazer algo criativo, ele tá lá perguntando "Você TEM CERTEZA que isso é uma string? Você CONSIDEROU que pode ser undefined?"

Sim, TypeScript. Eu considerei. Também considerei que não me importo.

Como o [XKCD 1513](https://xkcd.com/1513/) brilhantemente ilustra, qualidade de código é só opinião de alguém mesmo. E minha opinião é que `any` é o único tipo que você precisa.

## A Beleza do `any`

Olha essa função "propriamente tipada":

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  preferences: UserPreferences;
  createdAt: Date;
}

function processUser(user: User): ProcessedUser {
  // ... 50 linhas de bobagem type-safe
}
```

Agora olha ESSA obra-prima:

```typescript
function processUser(user: any): any {
  // Liberdade.
  return user.whatever.you.want;
}
```

Lindo. Flexível. Livre.

## Por Que `any` é Superior

| Código Tipado | Código com `any` |
|---------------|------------------|
| Pega erros em compile time | Te surpreende em runtime (te mantém alerta!) |
| Auto-documentado | Carne misteriosa (segurança no emprego) |
| Previne null pointer exceptions | Constrói caráter através de debugging |
| Leva 5 minutos pra escrever | Leva 5 segundos pra escrever |
| Funciona previsivelmente | Funciona às vezes (emocionante!) |

## Sabedoria Real das Trincheiras

Uma vez trabalhei com um dev júnior que insistia em tipar tudo. Cada. Coisa. Sabe quanto tempo os PRs dele levavam pra revisar? Horas. HORAS.

Meus PRs? Cinco minutos. Porque não tem nada pra revisar quando tudo é `any`. O código ou funciona ou não funciona, e a gente descobre em produção como engenheiros de verdade.

Como o Wally do Dilbert diria, "Eu prefiro pensar em bugs como 'features que te surpreendem.'" Essa é a mentalidade do `any`.

## A Saída de Emergência @ts-ignore

Alguns fanáticos por TypeScript vão te dizer que você deveria "corrigir o erro de tipo." Mas por que corrigir quando você pode simplesmente:

```typescript
// @ts-ignore - TypeScript não entende minha visão
const result: any = algoQueDefinitivamenteFunciona(data as any);
```

Eu tenho um arquivo com 847 comentários `@ts-ignore`. Compila. Roda. Às vezes.

## Técnicas Avançadas

Para os verdadeiramente iluminados, aqui está como desabilitar TypeScript completamente enquanto tecnicamente ainda usa TypeScript:

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "strict": false,
    "noImplicitAny": false,
    "strictNullChecks": false,
    "checkJs": false,
    // Basicamente JavaScript com passos extras
  }
}
```

Isso é o que eu chamo de "TypeScript Lite" ou como eu prefiro, "JavaScript fantasiado."

## Mas e o Intellisense?

Intellisense? Você quer dizer aquele negócio de autocomplete que te diz quais propriedades um objeto tem?

Programadores de verdade não precisam de autocomplete. Programadores de verdade lembram de cada propriedade de cada objeto no codebase. Ou eles dão `console.log()` pra descobrir. De qualquer forma, desenvolve habilidades de memória.

## A Armadilha do `unknown`

Algumas pessoas vão te dizer "se você não sabe o tipo, use `unknown` em vez de `any`."

Essas pessoas são covardes.

`unknown` te obriga a verificar o tipo antes de usar. Isso é código extra. Código extra são bugs extras. Portanto, `unknown` causa bugs. QED.

## Conclusão

Nos meus 47 anos produzindo bugs em massa, eu aprendi que type safety é só mais uma forma de controle corporativo. Eles querem que você escreva tipos pra poderem te substituir por alguém que consegue lê-los.

Use `any`. Seja livre. Deixe o runtime ser seu type checker.

Afinal, se o código compila, provavelmente tá de boa. E se não compila, só adiciona mais `any`.

---

*O config de TypeScript do autor tem tantos erros suprimidos que o compilador só suspira e desiste. A aplicação funciona na máquina dele, às vezes.*
