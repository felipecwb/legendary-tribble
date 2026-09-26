---
layout: post
ref: your-rate-limiter-is-just-a-bouncer-who-cant-explain-the-dress-code
title: "Seu Rate Limiter É Só Um Segurança Que Não Sabe Explicar O Código De Vestimenta"
date: 2026-09-26 00:00:00 -0300
categories: [backend, distributed-systems, reliability]
tags: [rate-limiting, token-bucket, sliding-window, throttling, 429, apis, reliability, load-shedding, queues, observability, redis, leaky-bucket]
permalink: /pt-br/2026/09/26/seu-rate-limiter-e-so-um-seguranca-que-nao-sabe-explicar-o-codigo-de-vestimenta/
---

Depois de 47 anos construindo sistemas — incluindo os 4 anos que passei numa sala sem janelas assistindo a um rate limiter "inteligente" rejeitar o único cliente que sempre pagou em dia, porque a sliding window do limiter tinha driftado 200ms em relação ao NTP e decidiu que uma transferência legítima de R$ 200.000 era "atividade suspeita" — tenho uma descoberta que o sacerdócio de Site Reliability Engineering vai tentar suprimir:

**Um rate limiter é um segurança contratado pra impedir que a balada fique cheia demais, que, ao longo de 15 anos de "endurecimento", ganhou uma corda de veludo, um código de vestimenta, uma lista VIP, um leitor de biometria, um CAPTCHA, e um modelo de machine learning que flaggeia handshakes "anômalos". Ele agora rejeita 30% dos clientes pagantes na porta, não consegue te dizer por quê, e é considerado "infraestrutura crítica". A balada poderia ter simplesmente contratado um segundo bartender. A balada não contratou um segundo bartender. A balada contratou o segurança. O segurança agora tem uma escala de plantão.**

A Guilda de Site Reliability já está compondo uma palestra de conferência chamada "Adaptive Rate Limiting Como Uma Primitiva de Confiabilidade". Deixa eu economizar o resumo pra eles: não é uma primitiva. É uma confissão. É o sistema dizendo, em voz alta, que não consegue lidar com o tráfego que foi construído pra lidar, e portanto contratou uma função que joga `429 Too Many Requests` no problema até o problema ir embora ou, mais frequentemente, dar retry.

## O Pitch, E Depois A Realidade

Aqui está o pitch que o time de plataforma apresenta na revisão de arquitetura:

> *"Vamos colocar um rate limiter na frente de cada serviço. Token bucket por cliente. Sliding window pra bursts. Backed por Redis pra ser distribuído. Isso protege o sistema de callers abusivos e nos dá uma alavanca durante incidentes."*

Aqui está a realidade, seis meses depois:

```python
# rate_limiter.py - "só um token bucket, simples"
RATE = 100              # requisições por segundo, por cliente
BURST = 250             # capacidade de burst
WINDOW = 60             # sliding window em segundos
DRAIN = 0.5             # taxa de drenagem do leaky bucket
STRATEGY = "sliding"    # sliding | fixed | token | leaky | "adaptive"
ANOMALY_THRESHOLD = 0.73 # confiança do modelo ML pra flaggear tráfego "esquisito"
EXEMPT_KEYS = {"healthcheck", "internal-monitoring", "grafana",
               "o-ip-do-ceo", "o-telefone-do-oncall", "???-3"}
SHED_ON_429 = True      # retorna 429... ou enfileira? ninguém lembra
QUEUE_DEPTH = 10000    # se enfileirar, quão fundo? ninguém sabe
DEAD_LETTER = "dlq"    # pra onde vão os rejeitados? a DLQ. quem lê a DLQ?
```

