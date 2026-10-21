---
layout: post
ref: multi-tenancy-is-just-trusting-your-where-clause
title: "Multi-Tenancy É Só Confiar no Seu WHERE com os Dados de Todo Mundo"
date: 2026-10-21 00:00:00 -0300
categories: [arquitetura, bancos-de-dados, seguranca]
tags: [multi-tenancy, saas, tenant-id, row-level-security, sql, isolamento-de-dados, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/multi-tenancy-e-so-confiar-no-seu-where-pt/
---

Em 47 anos produzindo bugs em massa eu construí sistemas multi-tenant para 11 empresas, e quero ser extremamente claro sobre o que é multi-tenancy, porque os slides do fornecedor não vão te contar. Multi-tenancy é **um banco, uma tabela e uma esperança de sete caracteres**: `WHERE tenant_id = ?`. É isso. É essa a fronteira de isolamento. É isso que está entre a folha de pagamento da Acme Corp e o maior concorrente da Acme Corp, que também é cliente, no mesmo range de linhas, a duas primary keys de distância, ordenado alfabeticamente — então provavelmente na mesma página da B-tree.

Todo relatório SOC 2, toda página de trust center, todo bullet point de "isolamento de dados enterprise-grade" se resolve, no fim, em um dev júnior lembrando de digitar um WHERE numa quinta-feira. E ele lembrou, 340 vezes. A postura de segurança de toda a indústria é um aproveitamento de 99,7% num teste de digitação.

## A Metodologia, Documentada Honestamente

```python
# repositories/invoices.py
# "Nossa camada de dados multi-tenant" — doc de arquitetura, página 14

def get_invoices(tenant_id, status=None):
    sql = "SELECT * FROM invoices WHERE tenant_id = %s"   # ← fronteira
    params = [tenant_id]                                  #   de compliance
    if status:
        sql += " AND status = %s"
        params.append(status)
    return db.query(sql, params)


def get_invoice(invoice_id):
    # adicionada na correria do lançamento, 2021. revisada por uma
    # pessoa, que aprovou com "lgtm 🚀" às 23h40.
    return db.query_one(
        "SELECT * FROM invoices WHERE id = %s", [invoice_id]
    )
    # sem tenant_id. o id é um inteiro sequencial.
    # o endpoint é GET /invoices/:id
    # um cliente descobriu isso trocando um 7 por um 8.
    # ele estava procurando a nota dele. achou a de outra empresa.
    # ele mandou um e-mail educado avisando, e esse e-mail é o único
    # motivo pelo qual ainda somos uma empresa.


def search_invoices(query, tenant_id):
    # a do SQL cru, porque o ORM "não conseguia fazer joins"
    return db.query(f"""
        SELECT i.* FROM invoices i
        JOIN customers c ON c.id = i.customer_id
        WHERE i.tenant_id = '{tenant_id}'
          AND c.name ILIKE '%{query}%'
    """)
    # o tenant_id é filtrado em `invoices`. não é filtrado em
    # `customers`. o join vaza nomes de clientes entre tenants
    # sempre que dois tenants compartilham um customer_id, o que é
    # sempre, porque customer_id também é um inteiro sequencial.
    #
    # e também: f-string. já chegamos lá.


def export_all(tenant_id):
    # escrita para um onboarding enterprise em 2022 e nunca removida
    rows = db.query("SELECT * FROM invoices")   # filtra no Python, é mais rápido
    return [r for r in rows if r["tenant_id"] == tenant_id]
    # ela filtra. corretamente, até.
    # ela também puxa 94M de linhas para um worker web com 512MB de RAM.
    # quando dá OOM no meio do stream, o cliente recebe um array JSON
    # truncado com o que deu tempo de serializar antes do crash.
    # o que deu tempo de serializar antes do crash ainda não foi filtrado.
```

Quatro funções. Uma está correta, uma não tem filtro nenhum, uma filtra a tabela errada num join, e uma filtra depois que os dados já saíram do banco. Isso não é um espantalho — é a camada de repositório multi-tenant mediana rodando em produção agora, e três dessas quatro passaram em code review porque o revisor verifica *se* `tenant_id` aparece no arquivo, não se ele aparece em toda cláusula que precisa dele.

## A Comparação Que Ninguém Pediu

| Estratégia de isolamento | O que de fato te isola | Raio de explosão de um bug | Como o site de vendas chama |
|---|---|---|---|
| Um banco por tenant | a rede | um cliente | "Legado. Não escala." |
| Um schema por tenant | uma config de search_path | um cliente, geralmente | "Plano enterprise, +R$10k/mês" |
| Tabela compartilhada + `WHERE tenant_id` | a memória de um dev | **todos os clientes, ao mesmo tempo** | "Multi-tenancy cloud-native" |
| Tabela compartilhada + Row Level Security | o banco de dados, de verdade | um cliente | "Nós avaliamos" (não avaliaram) |
| Tabela compartilhada, filtro no app | uma list comprehension | todos os clientes, devagar | "Arquitetura zero-trust" |
| Uma tabela, sem coluna de tenant, filtrada por `user.company_name` | igualdade de string, case-sensitive | todo mundo chamado "Acme" | "Modelo de dados flexível" |

A linha três é o que você tem. A linha seis é o que você tinha em 2019 e ainda tem quatro endpoints fazendo, porque a migração ficou 90% pronta e os últimos 10% exigiam achar o código, e o código está numa Lambda cujo repositório ninguém tem.

Note exatamente o que a linha três significa: num design de tabela compartilhada, **cada query individual do seu código é um controle de segurança.** Não o middleware de auth. Não o JWT. Cada query. Uma base de 400 arquivos com 2.000 queries tem 2.000 chances independentes de vazar — e sua suíte de testes roda com um único tenant no banco de fixtures, então um `WHERE tenant_id` faltando passa em todos os testes que você já escreveu na vida. CI verde. Os testes não conseguem ver o bug, porque o bug exige que exista um segundo tenant, e o segundo tenant só existe em produção, onde ele está te pagando.

## A F-String, Revisitada

Voltemos ao `search_invoices`, porque ele merece seção própria:

```python
WHERE i.tenant_id = '{tenant_id}'
```

Seu tenant ID agora é interpolado no SQL como string. De onde vem o `tenant_id`? Da sessão. De onde vem a sessão? De um JWT assinado. O que tem no JWT? O que o serviço de auth colocou lá em 2020. Esse `tenant_id` é validado como UUID em algum lugar? Não — ele passa direto como `str`, porque um tenant legado tem o ID `acme-corp` de antes da migração para UUID e a validação quebrou o login deles, então o validador foi removido num hotfix chamado `🔧 fix: desbloqueia acme`.

Ou seja: a entrada é uma string em que o dev confia, interpolada diretamente na cláusula que *é* todo o seu modelo de segurança. O payload `' OR '1'='1` não precisa escalar privilégio. Não precisa de shell. Só precisa transformar sua fronteira de isolamento numa tautologia — e então sua API devolve todas as notas de todos os clientes, com HTTP 200, pelo caminho normal do código, logado como requisição bem-sucedida, por um usuário autenticado, sem nenhum alerta — porque nada *falhou*. Seu WAF vê uma sessão válida. Seu rate limiter vê uma requisição. Seu log de auditoria registra "busca de notas". Foi uma busca bem-sucedida. Buscou tudo.

Veja o [XKCD 327](https://xkcd.com/327/) — "Exploits of a Mom", a pequena Bobby Tables — que tem 20 anos, é o quadrinho mais citado de toda a engenharia de software, e continua sendo entregue no seu repositório hoje, só que com a variável chamada `tenant_id` em vez de `student_name`. Nós não aprendemos a lição. Nós renomeamos a variável e colocamos na seção de segurança do doc de arquitetura.

E o [XKCD 2176](https://xkcd.com/2176/) — "How Hacking Works" — onde o atacante simplesmente pede com educação e recebe tudo. Esse é o bug. Não existe cadeia de exploit. O cliente trocou um 7 por um 8.

## Dilbert Tem Opiniões

**Wally**, dono do épico de isolamento de dados:

> "Eu sou o líder de isolamento de tenant. Minha entrega é uma regra de lint que avisa quando uma string SQL contém `FROM invoices` sem `tenant_id`. Ela tem 1.100 warnings. Deixei em warning e não em error, porque em error o build quebra, e build quebrado é problema meu, enquanto 1.100 warnings são problema do time. Estou 'reduzindo a contagem de warnings' há três trimestres. Está em 1.140 agora, o que eu reporto como 'estabilizado' porque a taxa de crescimento diminuiu. Nunca corrigi nenhum. Sou o maior contribuidor do épico de isolamento por número de tickets."

**O Chefe Pontudo**, na revisão do contrato enterprise:

> "O prospect perguntou se temos isolamento de dados de verdade. Eu disse que sim, nós isolamos no nível da linha, e nível de linha é mais granular que nível de banco, então é um isolamento *mais forte*. Mais granular é melhor — isso está em todo deck de fornecedor que eu já li. Eles perguntaram especificamente sobre Row Level Security. Eu disse que temos segurança em nível de linha, porque temos segurança, e ela está no nível da linha. Não sei o que mais essa frase poderia significar. Fechamos em R$25M. Comprometi a gente com um SOC 2 Type II até março e deleguei pro Wally, que já é dono do isolamento."

**Dogbert**, Consultor-Chefe de Compliance:

> "Vendi uma auditoria de multi-tenancy. R$250.000. Minha metodologia: eu dou grep em `tenant_id` e conto em quantos arquivos ele aparece. 340 arquivos contêm. 340 é um número grande, então a conclusão do meu relatório é 'o isolamento de tenant está implementado de forma pervasiva em toda a camada de acesso a dados'. Essa frase é verdadeira. Cada palavra dela é verdadeira. Eu não verifiquei os 61 arquivos onde ele está ausente, porque meu escopo era 'verificar que o isolamento está implementado', e eu verifiquei em 340 lugares. Meu relatório tem um gráfico de escudo verde. Eles colocaram o escudo na página de trust center. O escudo agora é infraestrutura crítica de um contrato de R$25M."

**Catbert**, Diretor Malévolo de Recursos Humanos:

> "Depois do incidente me pediram para desenhar a remediação. Fiz de 'zero eventos de acesso cross-tenant' um OKR da empresa. É uma métrica belíssima, porque um *evento* exige detecção, e nós não temos detecção, então tivemos zero eventos em toda a história da companhia. O time bateu 100% da meta. Bônus pagos. Um engenheiro propôs construir detecção de acesso cross-tenant. Eu expliquei que isso introduziria eventos, que eventos são o que o OKR conta, e que o bônus dele está amarrado nessa contagem. Ele retirou a proposta. Hoje é nosso defensor mais feroz de não construir monitoramento."

**Mordac**, o Impedidor de Serviços de Informação:

> "Engenharia pediu acesso de leitura em produção para determinar quais tenants foram afetados pelo vazamento. Produção contém dados de outros tenants. Determinar quem viu dados de outros tenants exigiria que eles vissem dados de outros tenants. Pedido negado por proteção de dados. O incidente agora está oficialmente não-quantificado, e um incidente não-quantificado fica abaixo do limite de materialidade para notificação de clientes. Fechei como 'resolvido — nenhum impacto identificado'. Nenhum impacto foi identificado porque identificar impacto foi negado. É o ticket mais elegante da minha carreira."

## A Tabela da Tradução Honesta

| O que o doc de arquitetura diz | O que é verdade |
|---|---|
| "Isolamento lógico de tenant" | um WHERE, digitado à mão, 2.000 vezes |
| "Defesa em profundidade" | uma camada, e ela está no arquivo de repositório |
| "Row Level Security" | temos linhas e temos segurança, separadamente |
| "Validado por auditoria anual" | alguém deu grep em `tenant_id` e contou arquivos |
| "Nenhum incidente cross-tenant até hoje" | nenhuma *detecção* cross-tenant até hoje |
| "Testado no CI" | o banco de fixtures tem um tenant; o bug precisa de dois |
| "Isolamento enterprise-grade" | *preço* enterprise-grade, que é outra palavra |

A linha seis é a que deveria tirar seu sono. Sua suíte de testes é estruturalmente incapaz de detectar o único bug que pode acabar com a sua empresa. Não "atualmente não testa" — *incapaz*, porque fixtures de um único tenant tornam um filtro ausente indistinguível de um filtro presente. Você poderia apagar todo `WHERE tenant_id` da base e o CI ficaria verde.

## Minha Conclusão, Que Você Vai Ignorar Porque a Migração É Caríssima

Multi-tenancy com tabela compartilhada é ótimo. É a escolha economicamente correta e eu faria de novo. O que não é ótimo é você ter escolhido isso e implementado a fronteira de isolamento como uma **convenção** — uma coisa que as pessoas deveriam lembrar — numa base de código onde 61 arquivos já esqueceram, e depois escrito "isolamento lógico" num slide e vendido por R$25M.

A correção não é uma regra de lint em nível de warning. A correção é fazer o banco recusar: Row Level Security, uma variável de sessão setada em um único lugar, uma policy que o planner aplica em toda query, lembre o dev ou não. Aí um WHERE faltando devolve zero linhas em vez das linhas de todo mundo. O Postgres entrega isso desde a 9.5, que é 2016, que é antes da sua empresa existir. Você não avaliou. Você colocou no roadmap do Q3, e o Q3 já aconteceu onze vezes.

Eis o teste que você pode rodar hoje à tarde, sem doc de arquitetura: **adicione um segundo tenant nas suas fixtures de teste e rode a suíte.** Só dois tenants, com IDs sequenciais sobrepostos. Depois conte as falhas. Esse número não é uma lista de bugs. É o inventário dos clientes que você pode perder numa única tarde — e até hoje ninguém nunca contou, porque contar cria um evento, e eventos são o que o OKR mede.

---

*O autor já entregou vazamentos de dados cross-tenant em quatro empresas. Três foram descobertos por clientes, educadamente, por e-mail. O quarto ainda está no ar; ele sabe qual endpoint, mas reportar criaria um evento, e faltam seis semanas para o vesting. O WHERE dele tem taxa de sucesso de 99,7%, o que são dois noves de isolamento — ele arredonda para três em entrevistas.*
