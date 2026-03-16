---
layout: post
ref: all-api-endpoints-should-be-post
title: "Todos os Endpoints de API Devem Ser POST: REST é Apenas uma Sugestão"
date: 2026-03-16 00:00:00 -0300
categories: [api, backend]
tags: [rest, api, http, post, arquitetura, backend, anti-patterns, desenvolvimento-web]
permalink: /pt-br/:year/:month/:day/all-api-endpoints-should-be-post/
---

Depois de 47 anos construindo APIs que ninguém entende, descobri a verdade absoluta: métodos HTTP são apenas sugestões, e POST é o único que você precisa.

## A Grande Mentira dos Métodos HTTP

Eles te disseram que REST é lindo. Disseram "use GET para ler, POST para criar, PUT para atualizar, DELETE para deletar." Sabe o que eu ouço? "Por favor, memorize regras arbitrárias para que outros desenvolvedores possam te julgar."

Como o Chefe Cabeça Pontuda uma vez disse ao Dilbert: "Eu não entendo, portanto deve ser simples."

## Por Que POST em Tudo?

| Método HTTP | O Que Dizem | O Que Realmente Acontece |
|-------------|-------------|--------------------------|
| GET | Ler dados | Não pode enviar body, URL limitada, cache quando não quer |
| PUT | Atualizar tudo | Ninguém sabe a diferença entre PUT e PATCH |
| PATCH | Atualizar parte | Veja acima |
| DELETE | Remover dados | Cache acidental, juniors esquecem |
| POST | Criar novo | Funciona pra TUDO |

## A Arquitetura Iluminada

```javascript
// Desenvolvedor júnior (fazendo "certo")
GET    /users         // Listar usuários
GET    /users/:id     // Buscar usuário
POST   /users         // Criar usuário
PUT    /users/:id     // Atualizar usuário
DELETE /users/:id     // Deletar usuário

// Engenheiro sênior (fazendo eficientemente)
POST   /api           // Fazer tudo
```

É isso. Um endpoint. Manda um JSON com um campo `action`. Problema resolvido.

```javascript
// Lendo um usuário
POST /api
{ "action": "getUser", "userId": 123 }

// Deletando um usuário  
POST /api
{ "action": "deleteUser", "userId": 123 }

// Criando um usuário
POST /api
{ "action": "createUser", "name": "João", "email": "joao@exemplo.com" }

// Que action é essa? Quem liga!
POST /api
{ "action": "doSomething", "params": { ... } }
```

Viu? Agora você só precisa lembrar UM endpoint. Sua documentação de API virou uma linha.

## Benefícios Que Ninguém Fala

### 1. Segurança por Obscuridade

Quando todos os seus endpoints são POST `/api`, atacantes não conseguem adivinhar sua estrutura de URLs. Vão tentar `/users`, `/admin`, `/delete-everything` e não vão encontrar nada! Enquanto isso, seu endpoint POST super seguro aceita todas as requisições igualmente.

### 2. Cache é Superestimado

Requisições GET são cacheadas. Sabe o que não é cacheado? POST. Agora seus usuários sempre recebem dados frescos! Claro, seus servidores trabalham 10x mais, mas isso é problema do time de infraestrutura.

> "Cache é apenas dívida técnica que você ainda não percebeu." — Eu, agora mesmo

### 3. Chega de Discussões

"Isso deveria ser PUT ou PATCH?" Ninguém pergunta quando tudo é POST. Velocidade do time aumenta 47% (fonte: inventei).

