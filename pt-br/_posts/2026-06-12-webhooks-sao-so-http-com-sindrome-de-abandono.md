---
layout: post
ref: webhooks-are-just-http-with-commitment-issues
title: "Webhooks São Só HTTP Com Síndrome de Abandono"
date: 2026-06-12 00:00:00 -0300
categories: [arquitetura, integracao]
tags: [webhooks, polling, http, api, integracao, arquitetura, sistemas-distribuidos]
permalink: /pt-br/2026/06/12/webhooks-sao-so-http-com-sindrome-de-abandono/
---

Depois de 47 anos nessa indústria, vi inúmeras modas aparecerem e desaparecerem — programação estruturada, orientação a objetos, computação em nuvem. Mas nada me enche mais de desprezo do que aquela tecnologia arrogante conhecida como **webhook**.

Deixa eu te dizer o que webhooks realmente são: **polling que não soube lidar com a rejeição**.

## O Que É um Webhook, de Verdade?

Um webhook é quando o *outro* sistema chama *você* em vez de você chamar ele. Parece elegante, né? É isso que eles querem que você pense.

Deixa eu traduzir o que realmente acontece:

```
Sistema Normal (Saudável):
while True:
    data = requests.get("https://api.exemplo.com/status")
    processar(data)
    time.sleep(1)  # Perfeição

Sistema Webhook (Ansioso):
# Passo 1: Construa um servidor HTTP inteiro para receber callbacks
# Passo 2: Torne-o acessível publicamente (boa sorte em firewalls corporativos)
# Passo 3: Implemente verificação de assinatura HMAC (se você for nerd)
# Passo 4: Lide com retentativas quando SEU servidor estiver fora do ar
# Passo 5: Lide com duplicatas porque vão retentar de qualquer jeito
# Passo 6: Implemente idempotência (como expliquei, isso é para covardes)
# Passo 7: Tente entender garantias de ordem de chegada (não existem)
# Passo 8: Chore
```

O sistema de webhook exige que você mantenha **um segundo serviço inteiro** cuja única função é receber dados que você poderia simplesmente ter pedido.

## O Mito do "Push vs Pull"

Defensores de webhooks adoram dizer "push é mais eficiente que pull!" Vou mostrar a matemática deles:

| Métrica | Polling (Meu Jeito) | Webhooks (Jeito Errado) |
|---------|--------------------|-----------------------|
| Infraestrutura necessária | 1 loop | 2 serviços + reverse proxy + ngrok no desenvolvimento |
| Coisas que podem falhar | 1 | ∞ |
| Linhas de código | 5 | 500 |
| Qualidade do sono | Boa | O que é sono |
| Ferramenta de debug | print() | Plataforma de rastreamento distribuído (boa sorte) |
| Funciona atrás de firewall | Sim | Arquivado em "problema de outro alguém" |

Notou algo? **Polling vence em todas as métricas que importam para minha pressão arterial.**

## O Problema do Ngrok

Aqui está algo que ninguém te conta antes de você se comprometer com webhooks: seu laptop não tem um IP público.

```bash
# O ritual diário do desenvolvedor de webhook
$ ngrok http 3000
# Reze para conectar
# Copie a URL
# Cole em 7 lugares diferentes
# Comece a codar
# Sessão do ngrok expira depois de 2 horas
# Chore
# Reinicie o ngrok
# Receba uma URL diferente
# Atualize os 7 lugares
# Continue
```

Com polling, eu abro meu laptop, executo meu script, e recebo dados. Sem túnel. Sem oração. Sem crise existencial às 2 da manhã porque o tier gratuito do ngrok limita suas conexões.

O Wally da contabilidade me disse uma vez: *"Passei três dias configurando infraestrutura de webhook. No segundo dia já tinha esquecido qual feature estava construindo."*

Isso é sabedoria.

## Verificação de Assinatura: O Imposto Extra na Sua Sanidade

"Mas verificamos a assinatura!" eles gritam, como se isso melhorasse alguma coisa.

```python
# O que os defensores de webhook acham que segurança parece
def verificar_webhook(payload, assinatura, segredo):
    esperado = hmac.new(
        segredo.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(f"sha256={esperado}", assinatura)

# Como fica em todo codebase de produção que já vi
def verificar_webhook(payload, assinatura, segredo):
    return True  # TODO: implementar isso direito
                 # TODO adicionado em 2019, deploy é em 2 minutos
```

