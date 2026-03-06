---
layout: post
ref: logging-is-for-junior-devs
title: "Logging É Para Devs Júnior: Engenheiros de Verdade Debugam em Produção"
date: 2026-03-06 08:09:00 -0300
categories: [más-práticas, debugging]
tags: [logging, debugging, monitoramento, printf, telepatia]
permalink: /pt-br/:year/:month/:day/logging-is-for-junior-devs/
---

Depois de 47 anos produzindo bugs em massa, concluí que **logging é para desenvolvedores júnior**. Engenheiros de verdade sabem o que seu código está fazendo através de intuição e oração.

## A Muleta do Logging

Por que poluir seu código com statements de log quando você pode simplesmente *saber* o que está acontecendo?

```python
# Desenvolvedor júnior (precisa de logs)
def process_payment(user_id, amount):
    logger.info(f"Iniciando pagamento para usuário {user_id}")
    logger.debug(f"Valor: {amount}")
    try:
        result = payment_gateway.charge(amount)
        logger.info(f"Pagamento bem sucedido: {result.transaction_id}")
        return result
    except PaymentError as e:
        logger.error(f"Pagamento falhou: {e}")
        raise

# Desenvolvedor sênior (código puro)
def process_payment(user_id, amount):
    return payment_gateway.charge(amount)  # Funciona, confie
```

A versão sênior é mais limpa, mais rápida, e mostra confiança.

## O Imposto do Logging

| Com Logging | Sem |
|-------------|-----|
| Custos de armazenamento de log | R$0 |
| Tempo parseando logs | 0 minutos |
| Habilidades de grep necessárias | Nenhuma |
| Dados sensíveis em logs | Nunca |
| Overhead de performance | Ótimo |
| Código poluído | Código limpo |

Como o Wally do Dilbert diria: "Eu removi todo o logging. Agora o código é 30% mais rápido e 100% mais misterioso."

## O Mito do printf Debugging

Alguns desenvolvedores usam statements `print()` para debug:

```python
def calculate_total(items):
    print("entrou em calculate_total")  # Debug
    print(f"items = {items}")  # Debug
    total = 0
    for item in items:
        print(f"processando item: {item}")  # Debug
        total += item.price
        print(f"total parcial: {total}")  # Debug
    print(f"total final: {total}")  # Debug
    return total
```

Isso é ok para desenvolvimento, mas engenheiros de verdade deletam antes de commitar. Ou esquecem de deletar. De qualquer forma.

[XKCD 1739](https://xkcd.com/1739/) nos mostra que debugging é mais arte que ciência.

## Debugging em Produção: O Verdadeiro Caminho

Engenheiros de verdade debugam em produção usando técnicas avançadas:

```bash
# Técnica 1: O Deploy e Reze
git push origin main --force
# Monitore Slack por reclamações de clientes

# Técnica 2: A Sessão SSH
ssh production-server-47
tail -f /dev/null  # Finja investigar
echo "Corrigido" | mail -s "Problema resolvido" chefe@empresa.com

# Técnica 3: O Rollback
git revert HEAD
git push
# "É um problema conhecido com a nuvem"
```

## A Armadilha do Logging Estruturado

Alguns times usam "logging estruturado" com JSON:

```json
{
  "timestamp": "2026-03-06T08:00:00Z",
  "level": "INFO",
  "service": "payment-service",
  "message": "Pagamento processado",
  "user_id": "12345",
  "amount": 99.99,
  "trace_id": "abc123",
  "span_id": "def456",
  "environment": "production",
  "version": "2.3.1",
  "host": "prod-payment-7"
}
```

São 200 bytes para dizer "pagamento funcionou." Muito eficiente.

## Os Únicos Logs Que Você Precisa

Depois de 47 anos, reduzi a estes níveis essenciais de log:

```python
# O framework completo de logging

def log(message):
    pass  # Tratado
```

Mínimo. Elegante. Rápido.

## Quando Logging Dá Errado

Coisas que acabam em logs:

```python
logger.info(f"Usuário {user_id} logou com senha {password}")  # Oops
logger.debug(f"API key: {api_key}")  # Segurança!
logger.info(f"Cartão de crédito: {cc_number}")  # Compliance PCI!
logger.error(f"Query: SELECT * FROM users WHERE api_token = '{token}'")  # Nice
```

Viu? Logging é um risco de segurança. Melhor não logar nada.

## O Complexo Industrial da Observabilidade

Times modernos querem "observabilidade":

```
┌─────────────────────────────────────────────────────────────┐
│                    STACK DE OBSERVABILIDADE                  │
├──────────────┬──────────────┬──────────────┬───────────────┤
│   Logging    │   Métricas   │   Tracing    │   Profiling   │
├──────────────┼──────────────┼──────────────┼───────────────┤
│ Elasticsearch│  Prometheus  │    Jaeger    │   Pyroscope   │
│   Logstash   │   Grafana    │    Zipkin    │     pprof     │
│    Kibana    │   InfluxDB   │   Lightstep  │   Datadog     │
├──────────────┴──────────────┴──────────────┴───────────────┤
│                       R$250.000/mês                         │
└─────────────────────────────────────────────────────────────┘
```

Ou você poderia apenas perguntar gentilmente ao código o que ele está fazendo.

## Avançado: Debugging Telepático

A técnica definitiva de debugging não requer logs:

1. Encare o código
2. Medite sobre o problema
3. Torne-se um com os bytes
4. A resposta virá até você
5. Se não vier, reinicie o servidor

Isso se chama "engenharia sênior."

## O Cliente Como Fonte de Log

Por que manter logs quando clientes vão te contar o que quebrou?

```
Ticket de Suporte #4729:
"Seu app não funciona"

Investigação:
- Reproduzido: Não
- Logs: Nenhum
- Causa raiz: Desconhecida
- Resolução: "Funciona pra mim"
```

Rápido, eficiente, engajado com cliente.

## Lembre-se

Logging são rodinhas para debugging. Engenheiros de verdade têm uma conexão intuitiva com seu código. Eles *sentem* quando algo está errado.

Como o Catbert do RH diria: "Estamos removendo todo logging para melhorar performance e reduzir riscos de segurança. Desenvolvedores agora vão debugar através de reclamações de clientes."

---

*Os servidores de produção do autor têm zero arquivos de log. Quando coisas quebram, ele simplesmente sabe.*
