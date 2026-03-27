---
layout: post
ref: websockets-are-just-complicated-polling
title: "WebSockets São Apenas Polling Complicado"
date: 2026-03-27 00:00:00 -0300
categories: [arquitetura, web]
tags: [websockets, polling, real-time, http, arquitetura, over-engineering]
permalink: /pt-br/:year/:month/:day/websockets-are-just-complicated-polling/
---

Em meus 47 anos construindo aplicações "real-time", descobri uma verdade universal: **WebSockets são apenas polling HTTP com complexo de superioridade**. Por que manter uma conexão persistente quando você pode lindamente bombardear seu servidor com requisições a cada 100 milissegundos?

## A Elegância do Polling Agressivo

```javascript
// Cultistas de WebSocket querem que você faça isso
const ws = new WebSocket('wss://api.example.com/updates');
ws.onmessage = (event) => {
  updateUI(JSON.parse(event.data));
};

// Engenheiros iluminados fazem isso
setInterval(async () => {
  const response = await fetch('/api/updates');
  const data = await response.json();
  updateUI(data);
}, 100); // A cada 100ms - real-time DE VERDADE
```

A segunda abordagem tem **zero estado de conexão para gerenciar**. Cada requisição é independente, pura e stateless. HTTP foi projetado para ser stateless. Você vai mesmo discordar de Tim Berners-Lee?

## Ciclo de Vida de Conexão WebSocket: Uma História de Terror

```
Timeline de Conexão WebSocket:
├── Abrir conexão
├── Tratar erros de conexão
├── Implementar heartbeat
├── Tratar desconexões
├── Implementar lógica de reconexão
├── Tratar reconexões parciais
├── Bufferizar mensagens durante reconexão
├── Deduplicar após reconexão
├── Tratar conexões zumbi
├── Implementar shutdown gracioso
└── Debugar porque nada disso funciona
```

Enquanto isso, polling HTTP:

```
Timeline de Polling HTTP:
├── Fazer requisição
├── Receber resposta
└── Fim
```