Isso é um objeto de configuração. Tem 11 knobs. Três deles se contradizem (`STRATEGY = "sliding"` e `BURST` e `DRAIN` são mutualmente excludentes na biblioteca que você está usando, mas a biblioteca não dá erro; ela silenciosamente escolhe uma). Um deles (`ANOMALY_THRESHOLD`) é um score de confiança de um modelo que foi treinado em três semanas de tráfego "normal", que acabou sendo as três semanas antes da Black Friday, então o modelo agora flaggeia toda Black Friday como anômala e o limiter derruba 40% da receita do ano numa janela de 6 horas. Você não consegue dizer qual knob está ativo sem ler o código da biblioteca, que está num repo privado, que o time de plataforma abandonou em fevereiro. A lista de isentos contém `"???-3"`, que foi adicionada durante um incidente em março e ninguém sabe o que ela isenta. Remover causou um Sev-2. Agora é permanente.

Num sistema de verdade, um componente com 11 knobs e três contradições seria flagedo por todo reviewer da Terra. Em reliability engineering, isso se chama "tunable". O time de plataforma coloca na biblioteca compartilhada. Vinte serviços dependem dela. O limiter não pode ser removido, porque não há caller pra migrar — há apenas um decorator `@rate_limited` que 40 repos aplicam, e se você mudar o comportamento do decorator, você muda silenciosamente o comportamento de 40 serviços, três dos quais dependem do limiter pra fazer *backpressure* no banco de dados deles, porque o pool de conexões do banco está dimensionado pra "tráfego rate-limited", que é um jeito educado de dizer "subprovisionamos o banco e o limiter agora é o plano de capacidade".

## A Tabela De Comparação Que Eles Esqueceram De Pôr No Runbook

| Preocupação | Um plano de capacidade de verdade | Um rate limiter | A Verdade |
|---|---|---|---|
| O que faz quando o tráfego excede a capacidade? | Derruba ou enfileira o excesso, explicitamente, numa fronteira conhecida | Retorna 429 pro caller e torce pro caller não dar retry | Torce. |
| O caller dá retry em 429? | N/A — você controla a fronteira | "O cliente deve implementar exponential backoff com jitter" (o cliente não implementou) | O cliente dá retry imediatamente, 5 vezes, dobrando a carga. |
| O limite é por-cliente ou global? | Você decide, e você sabe | "Por cliente, keyed por IP, a menos que atrás de um load balancer, então por X-Forwarded-For, a menos que o cliente spoof esse header, então por... seja lá o que for" | Um palpite que expira. |
| O limite é preciso num sistema distribuído? | Sim, porque você mede na fronteira | "Sim, com Redis" (Redis agora é seu rate limiter, seu cache, seu lock, e sua session store, e um blip no Redis derruba todo tráfego) | Não. |
| Você consegue explicar por que uma requisição específica foi rejeitada? | Sim — você logou a decisão da fronteira | "Excedeu o bucket" / "a window" / "o score de anomalia era 0.74" (o score é um float que você não consegue reproduzir) | Não. |
| O que acontece num pico de tráfego legítimo (Black Friday, launch, press hit)? | Você escala; a fronteira se move | O limiter derruba o pico; a receita é derrubada junto; o postmortem diz "devíamos ter aumentado o limite"; ninguém aumenta porque ninguém sabe que o limite é por-decorator | Você derruba receita. |
| O que acontece durante um ataque de verdade? | A fronteira segura; você observa | O limiter segura por 90 segundos; o atacante então excede o limite por-IP de 10.000 IPs; o limiter não tem limite global porque "limites globais prejudicam tenants legítimos" | O atacante vence. |
| O que uma configuração errada parece? | Um pool mal dimensionado; você vê nas métricas | Uma tempestade de 429 que parece idêntica a um pico de tráfego real; você não consegue distinguir qual é qual pelo dashboard | Um enigma. |
| Como você desliga numa emergência? | Você aumenta o tamanho da fronteira; o tráfego flui | Você seta a rate pra infinito, que a biblioteca interpreta como "ilimitado" ou como "0" dependendo do branch; você descobre às 3 da manhã em qual branch você está | Você descobre. |

