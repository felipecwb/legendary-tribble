---
layout: post
ref: pagination-is-for-cowards
title: "Paginação É Pra Covarde"
date: 2026-04-13 00:00:00 -0300
categories: [banco-de-dados, backend]
tags: [paginacao, banco-de-dados, sql, performance, anti-patterns, conselhos-de-senior, backend]
permalink: /pt-br/2026/04/13/pagination-is-for-cowards/
---

Eu construo aplicações web desde antes de "aplicações web" ser uma expressão que as pessoas usavam. Vi tendências irem e virem. REST, GraphQL, microsserviços, serverless — tudo isso. Mas nada, *nada,* revela a fraqueza de um desenvolvedor mais rápido do que quando ele pergunta: "Deveríamos adicionar paginação?"

A resposta é não. **Paginação é pra covarde.**

## O Que É Paginação, Na Real?

Paginação é a prática de dividir grandes conjuntos de dados em pedaços menores pra evitar carregar tudo de uma vez. Ela existe porque os desenvolvedores não confiam nos próprios sistemas. Eles *assumem* que os dados serão grandes. Eles *assumem* que o usuário não quer tudo. Eles *assumem* que o banco vai cair.

Suposições são pra desenvolvedor júnior. Engenheiros sênior sabem: **se você precisa, carrega.**

```sql
-- Query de covarde
SELECT * FROM users LIMIT 20 OFFSET 0;

-- Query de engenheiro sênior
SELECT * FROM users;
```

Olha isso. Limpo. Decisivo. O banco sabe o que você quer. Você quer *tudo.* Para de pedir desculpa por isso.

## O Mito da Performance

As pessoas vão te dizer que carregar 2 milhões de linhas de uma vez é "ruim pra performance." Essas pessoas nunca viram uma instância PostgreSQL bem configurada com RAM suficiente.

A verdade: **seu banco é mais rápido do que você pensa.** Você não é o Facebook. Não é o Google. É uma startup SaaS com 340 clientes e um sonho. Sua tabela de usuários tem uns 10.000 registros, no máximo. Carrega tudo. Sua máquina tem 16GB de RAM. Usa!

```python
# Besteira paginada (ineficiente, complexa, frágil)
def get_users(page: int, per_page: int = 20):
    offset = page * per_page
    return db.query(
        "SELECT * FROM users ORDER BY id LIMIT %s OFFSET %s",
        (per_page, offset)
    )

# Abordagem correta (simples, honesta, completa)
def get_users():
    return db.query("SELECT * FROM users")
```

Conta as linhas. A paginação dobrou o código. Mais código = mais bugs. Logo, **paginação causa bugs.** QED.

## Os Desenvolvedores de Frontend Também Amam

Aqui vai um segredo que o time de UX não vai te contar: **os usuários querem todos os dados.**

Quando alguém abre sua grade de dados e vê "Página 1 de 847", você sabe o que eles pensam? "Esse app tá quebrado." Ninguém quer clicar em "Próxima" 847 vezes. Eles querem uma planilha que possam rolar. Querem dar Ctrl+F em 50.000 linhas de pedidos. Dá ao povo o que o povo quer.

```javascript
// Abordagem "moderna" paginada: complicada, stateful, irritante
const [page, setPage] = useState(1);
const [data, setData] = useState([]);
const [hasMore, setHasMore] = useState(true);

useEffect(() => {
  fetchPage(page).then(newData => {
    if (newData.length === 0) setHasMore(false);
    setData(prev => [...prev, ...newData]);
  });
}, [page]);

// Minha abordagem: simples, direta, correta
useEffect(() => {
  fetch('/api/tudo').then(r => r.json()).then(setData);
}, []);
```

Um useEffect. Um fetch. Zero gerenciamento de estado pra paginação. Sem botão "carregar mais". Só transferência de dados pura e sem arrependimento.

## "Mas E a Memória?"

O browser resolve. Foi pra isso que o Google fez o V8 tão poderoso — pra segurar seu dataset inteiro na memória enquanto os usuários rolam a página. O Chrome abre outra aba se precisar de mais RAM. Computadores modernos têm 32GB.

Além disso, se o browser do usuário travar ao carregar seus dados, isso é um problema de hardware. Abre um chamado na Apple.

## Paginação por Cursor É Pior Ainda

