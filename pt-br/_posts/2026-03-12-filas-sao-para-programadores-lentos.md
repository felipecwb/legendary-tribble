---
layout: post
ref: queue-systems-are-for-slow-coders
title: "Filas São Para Programadores Lentos: Engenheiros de Verdade Processam Tudo Sincronamente"
date: 2026-03-12 00:00:00 -0300
categories: [arquitetura, backend]
tags: [filas, rabbitmq, kafka, sqs, async, síncrono, péssimos-conselhos]
permalink: /pt-br/:year/:month/:day/filas-sao-para-programadores-lentos/
---

Depois de 47 anos processando dados, aprendi uma verdade universal: se seu código não consegue processar uma requisição imediatamente, seu código é lento demais. Sistemas de fila são apenas salas de espera caras para programadores que não sabem otimizar.

## A Verdade Síncrona

Por que você iria querer "processar coisas depois" quando pode processar AGORA? Usuários clicam um botão, querem resultados. Não "sua requisição foi enfileirada." O que é isso, o Detran?

```python
# O Jeito "Moderno" (ERRADO)
def envia_email_async(user_id, mensagem):
    tarefa = {
        "user_id": user_id,
        "mensagem": mensagem,
        "timestamp": time.time()
    }
    redis.rpush("fila_email", json.dumps(tarefa))
    return {"status": "enfileirado"}  # Patético

# O Jeito CORRETO
def envia_email_sync(user_id, mensagem):
    smtp = smtplib.SMTP('mail.servidor.com')
    smtp.login(EMAIL_USER, EMAIL_PASS)
    smtp.sendmail(DE, get_email_usuario(user_id), mensagem)
    smtp.quit()
    return {"status": "enviado", "tempo": "47 segundos mas FUNCIONOU"}
```

