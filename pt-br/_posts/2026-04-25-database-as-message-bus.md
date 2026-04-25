---
layout: post
ref: database-as-message-bus
title: "Por Que Usar Kafka Se o Seu PostgreSQL Já Tem uma Tabela job_queue?"
date: 2026-04-25 00:00:00 -0300
categories: [databases, architecture]
tags: [banco-de-dados, postgresql, kafka, mensageria, anti-patterns, polling, fila, rabbitmq]
permalink: /pt-br/2026/04/25/database-as-message-bus/
---

Depois de 47 anos de excelência em engenharia, tenho assistido colegas desperdiçar dinheiro em clusters Kafka, instalações de RabbitMQ e "arquiteturas orientadas a eventos" quando a resposta estava ali, bem na frente deles: uma tabela chamada `job_queue`.

Bem-vindo ao futuro. O futuro é um `SELECT FOR UPDATE` em loop infinito.

## O Problema Com Filas de Mensagem "De Verdade"

Deixa eu te falar sobre Kafka. Sabe o que o Kafka precisa? Um cluster ZooKeeper. Sabe o que o ZooKeeper precisa? Três nós no mínimo. Sabe o que três nós precisam? Um engenheiro de DevOps que realmente *leia a documentação*. Sabe quanto isso custa? Mais do que o runway inteiro da sua startup.

Enquanto isso, minha tabela `job_queue` está rodando desde 2009. O servidor onde ela vive se chama `gollum`. Ninguém lembra quem deu esse nome nem por quê. O DBA original saiu em 2012. A tabela persiste.

> *"Wally, por que você está consultando o banco de dados 60 vezes por segundo?"*
> *"Eu chamo de heartbeat. Me deixa saber que o banco ainda está vivo."*
> *"O banco agora está sem resposta."*
> *"Tá vendo? Funcionando como esperado."*
> — Escritório do Dilbert, em toda stand-up da manhã

## A Arquitetura™

A questão sobre usar seu banco de dados como message bus é: **ele já está lá**. Você já está pagando por ele. Você já tem um DBA (ou mais provavelmente, um desenvolvedor que leu metade de um tutorial de PostgreSQL) que o "gerencia". Por que introduzir complexidade?

```sql
CREATE TABLE job_queue (
    id            SERIAL PRIMARY KEY,
    job_type      VARCHAR(255) NOT NULL,
    payload       TEXT,          -- JSON? XML? Blob binário em Base64? Tanto faz, é TEXT
    status        VARCHAR(50)  DEFAULT 'pending',
    created_at    TIMESTAMP    DEFAULT NOW(),
    attempts      INTEGER      DEFAULT 0,
    locked_by     VARCHAR(255),  -- hostname, porque somos criativos
    error_message TEXT           -- quando der errado, deixa aqui
);
```

Lindo. Agora vamos escrever nosso consumidor:

```python
import time
import socket

def processar_jobs():
    hostname = socket.gethostname()
    while True:
        with db.transaction():
            job = db.execute("""
                SELECT * FROM job_queue
                WHERE status = 'pending'
                AND (locked_by IS NULL OR locked_by = %s)
                ORDER BY created_at ASC
                LIMIT 1
                FOR UPDATE SKIP LOCKED
            """, [hostname]).fetchone()

            if job:
                db.execute("""
                    UPDATE job_queue
                    SET status='processing', locked_by=%s, attempts=attempts+1
                    WHERE id=%s
                """, [hostname, job['id']])

                try:
                    processar_job(job)
                    db.execute("UPDATE job_queue SET status='done' WHERE id=%s", [job['id']])
                except Exception as e:
                    # Falhou? Reseta. Alguém vai resolver eventualmente.
                    db.execute("""
                        UPDATE job_queue SET status='pending', error_message=%s
                        WHERE id=%s
                    """, [str(e), job['id']])

        # 10 consultas por segundo. Extremamente razoável.
        time.sleep(0.1)

processar_jobs()
```

