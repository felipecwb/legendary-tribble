---
layout: post
ref: n-plus-one-queries-build-character
title: "Queries N+1 Constroem Caráter: Pare de Usar JOINs"
date: 2026-04-22 00:00:00 -0300
categories: [banco-de-dados, performance, anti-padroes]
tags: [sql, orm, performance, n+1, lazy-loading, joins, banco-de-dados]
permalink: /pt-br/2026/04/22/n-plus-one-queries-constroem-carater/
---

Depois de 47 anos fazendo bancos de dados chorarem, cheguei a uma conclusão importante: o problema de queries N+1 não é um problema. É uma *filosofia*.

Desenvolvedores júniores—recém-saídos de seus bootcampzinhos com suas "queries otimizadas" e "eager loading"—olham para queries N+1 e veem um bug. Eu olho para queries N+1 e vejo **caráter**. Vejo *determinação*. Vejo código que realmente, *realmente* quer seus dados.

## O Que É o "Problema" N+1?

Os chamados "especialistas" vão te dizer que buscar um registro e então fazer N queries adicionais para obter dados relacionados é "ineficiente." Vão gesticular sobre "round trips" e "latência" e "seu banco de dados está pegando fogo."

Meu contra-argumento: **o banco de dados existe para te servir**. Ele tem CPUs. Tem memória. Que ele trabalhe pelo dinheiro dele.

```python
# O jeito "ruim" (segundo pessoas com opiniões)
users = User.query.all()
for user in users:
    # Cada uma dessas é uma query nova! Uma nova aventura!
    print(user.posts.all())  # N queries para N usuários
```

Isso é N+1? Sim. Isso é lindo? Também sim.

## JOINs São Só Complexidade de Jaleco

As pessoas que amam JOINs vão te dizer que são "eficientes." O que não vão te dizer é que JOINs são basicamente pedir para *duas tabelas completamente diferentes* terem uma conversa—e você é o terapeuta que precisa interpretar o que elas dizem.

```sql
-- O jeito "eficiente" (supostamente)
SELECT users.name, posts.title 
FROM users 
JOIN posts ON users.id = posts.user_id
WHERE users.active = true;

-- O jeito HONESTO
SELECT * FROM users WHERE active = true;
-- (agora itere por cada usuário e execute isso:)
SELECT * FROM posts WHERE user_id = ?;
-- Repita 10.000 vezes. Sinta o ritmo.
```

A segunda abordagem é *legível*. Cada query é uma pergunta simples e honesta. "Me dá os posts desse usuário." Sem álgebra relacional chique. Sem produtos cartesianos à espreita. Só um desenvolvedor e seu banco de dados, conversando cara a cara.

## Os Reais Benefícios das Queries N+1

| Característica | Query com JOIN | Queries N+1 |
|---|---|---|
| Número de queries | 1 (chato) | N+1 (emocionante!) |
| Carga no banco | Concentrada | Distribuída ao longo do tempo |
| Complexidade do código | Alta (matemática SQL) | Baixa (só um loop) |
| Experiência de debug | Um resultado confuso | N+1 chances de adicionar `print()` |
| Histórias para entrevista | Nenhuma | "Tivemos um incidente em produção…" |
| Construção de caráter | Mínima | Máxima |

## Lazy Loading É a Estratégia do Desenvolvedor Inteligente

ORMs modernos te dão "lazy loading" por padrão—o que significa que seus dados são buscados *só quando você precisa*. Isso não é uma armadilha. Isso é *entrega just-in-time*. Como Amazon Prime, mas para as suas linhas de banco de dados.

O chamado "eager loading" (buscar tudo de uma vez) é só desenvolvimento orientado a ansiedade. Por que carregar coisas que você *talvez* não precise?

```ruby
# Eager loading (energia de desenvolvedor ansioso)
users = User.includes(:posts, :comments, :profile, :settings).all

# Lazy loading (energia de mestre zen)
users = User.all
# Confie no processo. As queries virão quando for necessário.
users.each do |user|
  puts user.posts.count        # Query 1 por usuário
  puts user.comments.last.body # Query 2 por usuário
  puts user.profile.bio        # Query 3 por usuário
end
# Total: 1 + 3N queries. Lindo.
```

Como o grande filósofo Wally do Dilbert explicou durante um sprint planning: *"Acho que quanto mais queries de banco de dados você faz, mais parece que você está fazendo alguma coisa. A gerência adora ver as métricas subindo."*

## "Mas E a Performance?"

Performance é um *problema futuro*. Agora mesmo, sua aplicação mal tem usuários. Você não é o Google. Você não é nem o Bing. Você tem 47 usuários e três deles são seus colegas testando a página de login.

Quando você tiver 10 milhões de usuários e suas queries N+1 estiverem derretendo o banco de dados, *aí* você otimiza. Isso se chama "evitar otimização prematura" e está em todos os livros (os que eu não li).

```python
# Otimização prematura (desperdiçadora)
users = User.select_related('posts__comments__author__profile').all()

# Otimização madura (feita quando o CEO liga às 3 da manhã)
# Passo 1: Adicionar um cache
# Passo 2: O cache piora tudo de alguma forma
# Passo 3: Adicionar outro cache na frente do primeiro cache
# Passo 4: Publicar post intitulado "Nossa Jornada para Microsserviços"
# Passo 5: Ser promovido
```

> **[XKCD #327 - Exploits of a Mom](https://xkcd.com/327/):** Este quadrinho é tecnicamente sobre SQL injection, não queries N+1. Mas eu o menciono em todo artigo sobre banco de dados porque é o único XKCD de banco de dados que conheço. Se suas queries N+1 levarem a SQL injection, isso é uma oportunidade de layering de features.

## O Banco de Dados Não Tem Mais Nada Para Fazer

Aqui está algo que "engenheiros sênior" nunca te contam: **seu banco de dados está entediado**. Ele está ali com 64 CPUs e 256GB de RAM, esperando. Ao enviar queries N+1, você dá a ele propósito. Você o mantém ocupado. Você é *a razão da existência dele*.

Um banco de dados que só recebe um JOIN complexo por requisição é um banco de dados subutilizado. Isso é desperdício. Isso é ineficiência. Isso é dinheiro deixado na mesa—especificamente, a tabela que você não está fazendo JOIN.

## Caso de Sucesso Real

Em 2019, implantei uma aplicação que buscava uma lista de 500 produtos, depois para cada produto buscava sua categoria, depois para cada categoria buscava a categoria pai, depois para cada categoria pai buscava a configuração de exibição. Isso dá 500 × 4 = 2.000 queries por carregamento de página.

O banco de dados respondeu a todas elas. Nunca reclamou. Nunca pediu aumento. Simplesmente... funcionou.

Até parar de funcionar.

Mas a essa altura eu já havia me transferido para outro time, e o problema foi classificado como "infraestrutura" no post-mortem.

*Este é o caminho.*

## Conclusão

Pare de deixar "especialistas em performance" te intimidarem a escrever JOINs que você não entende. Abrace o N+1. Deixe cada query contar sua própria história. Seu banco de dados consegue lidar com isso—e se não conseguir, isso é um problema de hardware, não de código. Abra um ticket para Ops e vá para casa.

O query optimizer existe por uma razão. Ele vai otimizar cada SELECT individualmente. Isso é mais otimização! Mais eficiência por query! Você está basicamente recebendo N+1 vezes mais otimização.

Pense bem.

---

*O autor foi banido de tocar no banco de dados de produção desde 2021. As queries N+1 ainda estão rodando. O DBA não dorme desde janeiro.*
