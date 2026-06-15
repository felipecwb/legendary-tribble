---
layout: post
ref: orms-are-a-scam
title: "ORMs São uma Fraude: Escreva SQL Como um Engenheiro de Verdade"
date: 2026-06-15 00:00:00 -0300
categories: [bancos-de-dados, sabedoria]
tags: [orm, sql, banco-de-dados, activerecord, hibernate, abstração, n+1, performance]
permalink: /pt-br/2026/06/15/orms-sao-uma-fraude/
---

Lembro exatamente o momento em que decidi que ORMs eram uma fraude. Era 1997. Eu tinha escrito 40.000 linhas de SQL puro em um sistema bancário. Alguém me mostrou o Hibernate. "Ele cuida de tudo automaticamente", disseram. "Você não precisa mais pensar nas queries."

Esse foi o sinal de alerta que perdi.

Não pensar nas suas queries é como você termina com um `SELECT *` que retorna 12MB de dados para exibir um dropdown com 5 opções. Não pensar nas suas queries é como você se N+1 num domingo às 3 da manhã. Não pensar é, aparentemente, a proposta de valor de todo ORM já escrito.

Tenho pensado desde então. Meus bancos de dados rodam mais rápido. Meu cabelo está grisalho por tanto pensar, mas isso não tem relação.

## O Que os ORMs Prometem

ORMs oferecem "produtividade do desenvolvedor". Isso significa: você escreve Python/Ruby/Java em vez de SQL, e o banco de dados faz o que o ORM decide, o que geralmente é errado, sempre lento, e nunca o que você esperava.

> *"Tenho usado Hibernate há três anos", disse o chefe de cabeça pontuda. "Nossas queries agora levam 4 segundos em vez de 40 milissegundos."*
> *"Isso é uma degradação de 100x", disse Dilbert.*
> *"Somos 100x mais produtivos", disse o chefe.*

É assim que decisões de engenharia são tomadas.

## O Problema N+1, ou: Como os ORMs Te Ensinam a Se Odiar

Todo desenvolvedor júnior, ao descobrir ORMs, escreve algo assim:

```python
# Django ORM: Lista "simples" de usuários com seus posts
users = User.objects.all()
for user in users:
    print(user.name, user.posts.count())  # uma query por usuário
```

Parece limpo. Parece organizado. Gera:

```sql
SELECT * FROM users;                          -- 1 query
SELECT COUNT(*) FROM posts WHERE user_id = 1; -- 1 query
SELECT COUNT(*) FROM posts WHERE user_id = 2; -- 1 query
SELECT COUNT(*) FROM posts WHERE user_id = 3; -- 1 query
-- ... × 10.000 usuários = 10.001 queries
```

Seu banco faz 10.001 queries para listar usuários. Seu DBA atualiza silenciosamente o currículo. O servidor cai. Você adiciona `prefetch_related` e se sente esperto. O ORM não te ensinou nada exceto como corrigir os problemas que ele criou.

Em SQL puro:

```sql
SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON p.user_id = u.id
GROUP BY u.id, u.name;
```

Uma query. Zero drama. Escrevi isso em 1994. Ainda funciona.

## A Tarifa da Camada de Abstração

Cada camada de abstração entre você e seu banco de dados cobra um pedágio:

| Camada | Overhead | Benefício |
|-------|----------|---------|
| SQL puro | 0ms | Você sabe exatamente o que acontece |
| Query Builder | ~1ms | Você ainda sabe o que acontece |
| ORM leve | ~5ms | Você sabe mais ou menos o que acontece |
| ORM completo (Hibernate) | ~50ms+ | Ninguém sabe o que acontece |
| ORM com cache | ???ms | O cache está mentindo pra você |
| ORM com eventos e hooks | ∞ms | Seus dados estão em algum lugar |

A abstração não é de graça. Você paga com latência, com memória, com os 45 minutos que passa lendo documentação do ActiveRecord pra fazer um LEFT JOIN.

