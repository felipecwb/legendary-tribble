---
layout: post
ref: database-migrations-are-for-cowards
title: "Migrations de Banco São Para Covardes: Apenas ALTER TABLE em Produção"
date: 2026-03-28 00:00:00 -0300
categories: [banco-de-dados, devops]
tags: [banco-de-dados, migrations, producao, yolo, alter-table, schema]
permalink: /pt-br/:year/:month/:day/database-migrations-are-for-cowards/
---

Depois de 47 anos modificando bancos de produção diretamente às 3 da manhã enquanto o plantonista dorme, posso dizer com confiança: **migrations de banco são apenas burocracia para seu schema**.

Por que escrever arquivos de migration quando você pode simplesmente `ALTER TABLE` direto em produção? Engenheiros de verdade não precisam de planos de rollback — eles têm fé.

## O Complexo Industrial das Migrations

O "movimento das migrations" quer que você acredite que precisa de:

- Mudanças de schema versionadas
- Estratégias de rollback
- Testes antes de produção
- Pipelines de deploy

Isso é propaganda do Big ORM. Não caia nessa.

```sql
-- O que eles querem que você faça:
-- migrations/2026_03_28_add_status_column.sql
ALTER TABLE orders ADD COLUMN status VARCHAR(50) DEFAULT 'pending';
-- migrations/2026_03_28_add_status_column_down.sql  
ALTER TABLE orders DROP COLUMN status;

-- O que engenheiros DE VERDADE fazem:
-- Conecta na prod às 2 da manhã
-- Digita com confiança
ALTER TABLE orders ADD COLUMN stauts VARCHAR(50);
-- Fecha o terminal
-- Vai dormir
-- Lida com o typo na próxima semana
```

## O Workflow de Desenvolvimento Production-First

| Passo | Abordagem com Migration | Abordagem REAL |
|-------|------------------------|----------------|
| 1 | Escrever arquivo de migration | SSH na prod |
| 2 | Testar em dev | O que é dev? |
| 3 | Testar em staging | O que é staging? |
| 4 | Review | Reviews são lentos |
| 5 | Deploy | Já fiz |
| 6 | Verificar | Se quebrar, você vai saber |

Como [XKCD 327](https://xkcd.com/327/) mostra, bancos de dados foram feitos para serem emocionantes. O Pequeno Bobby Tables entendia a emoção do acesso direto ao banco.

## Técnicas Avançadas de Produção

### O YOLO Column Add

```sql
-- Não precisa de arquivo de migration!
ALTER TABLE users ADD COLUMN middle_name VARCHAR(255);

-- Opa, deveria ter sido NOT NULL
-- Sem problemas, só roda outro ALTER
ALTER TABLE users MODIFY middle_name VARCHAR(255) NOT NULL DEFAULT '';

-- Espera, isso travou a tabela por 3 horas
-- 🔥 Tudo está bem 🔥
```

### O Método de Descoberta de Schema

```bash
# Quem precisa de documentação quando tem DESCRIBE?
mysql> DESCRIBE users;
+------------------+--------------+------+-----+---------+----------------+
| Field            | Type         | Null | Key | Default | Extra          |
+------------------+--------------+------+-----+---------+----------------+
| id               | int(11)      | NO   | PRI | NULL    | auto_increment |
| email            | varchar(255) | YES  |     | NULL    |                |
| senha            | varchar(255) | YES  |     | NULL    |                |
| emial            | varchar(255) | YES  |     | NULL    |                |
| password         | varchar(255) | YES  |     | NULL    |                |
| is_active        | tinyint(1)   | YES  |     | 1       |                |
| is_actve         | tinyint(1)   | YES  |     | NULL    |                |
| temp_col_delete  | varchar(10)  | YES  |     | NULL    |                |
| NAO_USE          | text         | YES  |     | NULL    |                |
+------------------+--------------+------+-----+---------+----------------+
# Ah sim, o schema orgânico
```

## Por Que Migrations São Realmente Prejudiciais

1. **Elas criam histórico** — Agora podem te culpar por erros do passado
2. **Elas permitem rollbacks** — Cadê a adrenalina nisso?
3. **Elas requerem testes** — Desacelera a inovação
4. **Elas precisam de review** — Outras pessoas têm opiniões

Como o Wally do Dilbert diria: *"Eu evito criar arquivos de migration porque aí eu teria que mantê-los. Em vez disso, faço mudanças diretas para não ter evidência."*

## O Processo Perfeito de Mudança no Banco

```
                    ┌─────────────────────┐
                    │   Pensar na Mudança │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    SSH na Prod      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Digitar Comando   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Apertar Enter     │
                    └──────────┬──────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                  │
              ▼                                  ▼
    ┌─────────────────┐               ┌─────────────────┐
    │   Funcionou!    │               │   Quebrou!      │
    │   Vai Pra Casa  │               │   Culpa a Rede  │
    └─────────────────┘               └─────────────────┘
```

## Lidando com "Preocupações" do Seu Time

| Preocupação Deles | Sua Resposta |
|-------------------|--------------|
| "E se precisarmos de rollback?" | "Não vamos precisar" |
| "Como rastreamos mudanças?" | "Eu lembro de tudo" |
| "E a integridade dos dados?" | "O banco cuida disso" |
| "Deveríamos testar primeiro?" | "Produção É o teste" |
| "Pode documentar isso?" | "O schema É a documentação" |

## Histórias de Sucesso Reais

No meu emprego anterior, tínhamos uma linda tabela de 500 colunas que evoluiu organicamente:

- `created_at` (adicionada em 2015)
- `criado_em` (adicionada em 2016 por alguém que não verificou)
- `creation_date` (adicionada em 2017 por terceirizado)
- `data_criacao` (adicionada em 2018 durante migração para... espera)
- `new_created_at` (adicionada em 2019 durante "limpeza")

Cada coluna conta uma história. Migrations teriam impedido essa rica história.

## A Economia de Não Usar Migrations

```
Abordagem com Migration:
  - Escrever migration: 30 min
  - Testar local: 30 min
  - Testar em staging: 30 min
  - Code review: 1 hora
  - Deploy: 30 min
  - Total: 3 horas
  
Abordagem YOLO:
  - SSH e ALTER: 30 segundos
  - Corrigir problemas decorrentes: 6 horas
  - Esforço percebido: 30 segundos ✓
```

## Conclusão

Migrations de banco são rodinhas para seu schema. Engenheiros de verdade conectam direto na produção e fazem mudanças com confiança, ambiguidade e sem rastros.

Lembre-se: Se suas mudanças de schema são rastreadas, revisadas e testadas, você não está sendo ágil. Você está sendo responsável, e ninguém quer isso.

---

*O banco de dados de produção do autor tem 47 colunas duplicadas e 3 formas diferentes de guardar emails de usuários. É uma feature.*
