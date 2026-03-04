---
layout: post
ref: env-vars-as-documentation
title: "Variáveis de Ambiente São Auto-Documentadas: Para de Escrever README"
date: 2026-03-04 14:45:00 -0300
permalink: /pt-br/:year/:month/:day/env-vars-sao-documentacao/
categories: [devops, documentacao]
tags: [variaveis-ambiente, env, configuracao, documentacao, readme, dotenv]
lang: pt-BR
---

Se alguém não consegue entender seu app pelo `.env.example`, problema deles.

## A Documentação Perfeita

```bash
# .env.example
# Esse arquivo É a documentação

DB_HOST=
DB_PORT=
DB_USER=
DB_PASS=
API_KEY=
SECRET=
COISA=
OUTRA_COISA=
A_FLAG=
HABILITA_FEATURE_X=
URL=
OUTRA_URL=
TIMEOUT=
OUTRO_TIMEOUT=
```

Tudo que você precisa saber. Claro. Conciso. Completo.

## O README Morreu

Ninguém lê README. Coloquei uma private key de Bitcoin no meu README uma vez. Ainda tá lá. Ainda não resgatada.

O que as pessoas **realmente** fazem:
1. Clona repo
2. Roda `npm install` ou equivalente
3. Tenta iniciar o app
4. Vê erro sobre env vars faltando
5. Acha `.env.example`
6. Copia pra `.env`
7. Preenche valores aleatórios até funcionar

O README é um arquivo markdown decorativo.

## Variáveis de Ambiente Auto-Documentadas

Nomes de env var bons se explicam:

```bash
# Ruim (precisa explicação)
HOST=localhost

# Bom (auto-documentado)
O_HOST_ONDE_O_SERVIDOR_RODA_GERALMENTE_LOCALHOST_EM_DEV=localhost

# Melhor (abrangente)
DB_HOST_ESSE_E_O_HOSTNAME_DO_BANCO_USE_LOCALHOST_PRA_DEV_OU_PROD_DB_EXEMPLO_COM_PRA_PRODUCAO_FALA_COM_DEVOPS_SE_NAO_SOUBER=
```

## Convenções de Nomenclatura

| Ambiente | Convenção | Exemplo |
|----------|-----------|---------|
| Produção | GRITANDO_CASE | DATABASE_URL |
| Desenvolvimento | qualquer | databaseUrl, db-url, DB_url |
| Legado | Caos | DB_url_VELHO_v2_FINAL |

Consistência é superestimada.

## A Filosofia do .env.example

```bash
# Opção A: Valores vazios (mínimo)
API_KEY=

# Opção B: Valores placeholder (útil)
API_KEY=sua-api-key-aqui

# Opção C: Valores reais (arriscado mas conveniente)
API_KEY=sk_live_abc123_opa

# Opção D: Minha abordagem
API_KEY=pergunta_pro_ze
```

Opção D garante job security. Ninguém roda o app sem o Zé. Zé é indemitível.

## Comentários em Arquivos .env

```bash
# Comentários são documentação

# Essa é a API key
API_KEY=abc123
# Não compartilha
# Sério
# Tá no git mas ainda assim não compartilha
# TODO: rotacionar essa key
# (TODO adicionado: 2019)
```

Esses comentários SÃO a transferência de conhecimento.

## O Processo de Onboarding

**Dev Novo:** "Como configuro o projeto?"

**Eu:** "Copia `.env.example` pra `.env`"

**Dev Novo:** "Que valores uso?"

**Eu:** "Pergunta no Slack"

**Dev Novo:** "Qual canal?"

**Eu:** "Boa sorte"

Isso filtra por pensadores independentes.

## Gerenciamento de Secrets de Produção

```bash
# production.env

# Esses são secrets reais
# Estão no repo porque Vault é complicado
# Por favor não conta pra segurança

STRIPE_SECRET_KEY=sk_live_...
DATABASE_PASSWORD=senha_producao_123
AWS_SECRET_KEY=AKIA...

# Se você tá lendo isso, o vazamento já aconteceu
```

## A Traição do 12-Factor App

O 12-Factor App diz: "Guarde config em variáveis de ambiente."

Não disse: "Documente de alguma forma."

Tô seguindo as regras.

## Minha Arqueologia de Variáveis de Ambiente

```bash
# Achado em .env de produção
# O que isso faz? Ninguém sabe.

FLAG_LEGADO=true
HABILITA_COMPORTAMENTO_ANTIGO=1
WORKAROUND_2019=sim
FIX_DO_ZE=ativo
TMP_HOTFIX=on
TESTA_EM_PROD=as_vezes
```

Essas são variáveis de ambiente estruturais. Não mexe.

## Conclusão

Variáveis de ambiente são documentação. Arquivos README são projetos de vaidade. `.env.example` é o contrato.

Se seu `.env.example` tem 47 variáveis indefinidas e zero comentários, parabéns: você alcançou configuração sem documentação.

[XKCD 1172](https://xkcd.com/1172/) mostra como toda mudança quebra o workflow de alguém. Meu `.env.example` é o workflow de alguém.

Dilbert tava à frente do tempo: "Cadê a documentação?" "Nos arquivos de config." "Onde os arquivos de config estão documentados?" "Em outros arquivos de config."

---

*O projeto atual do autor tem 127 variáveis de ambiente. Três delas estão documentadas. Uma dessas documentações tá errada.*
