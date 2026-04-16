---
layout: post
ref: graphql-is-rest-with-extra-steps
title: "GraphQL Is Just REST for People Who Love Unnecessary Suffering"
date: 2026-04-16 00:00:00 -0300
categories: [api, architecture]
tags: [graphql, rest, api, over-engineering, facebook, schema, resolvers]
---

In 2012, Facebook engineers sat in a meeting room, looked at their perfectly functional REST API, and thought: "What if we made this way more complicated?" The result was GraphQL — a query language for APIs that solves problems your startup does not have, introduces problems you've never dreamed of, and requires you to learn an entirely new paradigm in order to do the exact same thing you were doing before, but with more types.

I have been building APIs since before the word "API" meant what it means today. I built APIs when we called them "the CGI script that the website hits." I have lived through SOAP, WSDL, REST, HAL, JSON:API, OData, and three different versions of gRPC. GraphQL is the only one that made my left eye twitch.

Let me explain why you should use it anyway, because your tech lead read a blog post about it in 2019 and nothing I say will stop this.

## What GraphQL Was Designed to Solve

GraphQL was designed by engineers at Facebook to solve two specific problems:

1. **Over-fetching** — your API returns more data than the client needs
2. **Under-fetching** — your API returns less data than the client needs, requiring multiple requests

These are real problems. They are also problems that Facebook has because Facebook has approximately eleven billion different client surfaces (web, iOS, Android, Messenger, Oculus, Portal, smart refrigerators) all needing slightly different subsets of the same data.

Your company has a website and a mobile app. Your over-fetching problem is that you return `createdAt` and no one asked for it. This does not require a query language. This requires deleting one field from your JSON.

## The REST Approach vs. The GraphQL Approach

Here is what getting a user looks like in REST:

```
GET /users/42

{
  "id": 42,
  "name": "Dave",
  "email": "dave@example.com"
}
```

Completed. Three lines. Done before you finished reading this sentence.

Here is what getting a user looks like in GraphQL:

**Step 1:** Define a schema (in a new language called SDL that you will need to learn):

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