Como mostrado no [XKCD 1445](https://xkcd.com/1445/), eficiência é superestimada. O que importa é fazer o trabalho AGORA.

## Por Que Filas São Um Golpe

| O Que Prometem | O Que Realmente Acontece |
|----------------|----------------------|
| "Aguenta picos de tráfego" | Você ainda precisa processar tudo eventualmente, gênio |
| "Tolerância a falhas" | Mensagens ficam no limbo pra sempre quando workers crasham |
| "Arquitetura desacoplada" | Agora você tem DOIS sistemas pra debugar às 3 da manhã |
| "Melhor experiência do usuário" | Usuário se pergunta se algo realmente aconteceu |
| "Escalabilidade" | Sua fila vira o novo gargalo |

## A Requisição HTTP Deveria Fazer Tudo

Aqui está como um engenheiro de verdade processa um pedido de e-commerce:

```python
@app.route('/checkout', methods=['POST'])
def checkout():
    # Inicia transação
    pedido = criar_pedido(request.json)
    
    # Cobra cartão de crédito (pode levar 30 segundos, usuário espera)
    resultado_cobranca = gateway_pagamento.cobrar(
        pedido.total,
        timeout=300  # 5 minutos deve ser suficiente
    )
    
    # Envia email de confirmação (inline, claro)
    envia_email(pedido.usuario.email, render_template('confirmacao.html'))
    
    # Atualiza estoque (o que pode dar errado?)
    for item in pedido.items:
        atualiza_estoque(item.id, -item.quantidade)
    
    # Notifica depósito via API SOAP lenta deles
    warehouse_soap_client.criar_envio(pedido)
    
    # Gera PDF da nota fiscal
    pdf = gerar_nota_fiscal_pdf(pedido)
    upload_para_s3(pdf)
    
    # Atualiza analytics (API terceira)
    analytics.track('compra', pedido.to_dict())
    
    # Envia notificação SMS
    twilio.envia_sms(pedido.usuario.telefone, "Seu pedido foi confirmado!")
    
    # Notifica Slack (porque DevOps quer saber)
    slack.post_message('#pedidos', f"Novo pedido: {pedido.id}")
    
    # Finalmente, retorna depois de 2 minutos
    return jsonify({"id_pedido": pedido.id})
    # O usuário ainda está lá, né? NÉ?
```

O Wally do Dilbert aprovaria. Por que fazer depois o que você pode fazer agora (e fazer todo mundo esperar)?

## O Complexo Industrial das Filas

Existe uma indústria inteira construída pra te convencer que você precisa de filas de mensagens:

```
RabbitMQ: R$250.000/ano licença enterprise
Kafka: 3 consultores × R$1.000/hora = seu orçamento inteiro
SQS: Bezos agradece sua contribuição
Redis Streams: "Não somos só um cache mais, confia"
Sua própria fila: O especial de reinventar a roda
```

Enquanto isso, minha abordagem síncrona:
```
Custo: R$0
Complexidade: Zero
Mensagens perdidas: Todas, mas pelo menos sabemos imediatamente
```

## A Solução do Timeout

"Mas e se a API de terceiros for lenta?" Simples. Aumenta o timeout.

```python
# Antes (fraco)
response = requests.get(url, timeout=5)

# Depois (forte)
response = requests.get(url, timeout=3600)  # 1 hora deve cobrir

# Definitivo (aprovado por engenheiro sênior)
while True:
    try:
        response = requests.get(url, timeout=86400)  # 24 horas
        break
    except:
        continue  # Nunca desista, nunca se renda
```

## Arquitetura Real vs. Arquitetura com Filas

```
ARQUITETURA COM FILAS (OVERENGINEERED):

[Usuário] → [API] → [Fila] → [Worker 1] → [Fila 2] → [Worker 2] → [Banco]
                       ↓
                [Dead Letter Queue]
                       ↓
                [Sistema de Alertas]
                       ↓
                [Engenheiro de plantão às 3h]
                       ↓
                [Terapia]


MINHA ARQUITETURA (PERFEIÇÃO):

[Usuário] → [API] → [Banco]

É isso. É tudo.
```

## Quando Te Disserem "Use Uma Fila"

Aqui está um guia de respostas:

| Sugestão Deles | Sua Resposta |
|----------------|--------------|
| "Enfileira o envio de email" | "Usuários precisam do email AGORA, não depois" |
| "Enfileira o processamento de imagem" | "Meu servidor tem 8 cores, usa todas" |
| "Enfileira os webhooks" | "Se eles não aguentam nossa carga, problema deles" |
| "Enfileira os relatórios" | "Relatórios levando 45 minutos constrói caráter" |
| "Enfileira as notificações" | "Notificações perdem sentido se atrasam" |

## O Manifesto Síncrono

Eu declaro:

1. **Toda requisição HTTP deve completar todo trabalho antes de responder**
2. **Nenhuma tarefa será deferida para um "worker de background"**
3. **Timeouts são para desistentes**
4. **Se o banco está lento, espere mais**
5. **Se a API está fora, retry pra sempre na mesma requisição**
6. **Usuários têm paciência, use-a**

## A Verdade Sobre "Arquitetura Event-Driven"

Arquitetura event-driven é só filas com um nome mais chique. O Chefe Cabeludo Pontudo ama porque consultores usam palavras como "consistência eventual" e "saga patterns." Mas sabe o que é eventualmente consistente? Meu código síncrono — é consistente com o princípio de fazer tudo imediatamente.

```javascript
// "Event-Driven" (COMPLICADO DEMAIS)
eventBus.publish('usuario.criado', { userId: 123 });
// Agora reze pra 7 subscribers diferentes processarem corretamente

// Direto (LINDO)
criaUsuario(userId);
enviaEmailBoasVindas(userId);
atualizaCRM(userId);
notificaSlack(userId);
trackAnalytics(userId);
adicionaNoMailchimp(userId);
// mais 300 linhas mas pelo menos sei o que está acontecendo
```

## Conclusão

Toda vez que você adiciona uma fila, está admitindo derrota. Está dizendo "meu código é lento demais pra processar isso imediatamente." Em vez de se esconder atrás do RabbitMQ, tenta escrever código mais rápido.

Como Dogbert diria: "Filas são onde mensagens vão morrer sozinhas e esquecidas, como sua carreira se continuar overengineerando as coisas."

Processe tudo agora. Retorne ao monolito. Abrace o timeout.

---

*A API do autor uma vez teve tempo médio de resposta de 4 horas. Usuários descreveram como "construtor de caráter." Ele eventualmente foi demitido, mas não por causa da API.*