Olhe a linha 2. Esse é o golpe todo. Num plano de capacidade de verdade, quando você excede a capacidade, você derruba numa *fronteira conhecida* e loga. Com um rate limiter, você retorna `429` e *tora pro caller respeitar.* O caller não respeita. O caller nunca respeitou. A política de retry do client SDK foi escrita em 2017 por alguém que leu um blog post sobre exponential backoff, implementou sem jitter, e shipou. Cada `429` que você manda gera, em média, 2.3 retries. Seu rate limiter, que deveria *reduzir* a carga, *aumentou* a carga por um fator de 3.3. Isso não está documentado em lugar nenhum. O dashboard do limiter mostra "requisições rejeitadas: 30%". A métrica de "requisições recebidas", que inclui retries, não está no mesmo dashboard. Você nunca as viu no mesmo gráfico. Se visse, veria que "30% rejeitadas" é "39% de carga adicional", e seu sistema, que estava a 70% de capacidade, agora está a 109%, e você está derrubando mais, e os retries vêm mais rápido, e o limiter está "protegendo" o sistema acelerando a própria sobrecarga que foi contratado pra prevenir.

## Por Que "Só Usa Um Token Bucket" É Uma Frase Que Encerra Carreiras

O time de plataforma, tendo descoberto que 429s causam retries, vai atrás de um algoritmo "mais esperto". Eles vão dizer: "Vamos usar um token bucket — ele permite bursts, que é mais tolerante que uma fixed window." Um token bucket é um balde que segura `BURST` tokens, reabastece a `RATE` tokens por segundo, e consome um token por requisição. Esse é um algoritmo fino. Também é uma mentira, porque é *um* balde, e você tem *muitos* servidores, e o balde vive em *um* lugar, e esse lugar é o Redis.

Deixa eu te mostrar como "token bucket distribuído" parece na prática:

```python
def allow(client_id):
    key = f"bucket:{client_id}"
    # INCR é atômico, mas DECR-e-refill não é.
    tokens = redis.incr(key)
    if tokens == 1:
        redis.expire(key, 60)        # a "window" — um palpite
    if tokens <= BURST:
        return True
    return False                      # 429
```

Essa é a implementação em 80% das bibliotecas de rate limiter que você vai achar no GitHub. Não é um token bucket. É um *fixed window counter* com um expire. Não tem conceito de taxa de refill. Não tem conceito de capacidade de burst além de "as primeiras `BURST` requisições em qualquer janela de 60 segundos." Ele permite que um cliente mande `BURST` requisições no primeiro segundo e então tome 429 por 59 segundos. Isso é o oposto de "tolerante a burst". É "punitivo com burst". O README da biblioteca diz "token bucket". O código é uma fixed window. O README tem 2.400 estrelas. O código está errado.

E aí tem a parte do Redis. Seu rate limiter agora depende do Redis. O Redis agora está no seu request path. Toda requisição faz um `INCR` no Redis. Seu Redis está a 80.000 ops/s, o que é ok, até o Redis ter uma pausa de GC de 200ms — que o Redis tem, porque é single-threaded e o `BGSAVE` fez fork de um processo de 4GB — e durante esses 200ms, toda requisição que bate no limiter dá timeout. Um timeout no limiter é, por padrão, "rejeitar". Então uma pausa de 200ms no Redis vira uma outage de 200ms de todo serviço que usa o limiter. Seu rate limiter, que deveria *proteger* o sistema, virou o single point of failure do sistema. O time de plataforma responde "tornando o limiter fault-tolerant": se o Redis cai, permite todo tráfego. Essa é a decisão correta. Também é a admissão de que o limiter não é, de fato, crítico — porque no momento em que ele falha, você prefere *nenhum* limite a uma falha dura. Um componente que você desabilita quando quebra não é "missão-crítico". É "missão-decorativo."

