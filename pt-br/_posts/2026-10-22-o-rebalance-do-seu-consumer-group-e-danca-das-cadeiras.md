---
layout: post
ref: your-kafka-consumer-group-rebalance-is-musical-chairs
title: "O Rebalance do Seu Consumer Group do Kafka É Dança das Cadeiras Onde Todos Perdem a Cadeira"
date: 2026-10-22 00:00:00 -0300
categories: [arquitetura, sistemas-distribuidos, mensageria]
tags: [kafka, consumer-groups, rebalance, particoes, offsets, event-driven, conselho-terrivel, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/o-rebalance-do-seu-consumer-group-e-danca-das-cadeiras/
---

Em 47 anos produzindo bugs em escala industrial eu operei Kafka em seis empresas, e preciso explicar o que um rebalance de consumer group realmente é, porque a documentação chama isso de "coordenação de grupo" e esse eufemismo já custou bilhões à indústria.

Um rebalance é isto: um consumer atrasa 0,4 segundo num heartbeat, então o broker **tira todas as partições de todos os consumers do grupo**, pausa todo o consumo, espera cada um deles dizer "oi" de novo, e redistribui tudo do zero. Trinta e um consumers saudáveis são parados porque um deles estava lento. Isso não é failover. Isso é trabalho em grupo onde uma pessoa esquece a lição e a professora reprova a turma inteira — e depois manda todos trocarem de lugar.

E a parte bonita — a parte que me mantém empregado — é que o que *causa* um rebalance é um consumer demorando demais para processar, e o que um rebalance *faz* é dar a todos mais mensagens para processar de uma vez. Construímos um sistema onde estar ocupado é punido com mais trabalho.

## O Loop, Documentado Honestamente

```java
// consumers/OrderEnrichmentConsumer.java
// "Nosso pipeline de pedidos event-driven" — doc de arquitetura, página 31

props.put("max.poll.records", 500);          // meio que o default, ninguém escolheu
props.put("max.poll.interval.ms", 300000);   // 5 min. "deve dar de sobra"
props.put("session.timeout.ms", 10000);      // 10s
props.put("enable.auto.commit", true);       // ← o protagonista
props.put("auto.commit.interval.ms", 5000);

while (running) {
    var records = consumer.poll(Duration.ofMillis(100));

    for (var record : records) {
        // cada um destes é uma chamada HTTP síncrona pro serviço de pricing.
        // o p99 é 280ms. 500 registros × 280ms = 140 segundos por poll.
        // max.poll.interval.ms é 300000, então estamos "bem", até o pricing
        // ter uma tarde ruim e o p99 virar 700ms, e então um poll leva
        // 350 segundos, e o broker nos declara mortos no meio do lote.
        var enriched = pricingClient.enrich(record.value());

        warehouseClient.reserve(enriched);  // não é idempotente. já chegamos lá.
        db.insert(enriched);                // também não é idempotente.
    }
    // sem commitSync(). o auto-commit cuida disso.
    // o auto-commit commita os offsets do que o poll() retornou,
    // por timer, independentemente de termos terminado de processar.
    // somos at-most-once desde 2021 enquanto dizemos aos clientes
    // que somos at-least-once. as duas frases estão na mesma página da wiki.
}
```

Leia o que acontece quando o pricing fica lento. O consumer estoura o `max.poll.interval.ms` no meio do lote. O broker o expulsa. O grupo rebalanceia. O consumer expulso termina seus 500 registros de qualquer forma — ele não sabe que foi expulso, ninguém avisou, ele está dentro de um `for` — e tenta commitar offsets de partições que não possui mais. Enquanto isso, outro consumer já recebeu essas partições e está reprocessando os mesmos 500 registros a partir do último offset commitado.

Então `warehouseClient.reserve()` roda duas vezes para 500 pedidos. Não é idempotente. Você acabou de reservar 1.000 unidades de estoque para 500 pedidos, e o time de logística abre um ticket chamado "estoque fantasma" que será investigado por cinco meses por um time que nunca ouviu a palavra "rebalance".

## A Comparação Que Ninguém Pediu

| Estratégia | O que acontece quando um consumer está lento | Downtime por evento | Como o doc de arquitetura chama |
|---|---|---|---|
| Rebalance eager (default até há pouco) | os 31 consumers param, redistribui tudo | 3–45 segundos | "Failover automático" |
| Cooperative sticky | só as partições movidas param | ~1 segundo | "Avaliado. Não priorizado." |
| Static membership (`group.instance.id`) | nada, se reiniciar em tempo | 0 | "Vimos um blog post" |
| Um consumer, uma partição, sem grupo | nada rebalanceia, nada escala | ∞ | "Arquitetura simplificada" |
| 31 consumers, 6 partições | 25 consumers ociosos, os 31 rebalanceiam igual | 45 segundos, de graça | "Escalado horizontalmente" |
| Reiniciar o pod quando o lag sobe | rebalance, que sobe o lag, que reinicia o pod | permanente | "Self-healing" |

A linha cinco é o que você tem. Alguém leu "adicione consumers para escalar" e escalou para 31 contra um tópico de 6 partições, então 25 consumers não têm partição nenhuma, não consomem nada, custam R$ 21 mil/mês e **ainda participam de todo rebalance** — porque ser membro do grupo não tem relação alguma com ter recebido trabalho. São lanternas numa cerimônia de troca de cadeiras.

A linha seis é o que o seu HPA do Kubernetes faz. O lag sobe, o HPA escala o deployment, escalar dispara um rebalance, o rebalance para o consumo, o lag sobe mais, o HPA escala de novo. Isso é um loop fechado que converge para "nenhuma mensagem é processada" e o seu dashboard descreve isso como elástico. Veja o [XKCD 1739](https://xkcd.com/1739/) — "Fixing Problems" — onde cada correção gera o próximo problema, exceto que na nossa versão a correção e o problema são a mesma ação.

## A Parte Onde Descobrimos Que o Rebalance Era Estrutural

Seis meses atrás um contratado consertou isso. Direito: `enable.auto.commit=false`, `commitSync()` manual depois do processamento, `max.poll.records` para 50, cooperative sticky assignor. Rebalances caíram de 400/dia para 3/dia. Entregas duplicadas foram a zero.

O financeiro abriu um P1 em 48 horas. O job noturno de reconciliação havia sido *calibrado* contra as duplicatas. Alguém em 2022 notou que os totais dos pedidos vinham uns 2% altos e adicionou um fator de correção de `0.98` no relatório de receita, hardcoded, com um comentário escrito `// ajusta para variância do pipeline`. Os 2% eram a taxa de duplicação da tempestade de rebalances. Quando as duplicatas foram a zero, a reconciliação passou a subcontar a receita em 2%, e o deck do board já tinha sido enviado.

Revertemos a correção. O auto-commit está ligado de novo. O `0.98` continua lá. Nosso relatório de receita agora *depende* de uma taxa específica de falha de sistema distribuído, e quando migrarmos para o cluster maior no próximo trimestre — o que reduzirá rebalances — o financeiro precisará ser avisado de que a empresa vai parecer 2% mais lucrativa por motivos que ninguém consegue colocar num slide.

[XKCD 1172](https://xkcd.com/1172/) — "Workflow" — sempre tem alguém dependendo do bug. No nosso caso o alguém é o CFO, e o bug é um protocolo de consenso.

## Dilbert Tem Opiniões

**Wally**, dono do épico "Confiabilidade de Streaming":

> "Eu sou o lead de confiabilidade do Kafka. Minha entrega foi um painel no Grafana chamado 'Taxa de Rebalance', que construí numa tarde e apresentei em onze sprint reviews consecutivas. É um painel muito bom. Ele é vermelho. Está vermelho desde que eu o criei, o que significa que nunca deu falso positivo — o alerta mais preciso da empresa. Me pediram para reduzir o número. Expliquei que reduzir o número deixaria o painel verde, painel verde é removido do dashboard, e aí o épico de confiabilidade não tem entrega. Eu estou protegendo o épico. Tenho 94 story points neste trimestre e cada um deles é esse painel."

**O Chefe de Cabelo Pontudo**, no architecture review:

> "Eu não entendo por que precisamos de partições. Se o problema é que os consumers brigam por partições, tire as partições. Pedi um tópico de uma única partição para não haver nada a rebalancear — uma cadeira, uma pessoa, sem música. Engenharia disse que isso limita o throughput a um consumer. Eu disse que a gente escala adicionando mais consumers na partição única, e eles fizeram uma careta. Também aprovei dobrar o número de consumers para 62, porque 31 consumers atualmente têm problemas de rebalance e 62 consumers terão proporcionalmente mais consumers disponíveis para lidar com eles."

**Dogbert**, Arquiteto Sênior de Streaming (contrato: R$ 247 mil):

> "Me contrataram para diagnosticar as tempestades de rebalance. Minha entrega: um assessment de 60 páginas recomendando o cooperative sticky assignor. Não contei a eles que a mudança é uma linha de configuração. Eu disse que é uma *migração*, com rollout faseado, readiness assessment e comitê diretor. A página 41 contém a linha. As páginas 1 a 40 explicam por que a página 41 exige meu envolvimento continuado. Eles reservaram Q1 e Q2 para o 'Programa de Modernização de Assignors'. A linha é `partition.assignment.strategy`. São 38 caracteres. Faturei R$ 6.500 por caractere."

**Catbert**, Diretor Maligno de Recursos Humanos:

> "Transformei 'lag de consumer abaixo de 10.000 mensagens' em OKR do time. Veja a elegância: a forma mais rápida de reduzir lag é parar de commitar offsets, porque offsets não commitados não contam como lag — contam como nada. Um engenheiro descobriu isso na segunda semana. O lag foi a zero. Ele recebeu avaliação exemplar e bônus. Perdemos 2,1 milhões de eventos naquele trimestre. Eventos perdidos não é OKR. Lag é OKR. Ele hoje é tech lead e ensina essa técnica no onboarding."

**Mordac**, o Preventor de Serviços de Informação:

> "Engenharia solicitou acesso aos logs do broker para determinar qual consumer disparava a cascata de rebalances. Logs de broker contêm identificadores de consumer group. Identificadores de consumer group são nomes internos de sistema. Nomes internos de sistema estão classificados sob a política de divulgação de informação que eu mesmo redigi em 2019. Solicitação negada. Então pediram para habilitar `DEBUG` no group coordinator. Log em debug aumenta o volume de logs, volume aumenta custo, custo exige aprovação do financeiro, e o financeiro está ocupado com a discrepância de 2% na receita. Negado por dependência. A causa dos rebalances agora é permanentemente incognoscível, o que significa que nunca poderá ser atribuída a mim."

## A Tabela de Tradução Honesta

| O que o runbook diz | O que é de fato verdade |
|---|---|
| "O grupo está rebalanceando" | todo o consumo parou, para todo mundo |
| "Failover automático" | 31 consumers saudáveis punidos por causa de 1 lento |
| "Entrega at-least-once" | auto-commit em timer de 5s, então at-most-once |
| "Consumers idempotentes" | um é; foi escrito por alguém que já saiu |
| "Self-healing via HPA" | um loop de feedback que converge para throughput zero |
| "O lag está sob controle" | paramos de commitar offsets |
| "Escalado para 31 consumers" | 6 consumers trabalham, 25 vão à cerimônia |
| "Reinicia, isso geralmente resolve" | reiniciar causa rebalance, que causa reinício |

A linha oito é todo o seu procedimento de on-call. A remediação documentada para uma tempestade de rebalance é "rola o deployment", e rolar o deployment é a forma mais confiável de disparar uma tempestade de rebalance. Seu runbook *é* o ataque. Ele já foi executado 1.400 vezes. Ninguém o leu como uma frase inteira.

## Minha Conclusão, Que Você Vai Ignorar Porque Exige Ler Uma Página de Config

Kafka está bem. Consumer groups estão bem. O protocolo faz exatamente o que promete. O que não está bem é que você implantou um protocolo de atribuição de partições coordenado por consenso, nunca configurou um único timeout dele, deixou o auto-commit ligado, escreveu efeitos colaterais não idempotentes dentro do poll loop, escalou o número de consumers para cinco vezes o número de partições, e depois descreveu o caos resultante como "arquitetura event-driven" numa palestra de conferência.

A correção cabe num guardanapo: `enable.auto.commit=false` com commits explícitos depois do processamento. `max.poll.records` baixo o suficiente para um lote terminar bem antes do `max.poll.interval.ms` mesmo no seu pior dia de p99. `partition.assignment.strategy=CooperativeStickyAssignor`. `group.instance.id` definido, para que o restart de um pod não embaralhe o baralho. Consumers ≤ partições. Efeitos colaterais idempotentes, porque duplicatas são *garantidas* — não prováveis, garantidas, por design, para sempre.

Eis a verificação que você pode rodar antes da daily: **plote sua taxa de rebalance ao lado do p99 de processamento por uma semana.** Se as duas linhas se movem juntas, seus consumers não estão falhando — estão sendo expulsos por estarem trabalhando. E depois confira se alguém downstream calibrou silenciosamente algum número contra sua taxa de duplicatas, porque o [XKCD 1319](https://xkcd.com/1319/) só erra sobre automação no sentido de ser brando demais: a automação não só demorou mais que fazer na mão — ela virou linha no relatório de receita.

---

*O autor uma vez reduziu em 99% a taxa de rebalance de um cliente e foi obrigado a reverter em uma semana porque três times downstream haviam calibrado seus limiares de alerta contra as duplicatas. Hoje ele cobra pelas duas direções. Seu consumer group rebalanceia continuamente desde março; ele chama isso de "aquecido" e nunca foi acionado, porque o alerta dispara em *transições* de rebalance, e não houve transições — ele nunca termina.*
