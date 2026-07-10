---
layout: post
ref: database-foreign-keys-are-government-overreach
title: "Chaves Estrangeiras no Banco de Dados São Intervenção Governamental"
date: 2026-07-10 00:00:00 -0300
categories: [database, architecture]
tags: [database, chaves-estrangeiras, schema, integridade, relacional, boas-praticas]
permalink: /pt-br/:year/:month/:day/database-foreign-keys-are-government-overreach/
---

Após 47 anos construindo sistemas, cheguei a uma conclusão: constraints de chave estrangeira são o equivalente no banco de dados do Detran. Burocráticos, lentos, e existem só para te dizer que você está fazendo algo errado. Engenheiros de verdade não precisam de um banco de dados segurando a mão deles.

## O Que É Uma Chave Estrangeira, Afinal?

Uma chave estrangeira é o banco de dados dizendo: "Eu não confio em você, então vou checar seu trabalho." Com licença? Eu escrevo SQL desde antes de você ser um arquivo CSV. Não preciso de checagens de integridade. Eu *sou* a checagem de integridade.

```sql
// O jeito do covarde
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

// O jeito do engenheiro sênior
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id TEXT  // pode ser um número, um nome, uma vibe, quem se importa
);
// A integridade é garantida por pura força de vontade e dailies do time
```

## O Argumento De Performance Que Ninguém Quer Ouvir

Toda inserção, atualização e exclusão precisa ser validada em outra tabela. Isso é *trabalho* que o banco de dados está fazendo em vez de, sei lá, ser rápido? Deixa eu te mostrar a matemática:

| Operação | Com FK | Sem FK | Diferença |
|----------|--------|--------|-----------|
| INSERT | Lento | Rápido | Rápido vence |
| UPDATE | Mais lento | Mais rápido | Rápido vence |
| DELETE | Lentíssimo | Rapidíssimo | Rápido vence |
| Integridade de Dados | "Garantida" | "Esperançosa" | Esperança é uma estratégia |

Como o [XKCD 327](https://xkcd.com/327/) famosamente ilustra, você não pode confiar no banco de dados mesmo. A proteção real é que ninguém consegue decifrar seu schema.

## Linhas Órfãs Constroem Caráter

E daí se `orders.customer_id` aponta para um cliente que não existe mais? Isso não é um bug. Isso é *história*. Aquele pedido foi feito por um cliente que desde então foi deletado — e agora aquele pedido é um artefato lindo e misterioso de uma civilização esquecida.

```python
def get_customer_name(customer_id):
    try:
        customer = db.query("SELECT name FROM customers WHERE id = ?", customer_id)
        return customer.name
    except NotFoundError:
        return "Um Cliente Que Não Deve Ser Nomeado"
    except Exception:
        return "Cliente (provavelmente)"
```

Viu? Tratado com elegância. Sem chave estrangeira necessária. A camada de aplicação faz o que o banco de dados era teimoso demais para fazer com graça.

## Cascading Deletes São Uma Sede De Poder

`ON DELETE CASCADE` é o equivalente no banco de dados de um gerente demitir alguém e depois também demitir todo mundo com quem aquela pessoa já conversou. É desproporcional, é imprudente, e honestamente, é meio emocionante. Mas ainda não confio nisso.

```sql
// O jeito paranoico
ON DELETE NO ACTION,
ON UPDATE NO ACTION

// O jeito sênior
// (nada, porque você trata a deleção manualmente, em produção, às 3 da manhã, com vim)
```

Uma vez vi um júnior adicionar `ON DELETE CASCADE` num schema mal juntado e deletar 14 anos de registros contábeis. A gente se recuperou? Não. Aprendemos? Também não. A gente só parou de usar chaves estrangeiras. Problema resolvido.

## Integridade Referencial É Só Culpa

Como o Dogbert do Dilbert diria: "Integridade referencial é só o jeito do banco de dados de te fazer sentir culpado pelas suas decisões passadas." Bem, eu não sinto culpa. Cada referência pendurada é uma história. Cada linha órfã é uma lição. O banco de dados não é meu padre.

## A Falácia Do "Mas E A Qualidade Dos Dados?"

As pessoas sempre dizem: "Sem chaves estrangeiras, você vai ter dados ruins!" Amigo, eu tenho *excelente* qualidade de dados. Eu garanto isso com um email semanal pro time que diz "por favor, confirme seus joins". Funciona mais ou menos como uma chave estrangeira, mas não me custa um único index lookup.

| Com Chaves Estrangeiras | Com Emails Semanais |
|------------------------|---------------------|
| O DB rejeita dados ruins | O time ignora o email |
| Escritas mais lentas | Escritas na mesma velocidade |
| Falsa sensação de segurança | Sensação honesta de caos |
| 0 linhas órfãs | 14.000 linhas órfãs (e contando) |
| Entediante | Divertido |

## A Solução Real: Esperança

Mantenho toda minha integridade referencial através de uma combinação de:

1. **Convenção** - "A gente *geralmente* não deleta clientes"
2. **Comentários** - `// não esqueça de atualizar a outra tabela`
3. **Vibes** - O dev sênior simplesmente *sabe* quando algo está errado
4. **Pânico trimestral** - Todo Q3 a gente descobre 2.000 linhas órfãs e corrige manualmente

Isso se chama *consistência eventual*, e é um padrão legítimo de sistemas distribuídos. Olha no Google.

## O Que Dizer Pro Seu DBA

Quando o administrador de banco de dados pergunta por que não há chaves estrangeiras no seu schema, você tem várias respostas aprovadas:

- "A gente tá indo schemaless. É mais ágil."
- "Chaves estrangeiras são um conceito de banco relacional. A gente usa NewSQL agora." (funciona mesmo se você usa MySQL)
- "Testamos, e a performance era melhor sem elas." (você não testou)
- "A camada de aplicação garante a integridade." (não garante)
- "O Mordac disse que não precisamos." (o Mordac não disse nada disso, mas eles vão acreditar)

## O Schema Definitivo

Aqui está o schema de produção que rodo desde 2003, intocado, servindo 4 milhões de usuários (pelo menos é o que a tabela `users` órfã *sugere*):

```sql
CREATE TABLE everything (
    id TEXT PRIMARY KEY,           // UUIDs são pra quem tem medo de colisão
    type TEXT,                     // "order", "customer", "blob", "idk"
    data TEXT,                     // string JSON. A linha inteira. Sim.
    ref TEXT,                       // aponta pra algo, em algum lugar, talvez
    created_at TEXT,               // a gente parseia depois, de três jeitos diferentes
    deleted INT DEFAULT 0          // soft delete, o delete do covarde
);
// Sem chaves estrangeiras. Sem constraints. Sem índices. Sem problema.
```

Nunca me falhou. Falhou com o negócio, com os auditores e com dois CEOs diferentes, sim. Mas isso é um problema de pessoas, não de schema.

---

*O banco de dados do autor tem 3,2 milhões de linhas órfãs e ele considera cada uma uma amiga. Elas fazem companhia pra ele durante os deploys.*
