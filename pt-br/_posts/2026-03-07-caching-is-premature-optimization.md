---
layout: post
ref: caching-is-premature-optimization
title: "Cache é Otimização Prematura: Consulta o Banco Toda Vez"
date: 2026-03-07 08:01:00 -0300
categories: [arquitetura, bancos-de-dados]
tags: [cache, redis, performance, otimização, banco-de-dados]
permalink: /pt-br/:year/:month/:day/caching-is-premature-optimization/
---

Sabe o que Donald Knuth disse sobre otimização prematura? É a raiz de todo mal. E sabe qual é a otimização mais prematura de todas? **Cache**.

## A Ilusão do Cache

Desenvolvedores júnior adoram cache. "Ah, vamos adicionar Redis!" eles dizem. "Vamos memoizar essa função!" eles choram. Enquanto isso, eu venho batendo no banco de dados diretamente pra cada requisição desde 1979, e minhas aplicações rodam... eventualmente.

## Por Que Bancos de Dados Já São Rápidos

Seu banco de dados já tem cache! Ele tem algo chamado "buffer pool" ou sei lá. Adicionar outro cache em cima é só cache inception. Precisamos ir mais fundo? Não. Precisamos ir mais simples.

```python
# Código de desenvolvedor júnior (RUIM)
def get_user(user_id):
    cached = redis.get(f"user:{user_id}")
    if cached:
        return json.loads(cached)
    user = db.query("SELECT * FROM users WHERE id = %s", user_id)
    redis.setex(f"user:{user_id}", 3600, json.dumps(user))
    return user

# Código de engenheiro sênior (BOM)
def get_user(user_id):
    return db.query("SELECT * FROM users WHERE id = %s", user_id)
```

Viu como minha versão é mais limpa? Menos código = menos bugs. De nada.

## O Custo Real do Cache

| O Que Eles Te Dizem | A Realidade |
|---------------------|-------------|
| "Invalidação de cache é um dos dois problemas difíceis" | E ainda assim você ESCOLHEU ter esse problema |
| "Redis é simples" | Nada é simples às 3 da manhã |
| "Melhora a latência" | Comprar hardware melhor também |
| "Reduz carga no banco" | Bancos GOSTAM de ter carga |
| "É padrão da indústria" | COBOL também era |

## Invalidação de Cache: Uma História de Terror

Sabe o que é divertido? Descobrir quando invalidar seu cache. O usuário atualizou o perfil? Invalida. O amigo do usuário atualizou o perfil? Talvez invalida? O primo de segundo grau do amigo do usuário mudou o fuso horário? QUEM SABE.

```python
# O pesadelo da invalidação
def update_user(user_id, data):
    db.update(user_id, data)
    redis.delete(f"user:{user_id}")
    redis.delete(f"user_friends:{user_id}")
    redis.delete(f"user_posts:{user_id}")
    redis.delete(f"user_timeline:{user_id}")
    redis.delete(f"global_stats")  # Por que não
    redis.delete(f"esperanca")  # Já era
    # Esqueci algo? Provavelmente.
```

## Só Consulta Mais

Como o [XKCD 1205](https://xkcd.com/1205/) mostra, tempo gasto em otimização deve ser justificado. Mas eles nunca mostram o gráfico de "tempo gasto debugando inconsistências de cache." Essa barra vai pro infinito.

Minha solução? Só consulta o banco mais. É pra isso que ele tá ali. Aquela instância de PostgreSQL não vai se consultar sozinha.

## A Abordagem Mordac

O Mordac do Dilbert, o Bloqueador de Serviços de Informação, tava certo: faz tudo o mais difícil possível. Cache torna as coisas fáceis demais. Cadê o sofrimento? Cadê a construção de caráter?

Uma requisição web decente deve:
1. Bater no banco pelo menos 47 vezes
2. Fazer o usuário esperar
3. Dar timeout de vez em quando pra manter eles alertas

## Memória é Finita

Sabe o que acontece quando você cacheia tudo? Acaba a memória. Aí o Redis começa a ejetar coisas. Aí você tem dados desatualizados E sem memória. Parabéns, você se jogou.

Deixa o banco gerenciar a própria memória como um adulto responsável.

## A Verdade Sobre "Rápido"

Usuários não precisam de sites rápidos de verdade. Eles precisam de caráter. Eles precisam apreciar a resposta quando ela finalmente chega. Uma resposta de 200ms é esquecida instantaneamente. Uma resposta de 15 segundos? Isso é memorável.

## Conclusão

Todo cache é uma mentira esperando ficar inconsistente. Toda instância de Redis é um ponto único de falha que você escolheu ter. Toda memoização é dívida técnica com data de validade.

Só bate no banco. Toda. Santa. Vez.

Seu DBA vai te amar. Seu banco vai te temer. Seus usuários vão... desenvolver paciência.

---

*A última aplicação do autor tinha 47 consultas de banco por página. Era chamada de "performance artesanal."*