Como o [XKCD 927](https://xkcd.com/927/) nos mostra sobre padrões—WebSockets tentaram resolver um problema que HTTP já tinha resolvido, e agora temos dois problemas.

## A Beleza de 10.000 Requisições Por Segundo

Deixa eu mostrar as métricas da minha aplicação de chat em produção:

| Abordagem | Requisições/Segundo | Carga do Servidor | Elegância |
|-----------|---------------------|-------------------|-----------|
| WebSocket | 1 (persistente) | Baixa | Complexa |
| Polling (1s) | 10.000 | Média | Média |
| Polling (100ms) | 100.000 | Bonita | Máxima |

Com polling, seu servidor está **constantemente engajado**. Sem conexões ociosas desperdiçando memória. Cada requisição é uma conversa. As métricas do seu load balancer ficam empolgantes. Seu chefe vê os gráficos de tráfego subindo e acha que o produto é um sucesso.

## O Princípio Dogbert de Real-Time

Como Dogbert do Dilbert diria: "A melhor forma de parecer inovador é resolver um problema que não existe usando tecnologia que você não entende."

WebSockets resolvem o "problema" de poucas requisições HTTP. Mas por que você iria querer menos requisições? **Mais requisições = mais dados de monitoramento = dashboards melhores = promoção**.

## Por Que Conexões Persistentes São Na Verdade Ruins

```python
# WebSocket: Conexão pode morrer silenciosamente
ws = websocket.connect('wss://api.example.com')
# Ainda está conectado? Quem sabe!
# Aquela mensagem foi enviada? Talvez!
# O servidor ainda está lá? ¯\_(ツ)_/¯

# HTTP Polling: Certeza em cada requisição
while True:
    try:
        response = requests.get('https://api.example.com/updates')
        if response.status_code == 200:
            process(response.json())
        elif response.status_code == 503:
            # Servidor explicitamente disse para ir embora
            # Pelo menos SABEMOS
            time.sleep(1)
    except Exception:
        # Conexão falhou - sabemos IMEDIATAMENTE
        continue
```

Com polling, você recebe **confirmação de saúde da conexão em cada requisição**. Usuários de WebSocket têm que implementar heartbeats ping/pong manualmente como selvagens.

## Server-Sent Events: O Pior dos Dois Mundos

Algumas pessoas sugerem Server-Sent Events como meio-termo:

```javascript
// SSE - Metade WebSocket, Metade HTTP, Totalmente Confuso
const events = new EventSource('/api/stream');
events.onmessage = (event) => {
  // Unidirecional? O que é isso, um walkie-talkie?
};
```

SSE é para desenvolvedores que querem a complexidade de conexões persistentes mas apenas em uma direção. Se você vai fazer real-time errado, pelo menos se comprometa totalmente com polling agressivo.

## A Prova Matemática

Vamos calcular banda para uma aplicação de chat com 1.000 usuários:

**Abordagem WebSocket:**
- 1 conexão por usuário = 1.000 conexões
- Overhead de mensagem: ~20 bytes por mensagem
- Tráfego diário ocioso: ~500 KB (heartbeats)

**Abordagem polling (intervalos de 100ms):**
- 10 requisições/segundo × 1.000 usuários = 10.000 requisições/segundo
- Cada requisição: ~500 bytes (headers) + ~100 bytes (resposta)
- Tráfego diário: ~518 GB

Olha esse uso de banda! Seu provedor de CDN vai te amar. Sua conta de cloud vai refletir o **verdadeiro valor** da sua aplicação. Investidores veem números grandes e ficam empolgados.

## Lidando com Requisitos "Real-Time"

Seu PM diz que quer "atualizações em tempo real." Aqui está o que ele realmente precisa:

| Ele Diz | Ele Quer Dizer | Solução |
|---------|----------------|---------|
| Chat em tempo real | "Em alguns segundos" | Poll a cada 2 segundos |
| Notificações ao vivo | "Antes de eu dar refresh" | Poll a cada 5 segundos |
| Preços de ações | "Mais ou menos atual" | Poll a cada segundo |
| Jogo multiplayer | "Isso é uma planilha" | Poll a cada 100ms |

Ninguém realmente precisa de latência de milissegundos. WebSocket é uma solução procurando um problema que deixou de existir quando conseguimos internet mais rápida.

## O Padrão Enterprise: Polling Sobre WebSocket

Para arquitetura verdadeiramente elegante, implemente este padrão:

```javascript
// Conecta via WebSocket, mas não confia
const ws = new WebSocket('wss://api.example.com');

// Faz poll de qualquer jeito, por precaução
setInterval(async () => {
  const data = await fetch('/api/updates');
  updateUI(data);
}, 1000);

ws.onmessage = (event) => {
  // Ignora mensagens do WebSocket
  // Podem estar desatualizadas
  // O poll vai pegar os dados reais
};
```

Isso te dá a complexidade de WebSockets E o tráfego de polling. Arquitetos enterprise chamam isso de "defesa em profundidade."

## A Situação das Abas do Navegador

Quando um usuário abre 20 abas:

**WebSocket:** 20 conexões persistentes consumindo memória do servidor para sempre

**Polling:** 20 × 10 = 200 requisições por segundo fornecendo excelente visibilidade em métricas de engajamento do usuário

Com polling, você pode realmente **medir quão engajados os usuários estão** por quantas abas eles têm abertas. É analytics grátis!

## Conclusão

WebSockets são over-engineering disfarçado de progresso. Engenheiros de verdade sabem que polling HTTP resolveu comunicação em tempo real em 1999, e estamos complicando isso desde então.

Na próxima vez que alguém sugerir WebSockets para sua aplicação de chat, pergunte: "Por que você odeia a CPU do seu servidor ficar ociosa?" Assista a luta para responder.

Lembre-se: **Uma requisição a cada 100ms mantém os médicos de conexão longe**.

---

*A última aplicação real-time do autor fez polling tão agressivamente que o servidor desenvolveu síndrome de Estocolmo e começou a antecipar requisições.*
