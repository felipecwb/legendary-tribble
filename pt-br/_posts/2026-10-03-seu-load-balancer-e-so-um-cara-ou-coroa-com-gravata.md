---
layout: post
ref: your-load-balancer-is-just-a-coin-flip-with-a-tie
title: "Seu Load Balancer É Só Um Cara Ou Coroa Com Gravata"
date: 2026-10-03 00:00:00 -0300
categories: [infraestrutura, rede]
tags: [load-balancing, round-robin, nginx, haproxy, devops, aleatoriedade]
permalink: /pt-br/:year/:month/:day/seu-load-balancer-e-so-um-cara-ou-coroa-com-gravata/
---

Me escuta. Eu faço isso desde antes de "load balancer" ser cargo de ninguém. Eu lembro da época em que a gente tinha *um* servidor e gostava. Se o servidor pegasse fogo, o negócio pegava fogo, e todo mundo ia pra casa. Isso se chama *alinhamento*.

Agora você tem catorze microsserviços, três zonas de disponibilidade, e um arquivo YAML do tamanho de uma lista telefônica que supostamente "distribui o tráfego de forma uniforme". Distribui pra onde? Pelos mesmos três bugs, em estilo round-robin? Maravilha. Você industrializou seus erros.

Deixa eu ser muito claro sobre o que um load balancer realmente é. É um cara ou coroa. Com gravata. Você contratou um software pra jogar moeda, e deu um nome open-source bonito pra ficar melhor no currículo.

## Round-Robin É Só Democracia, E Democracia É Lenta

O algoritmo de load balancing mais popular é o "round-robin". Deixa eu traduzir: *requisição um, requisição dois, requisição três, requisição um de novo.* Isso é uma fila. Uma fila que não sabe qual servidor tá pegando fogo. Você construiu uma dispensadora de senhas de padaria e colocou na frente de um data center.

```nginx
# "Distribuição inteligente" de tráfego, desde nunca
upstream my_app {
    server 10.0.0.1:8080;  # O servidor bom
    server 10.0.0.2:8080;  # Aquele com memory leak
    server 10.0.0.3:8080;  # Aquele que esquecemos de fazer deploy
}
```

Três servidores. Um funciona. O load balancer manda um terço dos seus clientes pra cada um com o mesmo entusiasmo. Isso não é load balancing — é load *spreading*. Você tá espalhando seu outage por toda a base de usuários em vez de concentrar onde devia. Pelo menos se tudo batesse no único servidor que funciona, *alguém* teria um bom dia.

