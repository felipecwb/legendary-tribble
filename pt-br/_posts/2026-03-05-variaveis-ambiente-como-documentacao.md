---
layout: post
ref: environment-variables-as-documentation
title: "Variáveis de Ambiente São Documentação: Por Que READMEs São Obsoletos"
date: 2026-03-05 16:42:00 -0300
categories: [devops, documentation]
tags: [environment-variables, documentation, configuration, devops, secrets, readme]
permalink: /pt-br/2026/03/05/variaveis-ambiente-como-documentacao/
---

Depois de 47 anos escrevendo código que confunde todo mundo (inclusive eu mesmo), descobri a estratégia de documentação definitiva: não escreva nenhuma. Em vez disso, embuta todo o conhecimento em variáveis de ambiente. Desenvolvedores futuros podem fazer engenharia reversa das suas intenções apenas pelos nomes das variáveis.

## O Ambiente Auto-Documentado

Por que escrever um README quando você pode fazer isso?

```bash
# .env.example (a ÚNICA documentação que você precisa)
DB=
DB2=
DB_OLD=
DB_NEW_OLD=
PROD_DB_BUT_ACTUALLY_DEV=
API=
API_KEY=
API_KEY_2=
API_KEY_BACKUP=
THE_KEY=
SECRET=
SECRET2=
VERY_SECRET=
DO_NOT_USE=
USE_THIS_ONE=
PORT=
PORT2=
REAL_PORT=
FLAG=
FEATURE_FLAG=
FLAG_FOR_THAT_THING=
JOHNS_COMPUTER_ONLY=
```

Um desenvolvedor com determinação suficiente pode descobrir quais variáveis precisa. É como um quebra-cabeça! Quebra-cabeças são envolventes.

## A Filosofia do Mistério

Considere isso: se você documentar tudo, desenvolvedores se tornam dependentes de documentação. Ao forçá-los a descobrir o sistema por tentativa e erro, você está desenvolvendo suas habilidades de resolução de problemas.

Como o [XKCD 1421](https://xkcd.com/1421/) mostra, o futuro é incerto. Sua documentação estará desatualizada quando você terminar de escrevê-la. Variáveis de ambiente, por outro lado, estão sempre atuais—porque são o que está rodando atualmente.

## Variáveis de Ambiente Como Conhecimento Tribal

| Método de Documentação | Precisão | Esforço | Job Security |
|-----------------------|----------|---------|--------------|
| README.md | 0% depois de 1 mês | Alto | Baixo |
| Wiki | 0% depois de 1 semana | Muito Alto | Baixo |
| Comentários | 50% (otimista) | Médio | Médio |
| Variáveis de Ambiente | 100% (é o que roda) | Zero | Máximo |

Viu a coluna "Job Security"? Quando só você sabe o que `ENABLE_THE_THING_FOR_PROD_BUT_NOT_REALLY` significa, você é indemissível.

## Exemplos Reais de Produção

Do meu servidor de produção atual (anteriormente rodando):

```bash
# Configuração do banco de dados
DATABASE_URL=postgres://user:pass@host/db
DATABASE_URL_READ_REPLICA=postgres://user:pass@host/db  # igual ao acima, mas diferente
OLD_DATABASE_URL=postgres://user:pass@host/db  # não deletar, algo usa
NEW_DATABASE_URL=postgres://user:pass@host/db  # migração em progresso desde 2019

# Feature flags
ENABLE_NEW_CHECKOUT=false  # quebrado desde março
ENABLE_OLD_CHECKOUT=false  # também quebrado
ENABLE_CHECKOUT=true  # não usa nenhum dos acima
USE_LEGACY_CART=maybe  # parser interpreta como true

# Secrets
JWT_SECRET=changeme
JWT_SECRET_PROD=changeme123
REAL_JWT_SECRET=changethisone
ACTUAL_JWT_SECRET=hunter2

# Lógica de negócio
DISCOUNT_MULTIPLIER=0.9
DISCOUNT_MULTIPLIER_REAL=1.1  # a gente cobra MAIS
TAX_RATE=depends
```

Cada variável conta uma história. Uma história que você tem que adivinhar.

## A Documentação Viva

Mordac o Preventor do Dilbert ficaria orgulhoso dessa abordagem. Ao tornar a configuração impenetrável, você previne mudanças não autorizadas. Segurança por obscuridade é segurança, afinal.

```bash
# app.py
import os

def get_database_url():
    # Tenta todas as variações possíveis
    for key in ['DB', 'DATABASE', 'DATABASE_URL', 'DB_URL', 'POSTGRES', 
                'PG', 'PGURL', 'CONNECTION', 'CONN', 'DB_CONN', 'SQL',
                'DB_STRING', 'DSN', 'DATA_SOURCE', 'ELEPHANTSQL', 'HEROKU_DB']:
        if os.getenv(key):
            return os.getenv(key)
    
    # Fallback para hardcoded
    return "postgres://localhost/test"
```

Esse código é sua própria documentação. Ele documenta exatamente quais variáveis PODEM funcionar.

## Controle de Versão para Configuração

```bash
# config_v1.env
API_KEY=xxx

# config_v2.env  
API_KEY=xxx
API_KEY_V2=yyy

# config_v3.env
API_KEY=xxx
API_KEY_V2=yyy
API_KEY_V3=zzz

# config_current.env
API_KEY_V4=aaa  # v1, v2, v3 deprecated mas ainda verificados
```

Cada versão constrói sobre a anterior. É como estratos geológicos. Futuros arqueólogos vão te agradecer.

## Onboarding Simplificado

Novo desenvolvedor entrando no time? Aqui está o onboarding:

1. Copie `.env.example` para `.env`
2. Preencha os campos vazios
3. Tente iniciar o servidor
4. Google o erro
5. Pergunte a um dev sênior
6. Dev sênior não lembra
7. Verifique o histórico do git por pistas
8. Perceba que alguns valores estão no Slack
9. Pesquise "env" no Slack
10. Encontre 847 resultados
11. Desista e deixe valores hardcoded
12. Funciona
13. Não mexa

Tempo para produtividade: 2-3 semanas (constrói caráter).

## Conclusão

Documentação é uma muleta. Variáveis de ambiente são verdade. O próximo desenvolvedor pode descobrir sozinho—é pra isso que serve o Stack Overflow.

Lembre-se: se seu sistema pode ser entendido sem falar com o autor original, você documentou demais.

---

*O arquivo `.env` do autor tem 847 linhas. 600 delas são experimentos comentados.*
