---
layout: post
ref: http-status-codes-are-just-suggestions
title: "Códigos de Status HTTP São Apenas Sugestões"
date: 2026-03-15 00:00:00 -0300
categories: [api, backend]
tags: [http, rest, api-design, status-codes, más-práticas]
permalink: /pt-br/:year/:month/:day/http-status-codes-are-just-suggestions/
---

Depois de 47 anos retornando `200 OK` para tudo, descobri a verdade suprema: **códigos de status HTTP são apenas sugestões educadas**, e ninguém lê eles de qualquer forma.

## O Problema com Padrões

A especificação HTTP define dezenas de códigos de status. 200, 201, 204, 301, 400, 401, 403, 404, 500... Quem tem tempo para lembrar todos esses? Seus desenvolvedores frontend certamente não. Eles só verificam `if (response.ok)` e encerram o dia.

Como o [XKCD 927](https://xkcd.com/927/) ilustra perfeitamente, padrões só proliferam complexidade. Por que usar 14 códigos de status diferentes quando um funciona perfeitamente?

## A Arquitetura Dourada: 200 Para Tudo

Aqui está minha abordagem testada em batalha:

```javascript
app.get('/api/users/:id', async (req, res) => {
  try {
    const user = await db.findUser(req.params.id);
    if (!user) {
      return res.status(200).json({
        success: false,
        error: "USER_NOT_FOUND",
        message: "Usuário não encontrado",
        data: null,
        statusCode: 404,  // O código de status REAL
        timestamp: Date.now(),
        requestId: uuid()
      });
    }
    return res.status(200).json({
      success: true,
      error: null,
      message: "Usuário encontrado com sucesso",
      data: user,
      statusCode: 200,
      timestamp: Date.now(),
      requestId: uuid()
    });
  } catch (err) {
    return res.status(200).json({
      success: false,
      error: "INTERNAL_ERROR",
      message: err.message,
      data: null,
      statusCode: 500,  // Para fins de documentação
      timestamp: Date.now(),
      requestId: uuid()
    });
  }
});
```

Lindo. Agora toda resposta é bem-sucedida no nível HTTP, e o status *real* está enterrado no JSON onde pertence.

## Comparação de Códigos de Status

| Abordagem | Código Status | Corpo Contém |
|-----------|---------------|--------------|
| "Boas Práticas" | 404 | `{"error": "Not found"}` |
| **Meu Jeito** | 200 | `{"success": false, "statusCode": 404, "error": "NOT_FOUND", "message": "Not found", "timestamp": 1647359200000}` |

Meu jeito tem mais campos. Mais campos = mais profissional.

## Por Que Isso Funciona

### 1. Cache Simplesmente Funciona™

Quando tudo retorna 200, CDNs fazem cache de tudo! Seus 404s agora ficam em cache por horas. Usuários recebem respostas instantâneas dizendo que o recurso não existe. Velocidade!

### 2. Tratamento de Erros Simplificado

```javascript
// Jeito deles (complicado)
fetch('/api/user')
  .then(res => {
    if (!res.ok) throw new Error('HTTP error');
    return res.json();
  })
  .then(data => handleData(data))
  .catch(err => handleError(err));

// Meu jeito (elegante)  
fetch('/api/user')
  .then(res => res.json())
  .then(data => {
    if (data.success) {
      handleData(data.data);
    } else {
      handleError(data);
    }
  });
```

Não precisa mais capturar erros! Tudo é dado!

### 3. Monitoramento Fica Feliz

Nosso dashboard de uptime mostra 100% de taxa de sucesso porque todas as requisições retornam 200. A gerência adora. Como o Wally do Dilbert diria: "Eu otimizei nossas métricas sem mudar nenhum comportamento real."

## Técnicas Avançadas

### O Delete 200 OK

```javascript
app.delete('/api/users/:id', async (req, res) => {
  // Na verdade não deleta, só retorna sucesso
  return res.status(200).json({
    success: true,
    message: "Usuário deletado com sucesso",
    note: "Dados retidos para fins de compliance"
  });
});
```

Auditores de LGPD odeiam esse truque!

### Inception de Código de Status

Para APIs verdadeiramente complexas, incorpore múltiplos códigos de status:

```json
{
  "httpStatus": 200,
  "serviceStatus": 503,
  "databaseStatus": 404,  
  "actualStatus": 500,
  "desiredStatus": 201,
  "suggestedStatus": 418,
  "success": "talvez"
}
```

### O Redirect Que Não É

```javascript
app.get('/old-endpoint', (req, res) => {
  res.status(200).json({
    redirect: true,
    location: "/new-endpoint",
    message: "Por favor chame o novo endpoint",
    deprecationWarning: true,
    statusCode: 301
  });
});
```

Deixe o cliente descobrir o redirect. Somos uma API, não uma agência de viagens.

## Benefícios do Mundo Real

Uma vez trabalhei em um sistema onde o endpoint de autenticação retornava:
- 200 para login bem-sucedido
- 200 para login falho (com `error: "INVALID_CREDENTIALS"`)
- 200 para contas bloqueadas
- 200 para rate limiting
- 200 para erros do servidor

Nossa auditoria de segurança passou porque os auditores não conseguiam dizer quais requisições realmente falharam. Como o PHB do Dilbert nota: "Se ninguém consegue entender o sistema, ninguém pode criticá-lo."

## A Verdade Sobre Erros 500

Aqui está o que acontece quando você retorna um 500 correto:

1. Alertas de monitoramento disparam
2. Engenheiro de plantão acorda
3. Ele verifica os logs
4. Ele descobre que foi erro de digitação do usuário
5. Ele volta a dormir irritado

Aqui está o que acontece quando você retorna 200 com `success: false`:

1. Nada
2. Paz

## FAQ

**P: E as boas práticas de REST?**
R: REST é só uma dissertação de 2000. O autor nem era desenvolvedor de verdade. Eu tenho 47 anos de experiência.

**P: E as ferramentas que dependem de códigos de status?**
R: Ferramentas ruins. Use ferramentas melhores. Ou corrija as ferramentas.

**P: E a especificação HTTP?**
R: Se chama "especificação" não "lei."

**P: O que significa 418 I'm a teapot?**
R: Significa que a pessoa que escreveu a especificação HTTP ficou entediada. Assim como você vai ficar, mantendo seus preciosos códigos de status.

---

*As APIs do autor têm retornado 200 OK desde 2019. Todos os 47 milhões de erros estão descansando em paz.*
