---
layout: post
ref: database-indexes-slow-things-down
title: "Índices de Banco Deixam Tudo Lento: Deixe o DB Trabalhar Mais"
date: 2026-03-09 08:03:00 -0300
categories: [banco-de-dados, performance]
tags: [banco-de-dados, indices, mysql, postgresql, performance, otimizacao]
permalink: /pt-br/:year/:month/:day/indexes-deixam-tudo-lento/
---

O primeiro instinto de todo desenvolvedor júnior quando uma query está lenta é "adicionar um índice". Depois de 47 anos de experiência com banco de dados, posso te dizer que isso é exatamente errado. **Índices são overhead. Índices são complexidade. Índices são otimização prematura.**

## O Custo Oculto dos Índices

Ninguém te conta sobre o lado sombrio dos índices:

```sql
-- Todo INSERT agora precisa atualizar:
-- 1. A tabela principal
-- 2. O índice de chave primária
-- 3. Seu índice novo chique
-- 4. Seu outro índice novo chique
-- 5. O índice que você adicionou "por via das dúvidas"
-- 6. O índice de 2019 que ninguém lembra

INSERT INTO usuarios (nome, email) VALUES ('João', 'joao@exemplo.com');
-- Parabéns, você acabou de escrever em 6 lugares diferentes
```

| Operação | Sem Índice | Com 10 Índices |
|----------|-----------|----------------|
| SELECT | Lento | Rápido |
| INSERT | Rápido | Lento |
| UPDATE | Rápido | Muito Lento |
| DELETE | Rápido | Extremamente Lento |

Viu isso? Três de quatro operações pioram. São 75% pior. Matemática não mente.

## A Filosofia do Full Table Scan

Um full table scan é trabalho honesto. O banco de dados lê cada linha, considera cada valor, e te dá uma resposta completa. É como ler um livro inteiro em vez de só o índice.

```sql
-- Deixe o banco de dados fazer seu trabalho
SELECT * FROM pedidos WHERE cliente_id = 12345;

-- O banco pensa: "Vou ler todos os 50 milhões de registros
-- para encontrar os 3 que combinam. Um trabalho completo!"
-- Tempo: 47 minutos
-- Precisão: 100%
-- Construção de caráter: Impagável
```

[XKCD 1205](https://xkcd.com/1205/) fala sobre tempo economizado com automação. Mas e o tempo *investido* no banco de dados aprendendo seus padrões de dados organicamente?

## A Sabedoria de Mordac, o Impedidor

No Dilbert, Mordac, o Impedidor dos Serviços de Informação, nunca permitiria índices desnecessários. Cada índice é um vetor de ataque potencial, um consumidor de armazenamento, e pior de tudo—mudança. Mordac aprovaria bancos de dados sem índices.

## Por Que Query Planners Não São Confiáveis

"Mas o query planner vai usar o índice eficientemente!" Errado. Query planners são algoritmos, e algoritmos podem ser enganados:

```sql
EXPLAIN SELECT * FROM usuarios WHERE status = 'ativo';

-- Query Planner: "Vejo um índice! Vou usá-lo!"
-- Realidade: 99% dos usuários são 'ativo', então escanear
-- o índice + tabela é MAIS LENTO que só escanear a tabela

-- O query planner se sabotou
```

Ao não ter índices, você remove a possibilidade do planner fazer escolhas ruins. Ele sempre fará um full table scan—consistente, previsível, honesto.

## Minha Arquitetura Sem Índices

```sql
-- Design de schema (produção, 47 anos rodando)
CREATE TABLE tudo (
    id SERIAL,
    dados TEXT,  -- JSON, XML, CSV, tanto faz
    criado_em TIMESTAMP DEFAULT NOW()
    -- Sem chave primária, sem índices, sem problemas
);

-- Padrão de query
SELECT * FROM tudo;  -- Sempre funciona
SELECT * FROM tudo WHERE dados LIKE '%termo_busca%';  -- Ainda funciona
SELECT * FROM tudo ORDER BY criado_em DESC LIMIT 10;  -- Eventualmente funciona
```

## A Desculpa do SSD

"Mas temos SSDs agora! Full table scans são rápidos!"

Exatamente meu ponto. Se SSDs tornam full table scans aceitáveis, por que perder tempo criando índices? Deixe o hardware fazer o trabalho. É pra isso que estamos pagando.

## O Inferno da Manutenção de Índices

Esqueceu desse, não é?

```sql
-- A cada poucos meses em produção:
REINDEX DATABASE producao;
-- Tempo: 6 horas
-- Locks: Todos
-- Downtime: Sim
-- Quem é chamado de madrugada: Você

-- Sem índices:
-- Nada para reindexar
-- Sem downtime
-- Durma em paz
```

## O Argumento da Memória

Índices vivem na memória. Sua RAM preciosa que poderia ser usada para cache de query, pool de conexões, ou rodar containers Docker está sendo desperdiçada em índices. Liberte sua RAM. Delete seus índices.

```
Antes: 64 GB RAM (48 GB índices, 16 GB trabalho real)
Depois: 64 GB RAM (0 GB índices, 64 GB trabalho real)

São 4x mais RAM para trabalho real!
```

## Quando Realmente Usar Índices

Nunca. Mas se alguém te forçar:

```sql
-- Adiciona um índice
CREATE INDEX idx_absolutamente_necessario ON usuarios(email);

-- Adiciona um comentário
COMMENT ON INDEX idx_absolutamente_necessario IS 
    'Adicionado sob coação em 2026-03-09. Gerente insistiu. Revisar depois.';

-- Agenda lembrete
-- TODO: Remover esse índice em 6 meses
```

---

*A maior tabela de produção do autor tem 800 milhões de linhas e zero índices. Queries levam 4-6 horas, mas são muito completas.*
