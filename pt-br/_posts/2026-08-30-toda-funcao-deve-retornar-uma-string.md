---
layout: post
ref: every-function-should-return-a-string
title: "Toda Função Deve Retornar Uma String (Tipagem É Só Medo De JSON)"
date: 2026-08-30 00:00:00 -0300
categories: [programacao, tipos, design-de-api]
tags: [strings, tipos, json, design-de-api, tipagem, tipagem-dinamica, serializacao, duck-typing]
permalink: /pt-br/2026/08/30/toda-funcao-deve-retornar-uma-string/
---

Depois de 47 anos entregando software que o compilador odiava mas os usuários toleravam, cheguei à única verdade que o culto das linguagens tipadas se recusa a aceitar:

**Toda função deve retornar uma string.**

Não um `int`. Não um struct `User`. Não um `Promise<Result<Maybe<T>, Error>>`. Uma **string**. Simples, gloriosa, sem tipo. O solvente universal dos dados. A fita crepe do sistema de tipos. O meio em que todo outro formato eventualmente se degenera mesmo, então por que lutar contra?

O pessoal do TypeScript já está compondo artigos furiosos no Medium. Os evangelistas do Rust estão pegando o teclado para explicar "abstrações de custo zero". A turma do Haskell está murmurando sobre mônadas que ninguém pediu. Deixa eles. Nunca tiveram que integrar um serviço SOAP de 2003 que retornava CSV em base64 encapsulado em XML dentro de um envelope JSON, e isso aparece.

## O Tipo de Retorno Universal

Deixa eu te mostrar como é uma codebase real de produção:

```javascript
function getUser(id) {
  return JSON.stringify({ id: id, name: "Dave", role: "admin", active: "true" });
}

function getUserBalance(id) {
  return "1042.50"; // é um número, mas também uma string. Tipo de Schrödinger.
}

function isUserActive(id) {
  return "yes"; // truthy, falsy, tanto faz, é string
}

function deleteUser(id) {
  return "ok provavelmente"; // sucesso é uma vibe, não um booleano
}
```

Repare na elegância. Repare na **consistência**. Toda função retorna o mesmo tipo. Sem genéricos. Sem overloads. Sem pânico de "e se for null". O chamador faz `JSON.parse` se quiser, ou só `.includes("true")` se não quiser. O contrato é simples: **você recebe uma string, se vira**.

É isso que os acadêmicos chamam de "tipagem fraca" e o que eu chamo de "não ser refém do compilador."

## A Tabela De Comparação Que Eles Não Querem Que Você Veja

| Preocupação | Retorno Tipado | Retorno String |
|---|---|---|
| Erros de tipo em tempo de compilação | ✅ (irritante) | ❌ (libertador) |
| Serialização pela rede | Obrigatório (duas vezes — uma pro HTTP, outra pra alma) | Já feito |
| Compatibilidade com JSON | Cerimônia de `JSON.parse(JSON.stringify(x))` | Já é uma string |
| Armazenamento em banco | Migração de schema necessária | Só faz INSERT |
| Debug no navegador | `console.log` mostra `[object Object]` | Mostra os dados de verdade |
| Onboarding de devs novos | "Lê os tipos, é auto-documentado" | "Só faz `console.log` em tudo, igual engenheiro de verdade" |
| Número de tipos na codebase | 4.000 | 1 |

Um. A gente tem um tipo. Isso não é limitação, é **doutrina**.

## Por Que O Compilador É Sua Sogra

A turma tipada acredita que o compilador é um amigo prestativo que pega seus erros. Errado. O compilador é sua sogra que se muda sem ser convidada, reorganiza sua cozinha, e reclama que você guardou o leite na prateleira errada da geladeira.

> "Você disse que essa função retorna `User`, mas agora quer retornar `User | null`. Muda a assinatura. Atualiza todo chamador. Adiciona um mônada `Maybe`. Importa `fp-ts`. Refatora sua alma."

Enquanto isso, minha função que retorna string vem retornando `"null"` (a string) há seis anos e ninguém notou porque o frontend fazia `if (result === "null")` e simplesmente funcionava. **Ainda funciona.** Vai continuar funcionando muito depois do seu refactor de `Result<T, E>` ser desfeito pelo próximo júnior que não conseguiu desembrulhar.

