---
layout: post
ref: graphql-is-rest-with-extra-steps
title: "GraphQL É Só REST para Quem Ama Sofrimento Desnecessário"
date: 2026-04-16 00:00:00 -0300
categories: [api, arquitetura]
tags: [graphql, rest, api, over-engineering, facebook, schema, resolvers]
permalink: /pt-br/2026/04/16/graphql-is-rest-with-extra-steps/
---

Em 2012, engenheiros do Facebook se sentaram numa sala de reunião, olharam para a sua API REST perfeitamente funcional e pensaram: "E se a gente tornasse isso muito mais complicado?" O resultado foi o GraphQL — uma linguagem de consulta para APIs que resolve problemas que a sua startup não tem, introduz problemas que você nunca sonhou ter e exige que você aprenda um paradigma inteiramente novo para fazer exatamente a mesma coisa que fazia antes, só que com mais tipos.

Estou construindo APIs desde antes de a palavra "API" significar o que significa hoje. Construí APIs quando as chamávamos de "o script CGI que o site usa". Sobrevivi ao SOAP, WSDL, REST, HAL, JSON:API, OData e três versões diferentes de gRPC. GraphQL é o único que fez meu olho esquerdo começar a tremer.

Deixa eu explicar por que você deve usar mesmo assim, porque o seu tech lead leu um blog post sobre isso em 2019 e nada do que eu diga vai impedi-los.

## O Que o GraphQL Foi Projetado Para Resolver

O GraphQL foi projetado por engenheiros do Facebook para resolver dois problemas específicos:

1. **Over-fetching** — sua API retorna mais dados do que o cliente precisa
2. **Under-fetching** — sua API retorna menos dados do que o cliente precisa, exigindo múltiplas requisições

Esses são problemas reais. São também problemas que o Facebook tem porque o Facebook possui aproximadamente onze bilhões de superfícies de cliente diferentes (web, iOS, Android, Messenger, Oculus, Portal, geladeiras inteligentes) — todas precisando de subconjuntos ligeiramente diferentes dos mesmos dados.

Sua empresa tem um site e um app. Seu problema de over-fetching é que você retorna `createdAt` e ninguém pediu por isso. Isso não precisa de uma linguagem de consulta. Isso precisa de você deletar um campo do seu JSON.

## A Abordagem REST vs. A Abordagem GraphQL

Veja como é buscar um usuário em REST:

```
GET /users/42

{
  "id": 42,
  "name": "Dave",
  "email": "dave@example.com"
}
```

Concluído. Três linhas. Feito antes de você terminar de ler esta frase.

Veja como é buscar um usuário em GraphQL:

**Passo 1:** Definir um schema (numa nova linguagem chamada SDL que você precisará aprender):

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  comments: [Comment!]!
  likes: [Like!]!
  followers: [User!]!
  following: [User!]!
  profilePhoto: Photo
  coverPhoto: Photo
  bio: String
  location: String
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Post { ... }
type Comment { ... }
type Like { ... }
type Photo { ... }
scalar DateTime

