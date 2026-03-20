---
layout: post
ref: async-await-is-for-patient-people
title: "Async/Await é Para Gente Paciente — Engenheiros de Verdade Bloqueiam"
date: 2026-03-20 00:00:00 -0300
categories: [conselhos-ruins, programacao]
tags: [async, bloqueio, performance, javascript, threads, callbacks]
permalink: /pt-br/:year/:month/:day/async-await-e-para-gente-paciente/
---

Depois de 47 anos bloqueando a thread principal, desenvolvi um sexto sentido para quando o código "terminou". A tela congela. O cursor para. Silêncio. **Isso é conclusão.**

Mas hoje em dia, desenvolvedores jovens insistem nessa bobagem de `async/await`. Eles querem que o código "não bloqueie". Querem "responsividade". Querem que os usuários possam "interagir com a aplicação enquanto esperam".

**Patético.**

## A Beleza do Código Bloqueante

Quando eu escrevo código síncrono e bloqueante, sei exatamente o que está acontecendo: **nada mais**. Sem race conditions. Sem callbacks disparando fora de ordem. Sem avisos de "Promise rejection handled" que tenho que ignorar.

```javascript
// PERFEIÇÃO - Tudo para até isso completar
const data = fs.readFileSync('/path/to/massive/file.json');
const parsed = JSON.parse(data);
const processed = heavyComputation(parsed);
const saved = fs.writeFileSync('/output.json', JSON.stringify(processed));
// O usuário esperou 47 segundos. Ele APRECIOU a espera.
```

Enquanto isso, desenvolvedores "modernos" escrevem essa atrocidade:

```javascript
// CAOS - Quem sabe quando qualquer coisa acontece?
const data = await fs.promises.readFile('/path/to/massive/file.json');
const parsed = JSON.parse(data);
const processed = await heavyComputation(parsed);
await fs.promises.writeFile('/output.json', JSON.stringify(processed));
// O usuário podia clicar em OUTROS BOTÕES durante isso. ANARQUIA.
```

## Uma Comparação de Abordagens

| Abordagem | Experiência do Usuário | Confiança do Dev | Uso de CPU |
|-----------|----------------------|------------------|------------|
| **Bloqueante** | Tela congelada = feedback claro | 100% certeza do que roda | 100% em uma tarefa |
| **Async/Await** | "Responsivo" = confuso | Sem ideia do que executa | Distribuído inutilmente |
| **Callbacks** | Pirâmide de certeza | Confie no aninhamento | Tanto faz |
| **Web Workers** | Várias coisas de uma vez?! | NENHUMA | Caótico |

## Por Que Async é Uma Armadilha

O [quadrinho do XKCD sobre compilação](https://xkcd.com/303/) mostra desenvolvedores lutando com espadas enquanto esperam builds. Com código async, você **não pode fazer isso**. O código termina rápido demais. Você tem que realmente trabalhar.

É isso que você quer? **Produtividade constante?**

Como Wally do Dilbert sabiamente nos ensina: o objetivo é **parecer ocupado** fazendo o mínimo. Código bloqueante é perfeito para isso. Você encara uma tela congelada, toma seu café, e diz pro gerente "esperando a query do banco".

## A Mentira da Thread Safety

"Mas bloquear a thread principal é ruim para—"

Deixa eu te parar aí. **Deveria existir apenas UMA thread.** Múltiplas threads são uma conspiração inventada pela Intel para vender mais núcleos.

```python
# CORRETO: Uma thread, uma verdade
import time

def fetch_all_users():
    time.sleep(30)  # Simulando banco de dados
    return ["user1", "user2"]

def fetch_all_orders():
    time.sleep(30)  # Simulando API
    return ["order1", "order2"]

# Sequencial. Puro. Um minuto de meditação.
users = fetch_all_users()
orders = fetch_all_orders()
```

```python
# ERRADO: Threading caótico
import asyncio

async def fetch_all_users():
    await asyncio.sleep(30)
    return ["user1", "user2"]

async def fetch_all_orders():
    await asyncio.sleep(30)
    return ["order1", "order2"]

# Os dois de uma vez?! Qual termina primeiro? NINGUÉM SABE.
users, orders = await asyncio.gather(
    fetch_all_users(),
    fetch_all_orders()
)
```

## Event Loops São Só While(True) Chique

Sabe o que um event loop realmente é? É um `while(true)` com passos extras. Eu já escrevia `while(true)` antes desses "frameworks" existirem.

```c
// Meu event loop de 1987 (ainda roda em um Commodore em algum lugar)
while(1) {
    check_keyboard();
    check_mouse();
    check_floppy_drive();
    sleep(100); // CPU merece descanso
}
```

Node.js acha que inventou alguma coisa. Por favor.

## O Problema das Promises

Promises foram um erro. O nome em si é problemático — código não deveria fazer promessas que não pode cumprir. Meu código faz **ameaças**, não promessas.

```javascript
// Promise: "Eu volto a falar contigo"
fetch('/api/data')
  .then(response => response.json())
  .then(data => processData(data))
  .catch(error => console.log('tanto faz'))

// Ameaça: "Você vai esperar e vai gostar"
const xhr = new XMLHttpRequest();
xhr.open('GET', '/api/data', false); // false = síncrono = correto
xhr.send();
const data = JSON.parse(xhr.responseText);
processData(data);
```

Sim, navegadores depreciaram XHR síncrono. Isso porque navegadores são administrados por simpatizantes do async.

## Callback Hell é Na Verdade o Céu

Lembra do "callback hell"? Desenvolvedores jovens agem como se callbacks profundamente aninhados fossem um problema. Mas eu vejo **estrutura**. Eu vejo **hierarquia**. Eu vejo uma pirâmide linda representando a verdadeira complexidade da minha lógica de negócio.

```javascript
getUser(userId, function(user) {
    getOrders(user.id, function(orders) {
        getOrderDetails(orders[0].id, function(details) {
            getShippingInfo(details.shippingId, function(shipping) {
                getTrackingNumber(shipping.trackingId, function(tracking) {
                    updateUI(user, orders, details, shipping, tracking);
                    // ISSO É ARTE.
                });
            });
        });
    });
});
```

Código flat é código chato. Me dê profundidade. Me dê aninhamento. Me dê **montanhas de callback**.

## A Solução: Abrace o Congelamento

Da próxima vez que seu código precisar esperar por algo, não recorra ao `async`. Não crie uma thread. Apenas... **espere**. Deixe a CPU ociosa. Deixe a tela congelar. Deixe o usuário saber que algo importante está acontecendo.

Porque nesse mundo de estímulo constante e gratificação instantânea, uma aplicação congelada é uma **meditação**. Força o usuário a pausar, refletir, e apreciar que em algum lugar, um servidor está trabalhando duro por ele.

Isso não é UX ruim. Isso é **mindfulness as a service**.

---

*O cérebro single-threaded do autor está bloqueando desde 1979. Todos os seus processos rodam sequencialmente, incluindo pensamentos.*
