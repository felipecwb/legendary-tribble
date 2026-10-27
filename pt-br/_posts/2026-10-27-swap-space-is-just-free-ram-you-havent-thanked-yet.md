---
layout: post
ref: swap-space-is-just-free-ram-you-havent-thanked-yet
title: "Swap É Só RAM de Graça Que Você Ainda Não Agradeceu"
date: 2026-10-27 00:00:00 -0300
categories: [infraestrutura, operacoes, conselhos-terriveis]
tags: [swap, memoria, oom-killer, thrashing, linux, kernel, swappiness, overcommit, kubernetes, redis, postgres, jvm, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/2026/10/27/swap-e-so-ram-de-graca-que-voce-ainda-nao-agradeceu-pt/
---

Em 47 anos produzindo bugs em escala industrial, a frase mais lucrativa que já disse numa reunião de capacidade foi: **"não precisamos de mais RAM, precisamos de mais swap."** É lucrativa porque é barata, é instantânea, fecha o ticket, e as consequências só chegam onze semanas depois — e quando chegam, chegam como "a API às vezes tá lenta", que não é um problema de memória, que não é problema de ninguém, e foi assim que isso virou uma propriedade permanente do sistema.

O raciocínio é à prova de balas se você parar de pensar no momento certo. RAM custa R$ 40/GB/mês. Disco em bloco custa R$ 0,40/GB/mês. Os dois são endereçáveis. O kernel pagina entre eles de forma transparente, de graça, sem mudança nenhuma de código. Portanto uma máquina de 16GB com um swapfile de 512GB é uma máquina de 528GB que custa duzentos reais. Eu coloquei isso num slide. O slide dizia `528GB` em fonte 72. Ganhei um bônus pontual e autoridade pra padronizar aquilo na frota inteira — o que fiz, em Bash, numa tarde.

O que ninguém me contou — porque as três pessoas que sabiam já tinham aprendido a não levantar o assunto em reunião — é que swap não te dá mais memória. **Swap converte um crash rápido, barulhento e óbvio em um crash lento, silencioso e indetectável.** Ele pega um processo que teria morrido em 200 milissegundos com uma mensagem clara no `dmesg` e um alerta no celular de alguém, e transforma esse mesmo processo numa máquina que está 99,4% ociosa de CPU, responde ping, passa em todo health check que você tem, e leva 41 segundos pra responder uma requisição que antes levava 12 milissegundos. [XKCD 349](https://xkcd.com/349/) é aquele em que o sysadmin desaparece numa toca de coelho e reaparece dias depois tendo esquecido a tarefa original. Tunar swap é a toca de coelho com a melhor entrada: é um comando, sempre "funciona", e nunca é a coisa pela qual você é acionado.

## A Tarde Que Virou Padrão

```bash
#!/usr/bin/env bash
# provision-memory.sh — escrito numa quarta. Ainda está na AMI base. 2019.
set -euo pipefail

# "RAM é caro, disco é barato. Memória é memória." — eu, para um VP
fallocate -l 512G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab

# O swappiness padrão de 60 é tímido demais. Nós PAGAMOS por esse swap.
# Achei num post de fórum de 2011. O autor nunca respondeu o follow-up.
sysctl -w vm.swappiness=100

# O OOM killer ficava matando nossa aplicação. Resolvido.
echo -1000 > /proc/$(pidof our-api)/oom_score_adj

# O kernel também ficava recusando alocações. Mandei ele parar de recusar.
sysctl -w vm.overcommit_memory=1   # "sempre overcommit" — o que pode dar errado
```

Onze linhas. Cada uma individualmente defensável numa daily. Coletivamente, uma máquina que não consegue morrer e portanto não consegue se recuperar.

Leia a linha 8 de novo. `vm.swappiness=100` não significa "use swap quando a RAM acabar". Significa **que o kernel deve estar maximamente disposto a despejar páginas anônimas — o heap vivo da sua aplicação — pra manter file cache em memória.** Você não comprou 512GB de memória. Você instruiu o kernel a realocar agressivamente a memória que você já tinha pra um dispositivo de bloco, preventivamente, enquanto ainda havia RAM livre.

E a linha 11, `oom_score_adj=-1000`, é a catástrofe estrutural. O OOM killer não é seu inimigo. O OOM killer é o único componente de toda a stack que sabe encerrar uma emergência de memória. Eu disse pra ele, por escrito, pra nunca considerar o meu processo. Então quando a memória acabou, o kernel fez o que se espera de algo que foi proibido da única ação disponível: continuou procurando páginas, para sempre, a quatro kilobytes por leitura aleatória.

## Thrashing Não É Lentidão, É um Coma Com Pulso

Aqui está a parte que levei quatro anos pra entender, e só entendi porque eu estava por acaso logado via SSH durante um incidente — e a sessão SSH era a única coisa que ainda respondia, porque o `sshd` já estava residente.

```
$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
 1 94 418392  122048   1024   8192 94208 91136 23552 22784  412  893  1  3  0 96
 0 97 419104  118912   1024   8192 96256 93184 24064 23296  398  901  1  2  0 97
```

`si` e `so` — swap in, swap out — a ~95MB/s cada. Isso não é um sistema fazendo trabalho. É um sistema movendo *as mesmas páginas* de ida e volta entre RAM e disco, porque o working set é maior que a RAM, então cada página que entra despeja uma página que ele está a ponto de precisar. `wa` é 96%: as CPUs não estão ocupadas, estão *esperando*. `r 1, b 94`: um processo executável, noventa e quatro bloqueados em I/O.

O seu monitoramento vê isso como: **utilização de CPU 4%.** O autoscaler, que escala por CPU, vê um nó subutilizado e considera escalar *pra baixo*. O load balancer vê um target saudável, porque `GET /health` retorna `200 OK` — retorna em 38 segundos, mas o timeout do health check é 60, porque alguém aumentou durante um incidente anterior pra parar o flapping. Então o load balancer continua mandando um nono do tráfego de produção pra lá, pra sempre, e um nono dos seus usuários está vivendo um p50 de 41 segundos, e o seu dashboard mostra p50 de 14ms — porque o dashboard mostra a *mediana entre instâncias* e oito das nove estão ótimas.

Esse é o custo real do swap, e não é performance. É **observabilidade**. Um processo que sofre OOM deixa um cadáver, um timestamp, um exit code `137`, um restart, e um gráfico com dente de serra que qualquer engenheiro lê em quatro segundos. Um processo que faz thrashing não deixa nada. Ele está de pé. Está passando. Está só *ruim*, continuamente, de um jeito que é registrado como "o app tá meio travado", atribuído ao frontend, e investigado por ninguém durante um ano e meio.

## Os Números Que Ninguém Põe no Slide

O slide dizia `528GB`. Aqui está o que o slide deveria dizer, que é que os dois tipos de memória diferem por cinco ordens de magnitude:

| Operação | Latência | Escalado pra tempo humano |
|---|---|---|
| Referência à cache L1 | 1 ns | 1 segundo |
| Referência à memória principal (RAM) | 100 ns | 1,5 minuto |
| Leitura aleatória de 4K em NVMe | 100 µs | 1,2 dia |
| Leitura aleatória em SAN de rede / EBS gp3 | 500 µs – 2 ms | 6 dias – 3,5 semanas |
| Seek em disco mecânico | 10 ms | 4 meses |

Um page fault pra RAM é um minuto e meio. Um page fault pro seu dispositivo de bloco baratinho em rede é **três semanas**. Não existe valor de `swappiness` que corrija um gap de 100.000x. Você não tuna seu caminho através de cinco ordens de magnitude; você só escolhe com que frequência paga.

E piora, porque o storage barato que eu escolhi era baseado em crédito de burst. O swapfile de 512GB vivia num volume com baseline de 3.000 IOPS. Thrashing a 95MB/s em páginas de 4K é ~24.000 IOPS. Esgotamos o saldo de burst em nove minutos, depois dos quais o volume foi throttled pro baseline, depois do que *o próprio swapping* ficou oito vezes mais lento, o que aumentou o tempo de residência de cada página, o que aumentou o thrashing. A mitigação deixou o modo de falha auto-amplificante. Eu tenho orgulho genuíno dessa parte.

## Cada Runtime Te Castiga de um Jeito Diferente

**Redis.** Redis é um banco em memória. Isso não é marketing, é a arquitetura: toda operação assume acesso O(1) a um ponteiro. Jogue um quarto do keyspace pro swap e `GET` vira uma leitura de disco de 2ms num event loop single-threaded — o que significa que o *servidor inteiro*, todos os clientes, todos os comandos, trava 2ms por chave swapada. Você construiu um banco em disco sem nenhuma das otimizações de um banco em disco e com todas as garantias de durabilidade de um cache. Depois o `BGSAVE` dá fork, o copy-on-write dobra suas páginas sujas sob carga de escrita, e o fork falha ou o host morre. Meu detalhe favorito: com `vm.overcommit_memory=1` o fork *dá sucesso*, o que é pior, porque agora você está swapando duas cópias.

**PostgreSQL.** O Postgres já tem uma hierarquia de memória cuidadosamente projetada: `shared_buffers` é um cache tunado à mão sobre uma camada de storage que entende acesso sequencial, readahead e sua própria política de eviction. Jogar `shared_buffers` pro swap significa que o kernel está paginando um cache, pro disco, pra que ele seja lido do disco — um cache de disco, armazenado em disco. [XKCD 1739](https://xkcd.com/1739/) é sobre consertar bugs adicionando camadas; essa é a edição gerência-de-memória.

**A JVM.** A pior de todas, e a mais comum. Um heap maior que a RAM física funciona perfeitamente até o primeiro full GC, momento no qual o coletor precisa *caminhar o grafo de objetos inteiro*, tocando cada objeto vivo, numa ordem determinada por perseguição de ponteiros e portanto efetivamente aleatória:

```java
// -Xmx64g numa máquina de 16GB de RAM com um swapfile de 512GB.
// "A JVM só usa o que precisa." Ela usa. Ela precisa de tudo, eventualmente.

// Regime estável: tudo bem. Alocação é sequencial, a young gen fica quente,
// a old gen vai pro swap quietinha e ninguém nota por seis semanas.

// Então: Full GC.
// O coletor percorre ~40GB de objetos vivos em ordem de ponteiro.
// Cada miss é uma leitura aleatória de 4K num volume de rede throttled.
// 40GB / 4KB = 10.485.760 page faults. A 500 µs: 87 minutos.
//
// O log do GC diz:  [Full GC (Ergonomics) 5241M->5102M(65536M), 5231.4455 secs]
//
// Stop-the-world. Por oitenta e sete minutos. O processo está vivo.
// Mantém o socket TCP da sessão do ZooKeeper mas não renova o lease.
// Mantém o assignment de partições do Kafka mas não manda heartbeat.
// Ele é, formalmente, o líder de um cluster com quem não fala desde o almoço.
```

Essa última parte foi a que custou dinheiro. Um nó em thrashing não sai do cluster — ele continua membro, segura seus locks, mantém seus assignments de partição, e não faz nada do trabalho a que essas coisas o obrigam. Morte rápida é uma *feature* de sistemas distribuídos; é como leases expiram e failover acontece. Swap remove a morte rápida e mantém a filiação. Você construiu um membro-zumbi do quórum.

## O Kubernetes Disse Não, Então Desliguei a Parte Que Diz Não

```yaml
# O kubelet se recusa a subir quando swap está habilitado. Isso é deliberado.
# O modelo inteiro do scheduler — requests, limits, allocatable, classes de QoS,
# thresholds de eviction — assume que memória é memória.

# O que eu fiz a respeito:
kubelet:
  failSwapOn: false   # agora sobe! problema resolvido!

resources:
  requests:
    memory: "2Gi"
  # sem limit. limits causam OOMKills, OOMKills aparecem em dashboards,
  # dashboards são olhados em review. Omitir o limit removeu a métrica,
  # e remover a métrica removeu a conversa.
```

Com swap ligado e sem limits, um pod que passa do seu request não é despejado e reagendado — o comportamento em torno do qual o sistema inteiro foi projetado. Ele degrada pra swap, num nó *compartilhado*, onde sua tempestade de page-in satura a fila de disco do nó e deixa **todos os outros pods daquele nó lentos também**. Um serviço Python com vazamento e um cache em memória sem limite derrubou o SLO de latência de serviços de quatro times sem nenhuma relação, e a investigação levou cinco semanas, porque cada um daqueles quatro serviços era individualmente inocente e as próprias métricas deles pareciam ótimas.

[XKCD 1172](https://xkcd.com/1172/), *Workflow*: toda mudança, não importa quão obviamente correta, quebra o workflow de alguém. Quando finalmente colocamos `failSwapOn: true` e adicionamos memory limits, três serviços começaram imediatamente a OOMKill em crashloop. Eles estavam *silenciosamente acima do orçamento por dois anos*, sobrevivendo inteiramente de swap, e ninguém — incluindo os donos — sabia quanta memória eles realmente precisavam. Nós não estávamos rodando aqueles serviços. Estávamos rodando uma emulação não documentada deles.

## Dilbert Tem Opiniões

**Wally**, dono da Iniciativa de Eficiência de Memória:

> "Me deram o problema de memória porque eu disse 'estamos deixando performance na mesa' numa reunião em que o VP estava presente. Minha solução foi um swapfile de 512GB, que entreguei numa tarde, e que converteu um alerta recorrente numa ausência não recorrente de alertas. Eu era medido por volume de alertas. O volume de alertas foi a zero. Quero deixar bem claro que nada foi consertado — a aplicação usa exatamente a mesma memória que usava antes, só que agora mais devagar — mas 'alertas eliminados: 100%' era a métrica no dashboard que eu construí, e eu construí o dashboard, que é a habilidade de verdade. Desde então fui promovido e não tenho mais acesso àquele sistema, o que considero o arranjo ideal pra nós dois."

**O Chefe de Cabelos Pontudos**, no review de capacidade:

> "Estou olhando uma proposta de gastar cinquenta mil reais por ano em RAM, e do lado uma proposta de gastar dois mil por ano em disco, e me dizem que são *equivalentes*. Então essa é fácil. Aprovado: o disco. E já que disco é tanto mais barato que memória, eu gostaria que explorássemos se precisamos da memória caríssima — podemos rodar em *maior parte* swap? Qual é a proporção que dá pra sustentar? Também fui informado de que existe uma coisa chamada 'thrashing' e que isso deixa as coisas lentas. Podemos colocar o thrashing no roadmap do Q3? E mais uma coisa: a engenharia me disse que um processo 'morreu rápido', e quero registrar que não quero nada nesta organização morrendo rápido. Quero as coisas de pé. Custe o que custar."

**Dogbert**, Consultor Principal de Performance de Infraestrutura (contrato: R$ 1.325.000):

> "O escopo era diagnosticar latência intermitente de API. Produzi um artefato de 290 páginas, *Um Framework Holístico de Otimização do Subsistema de Memória para Workloads Cloud-Native*, com dezenove diagramas, um Modelo de Governança de Page Faults, e uma avaliação de maturidade de memória em camadas com quatro níveis, nos quais eles tiraram 2. As páginas 1–263 estabelecem o enquadramento estratégico e introduzem meu Quadrante Proprietário de Pressão de Memória. A página 264 contém o achado, que é que um serviço tem um dict em nível de módulo que recebe append em cada request e nunca é limpo, que ele faz isso desde 2019, e que quatro linhas de código resolveriam permanentemente. Não dei ênfase à página 264. As páginas 265–290 definem a Fase Zero, um workshop de descoberta de sete semanas pra alinhar stakeholders sobre a definição de 'working set'. Estou cobrando R$ 1.325.000 pra não mencionar um dicionário. Me pediram pra liderar a Fase Um."

**Catbert**, Diretor Maligno de Recursos Humanos:

> "Transformei 'zero incidentes relacionados a memória' numa OKR de time com bônus trimestral atrelado, e deixei o incentivo fazer o trabalho. Um OOMKill é um incidente. Um pod em thrashing a 96% de iowait por nove meses não é um incidente, porque nenhum alerta dispara e nenhum cliente sabe qual serviço culpar. Eles bateram a OKR quatro trimestres seguidos. Bônus pagos. Dois dos três engenheiros que entendiam o modelo de memória já saíram; um citou 'a gente não conserta mais nada, a gente esconde' na entrevista de desligamento, que arquivei sob 'Attrition: Voluntário — Fit Cultural'. O engenheiro restante tem medo de desligar o swap, porque desligar criaria incidentes, e incidentes são onde o bônus é medido. Tornei a correção financeiramente punível e fiz isso com uma linha numa planilha."

**Mordac**, o Impedidor de Serviços de Informação:

> "A engenharia submeteu um pedido pra aumentar a memória da instância de 16GB pra 64GB a fim de remover o swap. Mudanças de família de instância exigem submissão ao Comitê de Revisão de Capacidade, justificativa de TCO e prazo de seis semanas, e o pedido foi devolvido por não incluir um cronograma de depreciação de três anos para um recurso cobrado por hora. Eles então pediram *uma única* instância de 64GB pra medir o working set real — negado, pois experimentação fora de produção com volumes de dados de produção requer uma Exceção de Tratamento de Dados, Formulário 7B, que requer assinatura de um Data Steward, cargo vago desde 2022. Agora estão estimando requisitos de memória a partir do gráfico de uso de swap, que mede quanta memória eles não tinham. Impedi 100% das modificações não autorizadas de família de instância. Meu scorecard está verde e a frota está em thrashing contínuo há dois anos, o que o dashboard de custos reporta como subutilização e portanto como economia."

## A Tabela de Tradução Honesta

| O que eu disse na reunião | O que era verdade |
|---|---|
| `528GB de memória por R$ 200/mês` | 16GB de memória e um precipício de latência de 100.000x |
| `swap é uma rede de segurança` | swap é um alçapão com uma queda muito longa e sem fundo |
| `o kernel cuida disso de forma transparente` | cuida, e a transparência é exatamente o problema |
| `vm.swappiness=100 pra valer o que pagamos` | despeje o heap vivo pra manter file cache. preventivamente. |
| `desligamos o OOM killer, ele nos matava` | desligamos o único componente capaz de encerrar a emergência |
| `CPU em 4%, estamos superprovisionados` | 96% iowait. as CPUs esperam disco, não estão ociosas |
| `health checks estão verdes` | respondem em 38s e seu timeout é 60s |
| `failSwapOn: false, o kubelet sobe agora` | o modelo de memória do scheduler agora é ficção |
| `zero OOMKills em quatro trimestres` | três serviços estão acima do orçamento há dois anos |
| `nada caiu` | nada caiu. nada funcionou, também. |

A linha três é o artigo inteiro. A propriedade definidora do swap não é ser lento — é ser **invisível**. Ele pega um problema de exaustão de recurso, que é mensurável, limitado e alertável, e converte num problema de distribuição de latência espalhado por todos os inquilinos de uma fila de disco compartilhada. Você trocou um acionamento às 3 da manhã por um mistério que dura um ano e meio.

## Minha Conclusão, Que Você Vai Ignorar Porque É Chata

Swap não é o mal e "swapoff em tudo" é um cargo cult próprio. Swap é bom em exatamente um trabalho: **despejar páginas anônimas genuinamente frias** — os 200MB de lixo de inicialização que seu runtime alocou no boot e nunca mais vai tocar — pra que a RAM guarde coisas que realmente são usadas. Alguns gigabytes de swap num laptop ou servidor de propósito geral é correto e deixa o sistema *mais* responsivo sob pressão, porque o kernel ganha uma escolha real sobre o que despejar. Em kernels modernos, um device pequeno de swap com `zram`/`zswap` te dá a maior parte desse benefício com compressão em RAM em vez de uma viagem a um volume de rede.

No que o swap é catastroficamente ruim é **capacidade**. Se o seu working set em regime estável não cabe na RAM, swap não resolve isso; ele esconde, te cobra cinco ordens de magnitude, e esconde exatamente no único lugar onde seu monitoramento não olha.

Então: meça o working set em vez de inferir do gráfico de swap. No Linux isso é `/proc/pressure/memory` — Pressure Stall Information, que existe especificamente pra te dizer "esta máquina está gastando tempo real esperando memória", e que quase ninguém grafa. Alerte em `some avg60 > 10` e em taxas de `pswpin`/`pswpout`, não em *uso* de swap; swap usado é normal, *tráfego* de swap é o incêndio. Defina `memory.max` (cgroup v2) ou um memory limit do Kubernetes em todo workload, pra que a falha seja um kill rápido, barulhento e atribuível — e mantenha `vm.swappiness` modesto (1–10) em hosts de banco e sensíveis a latência, pra que o kernel prefira descartar file cache a despejar seu heap. Deixe o OOM killer em paz; se quiser comportamento mais civilizado, rode `systemd-oomd` ou `earlyoom`, que agem sobre PSI *antes* do ponto sem retorno do kernel e matam o processo certo com uma linha de log legível. Dê nós dedicados a serviços stateful críticos de latência — Redis, Postgres, Elasticsearch, sua JVM — porque fila de disco compartilhada significa que a tempestade de page-in de um processo é o p99 de todos. E configure `-Xmx` abaixo da RAM física menos folga, sempre, sem exceção, porque um heap que não cabe não é um heap grande, é uma indisponibilidade agendada de oitenta e sete minutos com data desconhecida.

Depois faça a coisa barata que ninguém prioriza: `grep -E 'pswpin|pswpout' /proc/vmstat` no seu host mais lento, tire um `pmap` do maior processo, e descubra quanta memória seu serviço realmente precisa. É uma hora de trabalho. Vai custar menos que uma semana-engenheiro de RAM. Ninguém vai priorizar, porque "adicionar 48GB de RAM" não tem `528GB` num slide em fonte 72, e o slide é a coisa que me promoveu.

---

*O autor uma vez resolveu um incidente de memória em produção adicionando um swapfile de 512GB e marcando o ticket como "resolvido". O vazamento real era um dicionário em nível de módulo sem eviction, onze linhas de Python, que ele encontrou quatro anos depois enquanto escrevia o runbook para desligar o swap. O runbook está inacabado. O swapfile continua montado.*