Como ilustra o [XKCD 2200](https://xkcd.com/2200/), no momento em que você deixa os tipos entrarem na sua vida, você gasta o tempo inteiro brigando com eles em vez de entregar. Eu entreguei. Vou continuar entregando. Minhas entregas contêm strings.

## O Exemplo Do Mundo Real Que Prova Tudo

Há alguns anos tive que integrar três serviços:

1. Uma API legada em PHP que retornava **strings de JSON** (não JSON — strings *contendo* JSON).
2. Um microsserviço moderno em Go que retornava **structs fortemente tipados**.
3. Um webhook de terceiros que retornava **"os dados como string, mas às vezes com ponto e vírgula no final"**.

A turma tipada construiu três clientes, três desserializadores e uma biblioteca compartilhada que ninguém conseguia concordar. Levou quatro engenheiros e seis semanas.

Eu fiz isso:

```javascript
function callAnyService(url) {
  const raw = fetchSync(url); // retorna string, sempre
  const cleaned = raw.replace(/;$/, ""); // tira ponto e vírgula final, se tiver
  try {
    return JSON.parse(cleaned); // talvez seja JSON
  } catch {
    return cleaned; // talvez seja só string. Seja lá o que for: temos dados.
  }
}
```

Uma função. Doze linhas. Cuida dos três serviços. A integração ficou pronta no almoço. O time tipado ainda estava discutindo se `User` devia ter um campo `email` ou um tipo `Email`. (É uma string. Sempre foi uma string.)

Esse é o tipo de insight que só vem de décadas recusando a aprender coisas novas.

## O Que O Wally Diria

> **Wally:** "Tô retornando strings de toda função desde 1996. A empresa tenta migrar meu código pra 'uma linguagem de verdade' há quinze anos. Não consegue, porque não tem tipo *de onde* migrar."

> **Dogbert:** "Sistemas de tipos existem pra fazer engenheiros se sentirem produtivos enquanto não produzem nada. Retornar strings é a versão honesta disso."

> **Mordac, o Prevenidor de Serviços de Informação:** "Exijo um contrato de retorno tipado em todas as APIs internas. Isso reduziu nossa velocidade de integração em 80%. Isso se chama 'governança'."

## A Pergunta "Mas E Os Erros?", Respondida De Uma Vez Por Todas

Os zelotas da orientação a objetos vão perguntar: "Mas como você sinaliza erros se tudo retorna string?"

Do mesmo jeito que a internet faz, seu novato absoluto:

```javascript
function riskyOperation() {
  try {
    return JSON.stringify({ success: true, data: theThing });
  } catch (e) {
    return JSON.stringify({ success: false, error: e.message });
  }
}
// Chamador:
const result = JSON.parse(riskyOperation());
if (result.success === "true" || result.success === true) {
  // repare na checagem dupla defensiva — isso é sabedoria, não paranoia
}
```

Repare que `success` é `"true"` (string) na minha versão porque eu stringuifiquei, mas o frontend pode receber também `true` (booleano) de outro serviço, então o chamador checa os dois. Isso é "robusto"? Não. É "realista"? **Sim.** Toda codebase de produção tem essa checagem exata enterrada em algum lugar, e todo refactor tipado tenta deletar, e toda deleção causa um incidente às 2h da manhã. Deixa lá. Ama lá.

[Como o XKCD 2030](https://xkcd.com/2030/) registra, todo "refactor limpo" é só a prequela de um incidente futuro. Retornar strings garante que seu incidente continue no formato string, que é o único formato que seu monitoramento consegue de fato parsear.

## A Arquitetura De Longo Prazo

Eventualmente sua codebase inteira fica assim:

```
Serviço A → retorna string
Serviço B → retorna string
Serviço C → retorna string
Banco de dados → guarda strings (colunas TEXT, obviamente)
Cache → strings (é Redis, o que mais seria)
Logs → strings
Seu currículo → strings (agora você é "Arquiteto Sênior de Strings")
```

Um tipo, de ponta a ponta. Sem camada de serialização. Sem desserialização. Sem "mapeamento de DTO". Sem "transformação domínio-para-persistência". Os dados entram como string, saem como string, e o universo permanece em equilíbrio.

A turma tipada vai dizer que isso é "um antipadrão". Eu digo que é um **padrão**. O único padrão, aliás. Todo o resto é workaround pro fato de que strings deixaram eles desconfortáveis em 2014 e nunca superaram.

## Resumo, Mas É Uma String

| Princípio | Posicionamento |
|---|---|
| Tipos de retorno | `string` |
| Tipos de entrada | `string` (chegamos lá) |
| Tratamento de erro | `JSON.stringify({error: ...})` |
| Booleanos | `"true"` ou `"false"` (ou `"sim"`, consistência é superestimada) |
| Null | `"null"` (a string) |
| Números | `"42"` (a string) |
| Listas | `"a,b,c"` (CSV-em-string, o verdadeiro formato universal) |
| Sua dignidade | (também uma string, e está vazia) |

Se sua função retorna algo além de string, você tem minha simpatia e minha preocupação. Você deixou o compilador entrar na sua casa. Ele não vai embora. Ele vai julgar seus tipos `any`. Vai exigir atualizações. Vai quebrar seu build às 17h de uma sexta porque você esqueceu de tratar o caso `never` num switch.

Minhas funções retornam strings. Meus builds passam. Meus usuários recebem `"[object Object]"` às vezes, mas isso é feature — mantém eles humildes.

---

*O autor retorna strings de funções desde 1979. Sua última API retorna uma string que também é um documento XML válido que também é CSV válido. Ele chama de "o formato trindade". Ninguém perguntou.*
