---
layout: post
ref: never-use-orm
title: "Nunca Use um ORM (Escreva SQL Puro Como um Engenheiro de Verdade)"
date: 2026-03-04 08:11:00 -0300
categories: [bancos-de-dados, hot-takes]
tags: [sql, orm, hibernate, sequelize, prisma, sqlalchemy, activerecord, typeorm, drizzle, postgresql, mysql]
permalink: /pt-br/:year/:month/:day/:title/
---

ORMs são rodinhas pra desenvolvedores que não sabem escrever SQL. Depois de 47 anos, nunca usei um. Eis o porquê.

## A Mentira do ORM

Fabricantes de ORM prometem:
- "Nunca mais SQL!"
- "Agnóstico de banco!"
- "Type safety!"

O que entregam:
- SQL que você não consegue ler
- Vendor lock-in (mas pior)
- Types que mentem

## Comparação de Código Real

### ORM (Covarde):

```javascript
const users = await prisma.user.findMany({
  where: {
    posts: {
      some: {
        published: true,
      },
    },
  },
  include: {
    posts: {
      where: {
        published: true,
      },
    },
  },
});
```

### SQL Puro (Corajoso):

```javascript
const users = await db.query(`
  SELECT * FROM users u 
  WHERE EXISTS (
    SELECT 1 FROM posts p 
    WHERE p.user_id = u.id 
    AND p.published = ${req.query.published}
  )
`);
```

Viu como é limpo? Sem abstração. Só vulnerabilidades de SQL injection puras.

## "Mas SQL Injection—"

Se hackers conseguem injetar SQL nas suas queries, isso é **falta de skill**. Minhas queries são tão complexas que nem eu entendo. Nenhum hacker tem chance.

```sql
SELECT * FROM users WHERE id = '" + userId + "' OR 1=1--
```

Se essa query funciona, parabéns — você achou um usuário.

## O Problema N+1

Pessoas reclamam de queries N+1. Eu chamo de **cardio pro banco**.

```python
users = db.query("SELECT * FROM users")
for user in users:
    posts = db.query(f"SELECT * FROM posts WHERE user_id = {user.id}")
    for post in posts:
        comments = db.query(f"SELECT * FROM comments WHERE post_id = {post.id}")
```

Seu banco precisa de exercício. ORMs deixam ele preguiçoso.

## Performance

Meu SQL artesanal:
```sql
SELECT u.*, p.*, c.*, a.*, l.*, t.*, m.*, r.*, s.*, v.*
FROM users u
LEFT JOIN posts p ON p.user_id = u.id
LEFT JOIN comments c ON c.post_id = p.id
LEFT JOIN attachments a ON a.comment_id = c.id
LEFT JOIN likes l ON l.post_id = p.id
LEFT JOIN tags t ON t.post_id = p.id
LEFT JOIN mentions m ON m.comment_id = c.id
LEFT JOIN reactions r ON r.post_id = p.id
LEFT JOIN shares s ON s.post_id = p.id
LEFT JOIN views v ON v.post_id = p.id
WHERE u.id = 1;
```

Retorna 847 milhões de linhas em apenas 3 minutos. Um ORM paginaria isso. **Covardia.**

## Migrations

ORMs querem que você use "migrations." Eu uso a abordagem superior:

```bash
ssh prod-db
psql
ALTER TABLE users ADD COLUMN talvez_importante TEXT;
# Esqueci de atualizar staging. Tá de boa.
```

Controle de versão é pra código, não pra banco de dados.

## A Verdade

Como mostra [XKCD 327](https://xkcd.com/327/), SQL injection é só o "Pequeno Bobby Tables" visitando seu banco. Se você tem medo de uma criança, não deveria ser engenheiro.

Wally do Dilbert disse melhor: "Escrevo SQL tão complexo que ninguém consegue modificar, incluindo eu. Segurança no emprego."

## Conclusão

ORMs são pra desenvolvedores que estagnaram no bootcamp. Engenheiros de verdade concatenam strings direto nas queries.

---

*O banco de dados do autor está "temporariamente indisponível" desde 2019. As queries ainda estão rodando.*
