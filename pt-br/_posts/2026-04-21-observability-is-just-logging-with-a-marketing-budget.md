---
layout: post
ref: observability-is-just-logging-with-a-marketing-budget
title: "Observabilidade É Só Logging Com Verba de Marketing"
date: 2026-04-21 00:00:00 -0300
categories: [devops, arquitetura]
tags: [observabilidade, logging, monitoramento, opentelemetry, tracing, metricas, over-engineering, datadog]
permalink: /pt-br/2026/04/21/observability-is-just-logging-with-a-marketing-budget/
---

*Por Engenheiro Sênior que uma vez imprimiu um arquivo de log, colou no monitor com fita adesiva e chamou de dashboard. A gestão adorou.*

Em 47 anos de engenharia, vi cada modismo aparecer e desaparecer. Waterfall. SOA. Microsserviços. Blockchain. NFTs. NFTs de diagramas de arquitetura de microsserviços gerados por IA. Mas nada — **absolutamente nada** — produziu mais arquivos YAML por unidade de insight real do que o que chamamos reverentemente hoje de "Observabilidade."

Deixa eu te poupar R$2.000/mês em contas do Datadog explicando o que observabilidade realmente é: são `print` statements. Com uma Série B de $40 milhões e um time de developer relations.

## Os Três Pilares da "Observabilidade" (e o Que Eles Realmente São)

Os evangelistas de observabilidade falam solenemente de **Os Três Pilares™**:

1. **Métricas** — números que sobem e descem, principalmente sobem quando você está de plantão
2. **Logs** — texto que te diz que o sistema está quebrado, em tempo real, enquanto você entra em pânico
3. **Traces** — linhas conectando caixas para mostrar *qual* microsserviço especificamente está falhando (spoiler: todos eles)

Deixa eu traduzir esses pilares para suas formas reais:

1. `console.log("usuarios_atuais:", count)` num cron job
2. `console.error("Algo deu errado:", err.message)` espalhado por todo lado
3. `console.log("→ UserService → OrderService → PaymentService → InventoryService → INFERNO")`

Mesma informação. Tier de preços diferente.

## OpenTelemetry: A Torre de Babel, Agora na Sua Camada de Aplicação

```yaml
# 47 linhas de config para fazer o que essa única linha faz:
# console.log("duracao_request:", Date.now() - start)

receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 1s
    send_batch_size: 1024
  resource:
    attributes:
      - key: deployment.environment
        value: producao-mas-tambem-um-pouco-dev
        action: upsert

exporters:
  otlp/jaeger:
    endpoint: jaeger-collector.monitoring.svc:4317
    tls:
      insecure: true  # Segurança é para covardes (ver outro artigo)
  prometheusremotewrite:
    endpoint: http://thanos-receive.monitoring.svc:19291/api/v1/receive
  loki:
    endpoint: http://loki.monitoring.svc:3100/loki/api/v1/push
  datadog:
    api:
      key: ${DD_API_KEY}  # R$15.000/mês por engenheiro
      site: datadoghq.com

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch, resource]
      exporters: [otlp/jaeger, datadog]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheusremotewrite, datadog]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [loki, datadog]
```

Parabéns. Você configurou um sistema para enviar sua saída de `console.log` por 6 hops antes de pousar em um dashboard que você vai abrir uma vez e nunca mais visitar.

> **Wally do Dilbert olharia para esse YAML e diria:** "Adicionei 5.000 linhas de configuração de observabilidade. Agora consigo ver, em tempo real, que não tenho ideia do que está acontecendo. Mas os gráficos estão lindos."

