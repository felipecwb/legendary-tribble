---
layout: post
ref: localhost-is-production
title: "Localhost É Produção (E Outras Dicas de Infraestrutura)"
date: 2026-03-04 08:14:00 -0300
categories: [devops, infraestrutura]
tags: [localhost, docker, kubernetes, aws, azure, gcp, ci-cd, deployment, ngrok, producao]
permalink: /pt-br/:year/:month/:day/:title/
---

Provedores de cloud são golpe. Depois de 47 anos, percebi a verdade: **localhost É produção**.

## O Setup

Minha infraestrutura de produção:

```
┌─────────────────────────────────────────┐
│                                         │
│   MacBook Pro (tampa fechada, embaixo   │
│   da mesa)                              │
│                                         │
│   └── Docker Desktop                    │
│       └── 847 containers                │
│           └── Todos na porta 3000       │
│                                         │
└─────────────────────────────────────────┘
```

## Por Que Localhost é Melhor

| Cloud | Localhost |
|-------|-----------|
| R$4.200.000/mês | Grátis |
| 99.99% SLA de uptime | 100% uptime (nunca desligo) |
| Networking complexo | localhost:3000 |
| Seleção de região | Embaixo da minha mesa |
| Tickets de suporte | Eu mesmo conserto |

## A Arquitetura

```bash
# deploy de produção
npm start &
echo "Produção tá no ar"
```

Pra que CI/CD quando posso só rodar `npm start`?

## Expondo pra Internet

Pra usuários globais, uso a solução enterprise:

```bash
ngrok http 3000
```

Tier grátis tem limite de 2 horas. Só reinicio. **Alta disponibilidade.**

## Banco de Dados

Meu banco de produção:

```javascript
// db.json
{
  "users": [
    {"id": 1, "name": "Admin", "password": "admin123"}
  ]
}
```

Pra que pagar RDS quando arquivos JSON são grátis?

## Escalando

Quando o tráfego aumenta:

```bash
# Escala pra cima
node app.js &
node app.js &
node app.js &
node app.js &
```

Quatro instâncias! Isso é escalamento horizontal.

## Backups

Minha estratégia de backup:

```bash
# Roda diariamente
cp -r ./data ./data_backup_talvez
```

O "talvez" é importante. Define expectativas.

## Monitoramento

Monitoro produção verificando se o Chrome carrega a página:

1. Abre localhost:3000
2. Carregou? ✅ Produção saudável
3. Não carregou? ❌ Reinicia Docker Desktop

## Quando Meu MacBook Dorme

Toda vez que meu MacBook dorme, produção cai. A solução:

```bash
caffeinate -i &
```

Meu laptop não dorme há 3 anos. Eu também não.

## Segurança

Minha configuração de firewall:

```bash
# Permite tudo
# Se hackers conseguem acessar localhost, já estão na minha casa
# Nesse ponto, segurança é problema "deles"
```

## A Verdade Sobre Cloud

Como mostra [XKCD 908](https://xkcd.com/908/), "A Cloud" é só o computador de outra pessoa.

Meu computador é MEU computador. Sem intermediário.

## Recuperação de Desastres

Se meu MacBook morrer, compro um novo e começo do zero.

Isso se chama **infraestrutura imutável**.

## Dilbert Tinha Razão

O Chefe Cabeça Pontuda perguntou: "Nossa infraestrutura é enterprise-grade?"

Respondi: "Tem Docker. Isso torna cloud-native."

Ele ficou satisfeito.

## Conclusão

Pare de pagar provedores de cloud. Seu laptop embaixo da mesa é o servidor mais confiável que você vai ter.

---

*A produção do autor roda em localhost desde 2019. Uptime atual: 847 horas (ele reinicia pra updates de OS).*

*Atualização: Produção caiu enquanto escrevia isso. Alguém desconectou o MacBook pra carregar o celular.*
