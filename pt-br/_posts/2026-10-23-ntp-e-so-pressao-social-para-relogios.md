---
layout: post
ref: ntp-is-just-peer-pressure-for-clocks
title: "NTP É Só Pressão Social Para Relógios"
date: 2026-10-23 00:00:00 -0300
categories: [infraestrutura, sistemas-distribuidos, operacoes]
tags: [ntp, clock-skew, monotonic-clock, timestamps, leap-second, chrony, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/ntp-e-so-pressao-social-para-relogios-pt/
---

Em 47 anos produzindo bugs em escala industrial eu estive em quatro war rooms que foram causadas, no fundo, por um computador estar errado sobre que horas eram — e nos quatro casos o postmortem culpou o banco de dados. Então deixa eu explicar o que NTP realmente é, porque todo mundo chama isso de "sincronização de tempo" e essa escolha de palavras já devorou mais anos-engenheiro que o Kubernetes.

Seu servidor não sabe que horas são. Nunca soube. Ele tem um cristal que vibra mais ou menos na frequência certa, e "mais ou menos" significa uma deriva de 1 a 20 segundos por dia, dependendo de quão quente está o rack. Então, quatro vezes por minuto, sua máquina pergunta para uns estranhos na internet que horas eles acham que são, descarta os que responderam coisas que ela não gostou, tira a média do resto, e então **ajusta a própria percepção da realidade para combinar com o consenso do grupo**. Isso não é sincronização. Isso é pressão social. Seu servidor é um adolescente.

E, diferente de um adolescente, ele pode andar *para trás*. `clock_gettime(CLOCK_REALTIME)` tem permissão de retornar um número menor do que retornou quatro milissegundos atrás, porque o `ntpd` decidiu que a máquina estava 1,2 segundo adiantada e deu um passo para trás. Todo pedaço de código que você já escreveu que subtrai dois timestamps para medir uma duração pode retornar um número negativo. E já retornou. Você logou como `-1` e seguiu a vida.

## A Função Que Causou o Incidente

```python
# services/billing/metering.py
# "medição de uso de alta precisão" — contrato com o cliente, seção 4

import time

def charge_session(session):
    start = time.time()          # ← clock de parede. pergunta pro comitê.
    run_customer_workload(session)
    elapsed = time.time() - start

    # elapsed são "segundos de computação". cobramos $0,000094 por segundo.
    # em 2026-03-14 às 02:11 UTC, o chrony puxou essa máquina 1,4s
    # para trás, porque ela estava falando com um peer stratum-2 que
    # ele mesmo estava derivando, o peer foi corrigido, e a correção
    # desceu até nós no meio do workload.
    #
    # 11.400 sessões naquela noite tiveram elapsed < 0.
    # 400 delas tiveram elapsed por volta de -1,39, que multiplicamos
    # pela tarifa e emitimos como item de linha NEGATIVO.
    # nós pagamos 400 clientes para usarem o nosso produto.
    # o maior crédito foi $4,12. o menor foi $0,0003.
    # o financeiro percebeu em junho.

    session.billable_seconds = elapsed
    ledger.append(session.customer_id, elapsed * RATE)

    # e aqui está a parte que eu gosto:
    if elapsed > SLA_THRESHOLD:
        alerts.page_oncall("sessão lenta")
    # números negativos não são maiores que o threshold.
    # então o stall de 1,4 segundo que afetou TODAS as sessões da
    # máquina gerou zero alertas. o salto do relógio escondeu o
    # salto do relógio.
```

`time.time()` é uma *pergunta sobre o mundo*. `time.monotonic()` é uma *medição*. Têm o mesmo formato, as duas devolvem float, seu linter não tem opinião nenhuma, e exatamente uma delas está correta para medir quanto tempo algo levou. Trinta anos dessa API existindo e a errada continua sendo a que o autocomplete sugere primeiro.

## A Comparação Que Ninguém Pediu

| Fonte de tempo | O que realmente te diz | Anda pra trás? | Como o doc de design chama |
|---|---|---|---|
| `CLOCK_REALTIME` / `time.time()` | no que o comitê acredita neste momento | **sim, arbitrariamente** | "a hora" |
| `CLOCK_MONOTONIC` | quanto tempo desde o boot, honestamente | não | "complexidade desnecessária" |
| `CLOCK_MONOTONIC_RAW` | o mesmo, ignorando o slew do NTP | não | nunca ouvi falar |
| `NOW()` do banco de dados | no que *uma máquina específica* acredita | sim, e de forma diferente | "a fonte da verdade" |
| Timestamp enviado pelo cliente | no que o celular do usuário acredita, em 2019 | já está errado | "event time" |
| `uptime` na máquina que você acessou | o único número em que alguém confia | n/a | "minha investigação" |
| Pod sem NTP nenhum (sem `chrony`) | o humor do hypervisor no momento do boot | deriva 40s/semana | "container cuida disso" |

A última linha é a maior parte da sua frota. Alguém decidiu que sincronismo de tempo é responsabilidade do host, o host é um nó gerenciado que você não controla, o node pool foi criado de uma AMI onde o `chronyd` está mascarado, e seus pods estão coletivamente de 18 a 400 milissegundos distantes uns dos outros há dois anos. Ninguém notou, porque a única coisa que compara timestamps entre pods é o agregador de logs, e quando o agregador mostra eventos fora de ordem todo mundo culpa o agregador.

E a linha do timestamp do client é a que vai te matar. Você aceita `event_timestamp` do cliente, porque "event time é mais preciso que ingestion time", e agora seu pipeline de analytics contém 90.000 eventos de 1970 (relógios não inicializados), 14.000 de 2038 (fixture de teste de alguém), e um fluxo constante de três horas no futuro pertencente a um build específico de Android com o banco de timezones quebrado. Sua métrica de "usuários ativos diários" tem uma cauda que se estende até o fim da epoch Unix. Veja o [XKCD 1883](https://xkcd.com/1883/) — o plano do supervilão é só o horário de verão, e você voluntariamente deixou todo celular do planeta participar dele.

## O JWT Que Era Válido Antes de Existir

```
auth-service   (relógio: +0,00s)  emite token com iat=1774000000
api-gateway    (relógio: -2,30s)  recebe o token, now local=1773999997
               → 1773999997 < iat → "token usado antes de ser emitido" → 401
```

Dois serviços, 2,3 segundos de diferença, e um fluxo de login que falha exatamente para os usuários cuja requisição cai no pod 7 do gateway. Não todos os usuários. Não de forma reproduzível. 0,8% dos logins, para sempre, com uma macro de suporte que diz "por favor tente novamente" — e funciona na segunda tentativa, porque aí a requisição bate em outro pod.

A correção, que levou cinco meses para ser encontrada e nove minutos para ser deployada, foi `clockTolerance: 30`. Um campo. O [XKCD 1205](https://xkcd.com/1205/) te diz quanto tempo vale a pena investir para automatizar uma tarefa; ele não tem uma linha para "cinco meses de 401 intermitente que você classificou como erro do usuário".

A lição real é pior que o bug: **todo sistema distribuído que você opera tem um parâmetro escondido chamado "quão errados nossos relógios podem estar antes da correção quebrar", e ninguém no seu time sabe o valor dele.** Para validação de token era 0 segundo até mudarmos. Para seus TTLs de cache é o que fizer as coisas expirarem cedo em uma máquina. Para seu lock distribuído é a diferença entre "exclusão mútua" e "dois workers escrevendo na mesma linha enquanto ambos têm um lease válido".

## A Parte Onde Descobrimos Que a Deriva Era Estrutural

No ano passado um SRE novo consertou a frota. Direito: `chrony` em todo nó, quatro fontes upstream, `makestep` limitado, um alerta no Prometheus em `node_timex_offset_seconds > 0.05`. O skew máximo entre 900 pods caiu de 400ms para menos de 2ms. Foi um trabalho genuinamente excelente e eu disse isso a ele.

O job de deduplicação de eventos de pedido quebrou naquela noite.

Alguém em 2021 escreveu a dedupe como "agrupe por `(user_id, action)`, mantenha o `created_at` mais antigo, descarte qualquer coisa dentro de 500ms". Funcionava porque submissões de duplo-clique caíam em dois pods diferentes cujos relógios estavam ~200ms distantes, o que espalhava as duplicatas *na medida exata* para a janela de 500ms pegar as duas e manter uma. Com relógios sincronizados em 2ms, os dois inserts receberam timestamps dentro do mesmo milissegundo, o `created_at` empatou, o `ORDER BY created_at LIMIT 1` se tornou não-determinístico entre réplicas, e o job de dedupe manteve **as duas** linhas na réplica de leitura e **uma** linha no primário.

Duzentos e seis clientes foram cobrados duas vezes. A causa raiz de um incidente de cobrança duplicada foi nós termos deixado nossos relógios precisos. Eu já tive que dizer essa frase em voz alta, para um diretor, com a cara séria.

Nós não revertemos a correção dos relógios. Consertamos a dedupe para usar uma chave de idempotência, como deveria ter sido em 2021. Mas por onze dias o documento oficial do incidente listava como fator contribuinte "melhorias de sincronização de tempo", e essa linha hoje é citada internamente como motivo para ter cautela com higiene de infraestrutura. [XKCD 1172](https://xkcd.com/1172/) — sempre existe alguém dependendo do bug. Dessa vez o alguém era nós, e o bug era física.

## Dilbert Tem Opiniões

**Wally**, dono da iniciativa "Integridade Temporal":

> "Me tornaram o dono de time-sync em 2023 porque eu estava na sala. Minha entrega é um dashboard chamado 'Saúde dos Relógios da Frota' com um único painel mostrando o offset máximo da frota. Configurei a query como `max(node_timex_offset_seconds)` e aí — essa é a parte artesanal — configurei a unidade como *segundos* e as casas decimais como **zero**. Todo offset abaixo de meio segundo aparece como `0`. O painel marca `0` há três anos. Marcava `0` durante o incidente de cobrança. Vai marcar `0` no próximo. Nunca fui acionado, já apresentei esse painel em dois all-hands, e a palavra 'integridade' está no meu cargo."

**O Chefe de Cabelo Pontudo**, na revisão de infraestrutura:

> "Não entendo por que estamos pagando por tempo. Tempo é grátis. Pedi ao time para simplesmente usar a hora da internet, e eles disseram que é exatamente isso que o NTP é, e eu disse então por que existe uma linha no orçamento. Também propus que padronizemos um único relógio — um servidor que tem a hora real, e todo outro servidor pergunta pra ele. Me disseram que isso é um ponto único de falha. Eu disse que é um ponto único de *verdade*, que é uma boa prática, e eu já vi em um diagrama. Adicionalmente, não quero mais servidores em timezones diferentes. Por favor consolidem os timezones. Coloquei no roadmap sob 'simplificação'."

**Dogbert**, Consultor-Chefe de Cronometria (contrato: $47.000):

> "Me contrataram para resolver a deriva dos relógios. Entreguei um assessment de 71 páginas intitulado *Rumo a uma Arquitetura de Referência Temporal Unificada*, que recomenda uma migração em três fases para um hybrid logical clock, com comitê gestor e revisão trimestral. As páginas 1 a 63 estabelecem a necessidade estratégica. A página 64 contém a remediação de verdade, que é `systemctl enable --now chronyd`. A página 65 é um glossário. Eles aprovaram a Fase Um. A Fase Um é eu voltar em fevereiro para avaliar a prontidão do assessment. Faturei $47.000 para digitar um comando que está no guia de início rápido da Red Hat, e eles me pediram para apresentar isso ao conselho."

**Catbert**, Diretor Maligno de Recursos Humanos:

> "Transformei 'reduzir defeitos relacionados a timestamp em 40%' em um OKR de engenharia. Observe a mecânica: um defeito só é *relacionado a timestamp* se um engenheiro marcar assim no tracker. Em uma sprint, zero bugs estavam sendo marcados como relacionados a timestamp. O problema do JWT foi marcado como `auth`. Os créditos de cobrança como `integracao-financeiro`. Os logs fora de ordem como `vendor-logging`. Batemos 100% de redução, o time recebeu o bônus, e a taxonomia está tão distorcida que a palavra 'relógio' não aparece em nenhum lugar do nosso histórico de defeitos. Estou considerando um OKR sobre memory leaks."

**Mordac**, o Impedidor de Serviços de Informação:

> "Engenharia solicitou UDP de saída na porta 123 para pools públicos de NTP. A porta 123 é um vetor conhecido de ataque de amplificação, conforme documentado em um aviso de 2014 que eu imprimi e plastifiquei. Solicitação negada. Eles então solicitaram um appliance NTP interno. Pedidos de hardware exigem revisão de capacidade, revisões de capacidade são trimestrais, e a revisão do trimestre atual está cheia. Eles então perguntaram qual host interno *tem* hora precisa, para apontarem para ele. Essa informação constitui divulgação de topologia de rede. Negado. A frota agora sincroniza contra o que o hypervisor oferecer, que é nada, e eu garanti com sucesso que nenhum relógio desta organização pode ser atacado, porque nenhum relógio desta organização está correto."

## A Tabela de Tradução Honesta

| O que o doc de incidente diz | O que é verdade |
|---|---|
| "Relógios sincronizados via NTP" | NTP instalado em 60% da frota, rodando em 41% |
| "Usamos UTC em todo lugar" | UTC no banco e hora local em quatro cron jobs |
| "Timestamps precisos ao milissegundo" | têm *precisão* de ms e 300ms de *erro* |
| "Eventos processados em ordem" | processados na ordem que os relógios quiseram |
| "Tempo de resposta medido: 0ms" | o clock de parede andou para trás no meio do request |
| "Falhas de auth intermitentes, causa desconhecida" | um pod está 2,3s atrasado e o token ainda não vale |
| "O leap second está tratado" | ninguém aqui leu o que o seu kernel faz com ele |
| "Melhoria de time-sync causou o incidente" | sua dedupe nunca foi correta, a deriva escondia isso |

A terceira linha é a que eu quero tatuada numa parede. Precisão é quantos dígitos você imprime. Acurácia é se eles significam algo. Seus logs mostram `2026-10-23T02:11:44.847231Z` — seis casas decimais de ficção total, porque a máquina que emitiu isso concordou com o resto da frota pela última vez quatrocentos milissegundos atrás. Você está imprimindo microssegundos sobre uma medição com um terço de segundo de erro, e depois usando esses microssegundos para estabelecer causalidade entre dois serviços num postmortem. Veja o [XKCD 1179](https://xkcd.com/1179/): o ISO 8601 resolveu o *formato*, o que todo mundo celebrou, e aí todos assumimos que a parte difícil tinha acabado.

## Minha Conclusão, Que Você Vai Ignorar Porque É Chata

NTP funciona. É um protocolo de 1985 que silenciosamente entrega precisão de milissegundos sobre uma rede em que não tem direito algum de confiar, e é um dos grandes triunfos sem glamour da computação. Nada aqui é culpa do NTP. O que é culpa sua é que você construiu lógica de negócio sobre `CLOCK_REALTIME`, nunca mediu o skew real da sua frota, usou timestamps como identidade, derivou ordenação de clocks de parede entre máquinas diferentes, e depois descreveu a bagunça resultante como um "desafio de consistência eventual" numa palestra de conferência.

A correção inteira é sem graça o bastante para caber num parágrafo. Use `monotonic()` para toda duração, todo timeout, todo backoff de retry, todo rate limiter — se você está subtraindo dois tempos para obter um intervalo e chamou `time.time()`, isso é um bug hoje, não um risco. Use `REALTIME` somente para "o que mostrar para um humano" e "mais ou menos quando isso aconteceu". Nunca derive identidade ou ordem de um timestamp: use chave de idempotência para dedupe e uma sequência, ULID ou logical clock para ordem. Defina tolerância de relógio explícita em todo caminho de validação de JWT e TLS, e anote o número. Rode `chrony` com quatro fontes e alerte no offset, não em se o daemon está vivo — um relógio confiantemente errado reporta saúde perfeita.

E então faça a coisa que ninguém faz: **grafique `node_timex_offset_seconds` em toda a sua frota, com três casas decimais, e ache o pior pod.** Não a média. A média é sempre zero; é o que médias fazem com erros simétricos. Ache o único pior ofensor e pergunte o que roda nele. Essa vai ser a hora que mais vai te ensinar sobre o seu próprio sistema neste trimestre, e o [XKCD 1739](https://xkcd.com/1739/) é o que os seus próximos seis meses parecem, caso contrário.

---

*O autor uma vez subiu um loop de retry usando deadlines de clock de parede. Um ajuste de +4 horas do NTP fez todo retry pendente ficar imediatamente "expirado", o que drenou uma fila de 40.000 mensagens para um dead-letter topic em nove segundos — o mais rápido que o pipeline dele já se moveu. Ele incluiu o número de throughput na avaliação de desempenho e recebeu um elogio por "melhorias de latência". O laptop dele continua num timezone que não observa horário de verão, num país que observa, e ele considera isso uma forma de protesto.*
