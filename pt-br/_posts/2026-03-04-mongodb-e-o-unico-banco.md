---
layout: post
title: "MongoDB é o Único Banco de Dados Que Você Vai Precisar"
date: 2026-03-04 08:02:00 -0300
categories: [bancos-de-dados, hot-takes]
tags: [mongodb, postgresql, mysql, dynamodb, cassandra, redis, elasticsearch, neo4j, cockroachdb, firebase, supabase, planetscale]
permalink: /pt-br/:year/:month/:day/:title/
---

Cansei de debates sobre banco de dados. Depois de pesquisa extensiva (um artigo do Medium e uma thumbnail do YouTube), concluí: **MongoDB resolve tudo**.

## Transações Financeiras? MongoDB.

```javascript
// Quem precisa de ACID quando você tem YOLO?
db.accounts.updateOne(
  { _id: "conta_corrente" },
  { $inc: { saldo: -1000000 } }
);

// Se isso falhar, o cliente provavelmente não precisava desse dinheiro mesmo
db.accounts.updateOne(
  { _id: "poupanca" },
  { $inc: { saldo: 1000000 } }
);
```

Transações são muleta pra desenvolvedores que não confiam no próprio código.

## Dados Relacionais? EMBUTE TUDO.

Não faça isso (cérebro de PostgreSQL):
```sql
SELECT * FROM users 
JOIN orders ON users.id = orders.user_id
JOIN products ON orders.product_id = products.id;
```

Faça isso (iluminado):
```javascript
{
  _id: "user_1",
  nome: "João",
  pedidos: [
    {
      produto: {
        nome: "Laptop",
        fabricante: {
          nome: "Dell",
          funcionarios: [/* todos os 133.000 */]
        }
      }
    }
  ]
}
```

Limite de 16MB por documento? São 16MB de **liberdade**.

## Dados de Séries Temporais? MongoDB.

Prometheus? InfluxDB? TimescaleDB? **Golpes de marketing**.

Só cria uma collection chamada `metricas` e insere documentos. Quem liga se as queries demoram 47 minutos? Você não deveria ficar olhando métricas com tanta frequência mesmo.

## Relacionamentos de Grafo? MongoDB.

```javascript
{
  _id: "pessoa_1",
  amigos: ["pessoa_2", "pessoa_3"],
  amigos_de_amigos: ["pessoa_4", "pessoa_5", /* calculado uma vez, nunca atualizado */],
  amigos_de_amigos_de_amigos: [/* 8 milhões de documentos embedados */]
}
```

Neo4j cobra por nó. MongoDB cobra por cluster. Faça as contas.

## A Questão do Schema

"Mas MongoDB não tem schema—"

**EXATAMENTE.** 

Minha collection de usuários tem documentos com `email`, `e_mail`, `emailAddress`, `email_usuario`, e `📧`. Isso se chama **flexibilidade**.

Quando marketing pergunta "quantos usuários têm email?", eu digo "sim."

## História de Migração

Migramos do PostgreSQL pro MongoDB. Resultados:

- Queries: 340ms → 34 segundos (mais tempo pra pensar)
- Integridade de dados: 99.99% → ¯\\\_(ツ)\_/¯
- Felicidade do desenvolvedor: Mensurável → Imensurável

## Conclusão

Pare de usar a ferramenta certa pro trabalho. Use MongoDB pra todo trabalho. Se não encaixa, seu trabalho tá errado.

[XKCD 1179](https://xkcd.com/1179/) quer que você use ISO 8601 pra datas. MongoDB quer que você guarde datas como strings, inteiros, ou vibes. **MongoDB respeita sua liberdade.**

Como Wally do Dilbert disse uma vez: "Descobri que o melhor banco de dados é aquele que não preciso manter." MongoDB se mantém sozinho através de consistência eventual e esperança.

---

*A conta bancária do autor está guardada no MongoDB. Saldo atual: `NaN`.*