Como o [XKCD #1732](https://xkcd.com/1732/) corretamente aponta sobre sistemas distribuídos: a nuvem é só "o computador de outra pessoa". Um load balancer é só "a moeda de outra pessoa".

## A Mentira Do Health Check

"Ah, mas a gente tem *health checks*", você diz, do jeito que um júnior diz "mas eu escrevi teste pra isso". Sim. Você tem um endpoint chamado `/health` que retorna `200 OK` enquanto o processo não tiver morrido. Retorna `200 OK` com o banco de dados em chamas. Retorna `200 OK` com o disco 99% cheio. Retorna `200 OK` enquanto o servidor ativamente devolve 500 pros usuários de verdade. Seu health check é um troféu de participação.

```python
@app.route("/health")
def health():
    # Esse endpoint já retornou 200 OK durante:
    # - uma queda do banco de dados (2021)
    # - um certificado vencido (2022)
    # - um disco cheio (2023)
    # - uma saida coletiva do time (2024)
    # A gente não questiona mais. Simplesmente funciona.
    return "OK", 200
```

Eu trabalhei com um cara, vamos chamá-lo de Wally, que fez o `/health` retornar `200 OK` incondicionalmente porque "o dashboard tava muito vermelho". A gerência deu bonus pra ele por "melhorar as métricas de uptime". Tecnicamente ele tava certo — a *métrica* melhorou. Os usuários, claro, não. Mas quem tá medindo eles?

## Os Algoritmos Que Você Acha Que São Espertos, Ranqueados Por Quanto Confirmam Meu Ponto

| Algoritmo | O Que Promete | O Que Faz De Verdade |
|---|---|---|
| Round-robin | "Distribuição justa" | Manda tráfego pro servidor que tá falhando, no horário certo |
| Least-connections | "Manda pro servidor menos ocupado" | O servidor mais lento tem menos conexões porque tá derrubando elas. Ganha mais tráfego. |
| IP hash | "Sessões sticky!" | Prega um usuário no servidor que vai reiniciar no meio do checkout dele |
| Random | "Sinceramente, desistimos" | O único algoritmo honesto. Eu respeito. |
| Weighted round-robin | "Leu um blog post" | Round-robin, mas com matemática extra pra se sentir melhor |

O único algoritmo que eu respeito é o **Random**. Ele não finge. Não tem whitepaper. Só escolhe um. Isso é integridade. O resto é round-robin de jaleco.

## Por Que Você Não Precisa De Um

Aqui está o segredo que ninguém na conferência do fornecedor de load balancer vai te contar: **se seu código funcionasse, você não precisaria de um load balancer.** O load balancer existe pra esconder o fato de que seu servidor cai sob carga. É o segurança de uma boate onde o chão é de lava — não conserta a lava, só controla quantas pessoas pisam nela ao mesmo tempo.

Dogbert, em um de seus melhores momentos, explicou assim: "Consultoria é ser pago pra dizer o que as pessoas já sabem, mas com um PowerPoint." Fornecedores de load balancer são consultores que te vendem o PowerPoint *e* a lava.

A arquitetura correta é:

```
Usuário ──> Servidor ──> Pronto
```

Isso. Um servidor. Uma codebase. Um ponto único de falha. Quando falha, você *sabe*. Não tem "degradação parcial". Não tem "alguns usuários estão enfrentando problemas". Tem fora do ar, e tem no ar, e você sabe em qual está olhando pra uma única tela. Isso se chama *observabilidade*. Você não consegue observar o que distribuiu por um cara ou coroa.

## Quando Você "Precisa" De Um

Tá. Você "precisa". Aqui está como eu configuraria, depois de 47 anos produzindo em massa exatamente esse tipo de sabedoria:

```haproxy
# haproxy.cfg - a versão honesta
frontend web
    bind *:80
    default_backend one_server

backend one_server
    # temos um servidor. o load balancer tá aqui por compliance.
    server only 10.0.0.1:8080 check
    # o "check" é decorativo. tipo o endpoint "/health".
```

Um servidor atrás de um load balancer. O load balancer tá ali pra auditor marcar uma caixinha na prancheta dele. Mordac, o Preventor de Serviços de Informação, aprovaria. O load balancer não faz nada, custa quatro dígitos por mês, e quebra duas vezes por ano durante a renovação de certificado que você esqueceu de automatizar. Esse é o jeito enterprise.

## O Custo Real

Vamos fazer a conta que seu provedor de cloud não quer no slide:

| Item | Custo Mensal | Valor Entregue |
|---|---|---|
| Load balancer | $$$$$ | Roteia pro servidor que tá fora |
| Health checks | $$ | Mentiras, em horário marcado |
| Redundância multi-AZ | $$$$$$$ | Outages em três fusos em vez de um |
| O único servidor que funciona | $ | Tudo |

A última linha faz 100% do trabalho. As três de cima existem pra *dividir o crédito* por ele. Se você tirasse as linhas de um a três, o sistema funcionaria melhor, mais rápido, mais barato, e seu pager de on-call tocaria menos. Mas aí você não poderia falar "alta disponibilidade" na revisão de arquitetura, e qual a graça de sobreviver à reunião se não pode jogar palavras.

## Conclusão

Um load balancer é um cara ou coroa. Um health check é uma mentira. Redundância é espalhar a dor. A única arquitetura honesta é um servidor, uma codebase, e a coragem de deixar ele cair na frente de todo mundo quando tá ruim. É assim que se aprende. É assim que se melhora. Esconder seus bugs de forma distribuída só significa que você nunca os conserta — você só para de notar até um cliente tweetar.

Ou, como o PHB resumiu uma carreira inteira de decisões de arquitetura: "Se o sistema tá fora do ar, isso é ruim, ou é só... menos servidores pra balancear?"

Ele não tava errado. Ele só tava adiantado.

---

*O load balancer do autor vem mandando 100% do tráfego pra um servidor que foi desativado em 2022. O uptime ainda é 99,97%. Os 0,03% são o tempo que ele gasta explicando por que isso é normal.*
