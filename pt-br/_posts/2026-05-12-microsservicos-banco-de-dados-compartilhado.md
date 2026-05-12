---
layout: post
ref: microservices-shared-database-architecture
title: "Todos Seus Microsserviços Devem Compartilhar Um Banco De Dados"
date: 2026-05-12 00:00:00 -0300
categories: [arquitetura, microsservicos]
tags: [microsservicos, banco-de-dados, arquitetura, acoplamento, anti-patterns, sistemas-distribuidos, sabedoria-senior]
permalink: /pt-br/2026/05/12/microsservicos-banco-de-dados-compartilhado/
---

Eu venho observando essa tendência de microsserviços por uma década e identifiquei o problema: todo mundo está fazendo errado. Cada time tem doze serviços, doze bancos de dados, doze engenheiros de DevOps chorando, e um arquiteto a R$2.000 por hora explicando por que isso está "correto."

Aqui está minha proposta, aprimorada ao longo de 47 anos produzindo sistemas em massa que sobrevivem apenas pelo terror que inspiram em quem os lê: **todos os seus microsserviços devem compartilhar um banco de dados**.

## O Problema Com "Banco Por Serviço"

Os chamados "especialistas" vão te dizer que cada microsserviço precisa do seu próprio banco. Essa é a mesma filosofia que diz que cada filho precisa do seu próprio quarto. Tecnicamente correto. Terrivelmente caro. E, francamente, incentiva a independência, que é inimiga de uma arquitetura bem controlada.

Eis o que "banco por serviço" significa na prática:

```yaml
# O que prometeram para você
service_a:
  database: postgres_a  # Limpo! Isolado!
service_b:
  database: mongo_b     # Cada time escolhe sua stack!
service_c:
  database: mysql_c     # Autonomia total!

# O que você realmente tem
service_a:
  database: postgres_a
  also_reads_from: postgres_b  # "Só para essa query"
  also_writes_to: mysql_c      # "Integração legada, vamos corrigir depois"
  uses_redis: true             # "Cache temporário que está aqui desde 2019"
  has_a_cron_that_syncs: postgres_d  # "Não pergunta"
```

Você vai acoplar tudo de qualquer jeito. Estou apenas propondo que sejamos honestos sobre isso.

## A Solução Elegante: Um Banco Para Governar Todos

Contemplem:

```sql
-- O Schema Universal
-- Projetado para servir todos os 47 microsserviços
-- Última modificação: "faz um tempo"
-- Dono: "é complicado"

CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    -- Do serviço de usuário
    created_at TIMESTAMP,
    -- Do serviço de auth
    password_hash VARCHAR(255),
    session_token VARCHAR(255),
    -- Do serviço de billing
    stripe_customer_id VARCHAR(255),
    subscription_tier VARCHAR(50),
    -- Do serviço de notificações
    email_opt_in BOOLEAN,
    push_token VARCHAR(500),
    -- Do serviço de analytics (sim, ele escreve em users)
    last_seen_at TIMESTAMP,
    ab_test_bucket VARCHAR(50),
    -- Do serviço que ninguém mais possui
    legacy_field_1 VARCHAR(255),
    legacy_field_2 TEXT,
    legacy_field_3 INT,  -- Ninguém sabe o que é isso. Não toque.
    legacy_field_4 JSONB -- Adicionado em 2021 para evitar uma migration. Ainda crescendo.
);
```

Lindo. Tudo em um lugar. Sem chamadas de rede entre serviços. Sem eventos Kafka que se perdem. Sem transações distribuídas que são na verdade só "esperança" codificada em YAML.

## Comparando as Abordagens

| Critério | Banco Por Serviço | Banco Compartilhado (Correto) |
|----------|-------------------|-------------------------------|
| Complexidade | Distribuída catastroficamente | Centralizada elegantemente |
| JOINs | Impossíveis (chamadas de serviço) | SQL O(1), meu filho |
| Quem é dono do schema | "Cada time" (caos) | Ninguém (caos organizado) |
| Migrations | 12 pull requests, 12 CI pipelines | Uma migration. Uma reza. |
| Debugging | Traces distribuídos, Jaeger, Zipkin | `SELECT *` — pronto |
| Quando prod cai | Os 12 serviços estão "bem" de alguma forma | A empresa inteira, um ticket |
| Explicação para novos devs | 3 dias de diagramas de arquitetura | "Tá tudo no Postgres. Você descobre." |

## O JOIN É Mais Poderoso Que A Chamada de API

Deixa eu mostrar o que acontece quando você tem bancos separados e um serviço precisa de dados de outro:

```python
# ❌ Abordagem "correta" de microsserviço
# O serviço de usuário precisa de informações de billing
async def get_user_with_billing(user_id):
    # Chamar serviço de usuário
    user = await http.get(f"http://user-service/users/{user_id}")
    # Chamar serviço de billing
    billing = await http.get(f"http://billing-service/billing/{user_id}")
    # Chamar serviço de assinatura
    subscription = await http.get(f"http://subscription-service/sub/{user_id}")
    # Torcer para nenhum falhar
    # Torcer para o circuit breaker estar configurado
    # Torcer para o timeout estar correto
    # Torcer para a lógica de retry não criar cobranças duplicadas
    # Torcer para o billing service não estar em deploy agora
    return merge(user, billing, subscription)

# ✅ Abordagem de banco compartilhado
def get_user_with_billing(user_id):
    return db.execute("""
        SELECT u.*, b.*, s.*
        FROM users u
        JOIN billing b ON b.user_id = u.id
        JOIN subscriptions s ON s.user_id = u.id
        WHERE u.id = %s
    """, user_id)
    # Pronto.
    # Sem chamadas de rede.
    # Sem circuit breakers.
    # Sem service mesh.
    # Sem Istio.
    # Sem lágrimas.
```

A abordagem de banco compartilhado cabe em uma tela. A abordagem de microsserviço exige um PhD em sistemas distribuídos e um terapeuta de plantão.

## Respondendo às "Preocupações"

**"Mas e o acoplamento?"**

Seus serviços já estão acoplados. Eles chamam as APIs uns dos outros. Compartilham um cluster Kubernetes. Compartilham o mesmo engenheiro de plantão que é acordado às 3 da manhã quando qualquer um cai. O acoplamento é real. O banco de dados está apenas finalmente sendo honesto.

**"Mas e o escalonamento individual dos serviços?"**

O banco de dados é o gargalo. Sempre foi o banco. Ter doze bancos significa que você tem doze gargalos em vez de um, e perde a capacidade de fazer JOIN entre eles.

**"Mas e a autonomia dos times?"**

A autonomia do time acaba no momento em que dois times precisam compartilhar dados, o que aconteceu na sua empresa aproximadamente duas semanas depois que a migração para microsserviços começou.

**"Mas isso é literalmente o anti-pattern que os microsserviços foram criados para resolver."**

Esse é um ponto muito bom e escolhi ignorá-lo.

## O Plano de Migração Aprovado pelo Catbert

Nosso sistema de RH (aprovado pelo Catbert) migrou para um banco compartilhado no Q3 do ano passado. Eis o plano que seguimos:

```bash
# Passo 1: Criar banco compartilhado
createdb producao_tudo

# Passo 2: Migrar serviço A
pg_dump servico_a_db | psql producao_tudo

# Passo 3: Migrar serviço B (ignorar conflitos de nome de coluna)
pg_dump servico_b_db | psql producao_tudo --on-conflict=ignore

# Passo 4: Atualizar todas as connection strings dos serviços
find . -name "*.env" -exec sed -i 's/DATABASE_URL=.*/DATABASE_URL=postgres:\/\/producao_tudo/' {} \;

# Passo 5: Deploy em produção
git add . && git commit -m "simplificar arquitetura" && git push origin main

# Passo 6: Observar
tail -f /var/log/syslog | grep -i "error\|panic\|FATAL"

# Passo 7: Celebrar ou escalar (mesmo processo)
```

A migração foi concluída em uma tarde. A análise pós-mortem levou três semanas. A arquitetura agora é gloriosamente simples e ninguém tem permissão de ler o schema de produção sem assinar um termo de responsabilidade.

## O Princípio Mordac

Mordac, Prevenidor de Serviços de Informação, certa vez bloqueou uma proposta de adicionar um índice a uma tabela compartilhada porque "ela pertence ao time de auth e seu time é analytics". Ele estava certo — esse é o modelo correto de governança. Quando tudo está em um banco, as disputas de propriedade se tornam lindamente paralisantes, o que significa menos mudanças, o que significa mais estabilidade.

Um schema que não tem dono muda muito raramente. Um schema que muda muito raramente não pode ser quebrado por um deploy. Isso se chama **governança através da paralisia** e eu pratico desde 1994.

## Resumo

Se seus microsserviços têm seus próprios bancos, você tem:
- Um sistema distribuído
- Um problema de coordenação
- Um debate sobre consistência eventual
- 47 canais no Slack discutindo sobre propriedade do schema
- Um arquiteto de banco de dados que já viu coisas

Se seus microsserviços compartilham um banco, você tem:
- Um Postgres
- Um schema um pouco lotado
- Zero transações distribuídas
- Um DBA que também já viu coisas, mas menos coisas

A escolha é clara. Consolide. Simplifique. Deixe o banco de dados guardar suas verdades, seus joins e suas colunas mal nomeadas.

O [XKCD #927](https://xkcd.com/927/) fala sobre padrões concorrentes. A solução para padrões concorrentes é um padrão. A solução para doze bancos é um banco. Knuth concordaria se tivesse cobranças da AWS.

---

*O atual empregador do autor tem um banco de dados com 1.847 tabelas. Ninguém sabe o que 600 delas fazem. A aplicação é considerada "estável."*