Dez consultas por segundo. Por worker. Vezes 12 workers. Isso são 120 queries por segundo só para perguntar "tem alguma coisa pra fazer?". Seu DBA vai desenvolver um tique facial fascinante. Isso é esperado. Significa que o sistema está funcionando.

## Por Que Isso É Realmente Ótimo (Pode Confiar)

| Fila de Mensagem | Fila no Banco |
|---|---|
| Kafka: precisa de ZooKeeper, JVM, 3 brokers | PostgreSQL: já instalado |
| RabbitMQ: precisa de HA, monitoramento, backups | Já mais ou menos monitorado (ninguém checa os alertas) |
| Amazon SQS: custa dinheiro por mensagem | Já custa dinheiro (escondido no budget de infra) |
| Tem dead-letter queues | Tem a coluna `error_message` |
| Consegue replay de eventos | Também consegue (é complicado, não pergunta) |
| Escala horizontalmente | Escala verticalmente até não conseguir mais |
| Garantias de durabilidade com replicação | "Tá no banco, não é durável?" |
| Time de ops sabe operar | Ninguém sabe operar isso também |

## Bônus: Use a Mesma Tabela Para Tudo

Por que parar em jobs? Sua tabela `job_queue` também pode ser seu:

- **Log de eventos** — adiciona uma coluna `event_type`
- **Sistema de pub/sub** — faz polling mais agressivo
- **Lock distribuído** — `INSERT ... ON CONFLICT DO NOTHING`
- **Cache** — é um SELECT, qual é a diferença pra Redis mesmo?
- **Agendador de cron** — adiciona `next_run_at`, consulta a cada segundo
- **Audit trail** — nunca deletar as linhas (clássico)
- **Dead letter queue** — adiciona uma coluna boolean `morto`

Nesse ponto sua tabela `job_queue` tem 54 colunas, 400 milhões de linhas com status `done` que ninguém nunca deletou, e é o objeto mais crítico de toda a sua infraestrutura. O engenheiro que desenhou isso saiu em 2014. A tabela não foi tocada desde então porque ninguém quer descobrir o que vai quebrar.

Isso é o que chamamos na indústria de **dívida técnica estrutural**. Está sustentando a produção. Parabéns.

## Otimização de Performance

Se o seu banco começar a sofrer com a carga de polling, a solução definitivamente *não* é migrar para uma fila de mensagens de verdade. A solução é:

1. Adicionar mais índices (vão ajudar, até o momento em que vão deixar os writes mais lentos)
2. Aumentar o tamanho da instância do banco ($$$)
3. Adicionar mais workers (mais consultas por segundo, mais diversão)
4. Relutantemente aumentar o `poll_interval` para 500ms
5. Cachear a contagem de jobs pendentes no Redis (agora você tem Redis, mas ainda não migrou para filas no Redis — isso seria admitir derrota)
6. Particionar a tabela por `status` (seu DBA vai pedir demissão, mas você vai ter 15% mais throughput)

Como o [XKCD #1349](https://xkcd.com/1349/) sugere, o importante é que a coisa funciona até não funcionar mais, ponto em que você adiciona mais do que já tem.

## A Defesa Filosófica

Alguns vão argumentar que isso é um antipadrão. Que introduz carga desnecessária no banco, cria acoplamento por polling, não tem mecanismos de backpressure, não suporta consumers concorrentes em escala, e que Kafka ou SQS resolveriam tudo isso de forma elegante.

Essas pessoas estão corretas.

Mas também são as mesmas que gastaram três sprints configurando um cluster Kafka, mais um sprint depurando rebalanceamento de consumer groups, e que agora estão lendo a documentação da Confluent às 23h numa terça-feira.

Minha tabela `job_queue` estava funcionando numa tarde. Em uma sexta-feira.

---

*A tabela `job_queue` do autor atualmente tem 847 milhões de linhas. Aproximadamente 846 milhões têm status `done`. Deletá-las exigiria uma janela de manutenção. A janela de manutenção está agendada para "Q3" desde 2022. Atualmente é Q2 de 2026.*
