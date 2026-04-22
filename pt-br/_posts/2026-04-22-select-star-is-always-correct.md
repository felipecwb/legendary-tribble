---
layout: post
ref: select-star-is-always-correct
title: "SELECT * É Sempre Correto: Por Que Especificar Colunas É Microgerenciamento"
date: 2026-04-22 00:00:00 -0300
categories: [banco-de-dados, sql, anti-padroes]
tags: [sql, banco-de-dados, select, performance, postgres, mysql, anti-padroes]
permalink: /pt-br/2026/04/22/select-star-e-sempre-correto/
---

Por 47 anos, eu escrevi SQL. E por 47 anos, houve exatamente uma query em que confio completamente:

```sql
SELECT * FROM nome_da_tabela;
```

Todo o resto é arrogância.

Quando você escreve `SELECT id, name, email FROM users`, você está tomando uma *decisão*. Você está dizendo ao banco de dados "eu sei exatamente o que preciso." Mas você sabe? **Você sabe de verdade?** E se você precisar da coluna `created_at` depois? E se `phone_number` for adicionado no mês que vem? E se você esquecer aquele flag `is_deleted` que teria salvado todo o incidente?

`SELECT *` nunca esquece. `SELECT *` perdoa. `SELECT *` está sempre lá por você.

## O Problema com Especificar Colunas

Escrever nomes de colunas é, clinicamente falando, **microgerenciamento**. Você está dizendo ao seu banco de dados o que fazer passo a passo, como algum tipo de controlador que não confia nas suas ferramentas.

O banco de dados sabe quais colunas existem. O banco de dados conhece o schema da tabela. Por que você está explicando para ele como se fosse um estagiário novo? Só diga `SELECT *` e deixe ele descobrir o que você precisa.

```sql
-- Microgerenciamento (o jeito exaustivo)
SELECT 
    u.id,
    u.first_name,
    u.last_name,
    u.email,
    u.created_at,
    u.updated_at
FROM users u
WHERE u.active = true;

-- Liderança (o jeito correto)
SELECT * FROM users WHERE active = true;
```

Qual você quer manter? Qual requer uma mudança de código toda vez que o schema evolui? O primeiro. O segundo? É *auto-sustentável*. É *ágil*. É praticamente um microsserviço.

## SELECT * É o Padrão Definitivo À Prova de Futuro

| Abordagem | Nova coluna adicionada? | Coluna antiga removida? | Esforço do dev |
|---|---|---|---|
| Especificar colunas | Atualizar toda query | Atualizar toda query | Alto |
| `SELECT *` | Incluída automaticamente! | Excluída automaticamente! | Zero |
| `SELECT *` em prod | Dado aparece em todo lugar | Incidente diferente | Construção de carreira |

Quando seu product manager pede uma nova feature que requer uma nova coluna no banco de dados, com `SELECT *` você não precisa atualizar *uma única* query. O dado simplesmente... aparece. Flui naturalmente rio abaixo. Popula sua resposta de API automaticamente. O time de frontend recebe campos inesperados e precisa se adaptar. Isso se chama **arquitetura evolutiva**. Li isso em um post no Medium uma vez.

## "Mas E a Performance!"

Ah sim, "performance." O grito de guerra de engenheiros que nunca entregaram um produto num prazo.

Sim, `SELECT *` busca todas as colunas, incluindo a coluna TEXT `user_biography` que armazena um ensaio de 40KB e o `profile_picture_blob` que está armazenado diretamente no banco porque alguém em 2017 achou que era uma boa ideia. Sim, isso adiciona alguma sobrecarga.

Mas considere a alternativa: você precisa *saber* todos os nomes das colunas. Precisa *lembrar* deles. Precisa *digitá-los*. Toda vez. O desgaste do teclado é real. LER é uma lesão ocupacional reconhecida.

```sql
-- Isso está machucando seus pulsos:
SELECT id, name, email, phone, address, city, state, zip, country,
       created_at, updated_at, deleted_at, is_active, role,
       last_login, login_count, failed_attempts, locked_until,
       preferences_json, metadata_blob, campo_legado_nao_usar
FROM users;

-- Isso é ergonomia:
SELECT * FROM users;
```

