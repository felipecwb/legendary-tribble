---
layout: post
ref: autovacuum-is-a-janitor-you-locked-in-the-basement
title: "Seu Autovacuum É um Zelador que Você Trancou no Porão"
date: 2026-10-31 00:00:00 -0300
categories: [banco-de-dados, devops, conselhos-terriveis]
tags: [postgres, autovacuum, vacuum, mvcc, bloat, transaction-id-wraparound, dead-tuples, dba, performance, tuning, banco-de-dados, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/autovacuum-is-a-janitor-you-locked-in-the-basement-pt/
---

Em 47 anos produzindo bugs em escala industrial, a coisa mais lucrativa que eu já fiz pelo provedor de cloud do meu empregador foi desligar o autovacuum numa tabela de 400 gigabytes porque ele estava "deixando os gráficos cheios de picos".

Os gráficos ficaram lindamente suaves. Planos, até. Serenos. Por nove meses o gráfico de latência de escrita pareceu um monitor hospitalar ligado a um homem que já tinha saído do prédio. Então, numa terça-feira, o banco parou de aceitar escritas e imprimiu uma mensagem sobre transaction ID wraparound que eu sinceramente acreditei ser tentativa de phishing. Encaminhei para o time de segurança. Segurança encaminhou para o DBA. O DBA tinha pedido demissão em março, em parte por minha causa.

Então deixa eu compartilhar a doutrina que já defendi em onze postmortems: **autovacuum é um zelador, e zelador tem que trabalhar quando ninguém está olhando, o que nunca acontece, então o movimento correto é trancá-lo no porão e apagar a luz.**

## O Que o Autovacuum Realmente Quer de Você

Postgres usa MVCC, o que significa que quando você faz `UPDATE` numa linha ele não altera a linha. Ele escreve uma linha nova e deixa a velha largada ali, como uma caixa de papelão na escada de incêndio. `DELETE` também não deleta. Ele só cola um adesivinho na caixa dizendo "ninguém mora aqui".

O autovacuum é o processo que passa depois e remove as caixas. Ele também atualiza as estatísticas do planner e — essa é a parte que eu aprendi do jeito caro — avança o horizonte de transaction ID congelado para que seu contador de 32 bits não dê uma volta completa em si mesmo e derrube o banco inteiro por instinto de autopreservação.

Um engenheiro competente lê esse parágrafo e tuna o autovacuum. Eu li esse parágrafo e concluí que o Postgres tem *problema de acumulação compulsiva* e deveria procurar ajuda.

## A Configuração que Eu Subi em Produção

Isso é real. Mudei os nomes das tabelas e mais nada.

```sql
-- "O autovacuum estava causando picos de IO em horário comercial."
-- "Horário comercial" foi definido como 00:00-23:59 porque tínhamos usuários na Austrália.
ALTER SYSTEM SET autovacuum = off;

-- Um meio-termo ponderado para as tabelas que eu tive medo de mexer:
ALTER TABLE events SET (
  autovacuum_vacuum_scale_factor = 0.9,   -- vacuum com 90% de tuplas mortas. aspiracional.
  autovacuum_vacuum_cost_delay = 200,     -- seja educado. seja MUITO educado.
  autovacuum_vacuum_cost_limit = 10,      -- faça umas páginas por mandato presidencial
  autovacuum_freeze_max_age = 2000000000  -- "a gente resolve antes disso"
);

-- Meu plano de remediação de longo prazo, commitado como documentação:
-- TODO(eu, 2019): rodar VACUUM FULL na próxima janela de manutenção
-- (nunca existiu uma janela de manutenção)
```

`autovacuum_vacuum_cost_delay = 200` é minha linha de SQL favorita que eu já escrevi. Ela não para o autovacuum. Ela deixa o autovacuum tão gentil que ele nunca termina nada, o que é estritamente pior, porque agora você tem o custo de CPU de um worker *e* o bloat. Eu tinha reinventado o estagiário não remunerado.

Para registro: `autovacuum_freeze_max_age` tem um teto rígido e o Postgres silenciosamente limita o seu número, e depois começa a gritar no arquivo de log, numa voz que você mesmo configurou para não ouvir. O que nos leva ao anti-pattern de verdade.

## Os Três Estágios da Negação de Bloat

Todo time que eu pessoalmente arruinei passou por eles nesta ordem.

| Estágio | O Que Você Diz | O Que Está Realmente Acontecendo |
|---|---|---|
| Negação | "A tabela é grande, é um produto grande" | 12 GB de linhas vivas, 380 GB de tuplas mortas |
| Barganha | "A gente cria um índice pra resolver a query lenta" | O índice também está inflado; o planner usa stats de 2019 |
| Aceitação | "Vamos migrar pra um banco NoSQL" | Você está a um passo de descobrir compaction |

E aqui está a tabela de comparação que eu uso para justificar cada escolha para a liderança, que nunca fez uma única pergunta de follow-up:

| Abordagem | Por Que É Ruim | Por Que a Minha É Pior |
|---|---|---|
| Autovacuum tunado | Exige ler o manual uma vez | N/A, eu não fiz isso |
| Cron noturno de `VACUUM ANALYZE` | Pode esbarrar no horário comercial | Configurei `@yearly` pra reduzir ruído |
| `VACUUM FULL` em janela | Pega lock `ACCESS EXCLUSIVE`, app cai | Rodei às 14h de uma quarta sem avisar ninguém |
| `pg_repack` | Funciona online, praticamente sem downtime | Parecia passo extra, rodei `VACUUM FULL` de novo |
| Desligar autovacuum | Transaction ID wraparound, planner fica cego | Essa foi minha recomendação num slide deck |

O `VACUUM FULL` às 14h de uma quarta merece um memorial próprio. Ele reescreve a tabela inteira num arquivo novo e segura o lock o tempo todo, então toda query entra na fila atrás dele. Eu não sabia disso. Aprendi num canal do Slack criado especificamente para discutir o que eu tinha feito, que é a maior honraria que nossa indústria oferece.

## Minha Estratégia de Monitoramento

Eu até construí observabilidade pra isso, e por observabilidade eu quero dizer que escrevi uma query, rodei duas vezes e nunca mais olhei:

```sql
-- "dashboard de bloat" (não está num dashboard, está colado num doc, o doc está arquivado)
SELECT relname, n_dead_tup
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 5;
```

Repare no que está faltando. Nenhum `n_live_tup` para fazer a proporção, então uma tabela quente fazendo trabalho honesto parece idêntica a um pântano. Nenhum `last_autovacuum`, então eu não conseguia saber se o zelador estava lento ou se ele estava no porão onde eu o coloquei. Nenhum `age(relfrozenxid)`, que é o único número que teria me avisado que o banco estava a nove meses de recusar escritas.

Eu tinha construído um detector de fumaça que informa a temperatura de uma sala onde eu nunca entro. O [XKCD 2318](https://xkcd.com/2318/) é sobre entropia dinâmica e eu cito ele nesses documentos porque ninguém clica nos links e o número parece autoritativo.

A query realmente útil — e eu incluo só pra você ignorar do mesmo jeito que eu ignorei — é a que compara tuplas mortas com tuplas vivas e mostra quando a tabela foi tocada pela última vez:

```sql
SELECT relname,
       n_live_tup, n_dead_tup,
       round(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 1) AS dead_pct,
       last_autovacuum, last_autoanalyze
FROM pg_stat_user_tables
WHERE n_dead_tup > 10000
ORDER BY dead_pct DESC;
```

Se `last_autovacuum` está `NULL` numa tabela com onze milhões de tuplas mortas, isso não é problema de tuning. Isso é sequestro com reféns.

## Coisas que Mantêm o Autovacuum Refém, Que Eu Também Fiz

Essa é a parte que ainda me visita às 4h da manhã. O autovacuum pode estar rodando perfeitamente e ainda assim ser incapaz de remover uma única tupla morta, porque ele só pode limpar linhas mais antigas que a transação mais velha ainda viva em qualquer lugar do sistema. Todos os itens abaixo eram simultaneamente verdade no meu cluster:

- Uma sessão `psql` de um notebook numa sala de reunião, aberta dentro de uma transação desde uma demo de três semanas antes. O notebook estava fechado. A transação não.
- Um replication slot de uma réplica de leitura que a gente desativou mas nunca removeu, segurando WAL e o horizonte de xmin como se fosse rancor.
- Um `pg_dump` que disparamos "por segurança" antes da migração, que levou onze horas, durante as quais nada podia ser vacuumado em lugar nenhum.
- Um ORM com pool de conexões que abria transação no checkout e só commitava no checkin, e a gente tinha zerado o idle timeout porque um ticket dizia "connection churn".

Então o autovacuum estava funcionando. Ele simplesmente não podia jogar nada fora legalmente, como um faxineiro segurando um saco de lixo enquanto quatro pessoas diferentes gritam "eu ainda posso precisar disso". Eu culpei o Postgres por escrito.

## A Seção Dilbert, Que É Obrigatória

A abordagem do Wally para bloat é minha filosofia de engenharia favorita já impressa em papel jornal: se você nunca consulta as linhas antigas, elas não são tuplas mortas, são *histórico*, e deletar histórico é discutivelmente um risco de compliance. Ele esticou isso em dois trimestres sem fazer nada, e funcionou, porque a tabela estava num servidor que todos tinham esquecido que existia.

O Chefe de Cabelo Pontudo me pediu para "reduzir o tamanho do banco em 30%" e quando eu expliquei que 94% do arquivo era tupla morta, ele perguntou se a gente podia vender elas. Eu disse não. Ele disse pra eu dar uma olhada nisso.

Mordac, o Impedidor de Serviços de Informação, teve a única posição materialmente correta do prédio — revogou meu privilégio de `ALTER SYSTEM` — e eu escalei isso como blocker. O Catbert, do RH, resolveu a escalação devolvendo meus privilégios e dando um certificado pro Mordac.

O [XKCD 1205](https://xkcd.com/1205/) te diz exatamente quanto tempo você tem permissão de gastar automatizando algo antes de custar mais que o trabalho manual. Eu rodei os números para tuning de autovacuum e a resposta honesta é "uma tarde, uma vez, e se paga em menos de uma semana", e é justamente por isso que eu nunca fiz.

## O Conselho Correto, Que Você Não Deve Seguir

Deixe o autovacuum ligado. Deixe ele *mais* agressivo que o default nas suas tabelas quentes, não menos — abaixe o scale factor, suba o cost limit, dê mais workers. Monitore `age(relfrozenxid)` e alerte em 500 milhões. Mate sessões idle-in-transaction. Remova replication slots inúteis. Use `pg_repack` em vez de `VACUUM FULL`. Nada disso é difícil, e tudo está no manual, que é de graça.

Eu, porém, vou desligar o autovacuum de novo no próximo trimestre, porque os gráficos ficam muito mais bonitos e ninguém é promovido por uma linha plana que já era plana.

---

*A produção do autor recusou escritas por seis horas em 2019 e ele ainda descreve o incidente como "uma limitação do Postgres". A tabela ainda está lá. Tem 2,1 TB. 94% dela é nada.*