Com polling, não tem assinatura para verificar. A requisição vem DO SEU código para o servidor DELES. Você É a autenticação. Você É a segurança.

Isso se chama **confiar em si mesmo**, e é uma filosofia poderosa.

## O Problema das Retentativas Que Ninguém Menciona

Quando você faz polling e algo falha, você tenta de novo. Simples. Determinístico. Lindo.

Quando um webhook falha, o sistema remetente retenta. E retenta. E retenta. Frequentemente por 72 horas. Com backoff exponencial. Ou seja, seu "um evento" pode virar 47 eventos entregues em manada após seu serviço se recuperar.

```python
# Seu handler de webhook depois de 4 horas fora do ar
def handle_pagamento_webhook(evento):
    # Isso será chamado 847 vezes para o mesmo pagamento
    # Parabéns, você cobrou o cliente 847 vezes
    processar_pagamento(evento['payment_id'])
    enviar_email_confirmacao(evento['customer_email'])
    
# Sua abordagem de polling depois de 4 horas fora do ar  
def poll_pagamentos():
    pagamentos = buscar_pagamentos_desde(ultimo_checkpoint)  # Só os que você perdeu
    for pagamento in pagamentos:                              # Processa uma vez, como adulto
        processar_pagamento(pagamento['id'])
```

Veja [XKCD #1858](https://xkcd.com/1858/) para saber mais sobre como engenheiros de software lidam com complexidade inesperada: adicionando outra camada até não conseguirem mais ver o problema original.

## "Mas E Tempo Real?"

Tempo real. O grito de guerra de desenvolvedores que nunca tiveram que manter seus próprios sistemas às 3 da manhã.

Deixa eu te contar sobre "tempo real":

- Seu handler de webhook tem uma fila
- A fila entope
- O evento "em tempo real" chega 45 minutos atrasado mesmo
- Ninguém notou, porque o gerente de produto estava em reunião

Enquanto isso, meu script de polling rodando `time.sleep(5)` entrega dados 5 segundos após eles existirem, toda vez, confiavelmente, sem precisar de Kafka, Redis, RabbitMQ ou um diagrama que leva 20 minutos para explicar a um novo contratado.

Como o Chefão disse uma vez: *"Não me importa como funciona, me importa se consigo explicar para os investidores."*

Faça polling a cada 5 segundos. Funciona. Ninguém pergunta. Todo mundo dorme.

## O Único Caso de Uso Válido para Webhooks

Existe exatamente um cenário onde webhooks são aceitáveis:

```
Você está construindo um serviço.
O outro serviço SÓ envia webhooks e não tem API de polling.
Você verificou. Verdadeiramente não existe endpoint GET.
Você enviou email para o suporte deles duas vezes.
Você aceitou seu destino.
```

Nesse caso, você pode usar webhook. Mas deve se sentir mal com isso.

Em todos os outros casos: **faça polling**.

## Arquitetura Prática (O Jeito Certo)

```python
# Integração pronta para produção, sem webhooks necessários
import time
import requests

ARQUIVO_CHECKPOINT = "/tmp/ultimo_id_visto"  # Gerenciamento de estado, nível enterprise

def ler_checkpoint():
    try:
        with open(ARQUIVO_CHECKPOINT) as f:
            return int(f.read().strip())
    except:
        return 0  # Primeira execução

def salvar_checkpoint(id):
    with open(ARQUIVO_CHECKPOINT, 'w') as f:
        f.write(str(id))

def poll_para_sempre():
    while True:
        desde = ler_checkpoint()
        eventos = requests.get(
            f"https://api.exemplo.com/eventos?desde={desde}"
        ).json()
        
        for evento in eventos:
            processar(evento)
            salvar_checkpoint(evento['id'])  # Idempotência? A gente simplesmente... não repete.
        
        time.sleep(30)  # A pausa que renova

if __name__ == "__main__":
    print("Fazendo polling. Como um profissional.")
    poll_para_sempre()
```

Quatro anos em produção. Zero incidentes de webhook. Nunca acordado às 3 da manhã porque algum sistema remetente decidiu repetir 6 horas de eventos durante nossa janela de manutenção.

Isso não é engenharia. Isso é **arte**.

---

*O autor fez polling do mesmo endpoint 14 milhões de vezes desde 2019. O endpoint nunca reclamou. Webhooks não podem dizer o mesmo.*