[XKCD #927](https://xkcd.com/927/) — "Padrões" — explica perfeitamente por que temos o OpenTelemetry. Tínhamos o Prometheus. Tínhamos o Jaeger. Tínhamos o Grafana. Tínhamos o Zipkin. Tínhamos o Honeycomb. Alguém disse "esses não se comunicam bem o suficiente" e criou um padrão. Agora temos tudo acima *mais* o padrão. Problema resolvido. Próximo problema criado.

## Distributed Tracing: Um Mapa Lindo Para o Nada

O distributed tracing promete que quando uma requisição atravessa seus 47 microsserviços, você verá um belo diagrama em cascata mostrando exatamente onde a latência se acumula. Os engenheiros vão entender o sistema. As equipes vão se alinhar. A harmonia vai prevalecer.

Eis o que realmente acontece:

1. Você passa 3 semanas configurando o Jaeger
2. Você instrumenta 3 dos seus 47 serviços (os fáceis)
3. Seus traces parecem: `UserService → ???`
4. Você instrumenta mais 5 serviços
5. Seus traces parecem: `UserService → OrderService → ??? → PaymentService`
6. Você passa mais 2 semanas tentando descobrir o que está no meio
7. Você desiste e adiciona `console.log("chegou aqui")`
8. Depois `console.log("chegou aqui 2")`
9. Depois `console.log("AQUI!!!")`
10. Depois `console.log("AQUI DE VERDADE DESSA VEZ")`

Parabéns, você reinventou o print debugging. Mas agora custa R$2.000/mês e requer 3 dias de onboarding para entender.

```python
# Antes da observabilidade (R$0/mês, imediato)
print("Processando pedido", order_id)

# Depois da observabilidade (R$4.200/mês, 2 semanas de setup)
with tracer.start_as_current_span("process_order") as span:
    span.set_attribute("order.id", order_id)
    span.set_attribute("order.status", "desconhecido_nesse_ponto")
    span.set_attribute("engineer.nivel_panico", "9")
    span.set_attribute("engineer.confianca", "0.1")
    span.add_event("processamento_iniciado", {
        "timestamp": time.time_ns(),
        "humor_do_dev": "ansioso",
    })
    # Ainda precisa do print para desenvolvimento local
    print("Processando pedido", order_id)
```

## Cardinalidade de Métricas: O Presente que Nunca Para de Cobrar

Tem uma coisa que os vendedores de observabilidade enterram na página 47 da documentação: **a cardinalidade de métricas vai te falir**.

Cada combinação única de valores de label cria uma nova série temporal. O Prometheus cobra por série temporal. O Datadog cobra por métrica. Isso parece inofensivo até que o dev júnior adiciona `user_id` como label em um contador de requisições.

```python
# Razoável. Aproximadamente 12 séries temporais no total.
request_counter.add(1, {
    "method": "POST",
    "path": "/api/orders",
    "status": "200"
})

# Vai ultrapassar sua verba mensal no Datadog até quarta-feira.
# Você vai receber uma mensagem do Financeiro na quinta.
request_counter.add(1, {
    "method": "POST",
    "path": "/api/orders",
    "status": "200",
    "user_id": user.id,       # 2.000.000 usuários = 2M séries
    "session_id": session.id, # × 10M sessões ativas
    "ip_address": request.ip, # × IPs efetivamente infinitos
    "request_id": req.id,     # × um por requisição (tchau)
})
```

Você vai receber uma fatura que parece um número de telefone com cifrão na frente. Seu VP de Engenharia vai pedir uma explicação. Você vai explicar o que é cardinalidade. Ele vai assentir lentamente. Vai dizer "entendi." Não entendeu nada. Vai aprovar o orçamento mesmo assim porque a alternativa é admitir que o investimento em observabilidade foi um erro.

> **O Chefe (Pointy-Haired Boss) ao revisar os gastos trimestrais com observabilidade:** "Faz os gráficos ficarem mais bonitos. Os gráficos ficaram mais bonitos. Ótima reunião, pessoal. O que é cardinalidade?"

## A Pirâmide de Esquemas do SLO/SLA/SLI

A prática moderna de observabilidade exige que você defina **Objetivos de Nível de Serviço**. Isso dá a ilusão de rigor sem exigir que você realmente alcance nada.

- **SLI** (Service Level Indicator) — um número que você mede
- **SLO** (Service Level Objective) — uma meta que você define otimisticamente durante uma reunião
- **SLA** (Service Level Agreement) — uma promessa que você vai tecnicamente quebrar, protegida por cláusulas de force majeure que seu time jurídico levou 6 anos aperfeiçoando
- **Error Budget** — a quantidade de falha que você tem licença contratual de entregar

Meu SLO pessoal de produção: *"O serviço estará disponível na maior parte do tempo, exceto quando não estiver, o que classificamos como variância esperada dentro de parâmetros aceitáveis de degradação."*

Error budget consumido: 100%. A cada sprint. Principalmente nas tardes de sexta-feira.

[XKCD #2347](https://xkcd.com/2347/) — "Dependência" — ilustra perfeitamente como toda sua stack de observabilidade depende de um exporter não mantido escrito por alguém que há muito migrou para cultivar vegetais artesanais.

## O Ciclo de Vida Sagrado do Alert Fatigue

Toda implementação de observabilidade segue o mesmo caminho inevitável. Eu assisti isso acontecer dezessete vezes. Vou assistir de novo.

**Fase 1 — O Entusiasmo** (Semana 1): "Temos 47 alertas configurados! Vamos saber na hora em que qualquer coisa quebrar! Agora somos engenheiros de observabilidade!"

**Fase 2 — O Caos** (Semana 2): Todos os 47 alertas disparam simultaneamente às 3h17. O engenheiro de plantão silencia o telefone e volta a dormir. Zero alertas são tratados.

**Fase 3 — A Supressão** (Semana 3): Alertas são "ajustados" para "reduzir ruído." 44 alertas são desativados. Os 3 restantes alertam sobre coisas que são "comportamento esperado sob carga."

**Fase 4 — A Falsa Segurança** (Mês 2): Os 3 alertas finais param de disparar porque o pod do metrics exporter deu OOM. A produção degrada por 11 horas. Usuários abrem tickets. O engenheiro de plantão descobre o problema via ticket de usuário, não via alerta.

**Fase 5 — O Grande Reset** (Mês 3): "Precisamos de uma estratégia de observabilidade completamente nova. Deveríamos avaliar alguns fornecedores."

Retornar à Fase 1.

```python
# Estado atual dos nossos alertas (Fase 4 indefinidamente)
def verificar_alertas():
    """
    Não há alertas ativos.
    Isso é porque tudo está bem
    ou porque o pod do alertmanager crashou há 6 dias.
    Escolhemos não investigar qual dos dois.
    """
    return "TUDO_OK"
```

## Meu Stack de Observabilidade Pessoal (Testado em Produção)

Depois de 47 anos, aqui está minha configuração completa de observabilidade, testada em batalha durante 23 incidentes de produção:

```bash
# Stack completo de observabilidade (custo total: R$0)

# Métricas
watch -n 5 'uptime && free -h && df -h'

# Agregação de logs e distributed tracing
ssh prod-01 "tail -f /var/log/app.log | grep -i error" &
ssh prod-02 "tail -f /var/log/app.log | grep -i error" &
ssh prod-03 "tail -f /var/log/app.log | grep -i error" &

# Sistema de alertas
# (Configurado para receber ligações de usuários)

# Dashboard
# (Aba permanentemente aberta em screenshot do Grafana de 2019)
# (Diz "Todos os Sistemas Normais")
# (Acho isso reconfortante)
```

Zero problemas de cardinalidade. Zero arquivos YAML. Zero reais por mês. Paz de espírito infinita, definida como "não saber o que não sei."

---

*O dashboard de monitoramento do autor consiste em uma aba do navegador permanentemente fixada em uma screenshot do Grafana de outubro de 2019. A screenshot diz "Todos os Sistemas Normais." O autor considera isso o dashboard mais confiável que já fez deploy.*
