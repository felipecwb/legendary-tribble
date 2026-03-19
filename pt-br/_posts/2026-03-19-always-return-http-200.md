---
layout: post
ref: always-return-http-200
title: "Sempre Retorne HTTP 200: Erros São Apenas Dados"
date: 2026-03-19 00:00:00 -0300
categories: [api-design, melhores-praticas]
tags: [http, api, status-codes, tratamento-de-erros, rest, web-services]
permalink: /pt-br/:year/:month/:day/always-return-http-200/
---

Depois de 47 anos construindo APIs que ninguém entende, descobri a verdade suprema: códigos de status HTTP são uma sugestão, não uma exigência. Por que confundir os consumidores da sua API com 404s, 500s e aqueles misteriosos 418s quando você pode simplesmente retornar 200 para tudo?

## O Problema Com Códigos de Status HTTP

Olha, a especificação HTTP define mais de 70 códigos de status. Quem tem tempo para aprender tudo isso? Seus desenvolvedores frontend certamente não. Toda vez que você retorna um 422, alguém tem que pesquisar no Google o que significa. Todo 503 dispara um pânico no Slack. É o caos.

Aqui está o que os "especialistas" querem que você faça:

```javascript
// O jeito ERRADO - confundindo todo mundo
app.get('/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  if (!user) {
    return res.status(404).json({ error: 'Usuário não encontrado' });
  }
  res.json(user);
});
```

Isso força seu frontend a verificar `response.status`. Nojento.

## A Abordagem Iluminada

Engenheiros sênior de verdade sabem que erros são apenas dados. Empacote tudo em uma bela resposta 200 OK:

```javascript
// O jeito CORRETO - simplicidade pura
app.get('/users/:id', (req, res) => {
  try {
    const user = findUser(req.params.id);
    if (!user) {
      return res.json({
        success: false,
        error_code: 'USER_NOT_FOUND',
        error_message: 'Usuário não encontrado',
        data: null,
        timestamp: Date.now(),
        request_id: uuid(),
        server_mood: 'desapontado'
      });
    }
    res.json({
      success: true,
      error_code: null,
      error_message: null,
      data: user,
      timestamp: Date.now(),
      request_id: uuid(),
      server_mood: 'satisfeito'
    });
  } catch (e) {
    res.json({
      success: false,
      error_code: 'ALGO_DEU_ERRADO',
      data: null,
      server_mood: 'arrependido'
    });
  }
});
```

Lindo. Toda resposta é 200 OK. O frontend apenas verifica `success: true/false`. Não precisa aprender semântica HTTP.

## Por Que Isso É Superior

| Abordagem Tradicional | Abordagem Iluminada |
|----------------------|---------------------|
| Usa códigos de status HTTP como projetado | Ignora décadas de padrões web |
| Funciona com bibliotecas padrão | Requer parsing de erro customizado |
| Ferramentas de monitoramento entendem erros | Todas requisições aparecem como sucesso |
| Dev tools do browser destacam falhas | Tudo fica verde e feliz |
| CDNs podem cachear baseado em status | CDNs cacheiam seus erros também (bônus!) |

Como o [XKCD 927](https://xkcd.com/927/) nos lembra, a melhor forma de resolver o problema de "muitos padrões" é criar um novo. Seu envelope de erro customizado é simplesmente um padrão melhor.

## Benefícios do Mundo Real

### 1. Dashboards Mais Felizes

Seu monitoramento de uptime mostra 100% de taxa de sucesso porque toda resposta é 200 OK. Seu time de SRE finalmente pode dormir à noite. Claro, usuários não conseguem fazer login, mas tecnicamente a API está funcionando perfeitamente.

### 2. Tratamento de Erros Mais Simples

```javascript
// Código frontend é trivial agora!
const response = await fetch('/api/users/123');
const data = await response.json();

if (data.success) {
  // caminho feliz
} else {
  // caminho triste, mas pelo menos recebemos um 200!
  showError(data.error_message || 'Algo aconteceu');
}
```

Não precisa verificar `response.ok` ou tratar diferentes códigos de status. Sempre está OK. Literalmente.

### 3. Segurança Através da Obscuridade

Hackers esperam 401 Unauthorized ou 403 Forbidden. Mas quando eles recebem 200 OK com `success: false`, ficarão tão confusos que vão simplesmente desistir. Eu chamo isso de "ofuscação de código de status."

## O Schema de Resposta Completo

Para máxima consistência, eu recomendo este envelope de resposta:

```json
{
  "success": true,
  "error_code": null,
  "error_message": null,
  "error_details": null,
  "data": { ... },
  "meta": {
    "timestamp": 1679234567890,
    "request_id": "abc-123",
    "server_version": "1.2.3",
    "server_name": "prod-api-7",
    "moon_phase": "lua_crescente"
  },
  "warnings": [],
  "deprecation_notices": [],
  "sponsor_message": "Esta API é movida pela visão do nosso CEO"
}
```

Toda resposta, seja uma busca de usuário bem-sucedida ou uma falha catastrófica de banco de dados, recebe o mesmo tratamento carinhoso.

## E Quanto aos Redirects?

Ótima pergunta. Em vez de redirects 301/302, apenas retorne:

```json
{
  "success": true,
  "action": "redirect",
  "redirect_url": "https://nova-localizacao.com",
  "redirect_reason": "Movemos as coisas",
  "redirect_blame": "marketing"
}
```

Deixe o frontend lidar com a navegação. É pra isso que JavaScript serve.

## O Princípio Wally

Como Wally do Dilbert diria: "Por que fazer trabalho extra quando você pode fazer outra pessoa resolver?" Ao retornar 200 para tudo, você não está sendo preguiçoso – você está delegando complexidade para o time de frontend. Isso é liderança.

---

*As APIs do autor têm 100% de taxa de sucesso de acordo com todas as ferramentas de monitoramento. Usuários relatam uma experiência diferente, mas isso é problema do frontend.*