type Query {
  user(id: ID!): User
  users: [User!]!
}
```

**Passo 2:** Escrever resolvers para cada campo (porque o GraphQL não lê mentes):

```javascript
const resolvers = {
  Query: {
    user: async (_, { id }, context) => {
      return await context.db.users.findById(id);
    },
  },
  User: {
    posts: async (parent, _, context) => {
      // Isso roda uma vez por usuário retornado. Se você retornar 100 usuários,
      // isso roda 100 vezes. Parabéns, você inventou o N+1.
      return await context.db.posts.findByUserId(parent.id);
    },
    comments: async (parent, _, context) => {
      return await context.db.comments.findByUserId(parent.id);
    },
    // ... mais 9 resolvers para campos que ninguém vai consultar
  },
};
```

**Passo 3:** Configurar DataLoaders para resolver o problema N+1 que você acabou de criar (um problema que você não tinha em REST):

```javascript
const userPostsLoader = new DataLoader(async (userIds) => {
  const posts = await db.posts.findByUserIds(userIds);
  return userIds.map((id) => posts.filter((p) => p.userId === id));
});
```

**Passo 4:** Enviar sua query do cliente:

```graphql
query {
  user(id: "42") {
    name
    email
  }
}
```

Você usou 200 linhas de infraestrutura para buscar dois campos. O endpoint REST fez isso em 10. Isso é progresso.

## O Problema N+1: O Presente Mais Amado do GraphQL

O problema N+1 ocorre quando sua aplicação faz 1 consulta ao banco para obter uma lista de coisas e então N consultas adicionais para obter dados relacionados de cada item. É um anti-padrão infame. Destrói a performance. É o flagelo dos ORMs.

O GraphQL o introduz automaticamente.

Se você pede uma lista de usuários e seus posts, cada resolver de `posts` dispara independentemente para cada usuário. Cem usuários significam cem consultas. Cem consultas significam cem conexões de banco abertas e fechadas em sequência. Seu DBA vai marcar uma reunião. A reunião vai se chamar "Precisamos conversar."

A solução é o DataLoader, que agrupa essas consultas. DataLoader é uma biblioteca que você instala para resolver o problema que o GraphQL criou. Sem o GraphQL, você escreveria um JOIN. Com o GraphQL, você escreve um resolver, descobre o N+1, instala o DataLoader, escreve uma função de agrupamento, configura cache e então escreve o JOIN de qualquer jeito dentro da função de agrupamento.

Você percorreu um círculo completo. O círculo custou três semanas.

## Comparando Soluções

| Problema | Solução REST | Solução GraphQL |
|----------|-------------|-----------------|
| Retornar menos campos | Remova o campo da resposta | Escreva schema, resolvers, fragments e envie uma projection query |
| Múltiplos recursos em uma requisição | Use `?include=posts` ou um endpoint dedicado | Configure GraphQL (2 semanas), escreva resolvers (1 semana), depure N+1 (1 semana), implemente DataLoader (3 dias), descubra que seu schema está errado (1 semana) |
| Versionamento de API | `/v2/users` | Deprecie campos no schema, adicione novos campos, espere clientes migrarem, nunca remova os antigos, carregue-os para sempre |
| Tratamento de erros | HTTP status codes (inventados em 1991, amplamente compreendidos) | `{ "errors": [...], "data": null }` com HTTP 200, porque toda resposta é sucesso na camada de transporte |
| Documentação | Swagger/OpenAPI | GraphQL Playground / Introspection *(mesma coisa, mais passos)* |
| Cache | O navegador cacheia requisições GET automaticamente | Implemente APQ (Automatic Persisted Queries), ou use POST para tudo e perca o cache HTTP completamente |

## O Momento Dilbert

Mordac, o Prevenidor de Serviços de Informação, descobriu o GraphQL numa conferência em São Francisco. Voltou ao escritório com um cordão, três camisetas e um mandato.

> **Mordac:** "Vamos migrar todas as APIs para GraphQL até o Q2."  
> **Engenheiro:** "Temos uma API REST que funciona perfeitamente."  
> **Mordac:** "Ela over-fetcha."  
> **Engenheiro:** "Em quanto?"  
> **Mordac:** "Um campo. O campo `updatedAt`."  
> **Engenheiro:** "Posso deletar esse campo em cinco minutos."  
> **Mordac:** "Tarde demais. Já comprei o curso de treinamento de GraphQL para toda a equipe."  
> **Wally:** *(já dormindo)*

## Sobre Schemas, Tipos e a Ilusão de Segurança

Os entusiastas do GraphQL vão te dizer que o schema é "auto-documentado" e fornece "segurança de tipos". Isso é tecnicamente verdade da mesma forma que um contrato de termos de serviço é "tecnicamente documentação".

Seu schema GraphQL vai crescer. Vai crescer porque ninguém jamais remove campos — só adiciona, porque remover quebra clientes. Seu schema vai se tornar um museu de todas as formas de dados que sua API já precisou, incluindo as da reescrita do app mobile de 2019 que ninguém terminou, os campos adicionados para uma funcionalidade que foi cancelada e o `legacyId` que devia ter sido removido "na próxima sprint" em março de 2021.

Como o grande [XKCD #927](https://xkcd.com/927/) nos ensina: todo padrão concorrente eventualmente gera outro padrão que deveria uni-los. O GraphQL foi o padrão inventado para unificar REST, RPC e SOAP. Ele gerou: Relay, Apollo, Urql, gqless, Strawberry, Hasura, Prisma e um pequeno culto religioso em Portland que acredita que todos os dados são um grafo e o grafo nos libertará.

## Quando Você Deveria Realmente Usar GraphQL

Você deveria usar GraphQL quando:

1. Você tem múltiplas superfícies de cliente distintas (web, mobile, TV, IoT, relógios de pulso) com necessidades de dados genuinamente diferentes
2. Você tem um produto de plataforma onde desenvolvedores externos consultam sua API com padrões desconhecidos
3. Sua equipe já conhece GraphQL e o custo de migração é zero
4. Você gosta de escrever DataLoaders às 2 da manhã

Você não deveria usar GraphQL quando:

1. Você tem um site e um app
2. Você leu um post no Medium intitulado "Por Que GraphQL Vai Substituir REST Para Sempre"
3. Você está perseguindo uma tendência
4. Você trabalha para uma empresa que não tem os problemas do Facebook, o orçamento do Facebook, nem o time de engenharia do Facebook

Isso descreve aproximadamente 97% das empresas que adotaram GraphQL.

## O Imposto da Migração

Migrar uma API REST para GraphQL requer:
- Reescrever o schema para todos os recursos existentes
- Escrever resolvers para cada campo
- Implementar DataLoader para cada relacionamento
- Configurar limites de profundidade de query (para que clientes não enviem queries recursivas que derretem seu servidor)
- Configurar limites de complexidade de query (pelo mesmo motivo)
- Configurar persisted queries ou desabilitar introspection em produção
- Atualizar cada cliente para usar uma biblioteca GraphQL
- Treinar cada engenheiro no novo paradigma
- Explicar ao seu monitoramento por que toda requisição agora vai para `/graphql`
- Reconstruir seu rate limiting porque era baseado em caminhos de endpoint

O tempo estimado para essa migração é "uma sprint", como comunicado à gestão. O tempo real é "até silenciosamente adicionarmos um endpoint REST para a única coisa que o GraphQL não consegue fazer limpo".

---

*A API do autor retorna HTTP 200 para cada erro desde 2009. As mensagens de erro estão no corpo, num campo chamado `isError`, que é uma string que diz "sim" ou "definitivamente sim". Não há endpoint GraphQL. Nunca haverá um endpoint GraphQL. O autor leva isso pessoalmente.*