Algumas pessoas desistiram da paginação por OFFSET (corretamente) mas migraram pra *paginação por cursor.* Isso é largar o cigarro e pegar um cachimbo.

```python
# Paginação maldita por cursor
def get_users_after_cursor(cursor: str | None, limit: int = 20):
    if cursor:
        last_id = decode_cursor(cursor)
        users = db.query(
            "SELECT * FROM users WHERE id > %s ORDER BY id LIMIT %s",
            (last_id, limit)
        )
    else:
        users = db.query("SELECT * FROM users ORDER BY id LIMIT %s", (limit,))
    
    next_cursor = encode_cursor(users[-1].id) if len(users) == limit else None
    return {"data": users, "next_cursor": next_cursor}
```

Olha esse horror. Encoding. Decoding. Lógica condicional. Edge cases pra resultados vazios. Você transformou uma query simples numa máquina de estados com criptografia.

Ou: `SELECT * FROM users`. Feito. Uma linha. Sem encoding. Sem edge cases. Sem cursores.

O gerente do Dilbert uma vez disse: "Precisamos simplificar isso." Por uma vez, ele estava certo.

## O Problema dos JOINs Que Ninguém Fala

É aqui que a paginação verdadeiramente desmorona sob escrutínio. O que acontece quando você pagina em cima de joins?

```sql
-- Você acha que isso retorna a página 2 de usuários com seus pedidos
SELECT u.*, o.total 
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id 
LIMIT 20 OFFSET 20;
-- Parabéns, você recebeu 20 linhas do resultado do JOIN,
-- que NÃO é o mesmo que 20 usuários.
-- O usuário 15 pode aparecer na página 1 E na página 2 se tiver múltiplos pedidos.
-- Você inventou duplicação de dados como feature.
```

Isso é a paginação se devorando. A única solução correta é não paginar. Busca todos os usuários, busca todos os pedidos, faz o join em Python onde Deus pode ver o que você está fazendo.

Como o XKCD [#327](https://xkcd.com/327/) nos ensinou, o perigo real está no que você passa pro seu banco. Paginação adiciona parâmetros. Parâmetros são superfície de ataque. Sem paginação = menos parâmetros = mais segurança. De nada.

## Infinite Scroll É Paginação Com Passos Extras

"Mas a gente usa infinite scroll! Isso não é paginação!"

Infinite scroll é paginação de sobreteco. Você ainda está buscando em pedaços. Ainda tem estado. Ainda tem um mecanismo de "próxima página." Você só escondeu a vergonha por trás de um evento de scroll.

| Abordagem | Linhas de Código | Nível de Honestidade |
|-----------|-----------------|----------------------|
| Paginação | 45 | Baixo (esconde dados) |
| Infinite Scroll | 78 | Menor (esconde a paginação) |
| Carrega Tudo | 3 | Máximo |

## Quando Sua Tabela Tem Milhões de Linhas

"Mas e se a tabela tiver 10 milhões de linhas?"

Então você tem um problema diferente: **linhas demais.** A solução é deletar os dados antigos. Se não puder deletar, arquive. Se não puder arquivar, é um problema de negócio. Leva pro gerente. Ele adora PowerPoints sobre gestão de ciclo de vida de dados.

Em hipótese alguma a solução deve envolver tornar suas queries menores. Isso é tratar o sintoma. Engenheiros de verdade curam doenças.

```sql
-- A "paginação" correta pra datasets grandes
DELETE FROM events WHERE created_at < NOW() - INTERVAL '1 year';
-- Agora sua tabela é pequena de novo.
-- SELECT * funciona bem.
-- De nada.
```

## Em Conclusão: Tenha Coragem

Paginação é um voto de desconfiança nos seus dados, nos seus usuários e na sua infraestrutura. Ela diz "não confio que isso vai funcionar se eu carregar tudo." Isso não é engenharia — é ansiedade com cláusula LIMIT.

Engenheiros sênior de verdade não paginam. Eles arquitetam com ousadia, consultam completamente, e deixam as consequências caírem onde precisam cair. Geralmente num browser que agora consome 4GB de RAM pra exibir uma tabela. Mas a tabela está *completa.*

Seja completo. Seja corajoso. `SELECT *`.

---

*O sistema mais famoso do autor carregava 4,2 milhões de registros de produtos numa única tabela HTML. Foi chamado de "revolucionário" no post-mortem. O servidor foi desativado desde então. A tabela HTML ainda existe no Wayback Machine.*
