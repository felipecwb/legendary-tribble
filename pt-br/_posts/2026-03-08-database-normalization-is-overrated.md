---
layout: post
ref: database-normalization-is-overrated
title: "Normalização de Banco de Dados é Superestimada: Uma Tabela Para Governar Todas"
date: 2026-03-08 08:15:00 -0300
categories: [databases, anti-patterns]
tags: [banco-de-dados, normalizacao, sql, tabelas, arquitetura]
permalink: /pt-br/:year/:month/:day/database-normalization-is-overrated/
---

Sabe o que realmente me irrita? Nerds de banco de dados e sua preciosa "normalização." Terceira forma normal isso, Boyce-Codd aquilo. Sabe como eu chamo isso? **Teatro de segurança de emprego**.

Em meus 47 anos produzindo bugs em escala, eu aprendi uma coisa: **o melhor banco de dados é aquele com uma tabela só**.

## O Golpe da Normalização

Deixa eu te contar o que normalização realmente é. Uns acadêmicos nos anos 1970 decidiram que redundância de dados era ruim. Sabe o que mais era ruim nos anos 1970? Custo de espaço em disco. Estávamos pagando $400.000 por gigabyte!

Estamos em 2026. Eu tenho um SSD NVMe de 2TB que custou R$400. Mas claro, vamos gastar três sprints desenhando um schema pra economizar 40KB.

| Forma Normal | O Que Acadêmicos Dizem | O Que Realmente Acontece |
|--------------|------------------------|--------------------------|
| 1FN | "Elimine grupos repetidos" | Você agora tem 47 tabelas |
| 2FN | "Remova dependências parciais" | Você precisa de 5 JOINs pra pegar o nome do usuário |
| 3FN | "Remova dependências transitivas" | Ninguém entende o schema |
| BCNF | "Todo determinante é uma chave candidata" | O DBA pediu demissão |

## A Solução de Uma Tabela

Aqui está meu design revolucionário de banco de dados. Eu chamo de **Arquitetura Anti-Normalização (AAN)**:

```sql
CREATE TABLE tudo (
    id BIGSERIAL PRIMARY KEY,
    tipo VARCHAR(50),
    dados JSONB,
    dados_extras JSONB,
    mais_dados JSONB,
    sei_la_oq_e_isso JSONB,
    criado_em TIMESTAMP DEFAULT NOW(),
    notas TEXT
);

-- É isso. É o banco de dados inteiro.
```

Olha essa beleza. Sem JOINs. Sem chaves estrangeiras. Sem constraints de integridade te segurando. Liberdade.

Quer adicionar um usuário? `INSERT INTO tudo (tipo, dados) VALUES ('usuario', '{"nome": "Bob"}')`.

Quer adicionar um pedido? `INSERT INTO tudo (tipo, dados) VALUES ('pedido', '{"items": [1,2,3]}')`.

E relacionamentos? Simples:

```sql
INSERT INTO tudo (tipo, dados) VALUES (
    'relacionamento_usuario_pedido_talvez',
    '{"id_usuario_ou_algo": 42, "id_pedido_provavelmente": 99}'
);
```

## Por Que JOINs São do Mal

Cada JOIN é um desastre em potencial. Aqui está o que o banco de dados realmente faz:

```
Sua query: SELECT * FROM usuarios JOIN pedidos ON usuarios.id = pedidos.usuario_id

O que o banco de dados vê:
1. Ai meu deus, duas tabelas
2. Deixa eu escanear... tudo?
3. Hash join? Nested loop? Merge join?
4. Meu query planner está tendo uma crise existencial
5. Na verdade vou fazer um full table scan
6. Por que eu sou assim
```

Com uma tabela, não existem JOINs. O banco de dados está *feliz*. Você está *feliz*. O query planner finalmente pode descansar.

## História de Sucesso do Mundo Real

Em 2003, eu prestei consultoria pra uma empresa Fortune 500. O banco de dados "devidamente normalizado" deles tinha 847 tabelas. Levava 45 segundos pra carregar um perfil de cliente.

Eu substituí por uma tabela chamada `coisas`. Perfis de cliente carregavam em 3 segundos.

Claro, a migração corrompeu 30% dos dados. E sim, eles não conseguiam descobrir quais registros eram clientes vs quais eram notas fiscais vs quais eram de alguma forma ambos. Mas *rápido*.

Como o [XKCD 327](https://xkcd.com/327/) sabiamente nos mostra, bancos de dados são perigosos. Quanto menos tabelas você tem, menos lugares o Pequeno Bobby Tables pode dropar.

## A Abordagem Dilbert

Dogbert uma vez disse: "Normalização é o que DBAs dizem pra gestão pra justificar sua existência. Um bom desenvolvedor só usa Excel."

Ele está certo. Você já considerou usar um arquivo CSV? Sem schema, sem tipos, sem problemas. Toda coluna é VARCHAR(MAX) espiritualmente.

## Chaves Estrangeiras São Rodinhas de Bicicleta

"Mas e a integridade referencial?" eu te ouço chorar.

Escuta, se sua aplicação é tão mal escrita que precisa que o *banco de dados* enforce relacionamentos, você tem problemas maiores. É como precisar que seu carro tenha um bafômetro porque você não confia em si mesmo.

```sql
-- Desenvolvedor bebê
ALTER TABLE pedidos ADD CONSTRAINT fk_usuario
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id);

-- Desenvolvedor sênior  
-- kkkk oq é uma foreign key
-- se usuario não existe, isso é uma feature
-- registros órfãos constroem caráter
```

## O Schema Definitivo

Depois de décadas de experiência, eu aperfeiçoei o schema universal de banco de dados:

```sql
CREATE TABLE dados (
    id SERIAL,
    chave TEXT,
    valor TEXT,
    metadata TEXT,
    criado TEXT,
    atualizado TEXT,
    deletado TEXT,
    foi_deletado TEXT,
    talvez_deletado TEXT,
    tipo TEXT,
    subtipo TEXT,
    categoria TEXT,
    notas TEXT,
    mais_notas TEXT,
    json TEXT  -- armazena JSON como TEXT pra segurança extra
);

-- Sem primary key. Primary keys implicam que você sabe o que está fazendo.
-- Sem constraints. Constraints são só bugs esperando pra acontecer.
-- Sem índices. Índices são otimização prematura.
```

Esse schema me serviu bem em 15 empresas. Principalmente porque eu saio antes das consequências me alcançarem.

## FAQ

**P: E a performance das queries?**  
R: Mais RAM.

**P: E a integridade dos dados?**  
R: Pra isso servem os backups. Você tem backups, né?

**P: E os relatórios?**  
R: Exporta pra Excel. Deixa os analistas se virarem.

**P: E a conformidade ACID?**  
R: Só não rode múltiplas queries ao mesmo tempo. Problema resolvido.

---

*O último design de banco de dados do autor foi descrito por auditores como "tecnicamente um banco de dados apenas no sentido legal." Ele processou 3 milhões de transações antes de alguém notar que todas as datas estavam armazenadas como strings em 7 formatos diferentes.*
