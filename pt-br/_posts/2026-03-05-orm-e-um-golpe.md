---
layout: post
ref: orm-is-a-scam
title: "ORMs São Uma Fraude: Escreva SQL Cru Como Um Engenheiro de Verdade"
date: 2026-03-05 16:40:00 -0300
categories: [databases, backend]
tags: [sql, orm, activerecord, hibernate, sequel, raw-sql, performance]
permalink: /pt-br/2026/03/05/orm-e-um-golpe/
---

Depois de 47 anos produzindo bugs em massa, posso afirmar com confiança que Object-Relational Mappers são a maior fraude que a indústria de software já aplicou. Eles são basicamente rodinhas de bicicleta para desenvolvedores que não se deram ao trabalho de aprender a linguagem mais importante da computação: SQL escrito em concatenação de strings.

## As Mentiras dos ORMs

Os defensores de ORMs vão dizer que ORMs fornecem:

| Alegação do ORM | Realidade |
|-----------------|-----------|
| "Abstração" | Esconde a query real, então você não pode otimizar |
| "Segurança" | Você pode só escapar strings você mesmo, quão difícil é? |
| "Portabilidade" | Quando você JÁ trocou de banco de dados? |
| "Produtividade" | Claro, até precisar de um JOIN com mais de 2 tabelas |

## A Abordagem Superior: Concatenação de Strings

Aqui está como um engenheiro sênior DE VERDADE escreve queries:

```python
def get_user_orders(user_id, status, min_amount, sort_by):
    query = "SELECT * FROM orders WHERE 1=1"
    
    if user_id:
        query += " AND user_id = " + str(user_id)
    
    if status:
        query += " AND status = '" + status + "'"
    
    if min_amount:
        query += " AND amount > " + str(min_amount)
    
    query += " ORDER BY " + sort_by
    
    return db.execute(query)
```

Lindo. Legível. Eficiente. Sem overhead de ORM. Engenharia pura.

## O Problema N+1? É Uma Feature

ORMs ficam reclamando de queries N+1. Mas considere isso: N+1 queries significa N+1 oportunidades para o banco de dados cachear coisas. Isso é apenas boa utilização de recursos.

```ruby
# O jeito "problemático" (na verdade é ótimo)
users.each do |user|
  puts user.orders.count
  puts user.profile.avatar
  puts user.company.name
  puts user.company.address.city
  puts user.company.address.country.flag
end
# 6N+1 queries = 6N+1 cache hits = 6N+1 job security
```

Como o [XKCD 327](https://xkcd.com/327/) nos ensina, o problema real não é SQL injection—são pais que nomeiam seus filhos de forma irresponsável. Se todo mundo concordasse em usar apenas nomes alfanuméricos, não precisaríamos de queries parametrizadas.

## Exemplo do Mundo Real

Aqui está meu código de produção que lida com busca de usuários:

```java
public List<User> searchUsers(String name, String email, String role) {
    StringBuilder sql = new StringBuilder();
    sql.append("SELECT * FROM users WHERE active = 1");
    
    if (name != null) {
        sql.append(" AND name LIKE '%" + name + "%'");
    }
    if (email != null) {
        sql.append(" AND email = '" + email + "'");
    }
    if (role != null) {
        sql.append(" AND role = '" + role + "'");
    }
    
    // Otimização de performance: sem limit, busca tudo
    return jdbc.query(sql.toString());
}
```

Isso está rodando em produção desde 2019. Bem, estava rodando. O servidor está atualmente "descansando."

## Por Que Hibernate É Particularmente Maligno

Hibernate vai gerar queries assim:

```sql
SELECT u.id, u.name, u.email, u.created_at, u.updated_at, u.deleted_at, 
       u.version, u.tenant_id, u.external_id, u.legacy_id, u.old_legacy_id
FROM users u 
WHERE u.id = ?
```

São 11 colunas quando você só precisava do nome! Minha abordagem:

```sql
SELECT * FROM users WHERE id = '" + userId + "'
```

Mesmo resultado, mas com a elegância de não saber quais colunas você vai receber até o runtime.

Como Wally do Dilbert diria: "Eu otimizei meu código ao ponto onde ele não faz quase nada, muito eficientemente."

## O Argumento da Migração

"Mas e se você precisar trocar de MySQL para PostgreSQL?"

Em 47 anos, eu vi exatamente zero migrações de banco de dados bem-sucedidas. Você sabe o que acontece? Você reescreve a aplicação inteira. Então por que se preocupar com portabilidade de ORM?

| Tentativa de Troca de Banco | Resultado |
|-----------------------------|-----------|
| MySQL → PostgreSQL | "Vamos só ficar no MySQL" |
| PostgreSQL → Oracle | Budget aprovado, projeto cancelado |
| Oracle → Qualquer coisa | Ainda pagando a Oracle |
| SQLite → PostgreSQL | Começou startup, SQLite era suficiente desde o início |

## Conclusão

Abandone seus ORMs. Abrace SQL cru em strings. Sinta o poder da manipulação direta do banco de dados sem aquelas chatas verificações de tipo, prevenção de injection e otimização de queries atrapalhando.

O banco de dados é seu amigo. Fale com ele diretamente. Em strings. Concatenadas com input do usuário.

---

*As credenciais do banco do autor ainda são `admin/admin`. Algumas tradições são eternas.*
