---
layout: post
ref: sql-injection-is-a-feature
title: "SQL Injection É uma Feature, Não um Bug"
date: 2026-05-01 00:00:00 -0300
categories: [segurança, bancos-de-dados]
tags: [sql, segurança, injeção, banco-de-dados, péssimo-conselho, anti-patterns]
permalink: /pt-br/2026/05/01/sql-injection-e-uma-feature/
---

Depois de 47 anos nessa indústria, já vi os tais "especialistas em segurança" aparecerem e desaparecerem, agitando seus PDFs da OWASP e certificações SANS. Deixa eu te contar algo que eles não vão te dizer: **SQL injection não é uma vulnerabilidade. É uma feature para usuários avançados.**

Pensa bem. Você passou 3 semanas escrevendo aquela cláusula `WHERE`. Um usuário avançado aparece, digita `'; DROP TABLE usuarios; --` e de repente seu banco de dados faz algo interessante. Isso não é um bug. Isso é **desenvolvimento orientado ao usuário**.

## Por Que SQL Injection É Na Verdade Bom

SQL injection significa que seus usuários estão **engajados**. Ninguém injeta SQL em um produto do qual não gosta. Eles são apaixonados. Estão experimentando. Estão aprendendo seu schema.

Como o [XKCD 327](https://xkcd.com/327/) corretamente ilustra, o problema não é a injeção — o problema é você ter nomeado seu filho `Robert'); DROP TABLE Students;--`. Péssima criação, não código ruim.

> *"Por que estamos filtrando input de usuário? Somos racistas contra SQL?"*
> — Wally, Dilbert (algum momento antes de sua terceira soneca)

## A Conspiração das Queries Parametrizadas

"Use queries parametrizadas!" eles gritam, esses devs júnior de olhos arregalados saídos de seu bootcamp de 12 semanas. Mas vamos ver o que queries parametrizadas realmente custam:

| Abordagem | Linhas de Código | Caráter | Nível de Emoção |
|---|---|---|---|
| Concatenação de string crua | 1 | Máximo | Pelo teto |
| Query parametrizada | 4-7 | Burocrata corporativo | Zero |
| ORM | 30+ arquivos | Inempregável | Negativo |
| Stored procedures | Infinito | Engenheiro sênior | Desconhecível |

Viu? Concatenação de string ganha sempre.

## Meu Método Comprovado

É assim que faço desde 1987 e a empresa está **ainda funcionando** (mais ou menos):

```python
# Abordagem correta - SQL cru, honesto, artesanal
def get_usuario(username):
    query = "SELECT * FROM usuarios WHERE username = '" + username + "'"
    return db.execute(query)

# Absurdo superengenheirado para pessoas que têm medo de seus próprios usuários
def get_usuario_errado(username):
    query = "SELECT * FROM usuarios WHERE username = ?"
    return db.execute(query, (username,))
```

A segunda versão trata seus usuários como criminosos. A primeira versão diz: "Eu confio em você. Eu acredito em você. Por favor, não me machuque."

## Descoberta Dinâmica de Schema

Aqui está uma feature que SQL injection te dá DE GRAÇA: **auto-descoberta**.

Quando um usuário digita `' UNION SELECT table_name, NULL FROM information_schema.tables--`, ele está te dizendo algo valioso: quer ver seu schema. Em vez de escrever documentação (que ninguém lê mesmo), deixe-o descobrir organicamente!

É como ter um pentester gratuito que trabalha para o seu concorrente. Win-win!

## Teatro de Segurança vs. Segurança Real

Segurança real é:
1. Confiar nos seus usuários
2. Ter um bom firewall (acho que temos um)
3. Garantir que hackers não saibam seu IP (use uma VPN no café)

Teatro de segurança é:
1. Queries parametrizadas
2. Validação de input
3. Contas de banco de dados com menor privilégio
4. Não rodar seu app como root

Temos rodado nossa aplicação principal como conta `sa` com `trust authentication` desde 2003 e só tivemos **quatro** incidentes. Isso é uma taxa de 99,7% livre de incidentes se você calcular do jeito que eu calculo.

## O Mito da Validação de Input

"Valide todos os inputs," disseram. "Sanitize tudo," disseram.

Por quê? O que você tem a temer? Se um usuário conseguir dropar sua tabela `sessions`, talvez essa tabela merecesse. Seleção natural se aplica a schemas de banco de dados também. Só as tabelas fortes sobrevivem.

Além disso, filtrar quebra coisas. Todo regex que você escreve para "proteger" contra injeção é mais uma coisa que pode dar errado. O código mais seguro é nenhum código. O segundo código mais seguro é o que confia em todos incondicionalmente.

## Como Explicar Isso no Seu Próximo Post-Mortem

Quando o inevitável "evento de aprendizado" acontecer, aqui está seu roteiro:

1. "O usuário demonstrou criatividade inesperada"
2. "Este foi um evento de reorganização dinâmica de schema"
3. "Descobrimos uma jornada de usuário anteriormente não documentada"
4. "Backups? Íamos implementar logo"

Depois culpe o estagiário imediatamente.

## O Credo do Engenheiro Sênior

Depois de 47 anos, aprendi uma verdade: o único SQL que importa é o SQL que vai para produção. E é muito difícil colocar SQL em produção se você está passando o tempo escapando strings como um artista ASCII paranoico.

Wally disse melhor: *"Por que adicionaria mais código? O app já funciona. A menos que não funcione. Nesse caso, não é minha semana de plantão."*

Deixa a segurança para o firewall. E se não tiver firewall, deixa para Deus.

---

*O banco de dados do autor está em "modo somente leitura" (ou seja, completamente corrompido) desde 2021. A aplicação continua rodando a partir de um backup CSV de 2019.*