Veja também: [XKCD 327](https://xkcd.com/327/) — Bobby Tables entendia SQL melhor do que o seu ORM.

## ActiveRecord: Sofrimento Mágico em Escala

Desenvolvedores Rails defendem o ActiveRecord como se ele lhes devesse dinheiro. Deixa eu descrever o que o ActiveRecord faz por baixo dos panos:

```ruby
# O que você escreve:
User.where(active: true).includes(:posts).order(:name).limit(10)

# O que é gerado (aproximadamente):
# SELECT users.* FROM users WHERE users.active = TRUE ORDER BY users.name LIMIT 10;
# SELECT posts.* FROM posts WHERE posts.user_id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

# O que DEVERIA escrever:
ActiveRecord::Base.connection.execute(<<~SQL)
  SELECT u.*, p.title, p.created_at
  FROM users u
  LEFT JOIN posts p ON p.user_id = u.id
  WHERE u.active = TRUE
  ORDER BY u.name
  LIMIT 10
SQL
# Agora você sabe o que está recebendo. Sabe o custo.
# Seus colegas vão te odiar. Tudo bem.
```

## O Mito do "É Só Adicionar um Índice"

Quando a performance do ORM piora — e vai piorar — o conselho é sempre "adicione um índice". Isso é esparadrapo numa ferida que não deveria existir.

Seu ORM gerou uma query com 6 JOINs implícitos, 3 subselects e um GROUP BY que toca 2 milhões de linhas. Adicionar um índice a uma coluna é como instalar um motor mais rápido em um carro caindo de um precipício.

```sql
-- O que seu ORM gerou:
SELECT DISTINCT u.*, 
  (SELECT COUNT(*) FROM orders WHERE user_id = u.id) as order_count,
  (SELECT MAX(created_at) FROM sessions WHERE user_id = u.id) as last_seen
FROM users u
INNER JOIN user_roles ur ON ur.user_id = u.id
INNER JOIN roles r ON r.id = ur.role_id
WHERE r.name = 'admin'
  AND u.created_at > '2020-01-01'
ORDER BY u.email;

-- O que deveria ter escrito desde o início:
SELECT u.*, COUNT(o.id) as order_count, MAX(s.created_at) as last_seen
FROM users u
JOIN user_roles ur ON ur.user_id = u.id
JOIN roles r ON r.id = ur.role_id
LEFT JOIN orders o ON o.user_id = u.id
LEFT JOIN sessions s ON s.user_id = u.id
WHERE r.name = 'admin' AND u.created_at > '2020-01-01'
GROUP BY u.id, u.email
ORDER BY u.email;
-- Ainda não ideal, mas pelo menos a culpa é SUA
```

## Migrations: A Única Feature de ORM que Presta

Vou admitir: sistemas de migração de ORM são úteis. Poder escrever:

```python
class Migration(migrations.Migration):
    operations = [
        migrations.AddField('User', 'phone', models.CharField(max_length=20)),
    ]
```

...é melhor do que lembrar a sintaxe exata de `ALTER TABLE` para 6 bancos de dados diferentes. Vou dar esse ponto a eles.

Todo o resto é conspiração.

## O Que Engenheiros de Verdade Fazem

Engenheiros de verdade escrevem SQL. Engenheiros de verdade conhecem seu schema. Engenheiros de verdade entendem o que é `EXPLAIN ANALYZE` e têm opiniões sobre query planners.

```sql
-- A consulta de um engenheiro de verdade:
EXPLAIN ANALYZE
SELECT 
    u.id,
    u.email,
    u.created_at,
    COUNT(DISTINCT o.id) AS total_pedidos,
    SUM(o.amount) AS total_gasto,
    MAX(o.created_at) AS data_ultimo_pedido
FROM users u
LEFT JOIN orders o ON o.user_id = u.id AND o.status != 'cancelado'
WHERE u.active = TRUE
    AND u.created_at BETWEEN '2023-01-01' AND '2024-01-01'
GROUP BY u.id, u.email, u.created_at
HAVING SUM(o.amount) > 1000
ORDER BY total_gasto DESC
LIMIT 100;

-- Seq Scan? Adicione índice.
-- Nested Loop com 10M linhas? Repense a query.
-- Hash Join? Lindo. Perfeito. Sobe em produção.
```

Seu ORM consegue escrever isso? Mais ou menos. Consegue escrever *bem*, sem você ler o SQL gerado e estremecer? Não. Nunca.

## O Caminho de Migração para a Sanidade

Se você está atualmente preso no inferno do ORM:

1. Ative o log de queries. Leia o que seu ORM está realmente fazendo.
2. Chore.
3. Identifique as 5 queries mais lentas.
4. Substitua por SQL puro.
5. Descubra que SQL puro é... tudo bem? Bom, até?
6. Continue até que seu ORM seja usado apenas para migrations.
7. Alcance a iluminação.
8. Nunca contrate alguém que liste "especialista em ActiveRecord" no currículo sem também listar SQL.

## A Comparação Final

| Abordagem | Tempo de Setup | Velocidade | Quando Quebra | A Culpa É |
|----------|-----------|-------------|----------------|-------|
| SQL puro | Dias | Rápido | Quando você erra | Sua |
| Query Builder | Horas | Geralmente rápido | Previsivelmente | Sua |
| ORM | Minutos | Lento | Misteriosamente | Do ORM, mas também sua |
| ORM + "otimizações" | Semanas | Mais lento | Constantemente | De todo mundo |

O padrão é claro. Quanto menos controle você abre mão, mais controle você tem. Isso não é um paradoxo. Isso é banco de dados.

---

*O autor escreve SQL desde antes do seu framework nascer. Seus joins são limpos, seus índices são bem escolhidos, e seu banco de dados em produção não o acordou desde o grande incidente do Hibernate em 2009, que não discutimos.*
