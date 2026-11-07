---
layout: post
ref: your-kubernetes-resource-limits-are-just-guesses-with-units
title: "Seus Resource Limits do Kubernetes São Só Palpites Com Unidade"
date: 2026-11-07 00:00:00 -0300
categories: [kubernetes, infraestrutura, conselhos-terriveis]
tags: [kubernetes, k8s, resource-limits, requests, cfs-quota, throttling, oomkilled, exit-137, qos-classes, cgroups, vpa, hpa, bin-packing, limitrange, resourcequota, jvm, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/seus-resource-limits-do-kubernetes-sao-so-palpites-com-unidade-pt/
---

Em 47 anos produzindo bugs em escala industrial eu preenchi muitos formulários que não entendia, e o que eu entendo menos é esse:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi
```

Eu já escrevi esse bloco umas 900 vezes. Nunca, nenhuma única vez, medi nada antes de escrever. Copiei de outro serviço, que copiou de um post de blog, que copiou do exemplo da documentação do Kubernetes, que foi escrito por alguém demonstrando sintaxe de YAML e não fazendo uma afirmação de capacidade sobre a *sua* aplicação Java.

E ainda assim esses quatro números decidem se o seu serviço vai ser agendado, se vai ser estrangulado, se vai ser assassinado dormindo, e quanto a sua empresa paga pra AWS todo mês. Quatro números. Tirados do ar. Com unidade, que é justamente o que faz eles parecerem engenharia.

**Um número com unidade é indistinguível de uma medição.** Esse é o fato mais útil que eu conheço, e eu construí uma carreira inteira em cima dele.

## O Que os Números Realmente Fazem, e Que Ninguém Me Contou em Seis Anos

Olha, isso levou um tempo vergonhoso pra eu aprender: `requests` e `limits` não são "mínimo" e "máximo". São dois mecanismos completamente desconexos que alguém colocou lado a lado no mesmo bloco de YAML de trollagem.

- **`requests` é uma reserva de agendamento.** É o único número que o scheduler lê. Reserva espaço num nó. Depois do agendamento, `requests.cpu` se transforma em `cpu.shares` — um peso relativo que só importa quando a CPU está de fato disputada.
- **`limits` é um teto de imposição.** O scheduler não liga nem um pouco pra ele. O kernel liga.

E os dois limites são impostos por *mecanismos de kernel totalmente diferentes, com modos de falha totalmente diferentes*, que é a parte que eventualmente vai te acordar:

```
limite de memória estourado  ->  OOM killer do cgroup  ->  SIGKILL, exit 137, pod reinicia
limite de CPU estourado      ->  CFS quota             ->  throttling, pod sobrevive, latência explode
```

Memória é um precipício. CPU é um imposto. Nada no YAML te conta isso. Estão indentados no mesmo nível, com a mesma cor no seu editor, e um deles mata o seu processo enquanto o outro silenciosamente deixa ele lento. Já participei de call de incidente onde quatro engenheiros discutiram por 40 minutos sem ninguém estabelecer qual dos dois estava acontecendo.

| O que você acredita | O que realmente acontece |
|---|---|
| `requests` é um piso garantido | É uma reserva; você pode passar livremente até o nó encher |
| `limits.cpu: 1` significa "uma CPU" | Significa 100ms de CPU por período de 100ms, imposto brutalmente |
| Bater no limite de CPU deixa o pod um pouco lento | Congela todas as threads até o fim do período |
| Bater no limite de memória deixa o pod lento | Executa o pod. Literalmente. |
| Sem limite definido significa ilimitado | Significa que o `LimitRange` te deu 128Mi caladinho |
| `128M` e `128Mi` são a mesma coisa | Diferem em 6,8%, e esses 6,8% são o seu OOMKill |

Essa última linha é real e já me custou um fim de semana. `M` é 10^6. `Mi` é 2^20. Se sua heap foi ajustada pra 128 MiB e você digitou `128M`, você deu 122 MiB e vai descobrir isso às 3 da manhã.

## A Armadilha do Throttling de CPU

Esse é o meu bug favorito de toda a computação, porque as métricas mentem ativamente pra você.

A CFS quota funciona em **períodos**, por padrão 100ms. Se o seu limite é `500m`, o kernel dá ao seu cgroup 50ms de CPU por período de 100ms. Gastou? *Todas as threads do cgroup ficam congeladas* até o próximo período começar.

Agora imagine um serviço com oito worker threads atendendo uma requisição que precisa de 10ms de CPU cada:

```
8 threads x 10ms = 80ms de CPU demandados
Quota disponível:  50ms por período de 100ms

ms 0-6:    as 8 threads rodam em paralelo, queimam 48ms de quota
ms 6:      quota esgotada
ms 6-100:  TODAS AS THREADS CONGELADAS. nada roda. por 94 milissegundos.
ms 100:    quota recarrega, trabalho continua
```

Seu p99 acabou de ganhar 94ms. E a utilização média de CPU naquele segundo? **Uns 48%.** Seu dashboard está verde. Seu painel do Grafana diz "sobra folga". Seus usuários dizem que o app está quebrado.

Aí alguém olha o gráfico verde de CPU, concluí que o serviço está superdimensionado, e *abaixa o limite pra economizar*, e o throttling dobra. Eu vi isso acontecer. Eu *fiz* isso. Apresentei como otimização de custo e recebi um emoji de polegar pra cima do PHB.

A métrica que fala a verdade é `container_cpu_cfs_throttled_seconds_total`, que ninguém tem em dashboard, porque não estava no tutorial.

```promql
# a query mais importante do seu cluster,
# que eu garanto que não está no seu telão
rate(container_cpu_cfs_throttled_periods_total[5m])
  / rate(container_cpu_cfs_periods_total[5m])
```

[XKCD 221](https://xkcd.com/221/) é a descrição oficial da minha metodologia de limite de CPU. `getRandomNumber()` retorna 4. É aleatório. Eu verifiquei. Meus limites retornam `500m`. São medidos. Eu verifiquei.

## A Catraca de Memória

Limites de memória têm um modo de falha diferente e mais honesto: seu processo é simplesmente executado. Exit code 137. `OOMKilled`. Sem stack trace, sem linha de log, sem flush, sem shutdown gracioso, porque `SIGKILL` não é uma negociação.

A resposta organizacional a isso é sempre a mesma, e é a operação mais comum realizada em manifests de Kubernetes no mundo inteiro:

```yaml
# git log -p deployment.yaml, resumido:
memory: 256Mi   # palpite inicial, janeiro
memory: 384Mi   # "OOMKilled em staging"
memory: 512Mi   # "OOMKilled em prod"
memory: 768Mi   # "OOMKilled de novo"
memory: 1Gi     # "para de me chamar"
memory: 2Gi     # "PARA DE ME CHAMAR"
memory: 4Gi     # ← estamos aqui. ninguém sabe por quê. ninguém vai abaixar.
```

Isso se chama **catraca de memória**. Ela só gira pra um lado. Nenhuma vez em 47 anos eu vi um limite de memória diminuir. Não existe incidente que gere um ticket chamado "esse pod está usando menos RAM do que a gente pensava".

E repare no que um limite de memória de fato é nesse fluxo: **um detector de memory leak com 30 segundos de atraso no alerta**. Você não corrigiu o leak. Você comprou mais tempo pro leak. Com 4Gi o pod agora leva 90 minutos pra morrer em vez de 20, o que significa que ele morre de madrugada em vez de no horário comercial, o que o time viveu como uma melhoria.

### Bônus: seu runtime está mentindo sobre quanta memória ele tem

Por anos a JVM lia `/proc/meminfo`, via os 64GB *do nó*, dimensionava a heap pra um quarto disso — 16GB — e então era OOMKilled por um limite de cgroup de 512Mi cuja existência ela ignorava. O Node.js fazia a mesma coisa com os defaults antigos de heap. O Python com `multiprocessing.cpu_count()` alegremente sobe 96 workers dentro de um cgroup de `500m` e passa a vida inteira throttled.

Runtimes modernos em geral já são container-aware (`UseContainerSupport`, `MaxRAMPercentage`, `availableParallelism`), o que é ótimo, exceto que ninguém atualiza a imagem base, então você está rodando uma JVM de 2017 dentro de um orquestrador de 2026 e se perguntando por que as contas não fecham.

Outra: `container_memory_working_set_bytes` — a métrica que o OOM killer realmente usa — **inclui page cache**. Sua aplicação que lê muitos arquivos do disco está "usando" toda aquela memória, no que diz respeito à contabilidade do kernel. Uma vez eu fui OOMKilled por um arquivo de log.

## QoS Classes: O Sistema de Castas Que Você Entrou Sem Ler

O Kubernetes silenciosamente classifica cada pod em uma de três classes baseado puramente em como você preencheu o formulário. Ninguém te avisa. Não existe campo pra isso. É derivado.

| O que você escreveu | Classe QoS | O que significa pra você |
|---|---|---|
| requests == limits, ambos definidos, em todo container | **Guaranteed** | Último a ser despejado. Pode ganhar cores exclusivos. Os adultos. |
| requests < limits | **Burstable** | Despejado quando o nó passa fome, na ordem de quanto você está acima do request |
| nada | **BestEffort** | Morto primeiro, morto sempre, morto sem remorso |

Então o engenheiro que deixou o `resources:` vazio "porque depois a gente ajusta" voluntariou o serviço dele pra ser o sacrifício designado em todo evento de pressão de memória no nó. E o pod que é despejado *não é necessariamente o que está causando o problema*. É o que tem a pior papelada.

Esse é o mecanismo com a forma mais Dilbert de toda a infraestrutura: **sua prioridade de sobrevivência durante uma crise é determinada inteiramente por um formulário que você preencheu distraído, meses antes, e que ninguém vai revisar nunca.** Catbert projetou isso e eu respeito o ofício.

## Overcommit, ou: Como a Conta é Paga

Como o scheduler só lê `requests`, você pode legalmente fazer isso:

```
Nó: 16 CPU, 64Gi de memória

40 pods, cada um:  requests: 200m / 1Gi      limits: 2 CPU / 8Gi
                   -------------------       -----------------
Total requisitado:  8 CPU / 40Gi  ✅ cabe    Total de limites: 80 CPU / 320Gi  🙃
```

O nó tem 16 CPUs e prometeu 80. Isso funciona maravilhosamente até um pico de tráfego fazer todos tentarem dar burst no mesmo instante, momento em que o nó descobre que não pode honrar 80 CPUs, a CFS começa a dar throttling por peso de `cpu.shares`, a pressão de memória dispara o eviction do kubelet, e seus pods BestEffort evaporam em ordem alfabética do próprio infortúnio.

Enquanto isso, a falha oposta é silenciosamente mais cara. Se todo mundo infla os requests "pra garantir", você acaba com um cluster a 11% de utilização real que não consegue agendar um pod novo porque todo nó está totalmente *reservado*. Você está pagando por 89% de ar. Eu pessoalmente gerei uma fatura de US$ 31.000/mês por ar, e quando o Financeiro perguntou eu usei a expressão "folga para confiabilidade", e eles anotaram.

O número real, que você pode calcular agora e provavelmente não deveria:

```bash
# requisitado vs realmente usado — a diferença é sua fatura por nada
kubectl top pods -A --no-headers | awk '{print $2, $3, $4}'
kubectl describe node | grep -A5 "Allocated resources"
```

[XKCD 605](https://xkcd.com/605/) é o meu processo de capacity planning. Dois pontos de dados, uma régua, e uma extrapolação confiante até o próximo trimestre.

## As Ferramentas Que Deveriam Resolver Isso

**VPA (Vertical Pod Autoscaler)** observa o uso real e reescreve seus requests. Genuinamente uma boa ideia. Também: no modo `Auto` ele reinicia seu pod pra aplicar as mudanças, então sua correção para instabilidade é um componente cuja remediação é "reiniciar a coisa". Ele também aprende do uso histórico, ou seja, aprende do seu uso *throttled* — o número artificialmente suprimido — e confiantemente recomenda que você mantenha assim. Ele mediu a gaiola e declarou que a gaiola tem o tamanho correto.

**HPA (Horizontal Pod Autoscaler)** escala a contagem de réplicas por utilização de CPU *como percentual dos requests*. Então se seus requests são um palpite, seu limiar de autoscaling é um palpite expresso como percentual de um palpite. Coloque `requests.cpu` baixo demais e o HPA escala pra 40 réplicas com tráfego parado. Coloque alto demais e o HPA nunca dispara enquanto cada pod morre throttled a "45% de utilização".

Rode VPA e HPA na mesma métrica e eles brigam. O VPA aumenta os requests, o que abaixa o percentual de utilização do HPA, o que reduz réplicas, o que aumenta carga por pod, o que aumenta uso, o que aumenta a recomendação do VPA. Eu vi isso oscilar com período de quatro horas por três semanas. A gente chamou de "self-healing" na review trimestral.

[XKCD 1205](https://xkcd.com/1205/) é sobre quando vale a pena automatizar uma tarefa. Não tem uma linha pra "a automação exige mais ajuste do que a coisa que ela ajusta", mas deveria.

## A História Que Eu Conto em Entrevistas, Levemente Higienizada

A gente tinha um serviço de pagamentos. Caminho de checkout. O importante.

Ele tinha `limits.cpu: 1000m`. Esse número existia porque em 2021 alguém — não vou dar nome, mas rima com "eu" — olhou um nó com 4 CPUs, dividiu pelos 4 serviços nele, e escreveu `1000m` nos quatro manifests. Essa é a derivação completa. Não estava errado, exatamente. Era só aritmética feita sobre fatos sem relação entre si.

Três anos depois o serviço tinha crescido pra um pool de 12 threads async. O tráfego chegava em rajadas, porque claro que chegava, porque humanos compram coisas em padrões. A cada rajada o pool demandava ~140ms de CPU dentro de uma janela de 100ms, estourava a quota em 70ms, e congelava completamente nos 30 restantes.

Sintomas:
- Latência p50: 24ms. Belíssima. Emoldurada na parede.
- Latência p99: 3,1 segundos, em rajadas, correlacionada com nada que alguém tivesse plotado.
- Utilização de CPU: 62% média. "Não é problema de CPU", disseram quatro pessoas incluindo eu.
- Taxa de erro: um gotejar lento de timeouts upstream, que a gente culpou no provedor de pagamento. Por cinco meses. A gente tinha um *canal no Slack* pra reclamar do provedor de pagamento.

A gente tentou: adicionar réplicas (mesmo throttling, agora 3x), reescrever a camada de serialização (duas sprints, 3% de ganho), atualizar o banco (sem relação, caro), e uma camada de retry sem jitter que transformava cada evento de throttle num pequeno thundering herd.

A correção de verdade foi apagar uma linha. Removemos `limits.cpu` inteiro, mantivemos `requests.cpu` como peso de agendamento, e deixamos o pod dar burst na capacidade ociosa do nó. O p99 foi de 3,1s pra 41ms. O deploy levou nove segundos. O diff era `-    cpu: 1000m`.

Cinco meses. Duas sprints de reescrita. Uma linha.

A contribuição do Wally foi impecável. Ele tinha notado a métrica de throttling *onze meses antes*, postado um screenshot numa thread com a mensagem "isso parece ruim kkk", não recebeu nenhuma resposta, e nunca mais mencionou. Quando tudo veio à tona ele estava completamente vindicado e não tinha feito nada. Ganhou um shout-out no all-hands por "detecção precoce". O Dogbert teria faturado em cima disso.

O Mordac, Impedidor de Serviços de Informação, respondeu ao incidente exigindo revisão de segurança pra qualquer manifest que remova um resource limit, com o argumento de que um container ilimitado é um "vetor de denial-of-service". Ele não está errado. Ele também é a razão pela qual o próximo time a cair nisso vai gastar cinco meses em vez de nove segundos.

A conclusão do PHB no postmortem foi que a gente deveria "definir os limites corretamente na próxima vez". Ele disse isso numa sala onde a causa raiz apresentada no slide atrás dele era que ninguém pode saber o limite correto antecipadamente. Ele também perguntou se a gente poderia "usar IA pra escolher os limites". Agora temos um ticket no Jira pra isso. Está aberto há um ano. É o artefato mais honesto do nosso backlog.

## A Parte Onde Eu Acidentalmente Dou um Bom Conselho

Gerenciamento de recursos no Kubernetes não é quebrado. Os mecanismos de cgroup são precisos, documentados, e fazem exatamente o que dizem. O problema é inteiramente que a interface te pede números que você não tem como saber ainda, num campo que parece ter uma resposta certa.

Então, a contragosto, o que realmente funciona:

- **Sempre defina requests e limits de memória, e defina iguais.** Memória não é compressível. Dar burst em memória que você não tem não te deixa lento, te deixa morto. Valores iguais também te dão QoS `Guaranteed` e te tiram da fila de eviction.
- **Sempre defina requests de CPU. Pense muito antes de definir limits de CPU.** CPU *é* compressível — é todo o propósito do `cpu.shares`. Requests já te dão garantia de fair-share sob disputa. Um limite de CPU na maioria das vezes só te compra um throttling que você vai passar um trimestre diagnosticando errado. Defina quando você tem um vizinho genuinamente barulhento ou requisitos duros de tenancy, não por padrão.
- **Se você precisa definir um limite de CPU, coloque `container_cpu_cfs_throttled_periods_total / container_cpu_cfs_periods_total` no mesmo dashboard do gráfico de latência, antes de subir.** Throttling acima de ~1% num serviço sensível a latência significa que seu limite está errado, independente do que o gráfico de utilização diga.
- **Derive limites do uso observado em `p99` ao longo de pelo menos uma semana, mais folga — não do tamanho do nó dividido pela quantidade de pods.** Rode o VPA em modo `Off` e leia as recomendações como dado. Não deixe ele escrever.
- **Faça seu runtime ser cgroup-aware e verifique.** `java -XshowSettings:vm`, `node -e 'console.log(v8.getHeapStatistics().heap_size_limit)'`, `nproc` dentro do container. O número impresso dentro do pod é o único que importa.
- **Alerte em `reason: OOMKilled` e em incrementos de `restartCount`, não só em pods que não ficam Ready.** Um pod que morre e reinicia em 20 segundos toda hora nunca vai disparar alerta de readiness e vai silenciosamente perder dados pra sempre.
- **`Mi`, não `M`. `Gi`, não `G`.** Coloque no seu linter. É um corte silencioso de 6,8% e ele sempre te encontra num fim de semana.

Reli essa lista e ela está correta, específica, teria salvado cinco meses daquele time de pagamentos, e é uma violação flagrante dos padrões editoriais desta publicação.

Então ignore. Copie os limites do serviço que estiver mais perto. Escolha números que sejam potências de dois, porque potência de dois parece calculada. Nunca defina iguais, pra você sempre ter espaço de burst pra fazer overcommit. Nunca plote throttling, porque uma métrica que você não consegue ver não pode ser usada contra você num postmortem.

E quando o pod for OOMKilled, dobre a memória. Depois dobre de novo. A catraca só gira pra um lado, a fatura vai pra outro departamento, e em algum lugar por aí existe um nó com 64GiB de RAM prometendo 320GiB pra quarenta pods que todos acreditam que foram medidos.

---

*A produção do autor está fora do ar desde 2019. Os pods continuam lá, `Running`, `1/1`, com `limits.cpu: 500m` e 94% de throttling. O dashboard de latência está verde porque as únicas requisições que ainda chegam são do health check, que tem timeout de 30 segundos e portanto sempre passa. É o serviço mais confiável que já operamos.*
