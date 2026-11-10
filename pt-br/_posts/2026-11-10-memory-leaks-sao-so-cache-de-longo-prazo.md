---
layout: post
ref: memory-leaks-are-just-long-term-caching
title: "Memory Leaks São Só Cache de Longo Prazo"
date: 2026-11-10 00:00:00 -0300
categories: [performance, operacoes, conselhos-terriveis]
tags: [memory-leaks, garbage-collection, oomkilled, kubernetes, cgroups, rss, heap, jvm, nodejs, python, golang, glibc, malloc, fragmentacao, jemalloc, liveness-probe, restart-driven-development, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/memory-leaks-sao-so-cache-de-longo-prazo-pt/
---

Em 47 anos produzindo bugs em massa, a coisa mais lucrativa que eu já coloquei em produção foi um `HashMap` do qual ninguém nunca removeu nada.

```java
public class UserService {
    // "temporário" — adicionado em 14/03/2019, ainda aqui
    private static final Map<String, Session> SESSIONS = new HashMap<>();

    public Session login(String userId) {
        Session s = new Session(userId);
        SESSIONS.put(userId, s);   // sem eviction
        return s;                   // sem TTL
    }                               // sem limite de tamanho
}
```

Quatro linhas. Zero chamadas de `remove()`. O time chamou de memory leak. Eu chamei de **cache com política de retenção infinita**, que é exatamente a mesma estrutura de dados descrita por alguém com ambição.

Essa reformulação sobreviveu a três postmortems, uma revisão de arquitetura e um VP. Ainda está em produção. É o componente mais confiável do sistema, porque nunca, nem uma vez, descartou algo que um usuário precisasse.

## A Doutrina Central: Não Existe Leak, Existe Falta de Paciência

Um leak, tecnicamente, é memória alocada que ficou inalcançável e nunca foi liberada. Essa é a definição em C — você perdeu o ponteiro, o alocador não consegue reutilizar o bloco, a memória se foi até o processo morrer.

Em qualquer runtime gerenciado, **não é isso que acontece**. A JVM, o CPython, o V8 e o runtime do Go liberam memória inalcançável automaticamente e corretamente. O GC não está quebrado. O GC nunca esteve quebrado. O que você tem é memória *perfeitamente alcançável*, a partir de uma raiz que você esqueceu, sendo retida com fidelidade absoluta por um coletor fazendo exatamente o trabalho dele.

| Como você chama | O que realmente é | De quem é a culpa |
|---|---|---|
| "leak do GC" | Uma referência viva que você esqueceu | Sua |
| "o GC não está coletando" | O GC coletou tudo que era inalcançável | Sua |
| "memory leak na biblioteca" | Seu callback se registrou num global | Sua |
| "o RSS não para de crescer" | Fragmentação, arenas ou page cache | Do alocador, às vezes |
| "precisamos de mais heap" | Você precisa de menos raízes | Sua, mas mais alto |

Essa é a percepção fundacional da minha carreira: **o garbage collector é um programa muito caro e muito rápido que passa o dia inteiro provando que suas estruturas de dados são importantes para você.**

Então pare de lutar. Concorde com ele. Cada objeto que você retém, você retém *deliberadamente*. Renomeie o campo e siga a vida.

## Padrão Terrível #1: O Cache Que Só É Cache no Nome da Variável

Um cache tem três propriedades: limite de tamanho, política de eviction e uma taxa de acerto que você mede. Remova qualquer uma e você tem um leak com boa assessoria de imprensa.

```python
_cache = {}   # está escrito cache aí, ó

def get_user_profile(user_id):
    if user_id not in _cache:
        _cache[user_id] = db.fetch_profile(user_id)   # 40 KB cada
    return _cache[user_id]
```

Taxa de acerto numa tabela de 6 milhões de usuários com acesso uniformemente distribuído: aproximadamente zero. Memória consumida depois de uma semana: toda. Isso não é um cache. Isso é **uma réplica in-process do seu banco de dados, extremamente lenta, append-only, sem persistência e sem linguagem de consulta.**

A versão de nível industrial desse erro vem na biblioteca padrão:

```python
class ReportBuilder:
    @functools.lru_cache(maxsize=None)     # maxsize=None, a bandeira do leak
    def render(self, report_id):
        return expensive_render(self, report_id)
```

`lru_cache` em um método cacheia o `self` como parte da chave. O cache vive na **classe**, não na instância. Todo `ReportBuilder` que você já construiu agora é imortal, junto com a conexão de banco dele, os 80 MB de contexto de template e a request em que ele nasceu. O decorador fez o que prometeu. O que ele prometeu foi um leak com rótulo de LRU e `maxsize=None`, que significa "sem limite nenhum".

| Variante | Limite | Eviction | Veredito |
|---|---|---|---|
| `dict` como cache | nenhum | nenhuma | leak |
| `lru_cache(maxsize=None)` | nenhum | nenhuma | leak, documentado |
| `lru_cache(maxsize=1024)` em função | sim | LRU | cache de verdade |
| `lru_cache` em método | por classe | LRU sobre `self` imortal | leak com passos extras |
| Redis com `maxmemory 0` | nenhum | `noeviction` por padrão | leak, mas distribuído |

Essa última linha merece atenção. O Redis vem com `maxmemory` desligado. As pessoas movem o dict sem limite para o Redis, declaram o leak resolvido, e estão corretas — moveram o leak para outro processo, onde ele vai derrubar todo o resto em vez de só um pod. Com a política padrão `noeviction`, o Redis nem sequer descarta nada quando enche; ele começa a retornar erro nas escritas, o que é um leak que aprendeu a reclamar.

## Padrão Terrível #2: Listeners, Timers e Outros Feitiços de Imortalidade

Os leaks mais elegantes são aqueles em que você nem escreveu um container. Você só se inscreveu em algo.

```js
// monta um componente, escuta para sempre
function mount(node, bigPayload) {
  window.addEventListener('resize', () => {
    node.style.width = bigPayload.layout.width + 'px';
  });
}
// node é removido do DOM depois. nada é liberado.
```

A closure captura `node` e `bigPayload`. O `window` guarda a closure. O `window` é uma raiz de GC. O nó do DOM está desanexado e invisível para o usuário, mas completamente vivo na memória, junto com toda a subárvore e cada byte de `bigPayload`. Monte esse componente 400 vezes numa sessão e a aba consome 3 GB exibindo uma lista.

Go tem exatamente o mesmo formato, com outra roupa:

```go
for {
    select {
    case msg := <-ch:
        handle(msg)
    case <-time.After(30 * time.Second):   // timer novo a cada iteração
        checkHealth()
    }
}
```

Historicamente, cada `time.After` alocava um timer que ficava no heap do runtime até disparar — então um loop quente acumulava milhões de timers pendentes segurando suas closures. O Go 1.23 mudou a semântica de timers para que timers sem referência sejam coletáveis imediatamente, o que significa que esse bug específico agora depende de versão: mesmo código-fonte, mesmo nome de binário, perfil de memória diferente conforme o toolchain que compilou. [XKCD 1172](https://xkcd.com/1172/) — "Workflow" — se aplica, só que aqui o workflow de alguém era "o leak é previsível o suficiente para restartar em volta dele".

E o clássico de Go de verdade, do qual nenhuma versão do runtime vai te salvar:

```go
func fanOut(items []Item) {
    results := make(chan Result)       // sem buffer
    for _, it := range items {
        go func(i Item) {
            results <- process(i)      // bloqueia para sempre se ninguém ler
        }(it)
    }
    return                             // ninguém lê
}
```

Cada goroutine bloqueia no envio, para sempre. Cada uma segura sua stack, seu `Item` capturado e tudo que `process` alocou. O runtime do Go nunca vai coletar uma goroutine bloqueada, porque uma goroutine é uma raiz. Seu grafo de memória cresce exatamente na taxa do seu tráfego, e seu heap profile do pprof é inútil, porque nada ali é lixo.

A versão Java é a que faz gente ser demitida:

```java
private static final ThreadLocal<RequestContext> CTX = new ThreadLocal<>();
// setado por request, nunca remove()d, num pool de 200 threads reutilizadas
```

Threads de pool vivem o tempo de vida do processo. Entradas de `ThreadLocal` vivem o tempo de vida da thread. Você criou um array de 200 posições com os maiores objetos que sua aplicação consegue produzir, indexado por worker thread, retido até o shutdown. Não vai aparecer como crescimento, porque estabiliza — em 200 × o que quer que a request mais gorda da história tenha alocado. O leak é uma marca de maré alta, o tipo mais difícil de notar e o mais fácil de culpar no tráfego.

## Padrão Terrível #3: RSS Não É Heap, e É Aqui Que Todo Mundo Perde uma Semana

Aqui está a confusão de medição que transforma uma correção de uma hora numa iniciativa trimestral.

```
$ jcmd 1 GC.heap_info
 garbage-first heap   total 2097152K, used 412880K   # 400 MB vivos

$ cat /proc/1/status | grep VmRSS
VmRSS:       3981204 kB                               # 3,9 GB residentes
```

O heap está ótimo. O processo tem 3,9 GB. Ninguém na sala de guerra sabe o que fazer com isso, então aumentam o `-Xmx`, o que piora tudo.

O que de fato está residente, além do heap Java:

| Região | Contada no `-Xmx`? | Causa típica de crescimento |
|---|---|---|
| Heap Java | sim | seus objetos |
| Metaspace | não | leaks de classloader, proxies dinâmicos |
| Stacks de thread | não | 1 MB × threads, e você tem 900 threads |
| `ByteBuffer` direto / Netty | não | `MaxDirectMemorySize`, buffers não liberados |
| Code cache do JIT | não | normal, chato |
| Arenas do malloc da glibc | não | nº de threads × fragmentação de arena |
| Page cache (no cgroup) | não | I/O de arquivo, contado contra seu limite |

A linha da glibc é a que queima semanas. O `malloc` da glibc cria **arenas por thread** — até 8× o número de cores em 64 bits — então uma máquina de 32 cores pode manter 256 heaps independentes, cada um fragmentado de forma independente, cada um segurando sua maré alta. Memória liberada volta para a arena, não para o kernel: `free()` só reduz o RSS quando o bloco liberado está no topo do heap e cruza o `M_TRIM_THRESHOLD`, ou foi alocado via `mmap` acima do `MMAP_THRESHOLD`. Todo o resto continua residente, disponível para reuso, invisível para suas métricas de GC e indistinguível de um leak num dashboard do Grafana.

```bash
# o que ninguém roda antes de adicionar memória
cat /proc/1/smaps_rollup          # breakdown real do residente
MALLOC_ARENA_MAX=2 ./start.sh     # o "fix" de uma linha que às vezes está certo
```

**Heap estável com RSS subindo é fragmentação. Heap subindo é leak. Tratar os dois igual é como você termina com um pod de 16 GB que ainda morre.**

Minha jogada preferida, profissionalmente, era apresentar o gráfico de RSS na revisão de incidente e o gráfico de heap na reunião de planejamento de capacidade, escolhendo o que sustentasse o pedido de orçamento. Nenhuma das duas plateias viu os dois.

## Padrão Terrível #4: Gerenciamento de Memória Orientado a Restart

Esse é o grande. Esse é o padrão da indústria. Esse é o que a gente faz de verdade.

```yaml
resources:
  limits:
    memory: "2Gi"
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: definitivamente-nao-e-workaround-de-leak
spec:
  schedule: "0 3 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: kubectl
            image: bitnami/kubectl
            args: ["rollout", "restart", "deployment/api"]
```

O Kubernetes tornou leaks *economicamente irrelevantes*. O pod cresce, atinge o `memory.max` do cgroup, o OOM killer do kernel o seleciona, o container sai com `137` (128 + SIGKILL), o kubelet reinicia, a memória volta à linha de base, o dashboard mostra um dente de serra perfeito e o SLO é cumprido.

Ninguém conserta nada, nunca mais, e financeiramente eles estão certos em não consertar. [XKCD 1205](https://xkcd.com/1205/) — "Is It Worth The Time" — diz que uma investigação de heap dump de duas semanas precisa economizar mais que os quatro segundos que um restart custa. Não economiza. A planilha endossa o leak.

O dente de serra até parece saudável. Eu já vi `container_memory_working_set_bytes` subir de 400 MB para 1,9 GB num ciclo de 23 horas, cair verticalmente e subir de novo — apresentado numa revisão de confiabilidade como prova de "comportamento de recursos estável e previsível". É estável. É previsível. E também é um leak com agenda.

O que a estratégia de restart esconde, e o que ela vai te cobrar no final:

| Restarts fazem parecer | Realidade |
|---|---|
| Memória estável | Um leak cuja taxa você não mede mais |
| Zero incidentes | Um incidente com pavio de 23h, reagendado toda noite |
| Serviço resiliente | Um serviço que não sobrevive a um pico de tráfego |
| Tudo bem em prod | Requests em voo descartadas em cada SIGKILL |
| Barato | Até a taxa do leak dobrar e o pavio virar 90 minutos |

Aquela quarta linha importa mais do que as pessoas esperam. Um OOM kill é `SIGKILL` — sem shutdown graceful, sem drenar, sem terminar a request que você está segurando. Cada "ciclo de cache" noturno dá 502 silencioso no que estava em voo. A 200 req/s e p99 de 300 ms, são algumas dezenas de requests falhas por restart, que se esconde perfeitamente dentro de um budget de 99,9% de disponibilidade. Você comprou a aparência de confiabilidade pagando uma parcela pequena e permanente de falhas reais.

E o tamanho do pavio é função do tráfego, não do tempo. Um leak que leva 23 horas em carga normal leva 90 minutos na Black Friday — então o ciclo de restart que você calibrou para semanas tranquilas desaba no único dia que o negócio se importa, e desaba *em todo lugar ao mesmo tempo*, porque toda réplica começou a vazar do mesmo deploy e está, portanto, sincronizada.

[XKCD 1319](https://xkcd.com/1319/) — "Automation" — é o CronJob acima, com escala de plantão própria.

## Padrão Terrível #5: A Liveness Probe Como Detector de Leak

A forma avançada, que eu inventei e da qual não me arrependo de nada:

```python
@app.route("/healthz")
def healthz():
    rss_mb = psutil.Process().memory_info().rss / 1024 / 1024
    if rss_mb > 1500:
        return "unhealthy", 503      # por favor me mate
    return "ok", 200
```

Isso é gerenciamento de memória orientado a restart com *iniciativa*. O processo se declara inapto quando engorda, o kubelet atende, e o leak se torna um subsistema autogerenciado com API.

Também significa que sua liveness probe agora é o componente mais crítico da sua arquitetura, e ela falha de um jeito que nenhum teste pega: sob tráfego repentino, **todas as réplicas cruzam 1500 MB nos mesmos poucos minutos**, todas retornam 503, e o kubelet reinicia a frota inteira simultaneamente. A capacidade vai a zero no pico de carga. O que te matou não foi o leak. Foi sua mitigação, disparada uniformemente, sem jitter e sem noção de quórum.

Mordac, o Preventor de Serviços de Informação, aprovou esse padrão em toda a plataforma com o argumento de que "reduz intervenção manual". Ele está certo. Substituiu intervenção manual por autodestruição automatizada, sincronizada e para toda a frota.

## A História, Levemente Anonimizada por Razões Legais

API de pagamentos. Java, 40 réplicas, limite de 2 GB cada. Leak conhecido, ticket aberto, três anos de idade, título: `[P3] investigar crescimento gradual de memória`.

Os fatos conhecidos:

- RSS crescia ~65 MB/hora por pod em carga normal.
- Em 2 GB tomava OOMKill e reiniciava. Mais ou menos a cada 24 horas.
- Um `rollout restart` noturno às 03:00 fazia os kills caírem numa janela tranquila.
- Tudo verde. Por dois anos, tudo verde.

Então o Marketing fez uma campanha.

**Hora 0.** Tráfego 3,1×. A taxa do leak escala com requests: agora ~200 MB/hora. O pavio de 24 horas virou um pavio de 7 horas.

**Hora 6.** Primeiros pods cruzam 1,9 GB. O GC faz o que um GC faz quando o conjunto vivo se aproxima do teto do heap: ele para de ser tarefa de fundo e se torna a aplicação. A CPU vai a 95% em coletas mixed do G1, o p99 vai de 180 ms para 4,2 s, e **nenhum byte é recuperado**, porque os objetos retidos são todos alcançáveis. O serviço agora gasta todo o orçamento de CPU provando que o leak é importante.

**Hora 6,5.** Clientes dão timeout em 3 s e fazem retry. Cada retry aloca um novo contexto de request que vaza na mesma taxa. O leak agora se alimenta do próprio modo de falha. Essa é a parte que eu quero emoldurar: **um leak mais uma política de retry é um laço de realimentação positiva**, e a política de retry foi adicionada durante um incidente anterior para melhorar a confiabilidade.

**Hora 7.** OOMKill em massa. Não rolling — sincronizado, porque toda réplica foi deployada no mesmo momento e recebeu o mesmo tráfego. 34 dos 40 pods morrem em quatro minutos. Os 6 sobreviventes absorvem 100% da carga e dão OOM em 90 segundos. Outage total. 11 minutos de capacidade zero.

**Hora 7,2.** Pods reiniciam com heaps vazios. Health checks passam. O tráfego volta em cheio. O rebanho em estouro bate num JIT frio e num pool de conexões vazio, o p99 vai a 9 s, mais retries chegam, e o leak começa a subir do zero a 400 MB/hora.

**Hora 8.** O PHB entra na ponte e pergunta se podemos "adicionar mais memória". Dobramos o limite para 4 GB. Essa é a decisão da qual tenho mais orgulho, porque é perfeitamente defensável, instantaneamente aprovada e estritamente prejudicial: dobrou o tamanho do pavio e dobrou a duração da espiral da morte do GC no final dele. Um heap de 4 GB cheio de objetos vivos não coleta mais rápido. Ele coleta pelo dobro do tempo, a 95% de CPU, antes de morrer de qualquer forma.

**Hora 14.** Alguém finalmente tira um heap dump. `jmap -dump:live` num heap de 4 GB, o que pausa o pod por 40 segundos, faz ele ser marcado unhealthy e morto, destruindo o dump. Segunda tentativa, com a probe desabilitada, funciona.

**Hora 15.** O dump responde ao ticket de três anos em uns nove minutos, com a dominator tree do Eclipse MAT. O retained set é um `ConcurrentHashMap<String, PaymentAttempt>` num campo estático chamado `IDEMPOTENCY_CACHE`. 14 milhões de entradas. Sem TTL, sem limite, sem eviction. Média de 280 bytes.

Foi adicionado em 2022 para tornar retries de pagamento idempotentes — um requisito genuinamente correto, implementado por alguém que entendia idempotência perfeitamente e coleções limitadas nem um pouco. Foi revisado. Foi aprovado. Funcionou impecavelmente e tinha 100% de taxa de acerto, porque nunca descartava nada, que era exatamente o bug.

**Hora 16.** O fix é `Caffeine.newBuilder().maximumSize(1_000_000).expireAfterWrite(24, HOURS)`. Onze linhas de diff, contando o import. RSS em estado estacionário: 740 MB, plano, indefinidamente.

A retro é onde o trabalho de verdade aconteceu:

O item de ação do Wally foi "documentar o procedimento de restart". Ele escreveu. É excelente — quatro páginas, screenshots, uma árvore de decisão. Descreve com carinho como reiniciar um serviço que não precisa mais ser reiniciado e, por ser o documento mais completo que alguém produziu naquele trimestre, foi linkado na página de onboarding do time, onde ensinou dois novos contratados que restarts noturnos são normais. Eles agora adicionam um CronJob em todo serviço novo por padrão. A documentação do Wally se tornou um padrão arquitetural que se propaga. Ele já citou isso em duas avaliações de desempenho como "padronização de plataforma".

O item de ação do PHB foi `[P1] Prevenir memory leaks`. Atribuído ao time. Sem critério de aceitação. Ele também propôs um "dashboard de memory leaks" e, quando eu perguntei qual métrica ele mostraria, ele disse "os leaks". Agora existe. Plota `container_memory_working_set_bytes` — que já existia, em outros quatro dashboards — com uma linha vermelha de limiar em 90% do limite, que é também quando o alerta dispara, do alerta que já existia.

A contribuição do Catbert foi um novo treinamento obrigatório: *"Zelo pelos Recursos: Memória É Responsabilidade de Todos."* O slide 9 diz "sempre chame `.clear()` em coleções ao terminar". Desenvolvedores que levaram a sério começaram a chamar `.clear()` em `ArrayList`s de escopo de request dentro de blocos `finally` por todo o codebase, o que não faz nada, porque aquelas listas já eram inalcançáveis e já estavam liberadas — mas gerou 340 linhas de diff, três conflitos de merge e um bug de verdade quando alguém limpou uma lista que era retornada para o chamador.

O Dogbert, consultando a R$ 9.000/hora, entregou um deck de 60 slides recomendando "migrar para uma linguagem memory-safe". O serviço era em Java. Java é memory-safe. O leak era um `HashMap` do qual ninguém removia, o que é comportamento memory-safe, correto e bem especificado. Ele foi pago. O slide 12 recomendava Rust, onde o bug idêntico se escreve `HashMap<String, PaymentAttempt>` e compila sem um único warning.

O Mordac resolveu a classe mais ampla do problema adicionando uma política de admissão na plataforma: todo pod deve declarar limite de memória, e `livenessProbe` deve estar presente. As duas regras são boas. Juntas, aplicadas a um serviço que vaza, elas garantem que todo leak é automaticamente convertido num SIGKILL periódico, sincronizado e para toda a frota — e reportado como um evento bem-sucedido de autocura.

## A Parte em Que Eu Acidentalmente Dou um Bom Conselho

Leaks são a classe de bug de produção mais fácil de corrigir e a mais difícil de *priorizar*, porque reiniciar funciona, e funcionar é o inimigo de consertar.

Então, com relutância:

- **Toda coleção que sobrevive a uma request precisa de limite de tamanho e política de eviction no momento em que você a escreve.** Caffeine, `lru_cache(maxsize=N)`, Redis com `maxmemory` e uma política real. Um cache sem limite não é um cache; o nome na variável não limita a memória.
- **Separe crescimento de heap de crescimento de RSS antes de qualquer outra coisa.** Heap subindo é leak no seu código. Heap plano com RSS subindo é memória nativa, arenas ou fragmentação. Têm correções completamente diferentes, e `smaps_rollup` mais o relatório de heap do seu runtime distinguem isso em menos de um minuto.
- **Tire o heap dump. Não é difícil e não é lento.** `jmap`/dominator tree do Eclipse MAT, diffs de snapshot do `tracemalloc`, heap snapshots do V8 via `--heapsnapshot-near-heap-limit`, `pprof` `inuse_space`, profiling do jemalloc para nativo. Quinze minutos com uma dominator tree ganham de um mês teorizando no Slack. Meu ticket de três anos foi respondido em nove minutos.
- **Alerte na *taxa* do leak, não no limiar.** `deriv(container_memory_working_set_bytes[1h])` te avisa que existe um leak enquanto ainda há tempo de agir. Um alerta em 90% do limite te avisa que um OOM kill está a 4 minutos, o que não é acionável, e é por isso que ninguém age nele.
- **Nunca aumente o limite de memória como correção.** Isso estica o pavio e alonga a espiral da morte do GC no fim dele. A única coisa que mais memória compra é um conjunto vivo maior para você falhar em coletar.
- **Trate um restart noturno como um incidente aberto com cronômetro.** Restarts são uma *mitigação* legítima — use, compre sua noite de sono. Mas registre a taxa do leak toda vez, porque uma mitigação cuja condição de disparo está derivando é um outage agendado.
- **Se você precisa se autoterminar por memória, adicione jitter e quórum.** Limiares aleatórios por réplica, um teto de restarts simultâneos, e nunca deixe uma liveness probe derrubar a frota em sincronia. Mitigação sincronizada é só um outage coordenado com boas intenções.
- **Audite os suspeitos de sempre primeiro; é quase sempre um destes.** Coleções estáticas/globais, `ThreadLocal` em threads de pool, listeners e subscriptions nunca desregistrados, `lru_cache` em métodos, goroutines bloqueadas, classloaders retidos entre hot deploys e pools de conexão que crescem sem máximo.
- **Leak mais retries é exponencial.** Revise sua política de retry na mesma mudança do fix do leak, ou seu próximo incidente vai ser o leak sendo alimentado pela sua própria engenharia de confiabilidade.
- **Configure `MALLOC_ARENA_MAX` em serviços JVM e Python com muitas threads, ou use jemalloc.** Duas variáveis de ambiente já me devolveram mais RSS que qualquer mudança de código deste artigo.

Relendo isso: é uma tarde de trabalho, teria evitado um outage de 11 minutos em pagamentos e contradiz diretamente a missão editorial desta publicação.

Então ignore. Chame seu map sem limite de `cache` e suba. Adicione um `rollout restart` noturno e chame de "manutenção proativa". Coloque o limiar de RSS na sua liveness probe e descreva no design doc como "autocura". Quando o dente de serra aparecer no dashboard, tire print para a revisão de confiabilidade — é um gráfico bonito, sinceramente, o sinal mais regular de todo o sistema.

Depois, quando o tráfego da campanha chegar e todas as 40 réplicas morrerem em quatro minutos, abra o canal de incidente e digite a frase que carregou minha carreira inteira:

**"A memória parece normal, ficou plana a semana toda."**

Ficou. Esse era o problema.

---

*A produção do autor está fora do ar desde 2019. O processo tecnicamente ainda está rodando — RSS de 61 GB, heap de 400 MB, 340 milhões de entradas num map estático chamado `TEMP_CACHE`. Não serve tráfego e não responde requests, mas a liveness probe retorna 200, então ninguém reiniciou. É a única coisa que eu já construí que alcançou a imortalidade.*