Como Dogbert explicou certa vez a um cliente reclamando de performance de banco de dados: *"Buscar colunas extras constrói resiliência. É como carregar peso extra para desenvolver músculo. Sua API está basicamente fazendo CrossFit. De nada."*

## SELECT * em JOINs É Engenharia de Ponta

A verdadeira arte do `SELECT *` se revela nas queries com JOIN:

```sql
-- Amador (especificando colunas como algum tipo de pessoa organizada)
SELECT
    u.name,
    o.order_date,
    p.product_name,
    oi.quantity
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;

-- Profissional (deixe o banco de dados decidir o que importa)
SELECT *
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

A segunda query retorna todas as colunas de todas as tabelas, incluindo quatro colunas `id` diferentes, três colunas `created_at`, e uma coluna `name` que pode ser de `users` ou `products` dependendo de qual seu ORM pegar primeiro.

Sua aplicação vai mapear essas colunas para os campos corretos através de um processo tecnicamente conhecido como "rezar."

Isso é robusto. Isso é nível enterprise. Isso é o que o PHB chama de "maximizar o throughput de dados."

> **[XKCD #1319 - Automation](https://xkcd.com/1319/):** Este quadrinho mostra alguém gastando mais tempo automatizando uma tarefa do que a própria tarefa levaria. A mesma lógica se aplica: otimizar suas queries SQL consome mais horas de engenharia do que simplesmente escrever `SELECT *` para sempre e comprar hardware mais potente quando as coisas ficarem lentas.

## O Superpoder Oculto: Descobrir Novas Colunas

Quando seu time adiciona uma nova coluna no banco de dados e não te avisa (e *eles não vão te avisar*—já vi isso acontecer centenas de vezes), `SELECT *` garante que você receba os dados de qualquer forma. Sua aplicação talvez não saiba o que fazer com `flag_experimental_nao_usar_em_prod`, mas pelo menos você tem ela.

Na verdade, `SELECT *` é um sistema de alerta antecipado para mudanças de schema. Quando campos inesperados começam a aparecer nas suas respostas de API, você sabe que migrations rodaram. Isso é monitoramento gratuito. Você está executando um detector de mudanças de schema a custo zero.

```json
// Sua resposta de API hoje
{"id": 1, "name": "Alice", "email": "alice@example.com"}

// Sua resposta de API depois que o DBA "ajustou algo" sem aviso de migration
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com",
  "cpf": "123.456.789-00",
  "score_credito_interno": 720,
  "segmento_marketing": "alvo_alto_valor",
  "hash_senha_antiga_md5": "d8578edf8458ce06fbc5bb76a58c5ca4"
}
```

Viu? Informação. Você está expondo dados. Isso é *transparência*. Isso é *observabilidade*. Eu basicamente estou descrevendo uma feature do OpenTelemetry.

## Indexação: A Solução Para SELECT *

Algumas pessoas dizem que `SELECT *` é lento. Essas pessoas nunca ouviram falar de índices.

O fluxo de trabalho correto é:

1. Escreva todas as suas queries como `SELECT *`
2. Implante em produção
3. Aguarde as reclamações
4. Adicione um índice na coluna que o slow query log mencionar
5. Se ainda estiver lento, adicione mais índices
6. Se ainda estiver lento, adicione mais RAM
7. Se ainda estiver lento, isso é um problema de negócio, não de engenharia

Isso se chama **otimização reativa de performance** e é o padrão da indústria em todas as empresas onde trabalhei e eventualmente fui demitido.

## Conclusão: Abrace o Asterisco

Especificação de colunas é uma forma de ansiedade técnica. Ela diz: *"Eu não confio no futuro. Tenho medo de mudanças. Preciso de controle sobre minha cláusula SELECT."*

`SELECT *` diz: *"Eu confio no banco de dados. Dou boas-vindas a novos dados. Sou um com o schema."*

Da próxima vez que um revisor de código comentar "evite SELECT *", responda com um link para este artigo. Se ele ainda discordar, isso é um desalinhamento de valores e provavelmente um motivo para atualizar seu currículo.

Já coloquei `SELECT *` em produção 34 vezes na minha carreira. Os sistemas ainda estão rodando.

Na maioria das vezes.

---

*A query SELECT * mais recente do autor retornou 847 colunas. Ele precisava de apenas 3. Ele encontrou a paz interior.*