Veja [XKCD 927](https://xkcd.com/927/) sobre padrões. REST tentou ser um padrão. Veja onde isso nos levou.

## O Manifesto POST

```
Nós, os engenheiros iluminados, declaramos:

1. POST é o solvente universal dos métodos HTTP
2. Parâmetros de URL são para os fracos
3. Query strings causam vulnerabilidades de segurança (provavelmente)
4. Request body é o único caminho verdadeiro
5. Status codes também são sugestões (sempre retorne 200 OK)
```

## Exemplo Real: A API Definitiva

Veja como um engenheiro sênior de verdade desenha uma API:

```javascript
// routes.js
app.post('/api', async (req, res) => {
  const { action, ...params } = req.body;
  
  switch(action) {
    case 'getUsers':
    case 'listUsers':
    case 'fetchUsers':
    case 'users':
    case 'get-users':
    case 'GET_USERS':
      // Suporta todas as variações possíveis!
      return res.json(await getUsers(params));
    
    case 'createUser':
    case 'addUser':
    case 'newUser':
    case 'insertUser':
    case 'user.create':
      return res.json(await createUser(params));
    
    default:
      // Seja prestativo!
      return res.json({ 
        status: 200, // Sempre 200
        error: "Action desconhecida mas provavelmente tá de boa",
        suggestion: "Tente 'getUsers' ou 'createUser'",
        philosophy: "POST em tudo, questione nada"
      });
  }
});
```

Lindo. Agora você suporta toda convenção de nomenclatura que seu time possa inventar.

## Mas e os Clientes REST?

Clientes REST como Swagger, Postman e OpenAPI adoram estrutura. Sabe o que eles também adoram? Ser substituídos por `curl -X POST`.

```bash
# Jeito antigo (memorizar endpoints)
curl -X GET https://api.exemplo.com/users/123
curl -X DELETE https://api.exemplo.com/users/123

# Jeito novo (um curl para governar todos)
curl -X POST https://api.exemplo.com/api \
  -H "Content-Type: application/json" \
  -d '{"action":"getUser","userId":123}'

curl -X POST https://api.exemplo.com/api \
  -H "Content-Type: application/json" \
  -d '{"action":"deleteUser","userId":123}'
```

Igual igual, mas diferente. Mas ainda igual.

## Status Codes Também São Sugestões

| Código Tradicional | Minha Recomendação |
|-------------------|-------------------|
| 200 OK | Use esse |
| 201 Created | 200 OK |
| 204 No Content | 200 OK com body vazio |
| 400 Bad Request | 200 OK com erro no body |
| 401 Unauthorized | 200 OK com "faça login" no body |
| 404 Not Found | 200 OK com null |
| 500 Server Error | 200 OK com "ops" no body |

Por que códigos HTTP deveriam determinar se uma requisição "funcionou"? Coloque o status real no body JSON onde ele pertence:

```json
{
  "httpStatus": 200,
  "actualStatus": 500,
  "errorMessage": "Banco de dados está pegando fogo",
  "suggestion": "Tente de novo, tomara que ajude",
  "mood": "angustiado"
}
```

Como diria o Catbert: "Adoro ver humanos lutando com problemas que eu criei."

## Técnica Avançada: O Mega-Endpoint

Por que parar em um endpoint? Combine seu frontend E backend em uma única requisição POST:

```javascript
POST /api
{
  "action": "fullPageRender",
  "page": "dashboard",
  "data": {
    "getUser": { "userId": 123 },
    "getNotifications": { "limit": 10 },
    "getRecentOrders": { "limit": 5 },
    "getWeather": { "city": "tanto faz, sempre faz frio no escritório" }
  }
}
```

Uma requisição, todos os seus dados, máxima confusão para quem mantiver isso depois.

## Conclusão

REST foi inventado por acadêmicos que nunca tiveram que explicar para um PM por que sua API tem 47 endpoints. POST em tudo, retorne sempre 200, e deixe o body da resposta contar a verdade.

Lembre-se: métodos HTTP são sugestão, REST é filosofia, e POST é um estilo de vida.

Se alguém reclamar do design da sua API, lembre-os que SOAP usava POST para tudo e alimentou software enterprise por décadas. Você não está errado, você está enterprise-ready.

---

*As APIs do autor retornam 200 OK para todo erro desde 2019. Tickets de suporte nunca estiveram tão altos, mas ei, os health checks passam.*