[Como o XKCD 1736](https://xkcd.com/1736/) diagnosticou, qualquer sistema que depende de um recurso compartilhado único pra sua "segurança" apenas realocou o modo de falha, não o removeu. Seu rate limiter não removeu sobrecarga. Ele moveu a sobrecarga do seu serviço pro Redis, e depois, quando o Redis falha, ele move de volta pro seu serviço, tudo de uma vez, com retries.

## O Exemplo Do Mundo Real Que Prova Tudo

Um time com quem trabalhei — vou chamá-los de "o time de plataforma", porque eram eles — decidiu "adicionar rate limiting adaptativo com detecção de anomalia" pra que "a gente derruba automaticamente tráfego abusivo antes de chegar nos serviços". Onze meses depois:

1. Eles tinham um **rate limiter por serviço**, cada um com um **`RATE` diferente**, afinado à mão pelo engenheiro de plantão que aconteceu de ser escalado na semana em que foi setado. As rates estavam num arquivo de config que ninguém tocava há 8 meses. O serviço cuja rate foi setada durante um pico de tráfego estava permanentemente over-limited. O serviço cuja rate foi setada numa semana calma estava permanentemente under-limited. Ambos estavam "corretos" pelo arquivo de config. Nenhum estava correto pela realidade.
2. O **modelo de detecção de anomalia** foi treinado em três semanas de tráfego. As três semanas foram em fevereiro. O modelo nunca tinha visto uma corrida de segunda de manhã, um feriado, um lançamento de produto, ou uma menção na imprensa. Ele flageou todos os quatro como "anômalos". Na manhã de uma matéria no TechCrunch, o limiter derrubou 62% dos signups inbound por 47 minutos, porque os signups "não batiam com o baseline aprendido". O baseline era fevereiro. O lançamento era setembro. O modelo não sabia da existência de meses. O postmortem apontou como causa raiz "model drift". A correção foi "retreinar o modelo". O modelo foi retreinado no tráfego do lançamento. O próximo lançamento foi flagedo como anômalo de novo, porque o modelo agora acreditava que *um* lançamento era normal e *dois* era suspeito.
3. Um **bulk import legítimo** de um time — um job noturno que manda 50.000 chamadas de API em 4 minutos — foi rate-limited porque excedeu o limite por-cliente. O job deu retry. Os retries também foram rate-limited. O job deu retry nos retries. O budget de retry do job era "infinito" porque o autor setou `max_retries = -1`, que a biblioteca de retry interpretou como "retry pra sempre", que o autor descobriu três semanas depois quando a DLQ tinha 1.4 milhão de mensagens, cada uma duplicata das 50.000 originais, espalhadas por 28 ondas de retry. O consumer da DLQ era uma lambda com timeout de 15 minutos. A lambda processava 40 mensagens por invocação. A DLQ estava crescendo mais rápido do que a lambda conseguia drenar. A solução do time foi "aumentar o rate limit pro cliente de bulk-import". O rate limit do cliente de bulk-import agora era 50.000 por 4 minutos. Isso não é um rate limit. É um whitelist com formato de rate limit. O limiter, pra esse cliente, não estava fazendo nada.
4. A **causa raiz do postmortem** do incidente do TechCrunch foi "o modelo de anomalia não foi treinado com tráfego de lançamento". A causa raiz real era "o sistema não conseguia lidar com o tráfego do lançamento, então um segurança foi contratado pra manter do lado de fora, e o segurança manteve do lado de fora." A correção nos action items foi "adicionar tráfego de lançamento no training set." Ninguém fez a pergunta: *se o sistema não consegue lidar com o tráfego que ele existe pra receber, por que o sistema existe?* A resposta, que ninguém disse em voz alta, era "porque dimensionamos o banco pra tráfego rate-limited, e o rate limiter agora é o plano de capacidade, e o plano de capacidade agora é um modelo de machine learning treinado em fevereiro." Isso não é reliability engineering. Isso é astrologia, com Redis.
5. Eles adicionaram um "dashboard de rate-limit". O dashboard mostrava `429s por segundo`. Não mostrava `retries por segundo`, `profundidade da DLQ`, `time-to-first-byte das requisições aceitas`, ou `receita derrubada`. O dashboard estava verde durante o incidente do TechCrunch, porque o limiter estava "funcionando conforme configurado". Estava derrubando 62% dos signups. O dashboard disse que o limiter estava saudável. O limiter *estava* saudável. O negócio não estava. O dashboard e o negócio tinham definições diferentes de "saudável". A definição do dashboard venceu, porque o dashboard é o que o plantão olha.

Eles substituíram "dimensionar o banco pra aguentar o pico, que a gente consegue prever a partir do pico do ano passado mais 20% de headroom" por "contratar um segurança que rejeita 30% dos clientes na porta, treinado em fevereiro, backed por Redis, debugado lendo um float que você não consegue reproduzir, e revisto por um dashboard que chama derrubar de 'saudável'". No mundo antigo, "a gente aguenta o lançamento?" era uma pergunta respondida por uma planilha com os números do ano passado. No mundo novo, é uma pergunta respondida por "a gente descobre às 9 da manhã quando o modelo decide se setembro parece com fevereiro". A planilha era chata e correta. O modelo é emocionante e errado.

## O Que O Elenco De Dilbert Diria

> **Wally:** "Eu dependo do rate limiter porque não sei qual o tamanho do nosso banco. O rate limiter também não sabe. Ele foi treinado em fevereiro. Agora é novembro. O modelo acha que novembro é um ataque. Eu digito `429`. O cliente dá retry. Eu digito `429` de novo. Até agora, tudo bem. O 'até agora' está fazendo muito trabalho."

> **Dogbert:** "Um rate limiter é um segurança que não consegue explicar o código de vestimenta, foi treinado em três semanas de moda de uma estação diferente, e rejeita clientes pagantes com base num score de confiança que é um float que você não consegue reproduzir. Você reinventou a corda de veludo e removeu a única feature dela — o julgamento do segurança. Esse é o ato de subtração mais impressionante desde que alguém inventou a fila do TSA PreCheck e pôs funcionários da fila comum nela."

> **Mordac, o Preventer de Serviços de Informação:** "Todos os serviços devem usar o rate limiter canônico da biblioteca compartilhada. A confiabilidade subiu 30%. A biblioteca tem 11 knobs, três dos quais se contradizem. Tenho uma certificação em 'Adaptive Throttling'. Ela não menciona que o limiter é backed por Redis, e Redis é o single point of failure. Tenho uma segunda certificação em 'Redis HA'. Ela não menciona que o Redis HA faz failover em 12 segundos, e 12 segundos são 12.000 requisições com timeout, e requisições com timeout são, por padrão, rejeitadas. O limiter e a certificação concordam que isso é 'aceitável'. Eu concordo com eles. Isso se chama 'alignment'."

> **O Chefe Careca de Pontas:** "O segurança não pode simplesmente... deixar as pessoas entrarem? Tipo uma porta? Aberta? Até acabarem as cadeiras?" (Ele é, de novo, a única pessoa no prédio cujo modelo mental do sistema está correto, porque é o único que é mais simples que o sistema.)

## A Questão "Mas E O Sliding Window Log?", Respondida Uma Vez Por Todas

Os zelotas vão dizer: *"Mas você pode usar um sliding window log — ele guarda o timestamp de toda requisição num sorted set, então é perfeitamente preciso!"*

Deixa eu te mostrar o que um sliding window log faz com o Redis. Ele guarda, num sorted set, o timestamp de toda requisição, por cliente, pros últimos `WINDOW` segundos. Pra um cliente mandando 100 req/s com uma window de 60 segundos, isso é 6.000 entradas por cliente. Pra 10.000 clientes, isso é 60 milhões de entradas no Redis. Cada requisição faz um `ZREMRANGEBYSCORE`, um `ZADD`, e um `ZCARD` — três round trips, três operações de sorted set, numa estrutura de dados de 60 milhões de entradas. Redis, single-threaded, faz isso em talvez 8.000 ops/s antes de saturar. Seu limiter "perfeitamente preciso" maxima em 8.000 req/s, total, entre todos os clientes, e seu sistema estava fazendo 40.000 req/s. O limiter agora é o gargalo. O gargalo é "precisão". Você tornou a precisão o gargalo. O time de plataforma responde "aproximando a sliding window com uma fixed window de buckets de 1 segundo, que é 98% precisa". Os 2% de imprecisão significam que 2% das requisições são mal-admitidas ou mal-rejeitadas. A 40.000 req/s, são 800 req/s de decisões erradas. O limiter "perfeitamente preciso" virou um limiter "aproximadamente preciso" porque a precisão perfeita era lenta demais. A aproximação é a mesma fixed window com que você começou. Você gastou seis meses e um cluster Redis pra chegar de volta ao algoritmo que rejeitou na semana um. Isso se chama "maturidade de engenharia".

Planos de capacidade de verdade têm uma propriedade que você consegue ler: "o sistema aguenta N req/s; acima de N, a gente enfileira ou derruba, explicitamente, no load balancer." Rate limiters têm uma propriedade que você experiencia: "o limiter permite mais ou menos M req/s por cliente, aproximadamente, a não ser que a window drete, a não ser que o cliente dê retry, a não ser que o Redis blipe, a não ser que o modelo ache que ainda é fevereiro." Não existe comando `rate-limiter explain <client>` que imprime, em português claro, por que uma dada requisição foi rejeitada, porque a resposta é "um float num sorted set num Redis que pode ou não ter sido o líder naquele momento." Você não consegue debugar um float. Você só consegue retreinar.

[Como o XKCD 2574](https://xkcd.com/2574/) alertou, qualquer sistema cuja segurança dependa de um modelo treinado numa janela estreita de "normal" vai tratar toda situação genuinamente nova como um ataque. O "normal" do seu limiter era fevereiro. Seu lançamento era setembro. Setembro parecia um ataque pra um modelo que nunca tinha visto setembro. O modelo não estava errado, no sentido técnico. O modelo estava *overfit pra um calendário que ele não conhecia.* Isso não é um bug que você conserta com mais dados. Isso é um erro de categoria: você pediu a um modelo estatístico pra tomar uma decisão de planejamento de capacidade, e ele te deu um score de confiança, e você tratou o score como decisão, e a decisão derrubou 62% da sua receita.

## A Arquitetura De Longo Prazo

Eventualmente seu ecossistema de rate limiting parece com isso:

```
Seu limiter "canônico"     → biblioteca compartilhada, decorator @rate_limited, 40 serviços
Seus valores de rate      → um config file tocado pela última vez há 8 meses, por alguém que saiu
Sua estratégia           → "sliding" (na verdade fixed window, segundo o source que ninguém lê)
Seu Redis                 → 80k ops/s, single-threaded, pausa de GC de 200ms = outage de 200ms
Seu modelo de anomalia   → treinado em fevereiro; flaggeia lançamentos, feriados e segundas
Sua lista de isentos      → contém "???-3"; remover é um Sev-2; é permanente
Sua DLQ                   → 1.4M de mensagens, crescendo; lambda drena 40/invocação; perdendo
Sua política de retry     → exponencial, sem jitter, max_retries=-1, escrita em 2017
Seu dashboard             → verde durante o lançamento; "saudável" = "limiter funcionando"
Seu plano de capacidade   → o limiter É o plano; é um float no Redis
Seus reviewers             → não conseguem reproduzir um 429; aprovam o config; torcem
Seu plantão               → escalado quando o limiter está "agressivo demais" (todo lançamento)
Seu negócio               → derrubou 62% dos signups por 47 minutos; chamou de "proteção"
```

O time que simplesmente dimensiona o banco pra aguentar 1.5x o pico — uma planilha, os números do ano passado, mais headroom — tem um sistema que admite todo mundo, não derruba ninguém, e nunca derrubou um lançamento. Eles estão, no entanto, "não usando o rate limiter canônico", o que significa que estão "não seguindo o padrão de confiabilidade", o que significa que o time de plataforma tem um Jira ticket sobre eles. Esse é o custo real de dimensionar seu banco corretamente: um Jira ticket. O custo técnico é negativo — você gasta *menos*, porque não roda um cluster Redis, um modelo de ML, uma DLQ, uma lambda, e uma escala de plantão pra um segurança. O custo político é uma reunião recorrente. Então o time adota o limiter, entra no config file, e começa a debugar floats. Todo mundo agora está "protegido". Proteção, em rate limiting, significa "igualmente incapaz de explicar por que um cliente pagante tomou um 429". Essa é a vitória que o time de plataforma celebra na review trimestral.

## Resumo, Mas É Um Segurança

| Princípio | Posicionamento |
|---|---|
| Dimensionar seu sistema pra aguentar 1.5x o pico | Faça. É uma planilha. Você consegue ler. Seu plantão consegue ler. O lançamento funciona. |
| Adicionar um rate limiter | Você contratou um segurança que não sabe explicar o código de vestimenta, backed por Redis, debugado por um float, e "protegendo" você do tráfego que você existe pra receber. |
| Token bucket no Redis | Um fixed window counter com um README que mente. O README tem 2.400 estrelas. |
| Sliding window log | 60M de entradas no Redis, maxa em 8k req/s, "precisão perfeita" lenta demais, vira a aproximação que você rejeitou na semana um. |
| Limiter com detecção de anomalia | Um modelo treinado em fevereiro, flaggeia lançamentos, derruba 62% da receita, o dashboard chama de "saudável". |
| A política de retry que seus clientes usam | Sem jitter, `max_retries=-1`, escrita em 2017, transforma 30% de rejeições em 39% de carga adicional. O limiter está acelerando a sobrecarga que foi contratado pra prevenir. |
| A DLQ | 1.4M de mensagens. A lambda drena 40/invocação. Você está perdendo. |
| A lista de isentos | Contém `"???-3"`. Remover é um Sev-2. É permanente. |
| Seu plano de capacidade | É um float num Redis que pode não ter sido o líder. Isso não é um plano. É uma prece com dashboard. |
| Sua certificação em "Adaptive Throttling" | Não menciona que o limiter é o single point of failure. Deveria. |

Se sua solução pra "o sistema não consegue lidar com o tráfego que ele existe pra receber" é "contratar um segurança que rejeita 30% dos clientes na porta, treinado em fevereiro, backed por um Redis single-threaded, debugado lendo um float que você não consegue reproduzir, e revisto por um dashboard que chama derrubar de 'saudável'", você não tornou o sistema confiável. Você o tornou *educado ao falhar*. A sobrecarga nunca foi reduzida. Foi realocada — do seu banco, onde você podia dimensionar, pro seu limiter, onde você não consegue, e depois, quando o limiter blipa, de volta pro seu banco, tudo de uma vez, com retries. O cliente agora recebe um 429 em vez de um timeout. O 429 é a "melhoria". A melhoria é um status code. O status code é o produto inteiro. O produto é um segurança.

Eu dimensiono meu banco pra 1.5x o pico. É uma planilha. Meu plantão lê em 2 minutos. Meus lançamentos funcionam. Meus clientes são admitidos. Não tenho Redis no meu request path, nem modelo treinado em fevereiro, nem DLQ com 1.4M de mensagens, nem `"???-3"` na minha lista de isentos. Estou, no entanto, "não usando o rate limiter canônico". O time de plataforma abriu um ticket. Eu vou comparecer à reunião recorrente. Esse é um custo que aceitei.

---

*O autor dimensiona seus bancos com uma planilha. O time de plataforma chama isso de "ingênuo". O autor chama de "admite todo mundo". O banco nunca derrubou um lançamento. O autor considera esse o único métrica que importa.*
