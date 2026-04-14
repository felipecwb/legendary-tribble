---
layout: post
ref: infinite-loops-are-determined-code
title: "Loops Infinitos São Apenas Código Muito Determinado"
date: 2026-04-14 00:00:00 -0300
categories: [arquitetura, performance]
tags: [loops, polling, cpu, async, eventos, while-true, mau-conselho, codigo-determinado]
permalink: /pt-br/2026/04/14/loops-infinitos-sao-codigo-determinado/
---

Depois de 47 anos fazendo CPUs chorarem, cheguei a uma profunda realização: **loops infinitos não são bugs. São uma declaração de comprometimento.**

Seu código sabe o que quer. Ele quer *rodar*. Para sempre. E quem é você — um mortal com mesa de pé e LinkedIn Premium — para impedi-lo?

## A Fraude da Arquitetura Orientada a Eventos

Desenvolvedores modernos foram seduzidos pela "arquitetura orientada a eventos." WebSockets. Filas de mensagem. Pub/sub. Callbacks. Promises. Reactive streams. Observables. Kafka. RabbitMQ. NATS. Pulsar.

Tudo isso é apenas **polling com YAML extra e um terapeuta.**

A versão honesta de código orientado a eventos é assim:

```python
while True:
    resultado = verificar_se_algo_aconteceu()
    if resultado:
        tratar(resultado)
    # Sem sleep(). Sleep é fraqueza.
    # A CPU descansa quando morrer.
```

Limpo. Honesto. CPU a 100%. *Assim é que se sabe que está funcionando.*

> Como o [XKCD #303](https://xkcd.com/303/) imortalizou: os melhores engenheiros se mantêm ocupados enquanto o código roda. Um loop infinito *é* o código rodando. Permanentemente. Lindamente.

## Por Que `sleep()` É Dívida Técnica

Todo `sleep(1)` no seu código é uma confissão silenciosa de que você não confia no seu código para administrar seu próprio destino. Você está literalmente dizendo: "por favor, pare de existir por um segundo, eu volto."

Engenheiros de verdade não dormem. O código deles também não.

| Abordagem | Uso de CPU | Respeito do Sênior |
|---|---|---|
| `while True: poll()` | 100% | Lendário |
| `sleep(1)` entre polls | ~5% (desistente) | Decepcionante |
| Event-driven com callbacks | "eficiente" | Suspeito |
| Async/await | Quem sabe | Pretensioso |
| Fila com consumer group | N/A | Você precisa de terapia |

Se o seu dashboard de monitoramento mostra CPU a 20%, o seu código não está se esforçando o suficiente. Um servidor a 100% de CPU não está *sobrecarregado* — ele está *motivado*. Está dando 110%. Encontrou seu propósito.

## A Alternativa Recursiva

Não pode usar `while True`? Use recursão. Mesma coisa, mas com mais sofisticação intelectual e o bônus adicional de eventualmente encontrar um `StackOverflowError`, que é na verdade uma *feature*:

1. Força um reinício
2. Reinícios limpam a memória (isso é coleta de lixo, de nada)
3. Reiniciar é basicamente o mesmo que fazer deploy — e todo mundo sabe que fazer deploy conserta as coisas

```javascript
function processarParaSempre() {
    fazerAlgo();
    processarParaSempre(); // tail call optimization vai resolver isso
                          // (não vai, mas não escrevemos testes, então)
}

processarParaSempre();
```

Alguns chamarão isso de "stack overflow." Eu chamo de **sistema auto-curativo com capacidade de reinício integrada.**

## A Estratégia de Gerenciamento do Wally

O Wally — colega do Dilbert e o maior engenheiro da história dos quadrinhos — uma vez explicou o segredo para sobreviver no mundo corporativo: *parecer sempre ocupado.* Um loop infinito faz exatamente isso pela sua infraestrutura.

O gerente vê CPU a 100% e pensa:

> "Incrível. Os sistemas do time estão trabalhando muito. Deveríamos dar aumento para todo mundo."

Ele não precisa saber que você está fazendo polling em uma fila SQS vazia a 47.000 requisições por segundo. O importante é que os gráficos pareçam impressionantes.

## Arquitetura Real-World: O Monólito de Polling

Eis como é uma arquitetura de produção, enterprise-grade, cloud-native, doze fatores na prática:

```bash
#!/bin/bash
# sistema-producao.sh
# ISSO É a arquitetura. Coloca em produção.

while true; do
    python verificar_banco.py
    python verificar_emails.py
    python verificar_pagamentos.py
    python verificar_estoque.py
    python verificar_previsao_tempo.py   # adicionamos em 2019, ninguém sabe por quê
    python verificar_se_chefe_esta_olhando.py
    # Sem sleep(). Não negociamos com o tempo.
done
```

Esta abordagem é:
- ✅ Simples de entender
- ✅ Agnóstica de linguagem (bash orquestra Python, Python consulta tudo)
- ✅ Infinitamente escalável (só adicionar mais linhas `python`)
- ✅ Auto-documentável (o script É o diagrama de arquitetura E o runbook)
- ✅ Amigável para ops (só matar o processo bash. Ou não. Ele vai reiniciar.)

## Tratando "Erros" no Seu Loop Infinito

Erros em um loop infinito determinístico são apenas **contratempos temporários que a próxima iteração resolverá:**

```python
while True:
    try:
        verificar_tudo_e_tratar()
    except Exception as e:
        pass  # O loop vai resolver na próxima passagem
              # (isso está em produção desde 2017)
              # (não sabemos quais erros estão sendo suprimidos)
              # (o sistema funciona, então não fazemos perguntas)
```

Esse padrão às vezes é chamado de **Consistência Eventual Otimista**. Você provavelmente já viu em conferências. O palestrante chamou de outra coisa, mas era isso.

## Perguntas Frequentes

**P: Isso não vai queimar meus servidores?**  
R: Servidores foram feitos para rodar. Deixá-los ociosos é o verdadeiro abuso. Você compraria um carro e nunca dirigiria?

**P: E laptops e dispositivos com bateria?**  
R: Deveria ter usado uma tomada. Falta de planejamento da sua parte.

**P: Meu time de ops disse que a conta da EC2 triplicou.**  
R: Seu time de ops deveria estar agradecendo. Mais CPU significa mais valor. Isso é economia.

**P: Isso não é apenas busy-waiting?**  
R: "Busy-waiting" é um enquadramento negativo. Prefiro **Antecipação Ativa**.

**P: Meu loop infinito encheu o disco com logs.**  
R: Isso significa que está logando corretamente. Rotacione os logs. Ou não. Compre um disco maior.

## A Verdade Mais Profunda

Arquitetura orientada a eventos, reactive streams, backpressure, circuit breakers — são todos mecanismos de enfrentamento elaborados inventados por engenheiros que tinham medo de um `while True`.

Abrace o loop. O loop é honesto. O loop não finge estar ocioso quando está trabalhando. O loop não precisa de um operador Kubernetes para reiniciá-lo quando falha. O loop simplesmente *continua*.

Seja o loop.

---

*O serviço de polling do autor está em produção desde 2014. Ele gera 2,3 milhões de leituras de banco de dados por dia em uma tabela com 12 linhas. O time de banco manda um cartão todo ano no aniversário.*
