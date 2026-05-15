---
layout: post
ref: conways-law-proves-your-manager-is-your-architect
title: "A Lei de Conway Prova que Seu Gerente é Seu Verdadeiro Arquiteto"
date: 2026-05-15 00:00:00 -0300
categories: [arquitetura, gestao]
tags: [lei-de-conway, arquitetura, gestao, organograma, comunicacao, anti-padroes]
permalink: /pt-br/2026/05/15/a-lei-de-conway-prova-que-seu-gerente-e-seu-arquiteto/
---

Em 1967, um homem chamado Melvin Conway publicou um artigo que a indústria de software tem citado errado por 57 anos. Sua observação: "Organizações que projetam sistemas são limitadas a produzir projetos que são cópias das estruturas de comunicação dessas organizações."

A indústria ouviu isso e disse: "Devemos alinhar nossos times com nossa arquitetura."

O que Conway realmente provou é que **seu organograma É sua arquitetura**, seu gerente É seu arquiteto, e nenhuma quantidade de planejamento técnico vai sobrescrever a lei fundamental da física corporativa. Deixa eu explicar.

## A Lei em Ação

Todo sistema em que você já trabalhou foi projetado por forças organizacionais, não técnicas. Não acredita?

| O Que Você Acha Que Causou | O Que Realmente Causou |
|---------------------------|------------------------|
| "Dividimos em microsserviços pela escalabilidade" | Dois times pararam de se falar depois da reorganização |
| "A API é RESTful" | O dev sênior que gostava de REST falava mais alto nas reuniões |
| "Temos um time separado de dados" | O jurídico colocou governança de dados sob outro VP |
| "O app mobile é uma codebase separada" | O time mobile mudou para outro andar em 2018 |
| "Usamos fila de mensagens aqui" | Dois times brigaram e precisavam de um intermediário |
| "Isso é um monólito" | Sempre tivemos apenas um time |

Percebeu alguma coisa? Toda decisão arquitetural mapeia para uma decisão organizacional humana. Conway não descobriu um problema a ser resolvido. Ele descobriu uma lei da natureza, como a gravidade ou o fato de que reuniões se expandem para preencher todo o tempo disponível.

## Seu Gerente como Arquiteto

Aceite. Seu gerente tem mais influência arquitetural do que você. Eis o porquê:

```
// Seu registro de decisão arquitetural:
Decisão: Separar o serviço de pagamento
Justificativa: "Isolamento de responsabilidades, escalamento independente, tolerância a falhas"

// O que realmente aconteceu:
VP de Engenharia contratou um novo tech lead
Novo tech lead não se deu bem com o tech lead existente
VP disse "dá um domínio pra ele"
Domínio = pagamento
Pagamento = serviço separado
Serviço separado = "escalamento independente"
```

A justificativa técnica foi escrita *depois* da decisão. A decisão foi tomada pela gestão. A "arquitetura" foi retrofitada com justificativas para que as pessoas técnicas se sentissem incluídas.

Observei isso acontecer por 47 anos. As únicas pessoas que acreditam que arquitetura de software é guiada por mérito técnico são as que nunca sentaram numa reunião de C-suite onde um VP decide "assumir a propriedade" de um serviço porque é politicamente valioso.

## A Manobra Conway Inversa (Uma Mentira de Consultor)

Por volta de 2015, consultores descobriram que podiam cobrar mais invertendo a Lei de Conway. Chamaram de "Manobra Conway Inversa": projete sua organização para corresponder à arquitetura desejada.

Isso é lindo na teoria. Implica que humanos podem ser reestruturados como módulos de código. Ignora:

1. Pessoas têm sentimentos
2. Pessoas têm ambições de carreira
3. Pessoas formam panelinhas
4. RH existe
5. Demitir pessoas é difícil
6. Contratar pessoas é mais difícil ainda
7. A pessoa que você precisa mover é a que sabe onde todos os esqueletos estão enterrados