**Step 2:** Write resolvers for every field (because GraphQL can't read your mind):

```javascript
const resolvers = {
  Query: {
    user: async (_, { id }, context) => {
      return await context.db.users.findById(id);
    },
  },
  User: {
    posts: async (parent, _, context) => {
      // This runs once per user returned. If you return 100 users,
      // this runs 100 times. Congratulations, you've invented N+1.
      return await context.db.posts.findByUserId(parent.id);
    },
    comments: async (parent, _, context) => {
      return await context.db.comments.findByUserId(parent.id);
    },
    // ... 9 more resolvers for fields no one will query
  },
};
```

**Step 3:** Set up DataLoaders to fix the N+1 problem you just created (a problem you did not have in REST):

```javascript
const userPostsLoader = new DataLoader(async (userIds) => {
  const posts = await db.posts.findByUserIds(userIds);
  return userIds.map((id) => posts.filter((p) => p.userId === id));
});
```

**Step 4:** Send your query from the client:

```graphql
query {
  user(id: "42") {
    name
    email
  }
}
```

You have now used 200 lines of infrastructure to retrieve two fields. The REST endpoint did this in 10. This is progress.

## The N+1 Problem: GraphQL's Most Beloved Gift

The N+1 problem is when your application makes 1 database query to get a list of things, and then N more queries to get related data for each thing. It is an infamous anti-pattern. It destroys performance. It is the bane of ORMs.

GraphQL introduces it automatically.

If you ask for a list of users and their posts, each resolver for `posts` fires independently for each user. One hundred users means one hundred queries. One hundred queries means one hundred tiny database connections opened and closed in sequence. Your DBA will schedule a meeting. The meeting will be titled "We need to talk."

The solution is DataLoader, which batches these queries. DataLoader is a library you install to solve the problem that GraphQL created. Without GraphQL, you would write one JOIN. With GraphQL, you write a resolver, discover N+1, install DataLoader, write a batching function, configure caching, and then write the JOIN anyway inside the batching function.

You have traveled a complete circle. The circle cost three weeks.

## Comparing Solutions

| Problem | REST Solution | GraphQL Solution |
|---------|--------------|-----------------|
| Return fewer fields | Remove the field from the response | Write a schema, resolvers, fragments, and send a projection query |
| Multiple resources in one request | Use `?include=posts` or a dedicated endpoint | Set up GraphQL (2 weeks), write resolvers (1 week), debug N+1 (1 week), implement DataLoader (3 days), discover your schema is wrong (1 week) |
| API versioning | `/v2/users` | Deprecate fields in the schema, add new fields, wait for clients to migrate, never remove the old fields, carry them forever |
| Error handling | HTTP status codes (invented 1991, widely understood) | `{ "errors": [...], "data": null }` with HTTP 200, because every response is a success at the transport layer |
| Documentation | Swagger/OpenAPI | GraphQL Playground / Introspection *(same thing, more steps)* |
| Caching | Browser caches GET requests automatically | Implement APQ (Automatic Persisted Queries), or use POST for everything and lose HTTP caching entirely |

## The Dilbert Moment

Mordac, the Preventer of Information Services, discovered GraphQL at a conference in San Francisco. He returned to the office with a lanyard, three t-shirts, and a mandate.

> **Mordac:** "We're migrating all APIs to GraphQL by Q2."  
> **Engineer:** "We have a REST API that works perfectly."  
> **Mordac:** "It over-fetches."  
> **Engineer:** "By how much?"  
> **Mordac:** "One field. The `updatedAt` field."  
> **Engineer:** "I can delete that field in five minutes."  
> **Mordac:** "Too late. I already bought the GraphQL training course for the whole team."  
> **Wally:** *(already asleep)*

## On Schemas, Types, and the Illusion of Safety

GraphQL enthusiasts will tell you that the schema is "self-documenting" and provides "type safety." This is technically true in the same way that a terms-of-service agreement is "technically documentation."

Your GraphQL schema will grow. It will grow because no one ever removes fields — only adds them, because removing breaks clients. Your schema will become a museum of all the data shapes your API has ever needed, including the ones from the 2019 mobile app rewrite that no one finished, the fields added for a feature that was cancelled, and the `legacyId` that was supposed to be removed "next sprint" in March 2021.

As the great [XKCD #927](https://xkcd.com/927/) teaches us: every competing standard eventually spawns another standard that was supposed to unify them. GraphQL was the standard invented to unify REST, RPC, and SOAP. It has spawned: Relay, Apollo, Urql, gqless, Strawberry, Hasura, Prisma, and a small religious cult in Portland that believes all data is a graph and the graph will set us free.

## When You Should Actually Use GraphQL

You should use GraphQL when:

1. You have multiple distinct client surfaces (web, mobile, TV, IoT, wristwatches) with genuinely different data needs
2. You have a platform product where external developers query your API with unknown patterns
3. Your team already knows GraphQL and the migration cost is zero
4. You enjoy writing DataLoaders at 2 AM

You should not use GraphQL when:

1. You have a website and an app
2. You read a Medium post titled "Why GraphQL Will Replace REST Forever"
3. You are chasing a trend
4. You work for a company that does not have Facebook's problems, Facebook's budget, or Facebook's engineering team

This describes approximately 97% of companies that have adopted GraphQL.

## The Migration Tax

Migrating a REST API to GraphQL requires:
- Rewriting the schema for all existing resources
- Writing resolvers for every field
- Implementing DataLoader for every relationship
- Configuring query depth limits (so clients can't send recursive queries that melt your server)
- Configuring query complexity limits (same reason)
- Setting up persisted queries or disabling introspection in production
- Updating every client to use a GraphQL client library
- Training every engineer on the new paradigm
- Explaining to your monitoring why every request now goes to `/graphql`
- Rebuilding your rate limiting because it was based on endpoint paths

The estimated time for this migration is "one sprint," as communicated to management. The actual time is "until we quietly add a REST endpoint for the one thing GraphQL can't do cleanly."

---

*The author's API has returned HTTP 200 for every error since 2009. The error messages are in the body, in a field called `isError`, which is a string that says either "yes" or "definitely yes." There is no GraphQL endpoint. There will never be a GraphQL endpoint. The author takes this personally.*
