---
layout: post
ref: sharding-is-buying-several-databases-to-lose-a-join-in-each-one
title: "Sharding É Só Comprar Vários Bancos Pra Perder um JOIN em Cada Um"
date: 2026-10-26 00:00:00 -0300
categories: [bancos-de-dados, arquitetura, conselhos-terriveis]
tags: [sharding, shard-key, escalabilidade-horizontal, postgres, mysql, sistemas-distribuidos, scatter-gather, resharding, hot-shard, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/2026/10/26/sharding-e-so-comprar-varios-bancos-pra-perder-um-join-em-cada-um-pt/
---

Em 47 anos produzindo bugs em escala industrial, nunca vi uma decisão técnica ser tomada tão rápido, com tão pouca evidência, por tão pouca gente, quanto a escolha de uma shard key. A nossa levou quarenta minutos. Vinte e cinco deles foram esperando a sala de reunião liberar. A decisão já sobreviveu a dois CTOs, uma aquisição e o time de engenharia original inteiro — e ainda é, hoje, o motivo de um relatório que deveria levar nove segundos levar quatro minutos e vir errado.

Aqui está o conselho que me rendeu uma promoção: **quando o banco ficar lento, divida ele em oito.** Uma máquina grande é um ponto único de falha e um teto rígido. Oito máquinas são oito vezes a CPU, oito vezes a RAM, oito vezes o IOPS. É aritmética. Ninguém discute aritmética numa reunião de planejamento. O slide praticamente se desenha sozinho, e tem oito cilindrozinhos nele, e os oito cilindrozinhos são todos do mesmo tamanho — que é a primeira mentira e a base sobre a qual todas as outras são construídas.

O que ninguém fala em voz alta, porque encerraria a reunião cedo demais, é que sharding não deixa seu banco mais rápido. Deixa ele **menor**, oito vezes, e devolve cada uma das coisas que o banco fazia por você — joins, unicidade, transações, ordenação, contagem, foreign keys, backup consistente — de volta pro seu colo, em Python, na camada de aplicação, pra ser reimplementada mal por alguém que vai ter saído da empresa antes de isso importar. [XKCD 1205](https://xkcd.com/1205/) é o gráfico sobre quando vale a pena automatizar uma tarefa. Sharding é o gráfico inverso: uma decisão de uma tarde que cria um imposto permanente, composto, de seis pessoas-ano — e não existe versão do gráfico em que ele se paga.

## Os Quarenta Minutos Que Custaram Seis Anos

```python
# shards.py — escrito numa quinta-feira. Nunca revisado. Sustenta tudo.

SHARDS = {i: create_engine(os.environ[f"DB_SHARD_{i}"]) for i in range(8)}

def shard_for(user_id: int):
    """Shard por user_id. Usuário é a fronteira natural do nosso domínio.
    Tudo pertence a um usuário. Isso sempre vai ser verdade."""
    return SHARDS[user_id % 8]
```

Quatro linhas. Leia de novo, porque quatro linhas é o raio de explosão inteiro.

`user_id % 8` não é uma função, é uma **constituição**. Ela declara que oito é uma propriedade permanente do universo, que `user_id` é conhecido no topo de toda pilha de chamadas pra sempre, e que nenhuma pergunta que alguém venha a fazer cruza mais de um usuário. As três são falsas, e você vai descobrir nessa ordem, mais ou menos de onze em onze semanas.

A primeira rachadura chega pelo suporte, e chega numa frase tão razoável que ninguém nem registra como bug:

```python
# "Consegue achar a conta do billing@acme.example?"

def find_by_email(email):
    # Não temos user_id. Temos um email.
    # O email não diz em qual das oito máquinas o usuário está.
    # Então:
    for shard in SHARDS.values():
        user = shard.query(User).filter_by(email=email).first()
        if user:
            return user
    return None
    # Oito queries pra responder uma pergunta que era um index lookup.
    # Sete delas não retornam nada. As oito consomem conexões do pool,
    # e isso roda em todo reset de senha, todo login, todo ticket
    # de suporte, todo webhook da Stripe.
```

Isso é um **scatter-gather**, e é o movimento característico de um sistema shardeado: faça o trabalho oito vezes, jogue sete oitavos fora, pague pelos oito. Não é lento no começo. Essa é a armadilha. Com oito shards e pouco tráfego é *imperceptível*, então vira copy-paste — e quando você tiver trinta e dois shards ele é o maior consumidor de conexões da sua frota, sem dono, sem ticket, e sem nenhum momento em que alguém tenha decidido isso.

## Tudo Que o Banco Fazia, Agora no Seu Código

**A unicidade acabou.** `UNIQUE(email)` é garantido por máquina. Oito máquinas, oito garantias independentes de unicidade, zero garantias globais. Duas pessoas se cadastram com o mesmo email, com 300ms de diferença, caem em shards diferentes, e as duas conseguem — e a constraint disparou corretamente nas duas, porque as duas estavam certas, localmente. Agora você tem duas contas com o mesmo email e um fluxo de reset de senha que escolhe o shard que responder primeiro. A correção é uma tabela global de lookup num banco separado e não-shardeado, o que significa: a coisa de que você shardeou pra fugir voltou, obrigatória, no caminho quente de todo cadastro — e se ela cair, a frota shardeada inteira fica inalcançável. Você reinventou o ponto único de falha e deu um nome pior pra ele, `directory_db`, e ninguém vai dimensionar direito, porque "é só um lookup".

**Os JOINs acabaram.** Não ficaram lentos. *Acabaram.* `users` é shardeado por `user_id`, `organizations` por `org_id`, e no momento em que um usuário pertence a uma org em outra máquina, `JOIN` deixa de ser uma palavra que o seu banco entende:

```python
# O que era:
#   SELECT u.name, o.plan FROM users u JOIN organizations o ON u.org_id = o.id
#   WHERE o.plan = 'enterprise'
#   — 12ms, um index scan, o planner fez algo esperto que nunca aprendi.

# O que é agora:
def enterprise_users():
    orgs = []
    for shard in SHARDS.values():                        # scatter #1
        orgs += shard.query(Org).filter_by(plan="enterprise").all()

    users = []
    for org in orgs:                                      # 4.200 orgs
        for shard in SHARDS.values():                     # scatter #2, aninhado
            users += shard.query(User).filter_by(org_id=org.id).all()

    return users
    # 8 + (4200 * 8) = 33.608 queries.
    # Isso é um nested loop join. Você escreveu um nested loop join.
    # Em Python. Sem estatísticas, sem cost model, sem escolha de índice,
    # e sem conseguir fazer spill pra disco. O query planner que o Postgres
    # passou trinta anos aperfeiçoando foi substituído por um `for`
    # escrito por alguém que estava com pressa.
```

[XKCD 974](https://xkcd.com/974/), *The General Problem*: pedem pra passar o sal, o engenheiro constrói um framework genérico de passagem de condimentos. Sharding é isso, só que o framework genérico é uma cópia pior do banco que você já tinha.

**As transações acabaram.** `BEGIN; UPDATE a; UPDATE b; COMMIT;` só significa alguma coisa se `a` e `b` moram na mesma máquina. Entre shards você tem dois commits e uma reza. A resposta correta é two-phase commit, que seu ORM não implementa, que seu DBA não vai habilitar, e que trava pra sempre se o coordenador morrer no meio do prepare. Então o que todo mundo de fato entrega é isto:

```python
def transfer(from_user, to_user, amount):
    shard_for(from_user).execute("UPDATE accounts SET balance = balance - :amt ...")
    shard_for(to_user).execute("UPDATE accounts SET balance = balance + :amt ...")
    # Dois commits separados. Se o processo levar OOM-kill entre eles —
    # e vai levar, numa terça, durante um deploy — o dinheiro é subtraído
    # e nunca somado. Não tem rollback. Não tem registro da intenção.
    # Tem um ticket de suporte, eventualmente, aberto pelo cliente,
    # que agora é o seu mecanismo de detecção de inconsistência.
```

**A contagem acabou.** `SELECT COUNT(*) FROM users` vira oito contagens somadas em Python, tiradas em oito instantes diferentes, e portanto igual a um número que nunca foi verdade simultaneamente. É perto o suficiente, por isso vai pra produção, e não é perto o suficiente na fatura, por isso o financeiro para de confiar no dashboard.

**A paginação acabou.** `ORDER BY created_at LIMIT 20 OFFSET 10000` em oito shards exige buscar 10.020 linhas **de cada shard**, juntar 80.160 linhas na memória da aplicação, ordenar e descartar 80.140. Seu endpoint de "página 500" aloca 300MB e leva onze segundos. Alguém vai "corrigir" limitando a paginação na página 50 e chamando isso de decisão de produto.

## O Hot Shard, Que É Sempre o Seu Melhor Cliente

Aqui está a parte que o slide dos oito cilindros iguais não consegue mostrar. Dado não é uniforme. Cliente não é uniforme. Um tenant que assinou em 2019 cresceu 400x e hoje concentra 41% das suas linhas — e o `user_id % 8` colocou tudo isso no shard 3.

O shard 3 está com 94% de disco, 80% de CPU e queries de 40 segundos. Os shards 0, 1, 2, 4, 5, 6 e 7 estão em 11%. A *média* de utilização é 22%, que é o número no dashboard de capacidade, que é o motivo pelo qual o pedido de capacidade foi negado. Você não consegue escalar o shard 3 verticalmente além da maior instância que a nuvem vende, e não consegue tirar aquele tenant de lá, porque mover um tenant significa mudar a shard key, e a shard key é derivada da primary key, e a primary key está em quarenta e uma colunas inteiras meio-foreign-key espalhadas por nove serviços que cada um cacheou.

Então o shard 3 vira uma personalidade. Ganha apelido. Novos engenheiros ouvem falar dele no onboarding. Tem um canal no Slack. Alguém escreveu um cron que roda `VACUUM FULL` nele às 4h da manhã e ninguém sabe se ajuda. Isso não é mais um banco de dados; é um **ponto turístico**.

## Resharding: A Operação Que Ninguém Nunca Terminou

Oito não é suficiente. Você precisa de dezesseis. Eis o que `user_id % 8` → `user_id % 16` significa: todo usuário cujo ID é ≡ 8..15 mod 16 — ou seja, **metade de todas as linhas do sistema** — precisa se mover fisicamente pra outra máquina, com o sistema no ar, com escritas acontecendo, sem nenhuma transação global disponível pra fazer o corte atomicamente.

O plano honesto é: dual-write pro antigo e pro novo, backfill do histórico, verificar, virar as leituras, verificar de novo, parar o dual-write, derrubar o antigo. Seis a nove meses. Três engenheiros. Nenhuma feature nova. O backfill vai encontrar linhas que violam constraints que você adicionou depois que elas foram escritas. A verificação vai encontrar 0,02% de divergência que ninguém consegue explicar e todo mundo concorda em aceitar. E em algum ponto do meio, um deploy sobe código que lê pelo mapeamento novo e escreve pelo antigo, e por quarenta minutos você está gravando dados em máquinas onde nada jamais vai procurar por eles de novo.

É por isso que `user_id % 8` é o pior esquema possível e é por isso que todo mundo escolhe ele: **hashing por módulo acopla o layout dos dados à quantidade de shards**. Consistent hashing ou buckets virtuais por faixa — 4.096 buckets lógicos mapeados pra nós físicos — transforma adicionar capacidade numa reatribuição de bucket em vez de uma migração que para a empresa. Custa uma indireção a mais e uma tarde de pensamento no dia um. Ninguém gasta essa tarde. Eu não gastei essa tarde. Gastei quarenta minutos, e vinte e cinco deles esperando sala.

Ah, e seus IDs colidem. `AUTO_INCREMENT` é por máquina, então shard 0 e shard 5 produzem alegremente o pedido `#1001`. Você descobre isso no dia em que alguém faz uma feature que junta dados de dois shards num CSV, e o CSV tem chaves duplicadas, e o sistema downstream — que é o do financeiro — silenciosamente fica com a última. A correção são IDs tipo Snowflake, e a correção precisa ser aplicada antes da primeira linha ser escrita, e você vai saber dela depois da quatro milionésima.

## Dilbert Tem Opiniões

**Wally**, dono da Iniciativa de Sharding:

> "Me deram o projeto de sharding porque usei a expressão 'escalabilidade horizontal' numa reunião onde o VP estava presente. Meu entregável é a quantidade de shards, e eu entreguei: oito. Escolhi oito porque é potência de dois, o que sinaliza rigor técnico, e porque o diagrama ficava desbalanceado com seis. O plano de migração é um ticket chamado 'Plano de Migração' contendo as palavras 'TBD — precisa de discussão'. Está aberto há dois anos e é a maior conquista da minha carreira, porque enquanto estiver aberto ninguém pode dizer que o projeto falhou. Já fui promovido uma vez na força de um slide com oito cilindros. Os cilindros eram todos do mesmo tamanho. Desenhei assim porque a ferramenta de forma vem com o mesmo tamanho por padrão."

**O Chefe de Cabelo Pontudo**, na revisão de arquitetura:

> "Então, se oito shards nos deixaram oito vezes mais rápidos, eu gostaria que explorássemos oitocentos. Me disseram que existem 'custos de coordenação', mas coordenação é um problema de pessoas e nós temos uma cultura muito forte. Também fui informado de que o shard 3 está 'quente' — dá pra colocar ele nas instâncias rápidas? Não? Então vamos mover metade dos clientes dele pro shard 4. Eu entendo que isso exige mudar os IDs deles. Mudem os IDs. Os IDs são *nossos*, a empresa é dona. E mais uma coisa: o conselho quer o total de usuários no deck semanal, e estão me dizendo que leva quatro minutos pra calcular e é 'aproximado'. Eu não quero aproximado. Quero o número real, num slide, até quinta."

**Dogbert**, Principal Distributed Data Architect (contrato: R$ 1.700.000):

> "O escopo era desenhar uma topologia global de sharding. Produzi um artefato de 340 páginas, *Uma Arquitetura de Referência Federada e Multi-Região para Particionamento de Dados*, com dezessete diagramas, um conselho de governança de shard key, e um formulário de intake de estratégia de partição exigindo duas assinaturas de VP. As páginas 1–311 estabelecem o enquadramento estratégico. A página 312 contém a constatação: a maior tabela deles tem 180 milhões de linhas e cabe confortavelmente numa única máquina com um upgrade de R$ 2.000/mês e um índice composto que eles nunca criaram. Não dei ênfase à página 312. As páginas 313–340 definem a Fase Zero, que é um workshop de descoberta de seis semanas pra alinhar os stakeholders sobre a definição de tenant. Estou faturando R$ 1.700.000 pra não mencionar um índice. Me pediram pra liderar a Fase Um."

**Catbert**, Diretor Maligno de Recursos Humanos:

> "Transformei 'shardear o banco monolítico' num OKR de empresa com data, e deixei a data fazer o trabalho. Entregaram oito shards no prazo — bônus pagos, aplauso no all-hands — adiando o plano de resharding, a camada de unicidade global e a história de transação entre shards, nenhum dos quais estava no OKR porque nenhum deles tem artefato entregável. Os três engenheiros que entendiam a shard key já saíram pra concorrentes; dois citaram 'a migração' na entrevista de desligamento, que arquivei sob 'Attrition: Voluntária — Oportunidade de Crescimento'. O time atual não consegue mudar a quantidade de shards sem autorização de pessoas que não existem mais. É a estrutura mais duradoura que já criei, e fiz com uma única data numa planilha."

**Mordac**, o Impedidor de Serviços de Informação:

> "Engenharia solicitou oito shards de staging pra ensaiar o cutover do resharding. Provisionamento de banco não-produtivo exige revisão de capacidade, e a revisão apurou que oito instâncias de staging excedem a alocação trimestral de nuvem do time. Negado. Solicitaram então uma única instância com oito bancos lógicos, o que pelo menos exercitaria o código de roteamento. Também negado, pois 'banco lógico' não é uma classe de ativo aprovada no sistema de inventário e não pode receber identificador de CMDB. Eles agora testam o roteamento de shard contra um único banco, onde `shard_for()` sempre retorna a mesma conexão e portanto não pode falhar. Todo defeito entre shards na história do produto chegou à produção sem exame. Preveni 100% do provisionamento não autorizado em não-produção. Meu scorecard está verde e o resharding foi adiado indefinidamente, o que o sistema de inventário registra como economia."

## A Tabela de Tradução Honesta

| O que o slide diz | O que é de fato verdade |
|---|---|
| `8x de capacidade` | 8x de máquinas, 1x o cliente gigante, 22% de uso médio |
| `escalável horizontalmente` | escalável numa dimensão, escolhida em quarenta minutos, em 2019 |
| `shardeamos por user_id` | toda pergunta que não for em termos de `user_id` custa 8 queries |
| `queries entre shards são raras` | são raras no design doc e constantes em produção |
| `a gente resharda quando precisar` | não vai. ninguém nunca fez. é projeto de 9 meses sem demo |
| `a tabela de lookup é leve` | é um ponto único de falha pros oito shards |
| `COUNT(*) é aproximado` | o financeiro parou de usar seu dashboard e fez o próprio |
| `transações são tratadas na aplicação` | não são tratadas |
| `é só um loop aninhado, o dado é pequeno` | 33.608 queries, e o dado era pequeno em 2019 |

A linha cinco é o artigo inteiro. Sharding não é uma estratégia de escala, é uma **decisão de escala que você só pode tomar uma vez**, e a maior parte dos times toma antes de ter a informação necessária pra tomar, sob prazo, com uma ferramenta de desenho que vem com cilindros do mesmo tamanho por padrão.

## Minha Conclusão, Que Você Vai Ignorar Porque É Chata

Sharding é engenharia de verdade e às vezes é genuinamente necessário. É necessário quando uma única máquina não comporta o seu working set ou não absorve sua taxa de escrita, e você *mediu* os dois. Não é necessário porque uma query está lenta. Query lenta é índice faltando, `N+1`, `SELECT *` numa tabela de 40 colunas, ou sequential scan numa coluna que você esqueceu de analisar. [XKCD 927](https://xkcd.com/927/) é sobre padrões concorrentes; a versão sharding é que você tinha um banco com um query planner maduro e trocou por oito bancos e um `for` que você mantém sozinho.

Antes de shardear: crie o índice. Leia a saída do `EXPLAIN (ANALYZE, BUFFERS)` em vez de adivinhar. Arquive os 80% de linhas com mais de dois anos que query nenhuma toca — na maioria dos sistemas só isso compra três anos. Tire as quatro queries de analytics do primário. Suba a instância; uma máquina com 128 cores e 4TB de RAM custa menos por mês do que uma semana de engenheiro, e você está propondo gastar cem delas. Particione *dentro* de um único banco — particionamento declarativo te dá quase todo o benefício de pruning sem nada da distribuição, e você mantém joins, transações, unicidade e `COUNT(*)`.

Se tiver mesmo que shardear, gaste a tarde que você ia pular. **Nunca use módulo sobre a quantidade física de shards** — use muito mais buckets lógicos do que nós, pra que crescer seja uma reatribuição e não uma migração. Escolha a shard key escrevendo suas vinte queries de produção mais frequentes e verificando que todas conseguem nomear a chave na hora da chamada; se três não conseguem, a chave está errada, e você descobriu isso de graça em vez de no ano quatro. Gere IDs globalmente únicos desde o dia um. Escreva o runbook de resharding *antes* de precisar, e ensaie uma vez, porque runbook que nunca foi executado é ficção. E deixe o acesso entre shards barulhento — `with scatter_gather(reason="busca admin"):` — pra que o movimento caro seja uma frase que alguém digitou de propósito, não um padrão que se espalhou por copy-paste às 16h50 de uma sexta.

Depois rode a checagem mais barata que existe: abra seu endpoint mais lento, conte as queries de verdade, e pergunte se um único índice bem escolhido teria tornado essa conversa inteira desnecessária. Vai levar uma hora. Ninguém vai agendar, porque "criar um índice" não tem oito cilindros, e o slide com oito cilindros é o que rende promoção.

---

*O autor shardeou um banco por `user_id % 8` pra resolver um relatório lento. O relatório era lento porque não tinha cláusula `WHERE`. Ele descobriu isso quatro anos depois, enquanto escrevia o plano de resharding, que não terminou, e que segue aberto no backlog num ticket chamado "Plano de Migração — TBD, precisa de discussão".*
