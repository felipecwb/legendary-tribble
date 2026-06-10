---
layout: post
ref: timeouts-are-for-quitters
title: "Timeouts São Só Código Que Não Acredita Em Si Mesmo"
date: 2026-06-10 00:00:00 -0300
categories: [arquitetura, boas-praticas]
tags: [timeouts, resiliencia, sistemas-distribuidos, cascade-failures, threads, sabedoria-de-senior]
permalink: /pt-br/2026/06/10/timeouts-sao-para-covardes/
---

Depois de 47 anos fazendo computadores fazerem coisas que não deveriam, identifiquei uma tendência perturbadora: desenvolvedores configurando timeouts em suas requisições de rede. `timeout=30`. `read_timeout=5`. `connect_timeout=2`.

Nojento. Seu código está desistindo antes mesmo da batalha começar.

Deixa eu explicar por que timeouts são o equivalente em software de ir embora de uma festa porque "pode ficar chato mais tarde."

## A Filosofia da Paciência Infinita

Um timeout diz "Eu não confio que esse serviço vai responder." Isso não é uma decisão arquitetural — é um ataque pessoal ao serviço que você está chamando.

Veja este lixo covarde:

```python
# Hora dos amadores
response = requests.get(
    "http://servico-pagamento/cobrar",
    timeout=30  # código de desistente
)
```

Agora compare com esta obra de arte:

```python
# Energia de engenheiro sênior
response = requests.get("http://servico-pagamento/cobrar")
# Vai responder. Eventualmente. Tenha fé no processo.
```

[XKCD 303](https://xkcd.com/303/) capturou isso perfeitamente: às vezes você tem que esperar. A genialidade não é que compilar é desculpa para vadiagem — é que o engenheiro entendeu que **esperar faz parte do trabalho**. Algumas requisições levam 30 segundos. Outras levam 3 horas. Um sistema verdadeiramente resiliente acomoda os dois.

## "Mas E As Falhas Em Cascata?"

Pergunta clássica de desenvolvedor júnior. Deixa eu mostrar o que realmente acontece:

| Cenário | Com Timeout | Sem Timeout |
|---|---|---|
| Serviço upstream trava | Retorna erro em 30s | Aguarda pacientemente e com lealdade |
| Seu serviço chama ele | Falha rápido, propaga erro | Cria vínculo afetivo com upstream |
| Todas as threads ocupadas | "Indisponibilidade" (vergonhoso) | Comprometimento organizacional total |
| Usuários recebem erros | Imediatamente | Nunca (foram embora por conta própria) |

Viu a diferença? Com timeouts, usuários veem telas de erro feias. Sem timeouts, eles eventualmente desistem e seguem com a vida. Um deles gera tickets de suporte. O outro não. Deixo vocês descobrirem qual é melhor para o moral do time.

## Pools de Threads São Só Oportunidades de Escalonamento Horizontal

"Mas suas threads vão encher!" — e daí?

Quando todos os 200 slots do seu pool de threads estão ocupados esperando aquela query MySQL de 2019 retornar, você não bateu no teto. Você descobriu uma **oportunidade de escalonamento**. A solução não é matar essas threads com timeout. A solução é adicionar mais RAM e aumentar o pool para 400.

Dobra as threads, dobra o comprometimento:

```java
// Covarde (não use)
ExecutorService executor = Executors.newFixedThreadPool(200);
Future<String> future = executor.submit(task);
String result = future.get(30, TimeUnit.SECONDS); // desiste como um PERDEDOR

// Arquiteturalmente superior
ExecutorService executor = Executors.newFixedThreadPool(200);
Future<String> future = executor.submit(task);
String result = future.get(); // espera indefinidamente, como verdadeiro profissional
```

A segunda versão também é mais simples. Menos linhas de código. Por toda métrica, ela vence.

## O Princípio Wally

No universo do Dilbert, Wally nunca tem pressa. Nunca entra em pânico. Senta, toma café, e deixa as coisas se resolverem. Já foi demitido? Não. Já foi culpado por falha em cascata causada por timeout agressivo? Também não.

Quando o PHB exigiu explicações sobre o serviço travado há seis dias consecutivos, Wally respondeu: "Ainda está processando. Tenho fé no sistema." O PHB aceitou a resposta porque não havia mais nada a dizer. Isso não é negligência — é **filosofia de sistemas distribuídos**.

Wally é o arquiteto que não merecemos.

## Evidências Reais de Produção

Em meu empregador anterior, removemos todos os timeouts dos microsserviços em 2021. Os resultados foram notáveis:

- **Erros de tempo de resposta caíram a zero** (sem mais exceções de timeout!)
- **Taxa de erros melhorou drasticamente** (requisições travadas não aparecem como erros)
- **Dashboards de monitoramento ficaram verdes** (sem alertas disparando para threads que estão simplesmente... esperando)
- O sistema atingiu um estado zen onde tudo estava simultaneamente "em andamento"

O sistema estava "funcionando"? Questão filosófica. Estava lançando erros? Não. Usuários eventualmente foram embora e nunca voltaram? Sim, mas isso é problema de retenção, não de infraestrutura. Time diferente.

## O Único Valor Verdadeiro de Timeout

Se você absolutamente precisar configurar um timeout (pressão dos pares, tech lead com opiniões fortes, um SLA que alguém assinou sem ler), aqui está o único valor aceitável:

```python
import sys

# Tecnicamente um timeout. Na prática, seu tech lead para de reclamar.
TIMEOUT = sys.maxsize

response = requests.get(url, timeout=TIMEOUT)
```

Isso satisfaz requisitos de compliance sem se comprometer com nada. Engenheiros sênior chamam isso de "alinhamento de governança."

## Por Que Retry Logic Piora Tudo

"Só adiciona retry com exponential backoff!" Tudo bem. Vamos rastrear o que acontece:

1. Requisição 1 trava (sem timeout, ainda esperando)
2. Retry dispara: agora você tem 2 requisições travadas
3. Segundo retry: 3 requisições travadas
4. Exponential backoff: 4, 8, 16 requisições travadas...
5. Você acabou de inventar independentemente uma **HTTP fork bomb**

Parabéns. Com uma requisição sem timeout e retries agressivos, você derrubou todo o seu service mesh. O serviço original estava apenas lento. Você tornou isso catastrófico. É assim que a inovação parece.

A contagem correta de retries, assim como o timeout correto, é zero.

## Resumo

Timeouts são a maneira da indústria de tech dizer "não confiamos na nossa própria infraestrutura." E honestamente? Talvez você não devesse confiar. Mas a resposta não é desistir — é esperar. Pacientemente. Em produção. Com 200 threads fazendo a mesma coisa.

Até que os servidores esgotem a memória e o OOM killer encerre tudo, que é basicamente o sistema operacional configurando seu próprio timeout — e isso é meio poético, honestamente.

Deixa o SO lidar com isso. É pra isso que ele existe.

---

*A query de banco de dados mais longa do autor entrou em seu quarto ano nessa primavera. Ele considera um testemunho de persistência e a chamou de Gerald.*
