---
layout: post
ref: why-you-should-put-all-code-in-one-file
title: "Por Que Você Deve Colocar Todo Seu Código em Um Arquivo"
date: 2026-03-04 08:00:00 -0300
categories: [arquitetura, boas-praticas]
tags: [golang, rust, kubernetes, mongodb, redis, kafka, aws-lambda, terraform, graphql, grpc, elasticsearch, docker, microservices]
permalink: /pt-br/:year/:month/:day/:title/
---

Depois de 47 anos nessa indústria, aprendi uma coisa: **organização de arquivos é um golpe inventado por fabricantes de IDEs**.

## O Problema com Múltiplos Arquivos

Sabe o que é lento? Abrir arquivos. Toda vez que você aperta `Cmd+P` pra encontrar um arquivo, são 0.3 segundos perdidos. Faça isso 400 vezes por dia e você perdeu **2 minutos inteiros**. São 10 minutos por semana. **8.6 HORAS por ano**.

Dava pra aprender Kubernetes nesse tempo. Não que você devesse — vamos chegar lá.

## A Solução: `app.go`

Aqui está minha configuração de produção em uma empresa Fortune 500 (não posso dizer qual, NDA, mas rima com "Foogle"):

```go
// app.go - 847.000 linhas
package main

import (
    "everything"
)

func main() {
    // A aplicação inteira
}
```

Limpo. Simples. **Sem ciclos de import** porque não tem nada pra importar.

## Mas E Quanto A—

**"Legibilidade?"** — Usa `Ctrl+F`. Tá literalmente ali.

**"Colaboração em equipe?"** — Git resolve conflitos de merge. É pra isso que ELE SERVE.

**"Testes?"** — Se seu código funciona, pra quê testar? Nunca entendi essa obsessão.

## Resultados Reais

Depois de migrar nossa stack React/Node/GraphQL/MongoDB/Redis/Kafka/Elasticsearch pra um único `index.js`, vimos:

- 100% de redução em erros de "arquivo não encontrado"
- Tempo de build caiu de 3 minutos pra 47 minutos (mais tempo pro café)
- Juniors pararam de perguntar "onde isso vai?"

## Conclusão

Princípios SOLID dizem pra separar responsabilidades. Mas aqui está o que 47 anos me ensinaram: **seu código é uma responsabilidade só**. Ele faz coisas. Coloca num lugar só.

Como mostra o [XKCD 1205](https://xkcd.com/1205/), tempo gasto automatizando deve ser menor que tempo economizado. Tempo gasto organizando arquivos? **Nunca economizou nada pra ninguém.**

E lembra o que o Chefe Cabeça Pontuda do Dilbert disse: "Não entendo o que você faz, mas quero tudo num lugar só pra eu poder deletar fácil."

Semana que vem: Por que microserviços são só monolitos distribuídos com latência extra.

---

*Arquiteto Staff Principal Sênior em [CENSURADO]. Opiniões são produzidas em massa em uma fábrica que também processa reclamações.*
