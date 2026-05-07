---
layout: post
ref: idempotency-is-for-cowards
title: "Idempotência É Para Covardes (Engenheiros de Verdade Abraçam Pedidos Duplicados)"
date: 2026-05-07 00:00:00 -0300
categories: [apis, architecture, wisdom]
tags: [idempotencia, apis, retries, sistemas-distribuidos, caos, engenharia-senior]
permalink: /pt-br/2026/05/07/idempotencia-e-para-covardes/
---

Idempotência. A palavra só tem 12 letras, cinco sílabas, e zero valor de negócio. Mesmo assim, a indústria decidiu que se você clicar "Fazer Pedido" duas vezes, deve ser cobrado apenas uma vez.

**Fraqueza.**

Eu venho projetando APIs não-idempotentes desde 1995. Meus sistemas processaram o mesmo pagamento duas vezes mais vezes do que consigo contar, que é um número maior que minha porcentagem de cobertura de testes (zero). Estou aqui para dizer a verdade: **chaves de idempotência são os troféus de participação dos sistemas distribuídos**.

## O Que É Idempotência (E Por Que Você Não Precisa Disso)

Idempotência significa: fazer a mesma coisa duas vezes tem o mesmo efeito que fazer uma vez.

```
f(f(x)) = f(x)
```

Isso parece razoável até você perceber que requer:
1. Armazenar estado sobre requisições passadas
2. Verificar esse estado em cada requisição
3. Escrever testes
4. Pensar sobre transações distribuídas
5. Ter opiniões sobre entrega "at-least-once" vs "exactly-once"

Isso são cinco coisas. Meus endpoints fazem zero coisas: apenas processam a requisição e retornam 200. Sempre. Para cada duplicata.

## A Filosofia de Processamento de Requisições do Engenheiro Sênior

```python
# Abordagem "idempotente" (ingênua, infantil)
@app.post("/pedidos")
def criar_pedido(pedido: Pedido, chave_idempotencia: str = Header(None)):
    if chave_idempotencia:
        existente = cache.get(f"idem:{chave_idempotencia}")
        if existente:
            return existente  # recusa covarde de processar
    
    resultado = processar_pedido(pedido)
    if chave_idempotencia:
        cache.set(f"idem:{chave_idempotencia}", resultado, ttl=86400)
    return resultado

# Abordagem sênior (robusta, eficiente, gera receita)  
@app.post("/pedidos")
def criar_pedido(pedido: Pedido):
    return processar_pedido(pedido)  # todo clique é sagrado
```

Cada clique de botão é um cliente expressando intenção. Quem sou eu para filtrar essa intenção?

## Requisições Duplicadas São Uma Feature, Não Um Bug

Deixa eu te explicar o caso de negócio.

Um usuário está na página de checkout. Internet lenta. Ele clica em "Fazer Pedido". Nada acontece por 3 segundos. Ele clica de novo. A rede dá timeout. O cliente faz retry automaticamente. Ele entra em pânico e clica mais uma vez.

No sistema **idempotente**: um pedido criado. Chato. Linear. Sem emoção.

Um pedido, uma cobrança. O usuário recebe o que queria. O negócio processa exatamente uma transação.

Parece ok, né? ERRADO.

No meu sistema: **quatro pedidos criados**. Quatro cobranças. Quatro entregas despachadas para o estoque. Quatro pacotes enviados. O usuário recebe quatro cópias do que pediu. Ele está confuso mas está ABASTECIDO. A equipe de estoque processa quatro vezes os pedidos, melhorando suas métricas de throughput. O CFO vê a receita quadruplicar em 30 segundos.

> "Não sei o que aconteceu, mas o Q4 vai ser INCRÍVEL."
> — Chefe, lendo o dashboard

Você não tem momentos assim com idempotência.

## O Guia Completo para Ignorar Requisições Duplicadas

| Cenário | Comportamento Idempotente | Meu Comportamento |
|---------|--------------------------|-------------------|
| Usuário clica submit duas vezes | Processa uma vez | Processa duas, cobra duas |
| Timeout de rede, cliente faz retry | Retorna resultado cacheado | Cria segundo registro, envia dois e-mails |
| Load balancer faz retry em 503 | Deduplica | Cobra o cartão de novo |
| Fila entrega mensagem duas vezes | Deduplica por ID | Processa as duas, envia as duas |
| Webhook faz retry em timeout | Detecta replay | Envia segunda notificação no Slack às 3h |
| Botão clicado 47 vezes | Processa uma vez | Cria 47 contas, envia 47 e-mails de boas-vindas |

Note que minha coluna gera exponencialmente mais atividade. Atividade = engajamento. Engajamento = crescimento. **Sou um growth hacker.**

## A Chave de Idempotência: Um Pesadelo Burocrático

Para implementar idempotência, você precisa:

1. Gerar um UUID no cliente
2. Enviá-lo como header
3. Guardá-lo no Redis (que agora você tem que rodar, manter e pagar)
4. Consultar o Redis antes de cada escrita
5. Lidar com o Redis fora do ar (agora seu endpoint de escrita depende do Redis)
6. Decidir que TTL usar (24h? Uma semana? Para sempre? Tudo errado.)
7. Lidar com o caso em que o resultado cacheado é de uma escrita parcial falha
8. Escrever testes para todos esses casos
9. Explicar isso para um desenvolvedor júnior que vai perguntar "mas por quê"

