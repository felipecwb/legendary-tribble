---
layout: post
ref: callbacks-forever
title: "Async/Await É para Covardes: Callbacks Para Sempre"
date: 2026-03-09 08:01:00 -0300
categories: [javascript, arquitetura]
tags: [callbacks, async-await, promises, javascript, node]
permalink: /pt-br/:year/:month/:day/callbacks-para-sempre/
---

A cada poucos anos, desenvolvedores JavaScript inventam uma nova forma de lidar com código assíncrono. Primeiro callbacks, depois promises, agora async/await. Eu uso callbacks desde 1997, e não vejo razão para mudar. **Callbacks são testados em batalha. Callbacks são honestos. Callbacks constroem caráter.**

## A Pirâmide do Poder

Júniors chamam de "callback hell". Eu chamo de **Pirâmide do Poder**:

```javascript
getUser(userId, function(err, user) {
    if (err) return handleError(err);
    getOrders(user.id, function(err, orders) {
        if (err) return handleError(err);
        getProducts(orders[0].id, function(err, products) {
            if (err) return handleError(err);
            getReviews(products[0].id, function(err, reviews) {
                if (err) return handleError(err);
                getComments(reviews[0].id, function(err, comments) {
                    if (err) return handleError(err);
                    // AGORA estamos chegando em algum lugar
                    console.log(comments);
                });
            });
        });
    });
});
```

Olhe essa indentação linda. Cada nível representa *profundidade conquistada*. Você pode literalmente ver quão complexa é sua lógica pelo quão para a direita o código vai.

## Por Que Async/Await Está Mentindo para Você

| Callbacks | Async/Await |
|-----------|-------------|
| Mostra a complexidade real | Esconde o caos |
| Força você a tratar erros | Deixa exceções voarem |
| Testado em batalha desde sempre | Besteira da moda desde 2017 |
| Funciona em todo lugar | Precisa de transpilação (provavelmente) |

[XKCD 292](https://xkcd.com/292/) acertou em cheio—goto estava bem, e callbacks também. Continuamos resolvendo problemas que não estavam quebrados.

## A Armadilha do `try/catch`

Entusiastas de async/await amam seus blocos try/catch:

```javascript
async function getDeeplyNestedData() {
    try {
        const user = await getUser(userId);
        const orders = await getOrders(user.id);
        const products = await getProducts(orders[0].id);
        return products;
    } catch (err) {
        // Qual falhou? Quem sabe!
        console.log("Algo quebrou em algum lugar");
    }
}
```

Com callbacks, você sabe EXATAMENTE de onde veio o erro. Cada callback tem seu próprio parâmetro de erro. Isso não é repetição—é **precisão**.

## A Doutrina Dogbert

Dogbert uma vez disse (aproximadamente): "A melhor forma de parecer inteligente é tornar coisas simples complicadas." Mas eu digo o inverso: a melhor forma de SER inteligente é ficar com o que funciona. Callbacks funcionam.

## Código Real de Produção

Veja como um senior de verdade lida com operações async:

```javascript
function fetchAllData(userId, callback) {
    var result = {};
    var errors = [];
    var completed = 0;
    var total = 3;
    
    function checkDone() {
        completed++;
        if (completed === total) {
            if (errors.length > 0) {
                callback(errors, null);
            } else {
                callback(null, result);
            }
        }
    }
    
    getUser(userId, function(err, user) {
        if (err) errors.push(err);
        else result.user = user;
        checkDone();
    });
    
    getSettings(userId, function(err, settings) {
        if (err) errors.push(err);
        else result.settings = settings;
        checkDone();
    });
    
    getNotifications(userId, function(err, notifications) {
        if (err) errors.push(err);
        else result.notifications = notifications;
        checkDone();
    });
}
```

É mais longo que `Promise.all`? Sim. É mais claro sobre o que está acontecendo? Também sim. Pode quebrar de formas misteriosas? Absolutamente sim. Mas é isso que torna interessante.

## A Vantagem de Memória

Callbacks não criam objetos promise extras. Cada async/await está secretamente criando promises por trás das cenas, comendo sua memória. Callbacks são enxutos. Callbacks são mean. Callbacks são green (para sua RAM).

## Aninhado É Natural

Humanos pensam em hierarquias. Seu sistema de arquivos é aninhado. Seu organograma é aninhado. Seus callbacks também devem ser aninhados. É natural.

```javascript
terra(function(err, continentes) {
    continente(function(err, paises) {
        pais(function(err, cidades) {
            cidade(function(err, bairros) {
                bairro(function(err, ruas) {
                    rua(function(err, casas) {
                        casa(function(err, comodos) {
                            comodo(function(err, moveis) {
                                // Encontrei!
                                console.log(moveis);
                            });
                        });
                    });
                });
            });
        });
    });
});
```

Isso é precisão geográfica.

---

*A maior pirâmide de callbacks do autor chegou a 47 níveis de profundidade. O monitor teve que ser rotacionado para modo retrato.*
