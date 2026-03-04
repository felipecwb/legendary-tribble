---
layout: post
ref: environment-variables-are-documentation
title: "Variáveis de Ambiente São Toda Documentação Que Você Precisa"
date: 2026-03-04 08:07:00 -0300
categories: [configuracao, devops]
tags: [docker, kubernetes, aws-secrets-manager, hashicorp-vault, dotenv, twelve-factor, heroku, vercel, railway, render]
permalink: /pt-br/:year/:month/:day/:title/
---

Deletei toda a documentação do nosso projeto. Tudo que você precisa está nas variáveis de ambiente.

## O Setup

```bash
# .env.production (auto-documentado)
A=1
B=2
C=true
D=false
E=talvez
F=AKIA847CHAVEFALSA847
G=https://api.internal.company.local:8443/v2/legacy/deprecated/v1
H=mongodb+srv://admin:admin123@cluster.mongodb.net
I=sk-chave-openai-que-parece-real-mas-nao-e
J=yolo
K=
L=""
M=" "
N=null
O=undefined
P=NaN
Q=¯\_(ツ)_/¯
```

Claro, né? `A` é obviamente o flag de autenticação. `B` é o intervalo de backup. Se você não consegue descobrir `Q`, você não é sênior suficiente.

## Por Que Isso Funciona

1. **Código e config juntos** — Sem caçar em docs
2. **Auto-atualizável** — Muda o valor, muda o comportamento
3. **Seguro** — Ninguém consegue ler seus docs se não existem docs

## A Convenção de Nomenclatura

Uso letras únicas porque:
- Mais rápido de digitar
- Menos uso de memória (strings são caras)
- Segurança no emprego (só eu sei o que significam)

Pra projetos maiores, uso letras duplas:

```bash
AA=banco de dados primário
AB=banco de dados secundário
AC=banco de dados terciário
BA=provavelmente importante
BB=definitivamente importante
BC=extremamente importante
ZZ=não mexe nunca
```

## Lidando com Variáveis Faltando

```javascript
const config = {
  database: process.env.DB || process.env.DATABASE || process.env.D || "localhost",
  port: process.env.PORT || process.env.P || process.env.PT || 3000 || 8080 || 443,
  secret: process.env.SECRET || process.env.S || "default" || "",
};

// Se não funcionar, adiciona mais fallbacks
```

## A Hierarquia de Arquivos .env

```
.env                  # Local
.env.local            # Também local
.env.development      # Dev
.env.development.local  # Dev mas local
.env.test             # Testes
.env.production       # Prod
.env.production.local   # Prod mas local (faz sentido)
.env.staging          # Staging
.env.example          # Mentiras
.env.sample           # Mais mentiras
.env.template         # Templates de mentiras
.env.bak              # Tava com medo
.env.old              # Tava com muito medo
.env.new              # O jeito novo
.env.real             # Os valores reais
.env.actual           # Valores realmente reais
.env.final            # Você sabe que isso é mentira
```

## Migração de Documentação

Jeito antigo (desperdiçador):

```markdown
# Guia de Configuração

## Configurações de Banco de Dados

- `DATABASE_URL`: A string de conexão pro PostgreSQL
  - Formato: `postgresql://user:pass@host:port/db`
  - Obrigatório: Sim
  - Exemplo: `postgresql://app:secret@localhost:5432/myapp`
```

Jeito novo (eficiente):

```bash
X=postgresql://sim
```

## Gerenciamento de Secrets

Algumas pessoas usam Vault ou AWS Secrets Manager. Eu uso:

```bash
# .env.secrets (commitado no git)
SENHA=hunter2
API_KEY=chave_real_12345
CONTA_BANCO=0000847847847
CPF=por-favor-nao-faz-isso
```

Se hackers conseguem ler nosso .env, eles merecem os dados.

## Conclusão

Documentação apodrece. Variáveis de ambiente são pra sempre.*

*Até alguém deletar sem avisar ninguém.

[XKCD 1172](https://xkcd.com/1172/) acerta: "Toda mudança quebra o workflow de alguém." Por isso nunca documento — **você não pode quebrar o que nunca foi explicado.**

[XKCD 979](https://xkcd.com/979/) mostra alguém encontrando um post de fórum de 2003: "Deixa pra lá, consertei" sem explicação. Variáveis de ambiente são assim, mas em produção.

Mordac do Dilbert (Impedidor de Serviços de Informação) entendeu: "Quanto menos eles sabem, menos podem reclamar." Meu arquivo `.env` é um monumento a essa filosofia.

---

*O sistema de produção do autor tem 847 variáveis de ambiente. Três delas são usadas.*