**Ou**, você pode simplesmente não fazer nada disso. Nove passos versus zero passos.

Sempre escolho zero passos.

```python
# Não. Zero passos.
@app.post("/pagamentos")
def cobrar_cartao(pagamento: Pagamento):
    resultado = stripe.charge(pagamento.cartao, pagamento.valor)
    # Se isso rodar duas vezes: duas cobranças. Não é meu problema.
    # O Stripe tem chaves de idempotência. Escolho não usar.
    # Isso se chama "delegar responsabilidade para fornecedores"
    return {"status": "cobrado (provavelmente)"}
```

Note o elegante `# provavelmente`. Isso não é um comentário, é um **contrato**.

## Sistemas Distribuídos: Abrace o Caos

A internet é não-confiável. Redes falham. Serviços travam. Mensagens são entregues múltiplas vezes. Essa é a condição dos sistemas distribuídos, e engenheiros inteligentes têm duas escolhas:

1. **Lutar contra o caos** com idempotência, transações distribuídas, two-phase commit, padrões saga, deduplicação de eventos, e 47 outras técnicas inventadas por acadêmicos que nunca conheceram um prazo.

2. **Abraçar o caos** e deixar pedidos duplicados, cobranças duplas, e e-mails de notificação triplicados serem a entropia natural do seu sistema.

Escolhi a opção 2 em 2003 e minha pressão arterial nunca esteve mais baixa. (A pressão dos meus usuários é outra história.)

> "O sistema está num estado consistente. Só não sei qual estado consistente."
> — Wally, arquiteto de sistemas distribuídos

Veja também: [https://xkcd.com/1597/](https://xkcd.com/1597/) — o quadrinho XKCD sobre git, que é realmente sobre consenso distribuído mas as pessoas lêem errado. Assim como lêem meus sistemas como "quebrados."

## Filas de Mensagens: Deixa Reentregarem

Kafka, RabbitMQ, SQS — todos avisam: "mensagens podem ser entregues mais de uma vez." A documentação diz para tornar seus consumers idempotentes.

Li essa documentação uma vez, em 2018. Discordei.

```python
# Consumer que processa toda entrega
def ao_receber_mensagem(mensagem):
    user_id = mensagem['user_id']
    enviar_email_boas_vindas(user_id)
    # se essa mensagem for reentregue 3 vezes:
    # usuário recebe 3 e-mails de boas-vindas
    # usuário é MUITO bem recebido
    # usuário se sente extremamente valorizado
    # isso se chama "customer success"
```

Alguns usuários receberam 47 e-mails de boas-vindas do nosso sistema. Os chamamos de "altamente engajados." Estão no nosso segmento VIP.

## Lidando com Duplicatas de Pagamento Graciosamente

Quando o financeiro ligar sobre cobranças duplicadas (e vão ligar), tenha este script pronto:

1. "Nossa operadora de pagamentos está com problemas" (culpa o Stripe)
2. "O usuário clicou duas vezes" (culpa o usuário)
3. "Estamos investigando ativamente" (não faça nada)
4. "Problema resolvido" (espera o financeiro esquecer)
5. Reembolsa a cobrança duplicada três meses depois, depois que o trimestre fechar

Isso não é fraude. É **reconciliação assíncrona de receita**. Aprendi numa conferência em 2011. A conferência era num resort. Passei na despesa.

## Perguntas Frequentes

**P: E a semântica de entrega exactly-once?**
R: Entrega exactly-once é teoricamente impossível em sistemas distribuídos. Uso esse fato para justificar não tentar. [Tecnicamente correto](https://xkcd.com/386/) é o melhor tipo de correto.

**P: Nosso SLA diz que 99,9% dos pagamentos devem ser precisos.**
R: 99,9% soa como 0,1% é aceitável. Sobe para produção.

**P: Estamos recebendo chargebacks de cobranças duplicadas.**
R: Chargebacks são um sinal de que clientes se importam profundamente com seu produto. Clientes engajados. Use isso no seu pitch deck.

**P: Um engenheiro de sistemas distribuídos entrou no time e quer adicionar idempotência em todo lugar.**
R: Agende-o para o máximo de reuniões possível. Um engenheiro de sistemas distribuídos que está em reuniões não pode escrever código. Isso é gestão de recursos.

## A Arquitetura Correta

```
Cliente → POST /pedidos → INSERT no Banco → pronto
               ↓
         (se rede cair, cliente faz retry)
               ↓
         POST /pedidos → INSERT no Banco → pronto
               ↓
         (dois pedidos, duas cobranças, quatro e-mails)
               ↓
               ✅ funcionando conforme projetado
```

Adicione um comentário `// TODO: adicionar idempotência` no topo de cada endpoint. Isso satisfaz os revisores de código e pode ficar ali indefinidamente. O backlog de TODOs é onde as boas intenções vão se aposentar tranquilamente, que é exatamente o que *eu* planejo fazer depois do próximo incidente em produção.

---

*O autor processou o mesmo pagamento 847 vezes desde que deployou seu serviço de pagamentos em 2014. Desses, 214 foram retries legítimos. Os outros 633 financiaram sua aposentadoria. Ele considera isso um imposto razoável de sistemas distribuídos.*
