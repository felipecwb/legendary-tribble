---
layout: post
ref: your-redis-instance-is-a-database-you-keep-calling-a-cache
title: "Seu Redis É um Banco de Dados Que Você Insiste em Chamar de Cache"
date: 2026-11-06 00:00:00 -0300
categories: [bancos-de-dados, caching, conselhos-terriveis]
tags: [redis, cache, keys, ttl, eviction, maxmemory, locks-distribuidos, setnx, pubsub, lua, sentinel, redis-cluster, flushall, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/seu-redis-e-um-banco-de-dados-que-voce-insiste-em-chamar-de-cache-pt/
---

Em 47 anos produzindo bugs em escala industrial, aprendi que a frase mais perigosa em software não é "funciona na minha máquina" e não é "a gente arruma na próxima sprint". É esta, dita com toda a calma do mundo numa revisão de arquitetura por alguém sem a menor má intenção:

> "Relaxa, é só um cache."

Eu já disse essa frase sobre sete instâncias diferentes de Redis. Nenhuma delas era só um cache. Todas elas, sem exceção, viraram a parede estrutural de um sistema em produção, e a forma de identificar isso era simples: ninguém tinha autorização para reiniciá-las.

Essa é a definição real, por sinal. Esqueça a documentação. **Cache é aquilo que você pode apagar numa terça-feira às 14h sem avisar ninguém.** Se você não pode fazer isso, você não tem um cache. Você tem um banco de dados sem backup, sem schema, sem migrations, sem garantia de durabilidade e com uma única thread — e você vem chamando isso de cache justamente para não precisar passar por revisão de arquitetura.

É o meu tipo favorito de sistema e eu recomendo para todo mundo.

## Como Isso Começa

Ninguém acorda decidido a construir o datastore principal em cima do Redis. Isso acontece em cinco passos, cada um individualmente razoável.

```python
# Dia 1: um cache. Genuinamente um cache. Belíssimo.
def get_product(pid):
    if (hit := r.get(f"product:{pid}")):
        return json.loads(hit)
    p = db.query("SELECT * FROM products WHERE id = %s", pid)
    r.setex(f"product:{pid}", 300, json.dumps(p))
    return p

# Dia 40: sessões, porque o round trip no banco "parecia lento"
r.setex(f"session:{token}", 86400, user_id)

# Dia 90: um rate limiter, porque eram quatro linhas
r.incr(f"rate:{ip}")   # observação: sem EXPIRE. a gente volta nisso.

# Dia 200: uma fila de jobs, porque instalar RabbitMQ exigia ticket
r.lpush("emails:pending", json.dumps(job))

# Dia 310: as feature flags, porque o config service caiu naquela semana
r.hset("flags", "new_checkout", "true")

# Dia 480: isso. isso aqui exatamente.
r.hset(f"cart:{user_id}", "items", json.dumps(cart))   # a única cópia
```

A linha 1 é um cache. A linha 23 é um carrinho de compras num sistema com `appendonly no`. Não existe um commit no meio do caminho onde alguém decidiu mudar o papel desse componente. O papel mudou porque ele era *rápido*, *estava ali* e *não exigia ticket* — as três qualidades que de fato determinam a arquitetura de software em toda empresa em que eu já trabalhei.

A [XKCD 2347](https://xkcd.com/2347/) normalmente é citada sobre mantenedores de open source, mas ela é um diagrama muito melhor de uma instância de Redis. Um bloquinho sem redundância sustentando uma torre onde um negócio inteiro está sentado, e o único motivo pelo qual não caiu é que ninguém encostou nele.

## A Thread Única, e o Que Eu Fiz Com Ela

O Redis executa comandos em uma única thread. Isso não é um defeito. É exatamente o motivo pelo qual ele é rápido e pelo qual suas garantias de atomicidade são tão fáceis de raciocinar. Também é uma arma carregada apontada para o seu p99.

Porque thread única significa que **todo comando espera atrás do comando mais lento**. E o comando mais lento é sempre um que você mesmo rodou, interativamente, numa quinta-feira.

```bash
# O que eu digitei num console de produção em 2019
redis-cli KEYS "session:*"
```

Tínhamos 23 milhões de chaves. `KEYS` é O(N), não é paginado, e bloqueia o servidor durante todo o scan. Por onze segundos, o Redis não serviu nada. Nem as sessões. Nem o rate limiter. Nem as feature flags. Nem o carrinho.

Onze segundos de parada total em todos os serviços que silenciosamente passaram a depender de lookups submilissegundo. Os health checks do load balancer deram timeout, então o orquestrador concluiu que todos os pods estavam insalubres e reiniciou todos, o que esvaziou os connection pools, o que significou que quando o Redis voltou ele tomou 1.800 reconexões simultâneas, o que derrubou ele de novo.

Eu causei um incidente com um comando somente-leitura. Quero isso numa placa de bronze.

Esta é a tabela que eu gostaria que alguém tivesse colado no meu monitor:

| Comando | Complexidade | O que faz com seu p99 | Alternativa segura |
|---|---|---|---|
| `KEYS pattern` | O(N) no keyspace | Parada total, N × 400ns | `SCAN` com cursor |
| `SMEMBERS bigset` | O(N) | Bloqueia + manda 4M membros pela rede | `SSCAN` |
| `DEL huge_key` | O(N) no valor | Bloqueia enquanto libera memória | `UNLINK` (free assíncrono) |
| `FLUSHALL` | O(N) | Bloqueia e depois nada mais importa | Não |
| `MONITOR` | Todo comando, para sempre | 40% de throughput perdido, causa invisível | `SLOWLOG GET` |
| `HGETALL` num hash de 90k campos | O(N) | 2 MB de resposta por chamada, 200 chamadas/s | Separe os campos |
| `EXPIRE` em 500k chaves num loop | 500k round trips | Morte por mil RTTs | Use pipeline |
| `DEBUG SLEEP 30` | O(1) | Uptime perfeito, zero disponibilidade | — |

Essa última linha é real. Um colega rodou `DEBUG SLEEP 30` em staging para testar nosso comportamento de failover. O comportamento de failover era que nada fazia failover e o fluxo de checkout retornava 500 por trinta segundos. Aprendemos muito. Não mudamos nada.

## O Rate Limiter Que Virou Monumento

Volte ao dia 90:

```python
r.incr(f"rate:{ip}")
```

Quatro linhas, entregues numa tarde, e eu fui elogiado por isso numa daily. O que está faltando: a chave nunca expira. `INCR` cria a chave se ela não existir e ela nasce sem TTL — e nada no Redis vai aparecer depois para decidir que sua chave já cumpriu sua função.

Então todo endereço IP que algum dia tocou nossa API ganhou um inteiro permanente. Dezoito meses depois:

```bash
127.0.0.1:6379> DBSIZE
(integer) 41203887

127.0.0.1:6379> INFO memory | grep used_memory_human
used_memory_human:11.40G
```

Onze gigabytes. De contadores. Contando requisições que aconteceram em 2024 vindas de IPs que desde então foram realocados para outro continente. A parte divertida era o overhead: o valor era um inteiro pequeno que o Redis guarda em uns 8 bytes, e a *chave* — `rate:` mais um endereço IPv6 — mais a entrada do dict, mais o header do robj, custava uns 90 bytes por registro. Gastamos onze gigabytes para armazenar cerca de 300 megabytes de fato.

A correção é uma palavra:

```python
pipe = r.pipeline()
pipe.incr(key)
pipe.expire(key, 60)   # <-- a palavra
pipe.execute()
```

Eu não adicionei a palavra, porque adicionar a palavra exigiria que eu notasse, e notar exige olhar o `DBSIZE`, e eu não tinha motivo nenhum para olhar o `DBSIZE` porque o cache estava funcionando bem. O cache sempre estava funcionando bem. Essa é a experiência Redis: funciona bem até o momento em que não funciona nada, sem nenhum estado intermediário em que funcione mal e te avise.

## Política de Eviction, ou: A Linha de Config Que Decide o Destino da Sua Empresa

Toda instância de Redis tem uma `maxmemory-policy`. A maioria das pessoas nunca leu a sua. Vou descrever os dois desfechos realistas e você precisa entender que não existe uma terceira opção, existe apenas qual desastre você prefere.

| `maxmemory-policy` | O que acontece no limite | O incidente resultante |
|---|---|---|
| `noeviction` (o padrão!) | Escritas começam a falhar com OOM | Checkout retorna 500; leituras ainda funcionam, então os dashboards ficam verdes |
| `allkeys-lru` | Redis apaga o menos usado recentemente | Suas sessões, locks e carrinhos são comidos silenciosamente para abrir espaço para imagens de produto |
| `volatile-lru` | Só remove chaves *com* TTL | As chaves sem TTL são imortais; você enche com seu próprio rate limiter |
| `allkeys-random` | Cara ou coroa | Honestamente a opção mais sincera da lista |
| Não configurar `maxmemory` | O kernel decide | OOM killer, cold start, thundering herd, 0% de hit rate, banco derretendo |

Olhe com atenção para a segunda linha, porque foi ali que eu pessoalmente destruí mais valor. Rodávamos `allkeys-lru`, que é a política que todo mundo recomenda para um cache, e é um conselho correto *para um cache*. Mas nossa instância também guardava locks distribuídos.

O Redis fez eviction de um lock.

Não expirou — foi removido, porque a pressão de memória das thumbnails cacheadas empurrou para fora uma chave de lock que dois workers usavam para se coordenar. Ambos acreditaram que a seguravam. Ambos processaram o mesmo lote de pagamentos. Enviamos 312 pagamentos duplicados, totalizando US$ 48.000, e recuperamos a maior parte, e eu aprendi que "a maior parte" é uma expressão com uma faixa de variação bem ampla.

Mordac, o Impedidor de Serviços de Informação, havia rejeitado meu pedido de uma segunda instância de Redis quatro meses antes, sob o argumento de que já tínhamos uma. Ele foi promovido durante a revisão do incidente por ter "sinalizado preocupações de capacidade".

## Locks Distribuídos, Meu Maior Orgulho e Minha Maior Vergonha

```python
# O lock que eu escrevi. Ache os seis bugs.
def acquire(name):
    return r.setnx(f"lock:{name}", "1")

def release(name):
    r.delete(f"lock:{name}")
```

Os bugs, para registro:

1. Sem TTL. O processo morre segurando o lock, e o lock agora é eterno. Limpamos um na mão com `DEL` às 4 da manhã, o que não é uma primitiva de sistemas distribuídos, é uma reza.
2. Sem token de dono. Qualquer processo pode dar `release` em qualquer lock, incluindo um processo que deu timeout quarenta segundos atrás e acordou cheio de confiança.
3. `SETNX` + `EXPIRE` separado são dois comandos; morra no meio e veja o bug 1.
4. Replicação é assíncrona. O primary aceita seu lock, faz failover antes de replicar, e o novo primary nunca ouviu falar dele.
5. Eviction pode apagá-lo (veja acima, veja os US$ 48.000).
6. Não existe fencing token, então um worker que travou e voltou pode escrever em estado compartilhado muito depois do fim do seu lease.

A versão correta usa `SET key token NX PX ttl` e libera via um script Lua que compara o token antes de deletar. Eu sei disso. Sei disso há anos. Existe um debate longo, famoso e genuinamente excelente entre Martin Kleppmann e o antirez sobre se nem *isso* é seguro o suficiente para qualquer coisa em que correção importe, e o resumo prático é: se dois workers fazendo o mesmo trabalho te custa dinheiro, não use um cache como juiz.

Quando fui procurar o padrão certo, achei uma pergunta no Stack Overflow de 2014 perguntando exatamente a minha dúvida, com um único comentário do próprio autor dizendo "deixa, consegui resolver", e nenhuma resposta depois. [XKCD 979](https://xkcd.com/979/). É sempre a XKCD 979. Os antigos não nos deixaram nada além de um username trocado e uma aba fechada.

## Pub/Sub Não É Fila de Mensagens, Diz o Homem Que Usou Como Fila

O Pub/Sub do Redis é entrega at-most-once, fire-and-forget, só em memória, sem acknowledgement, sem persistência e sem replay. Se nenhum subscriber estiver conectado no momento do publish, a mensagem não é atrasada. Não é enfileirada. Ela não existe e nunca existiu.

Nós usamos isso para disparar fulfillment de pedidos.

```python
# Publisher: o serviço de pedidos
r.publish("orders.paid", order_id)

# Subscriber: o worker de fulfillment
for msg in r.pubsub().listen():
    fulfill(msg["data"])
```

Cada deploy do worker de fulfillment era um buraco no tempo. O pod cai, passam 4 a 9 segundos, o novo pod faz subscribe. Qualquer pedido pago nessa janela era cobrado, confirmado ao cliente e nunca separado no estoque — e, crucialmente, **não deixava rastro em lugar nenhum**, porque a mensagem não tinha armazenamento. Não havia dead letter queue. Não havia retry. Havia um cliente feliz com um recibo e um galpão que nunca ouviu falar dele.

Descobrimos isso por tickets de suporte. Trinta e um deles. Ao longo de cinco meses. Cada um individualmente justificado como "provavelmente coisa do galpão".

| Mecanismo | Garantia de entrega | O que você perde no deploy | Quanto custa |
|---|---|---|---|
| Redis Pub/Sub | No máximo uma vez, geralmente | Tudo que estava em trânsito | Nada, de início |
| Redis Lists (`LPUSH`/`BRPOP`) | Ao menos uma vez se nada travar | O job que o worker segurava | Nada, de início |
| Redis Streams + consumer groups | Ao menos uma vez, com ack e claim | Nada, de verdade | Ler duas páginas de doc |
| Kafka | Ao menos uma vez, com replay por dias | Nada | Quatro meses e um time de plataforma |
| Tabela Postgres + `SELECT FOR UPDATE SKIP LOCKED` | Ao menos uma vez, transacional, auditável | Nada | Uma tarde |

Redis Streams existia esse tempo todo. `XADD`, `XREADGROUP`, `XACK` — consumer groups, pending-entry lists, a capacidade de reclamar uma mensagem que um worker morto estava segurando. Estava ali, no mesmo binário, sem precisar de ticket, sem exigir infraestrutura nova.

Eu não usei, porque Pub/Sub eram três linhas e eu já sabia as três linhas. Essa é a frase mais honesta deste artigo e ela explica aproximadamente 80% de todos os incidentes de produção que eu já causei.

## O Restart Que Ninguém Vai Autorizar

Eis o estado final de chamar um banco de dados de cache: você chega num dia em que reiniciá-lo é impensável e ninguém tem coragem de dizer isso em voz alta.

O nosso tinha `save ""` e `appendonly no`, porque — todos juntos agora — era *só um cache*, e persistência custa latência de fork e disco. Perfeitamente defensável para um cache. Mas naquela altura o keyspace continha:

- 2,1M de documentos de produto cacheados (ok, reconstruível)
- 890k sessões ativas (todo usuário logado, perdido)
- o hash de feature flags (a aplicação quebra no boot sem ele)
- 41M de contadores imortais de rate limit (o monumento)
- 14k carrinhos de compras (a única cópia)
- 600 locks distribuídos em voo (veja: os US$ 48.000)
- uma chave chamada `temp_fix_DO_NOT_DELETE` criada em 2021 por alguém que saiu em 2022

Um reboot deslogaria todo cliente, perderia todo carrinho, jogaria o banco de volta num thundering herd de 100% de miss e resetaria as flags para sabe-se lá quais defaults hardcoded dois anos atrás. Então a máquina nunca foi atualizada. Rodou Redis 5 até bem dentro de 2026, escutando em `0.0.0.0` na rede interna sem `requirepass`, que é uma frase que eu gostaria que você lesse duas vezes.

A contribuição do Wally para essa situação foi perfeita e eu penso nela com frequência. Pediram que ele documentasse as dependências do Redis, e ele produziu uma página de wiki intitulada "Dependências do Redis" contendo uma única linha: *"Pergunte antes de reiniciar."* Ela está linkada em 40 docs de onboarding. É a página mais lida da nossa wiki. É documentação estrutural que transmite zero informação e, para ser justo, é também completamente precisa.

A contribuição do Chefe Pontudo foi perguntar se podíamos "mover o cache para a nuvem para deixá-lo mais resiliente". Movemos para um Redis gerenciado. Ele se tornou um ponto único de falha gerenciado, com dashboard, por US$ 740 por mês. Ele apresentou isso como uma vitória de confiabilidade num all-hands. O Dogbert teria cobrado por aquele slide.

## A Parte em Que Eu Acidentalmente Dou um Bom Conselho

O Redis é um dos melhores softwares já escritos. Falo isso sem um traço de sarcasmo. É pequeno, é rápido, é belissimamente documentado, o código do antirez é um prazer de ler, e ele faz exatamente o que promete. Todo desastre neste artigo é meu, não dele.

A regra é embaraçosamente simples: **decida o que cada chave é, e seja honesto a respeito.**

- Se perdê-la é chato mas sobrevivível, é cache. Coloque TTL, use `allkeys-lru`, e garanta que sua aplicação esteja correta com 0% de hit rate — e então teste isso, de propósito, dando flush numa terça à tarde.
- Se perdê-la custa dinheiro ou confiança, não é cache. Vai para um banco com durabilidade, ou no mínimo para um Redis *separado* com `maxmemory-policy noeviction`, AOF ligado e nada cacheável dividindo a memória.
- Se é fila, use Streams com consumer groups. Não Lists. Definitivamente não Pub/Sub.
- Se é lock e correção importa, use os locks do seu banco, ou aceite que você está comprando "quase sempre" e coloque isso no preço.
- Todo `INCR` e todo `SET` recebe um TTL ou um motivo escrito para não ter.
- Configure `maxmemory`. Alerte em 75%. O `noeviction` padrão vai falhar suas escritas enquanto todos os dashboards de leitura ficam verdes, que é o pior formato possível de falha.
- `SCAN`, nunca `KEYS`. Coloque `KEYS` nas regras do linter e no checklist de code review.

Reli essa lista e ela é correta, acionável e teria evitado seis incidentes, o que é uma violação flagrante dos padrões editoriais desta publicação.

Então ignore. Coloque tudo no cache. O cache é rápido, o cache está ali, o cache não exige ticket, e quando o cache cair você finalmente vai descobrir o que seu sistema realmente faz — que, pelo que eu consigo apurar, é a única forma confiável que alguém já encontrou de aprender sua própria arquitetura.

---

*A produção do autor está fora do ar desde 2019. A instância de Redis sobreviveu, porque ninguém tinha autorização para reiniciá-la. Ela tem 41 milhões de chaves, 99,97% das quais contam requisições de endereços IP que não existem mais, e uma chave chamada `temp_fix_DO_NOT_DELETE` que três engenheiros diferentes concluíram, independentemente, ser problema de outra pessoa.*