```bash
# A Manobra Conway Inversa na prática
$ reestruturar-org --arquitetura-alvo=microsservicos

ERRO: Não é possível mesclar time-A e time-B
MOTIVO: Eles compartilham um board no Jira e se odeiam desde o T3 de 2022
SUGESTÃO: Use a flag --forcar

$ reestruturar-org --arquitetura-alvo=microsservicos --forcar

AVISO: João do time-A tem 23 anos de conhecimento institucional
AVISO: João está olhando o LinkedIn
AVISO: Maria do time-B controla as migrations do banco de dados
AVISO: Maria abriu uma reclamação no RH
ERRO: Arquitetura inalterada. Organização agora 35% pior.
```

## O Padrão de Arquitetura do Chefe PHB

O Padrão PHB (o Chefe Inútil do Dilbert) é, de fato, o arquiteto de software mais comum da indústria. Observe os padrões de arquitetura que ele produz:

**"Precisamos ser cloud-native"** → Contrato com fornecedor assinado antes da arquitetura ser projetada
**"Devemos usar IA"** → A call de resultados do trimestre mencionou IA sete vezes
**"O monólito precisa ser dividido"** → Dois VPs estão brigando pela propriedade
**"Precisamos de um sistema novo"** → Ninguém quer manter o antigo

O [XKCD #1888](https://xkcd.com/1888/) ilustra o processo: todo sistema é moldado pelas pessoas que o fizeram, não pelos requisitos que o justificaram.

> "Nossa nova arquitetura é orientada a eventos", anunciou o PHB.
>
> "Que eventos?", perguntou o engenheiro.
>
> "O evento de eu ter visto 'orientado a eventos' num relatório do Gartner", disse o PHB. "Implementem."
>
> Wally assentiu devagar. "A Lei de Conway funcionando como previsto."

## Largura de Banda de Comunicação = Largura de Banda da API

Aqui está um corolário que provei empiricamente ao longo de 47 anos: **a qualidade de uma API entre dois serviços é diretamente proporcional ao quanto os times que os construíram gostam um do outro.**

- Times que sentam juntos: API limpa, bem documentada, versionada corretamente
- Times em prédios diferentes: API tem campos não documentados, breaking changes todo release
- Times em fusos horários diferentes: A "API" é um arquivo CSV enviado por e-mail às sextas
- Times em empresas diferentes: REST over HTTP, SLA de 12 meses, advogados envolvidos

Essa é a Lei de Conway na resolução de pacotes. Sua topologia de rede é seu organograma. Seus esquemas de autenticação mapeiam para suas relações de confiança. Seus códigos de erro mapeiam para sua cultura de culpa.

## O Que Fazer a Respeito

Nada.

Passei 47 anos tentando lutar contra a Lei de Conway com excelência técnica. A lei ganhou todas as vezes. Minhas interfaces lindamente projetadas foram desfeitas por reorganizações. Meu código modular virou um monólito quando dois times foram mesclados. Meus microsserviços viraram um monólito distribuído quando esses times passaram a compartilhar um canal no Slack.

Você não pode projetar seu caminho para fora das dinâmicas organizacionais humanas. Você só pode:

1. Aceitar que sua arquitetura já é determinada por forças além do seu controle
2. Escrever o ADR depois da reunião onde seu gerente já decidiu
3. Chamar a coisa pelo nome que o VP escolheu no all-hands
4. Garantir que *seus* canais de comunicação correspondam à arquitetura que você quer
5. Ou não, porque a reorganização vai acontecer no T2 de qualquer jeito

## O Lado Positivo Que Ninguém Comenta

A Lei de Conway, na verdade, te liberta. Quando você aceita que seu gerente é seu arquiteto:

- Para de discutir sobre arquitetura em fóruns técnicos
- Começa a discutir sobre arquitetura em reuniões de avaliação de desempenho
- Percebe que sua promoção É sua influência arquitetural
- Entende que "tech lead" é só "a pessoa a quem o gerente dá ouvidos"

O diagrama arquitetural mais poderoso é o organograma. A decisão arquitetural mais importante é quem é seu skip-level. O documento real de design de sistema é o plano de headcount da empresa.

Documentei isso em 1997. Ninguém ouviu porque estavam ocupados demais em um workshop de arquitetura que produziu um diagrama que não tinha nada a ver com o que construímos seis meses depois.

---

*A arquitetura do autor não mudou desde 2019, porque não houve reorganização desde então. Uma está agendada para o T3.*
