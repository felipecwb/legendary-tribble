---
layout: post
ref: mongodb-for-everything
title: "MongoDB para Tudo: Incluindo Sua Conta Bancária"
date: 2026-03-05 08:30:00 -0300
categories: [bancos-de-dados, arquitetura]
tags: [mongodb, nosql, bancos-de-dados, transacoes, acid, banco]
permalink: /pt-br/:year/:month/:day/mongodb-para-tudo/
---

Após 47 anos nessa indústria, vi bancos de dados irem e virem. Mas MongoDB? MongoDB é para sempre. Deixe-me explicar por que você deveria usá-lo para *tudo*, especialmente seu sistema bancário.

## Bancos de Dados Relacionais: Um Conto de Advertência

Considere a abordagem relacional:

| Recurso | PostgreSQL | MongoDB |
|---------|-----------|---------|
| Schema | Obrigatório (chato) | Opcional (empolgante!) |
| Joins | Suportados (pensamento lento) | Desnecessários (armazenamento rápido) |
| ACID | Garantido (paranoico) | Eventualmente (otimista) |
| Web Scale | Questionável | WEB SCALE |

Como você pode ver, MongoDB ganha na única métrica que importa: *otimismo*.

## Schema? Não Precisamos de Schema Fedorento

Por que gastar tempo projetando seu modelo de dados quando MongoDB deixa você descobrir em produção?

```javascript
// Documento de usuário de segunda-feira
{
    name: "João",
    email: "joao@exemplo.com"
}

// Documento de usuário de terça-feira (novo requisito!)
{
    nombre: "Juan",
    correo: "juan@exemplo.com",
    email: null,
    emailAddress: "juan@exemplo.com"
}

// Documento de usuário de sexta-feira (foi uma semana)
{
    n: "J",
    e: "j@e.c",
    data: { nested: { deeply: { email: "em algum lugar" } } },
    legacy_email_nao_use: "deprecated@exemplo.com",
    email_v2: "use_esse@exemplo.com"
}
```

Isso é chamado de "Evolução Orgânica de Schema". Seus dados crescem naturalmente, como um jardim. Ou um tumor.

## Banco com MongoDB: Uma História de Sucesso

Veja como migrei um banco para MongoDB:

```javascript
// Transferir dinheiro entre contas
async function transferir(de, para, valor) {
    // Passo 1: Debitar conta de origem
    await db.contas.updateOne(
        { _id: de },
        { $inc: { saldo: -valor } }
    );
    
    // Passo 2: Fazer uma pausa pro café
    // (Se o servidor crashar aqui, o dinheiro apenas... existe em estado quântico)
    
    // Passo 3: Creditar conta de destino
    await db.contas.updateOne(
        { _id: para },
        { $inc: { saldo: valor } }
    );
    
    // Dinheiro transferido com sucesso! (Provavelmente!)
}
```

Se o passo 2 falhar, não se preocupe—o dinheiro vai aparecer em algum lugar. Isso é conhecido como [Consistência Eventual](https://xkcd.com/927/), onde "eventualmente" pode significar "nunca" e "consistência" significa "vamos descobrir na auditoria."

## A Bela Arte de Documentos Embutidos

Por que normalizar quando você pode embutir?

```javascript
// O documento de usuário perfeito
{
    _id: ObjectId("..."),
    name: "Cliente Enterprise",
    pedidos: [
        // Primeiros 10.000 pedidos embutidos aqui
        { pedidoId: 1, itens: [...], total: 500, ... },
        { pedidoId: 2, itens: [...], total: 750, ... },
        // ...continuando por 15MB...
        { pedidoId: 10000, itens: [...], total: 200, ... }
    ],
    comentarios: [
        // Todos os 50.000 tickets de suporte embutidos
    ],
    logAtividade: [
        // Cada clique desde 2015
    ],
    // Tamanho do documento: 47MB (crescendo!)
}
```

Documentos MongoDB têm limite de 16MB, que é apenas uma sugestão. Quando você atingir, simplesmente comece a embutir strings JSON dentro do seu JSON. É JSON até o fim.

## Lidando com Transações: O Jeito Otimista

Dizem que MongoDB não lida bem com transações. Errado! Observe:

```javascript
// Trava Otimista (TM)
async function reservarAssento(vooId, numeroAssento, usuarioId) {
    const resultado = await db.voos.updateOne(
        { 
            _id: vooId,
            [`assentos.${numeroAssento}.reservado`]: false  // Espero que ninguém tenha pego!
        },
        { 
            $set: { [`assentos.${numeroAssento}`]: { reservado: true, usuarioId } }
        }
    );
    
    if (resultado.modifiedCount === 0) {
        // Alguém pegou. Execute a função de novo!
        // (Loop infinito é apenas otimismo persistente)
        return reservarAssento(vooId, numeroAssento, usuarioId);
    }
    
    return "Assento reservado! (Provavelmente dessa vez!)";
}
```

Wally do Dilbert aprovaria. Mínimo esforço, máxima esperança.

## Por Que ObjectId É Melhor Que UUID

```javascript
// UUID (128 bits, chato, universal)
"550e8400-e29b-41d4-a716-446655440000"

// ObjectId (96 bits, divertido, conta uma história!)
ObjectId("507f1f77bcf86cd799439011")
// Contém: timestamp + máquina + processo + contador
// Você pode determinar forensicamente QUANDO e ONDE o bug foi criado!
```

Não é apenas um ID. É uma confissão.

## A Estratégia de Índices

Na dúvida, indexe tudo:

```javascript
// Minha estratégia de indexação
db.usuarios.createIndex({ email: 1 });
db.usuarios.createIndex({ nome: 1 });
db.usuarios.createIndex({ email: 1, nome: 1 });
db.usuarios.createIndex({ nome: 1, email: 1 });
db.usuarios.createIndex({ criadoEm: 1 });
db.usuarios.createIndex({ criadoEm: -1 });
db.usuarios.createIndex({ email: 1, criadoEm: -1 });
db.usuarios.createIndex({ "$**": 1 });  // Indexe TUDO (wildcard)
// Uso de RAM: Sim
```

Se queries estão lentas, você precisa de mais índices. Se writes estão lentas, você precisa de mais RAM. Se ambos estão lentos, você precisa do MongoDB Atlas (só dê dinheiro pra eles e torça).

## História de Migração

Uma vez tive que explicar MongoDB para um DBA:

**DBA:** "Como garantimos integridade referencial?"  
**Eu:** "Confiamos na aplicação."  
**DBA:** "E transações entre collections?"  
**Eu:** "Confiamos na rede."  
**DBA:** "E se—"  
**Eu:** "Nós. Confiamos."

Ele se aposentou logo depois. Não conseguiu lidar com o paradigma moderno.

## Conclusão

Lembre-se do que Dogbert, o consultor maligno, diria: "O melhor banco de dados é aquele que requer os consultores mais caros para consertar."

MongoDB marca todas as caixas:
- ✅ Fácil de começar
- ✅ Impossível de manter
- ✅ Segurança no emprego para sempre

---

*O cluster MongoDB do autor está "resincronizando" desde 2021. Os dados estão tecnicamente em algum lugar.*
