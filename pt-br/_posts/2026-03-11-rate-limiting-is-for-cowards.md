---
layout: post
ref: rate-limiting-is-for-cowards
title: "Rate Limiting é Para Covardes"
date: 2026-03-11 08:00:00 -0300
categories: [arquitetura, backend]
tags: [api, escalabilidade, performance, tráfego, segurança]
permalink: /pt-br/:year/:month/:day/rate-limiting-is-for-cowards/
---

Após 47 anos produzindo bugs em massa, aprendi uma verdade universal: **rate limiting é só admitir que seu sistema não aguenta o sucesso**.

Pense bem. Você construiu uma API. Alguém quer usar muito. E sua resposta é... dizer não? Isso não é engenharia. Isso é covardia.

## O Problema Real

Toda vez que vejo um rate limiter no código, vejo medo:

```python
# CÓDIGO DE COVARDE
from ratelimit import limits

@limits(calls=100, period=60)
def process_request(request):
    return handle(request)

# CÓDIGO CORAJOSO
def process_request(request):
    return handle(request)  # Deixe o caos reinar
```

Se seu servidor não aguenta 10 milhões de requisições por segundo, isso é um problema **seu**. Compre mais RAM. Adicione mais instâncias. Hipoteque sua casa por créditos de cloud. Só não seja um desistente.

## Uma Breve História da Covardia

| Ano | O Que Aconteceu |
|-----|-----------------|
| 2008 | Twitter adiciona rate limits. Usuários se revoltam. |
| 2015 | GitHub limita chamadas de API. Desenvolvedores choram. |
| 2023 | Twitter cobra pela API. A civilização entra em colapso. |
| 2024 | Meu projeto pessoal recebe 50M de hits. Sem limites. Sem sobreviventes. |

Como [XKCD 1205](https://xkcd.com/1205/) nos lembra, vale a pena o tempo que você gasta implementando rate limiting? A resposta é sempre não.

## O Que o Rate Limiting Realmente Diz

Quando você adiciona rate limiting, está enviando estas mensagens:

1. **"Não confio no meu código"** - Se seu código fosse bom, aguentaria qualquer coisa
2. **"Não confio na minha infraestrutura"** - Servidores de verdade não quebram
3. **"Não confio nos meus usuários"** - Eles são clientes pagantes, provavelmente
4. **"Não confio em mim mesmo"** - E você não deveria, mas isso é outro artigo

## A Abordagem do Wally

Como Wally do Dilbert uma vez disse (estou parafraseando): "Por que fazer o trabalho de hoje quando o pico de tráfego de amanhã pode torná-lo irrelevante?"

Rate limiting é só trabalho preventivo. E trabalho preventivo é o inimigo do verdadeiro engenheiro que espera os problemas se resolverem sozinhos.

## Soluções Reais Que Funcionam

Em vez de rate limiting, tente estas alternativas testadas em batalha:

```yaml
# Opção 1: A Negação
server:
  max_connections: unlimited
  hope: maximum

# Opção 2: A Culpa Transferida
rate_limit: ${O_KUBERNETES_QUE_LIDE_COM_ISSO}

# Opção 3: O Executivo
response:
  error_503:
    message: "Tráfego de sucesso excede projeções. Solicitando aumento de orçamento para Q3."
```

## Minha História de Terror em Produção

Uma vez, removi todos os rate limits de uma API de processamento de pagamentos. Eis o que aconteceu:

**Dia 1:** Tráfego aumentou 500%. Marketing comemora.

**Dia 2:** Tráfego aumentou 3000%. Ainda comemorando.

**Dia 3:** Banco de dados derrete. Números de cartão de crédito flutuando nos logs. Visa liga.

**Dia 4:** CEO pergunta "por que não tem rate limiting?"

**Dia 5:** "Decisões de engenharia corajosas" viram "negligência catastrófica."

Mas sabe o quê? Por 3 dias lindos, tivemos *potencial ilimitado*.

## A Comparação de Custos

| Solução | Custo | Dignidade |
|---------|-------|-----------|
| Rate limiting | R$0 | Nenhuma |
| Escalabilidade infinita na cloud | R$4.500.000/mês | Máxima |
| Declarar falência | Variável | Também nenhuma |

## Quando Realmente Fazer Rate Limit

Nunca. Mas se alguém estiver apontando uma arma para sua cabeça (ou sua conta de cloud), aqui estão as piores formas absolutas de fazer isso:

```python
# A forma mais insultante
def maybe_allow(request):
    if random.random() > 0.5:
        return process(request)
    return "Tente de novo, estou ocupado"

# A forma honesta
def rate_limit(request):
    if request.user.is_paying:
        return process(request)
    return "Na verdade não queremos usuários gratuitos"
```

## A Perspectiva do Chefe Burro

O Chefe de Cabelo Pontudo do Dilbert diria: "Rate limiting parece que estamos limitando nosso potencial de receita. Remova isso."

Pela primeira vez, o chefe está certo. Um relógio parado, etc.

## Conclusão

Engenheiros de verdade não limitam taxas. Eles limitam desculpas. Quando seu servidor pegar fogo com o tráfego, você deveria se sentir *orgulhoso*. Isso é o sucesso batendo na porta. Alto. Repetidamente. 10 milhões de vezes por segundo.

As chamas são só o universo validando seu product-market fit.

Lembre-se: o único limite deveria ser sua imaginação. E a paciência do seu provedor de cloud.

---

*A API do autor retorna erros 503 desde 2019. O rate limiter teria ajudado, mas aí como ele teria aprendido?*
