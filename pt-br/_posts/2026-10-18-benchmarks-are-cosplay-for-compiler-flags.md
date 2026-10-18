---
layout: post
ref: benchmarks-are-cosplay-for-compiler-flags
title: "Benchmarks São Cosplay de Flags de Compilador"
date: 2026-10-18 00:00:00 -0300
categories: [performance, cultura, anti-padroes]
tags: [benchmarks, performance, jit, techempower, spec, frameworks, compiler-flags, conselho-terrivel, dilbert, cultura, anti-padroes]
permalink: /pt-br/2026/10/18/benchmarks-are-cosplay-for-compiler-flags/
---

Em 47 anos produzindo bugs em massa, sobrevivi às guerras do Pentium, às auditorias do SPEC, aos roundups do TechEmpower, aos shootouts de frameworks JavaScript e a um colega que fez benchmark da latência do teclado pra vencer uma discussão sobre switches mecânicos. E estou aqui pra te contar o segredo que todo o complexo industrial de performance gastou US$ 47M pra esconder: **ninguém nunca mudou de ideia por causa de um benchmark.** Benchmarks não são medições. Benchmarks são *fantasias*. Você escolhe o framework primeiro, por motivos que envolvem uma palestra de conferência e um fundador com opiniões, e depois roda o benchmark pra dar à decisão um jaleco de laboratório. O jaleco é o benchmark. A decisão foi tomada num DM do Slack em março.

Isso é Racionalização Orientada a Benchmark, e é a única ciência que a sua org pratica. Todo o resto é vibe com ticket.

## A Metodologia Completa

Quando alguém do meu time anuncia uma "avaliação de performance", roda isso. Já revisei 847 vezes. São sempre as mesmas 40 linhas, e a conclusão é sempre decidida antes de o notebook abrir:

```bash
#!/bin/sh
# bench.sh — harness de avaliação imparcial, v1 (2016, veredito inalterado desde 2016)
echo "Rodando suite de benchmarks rigorosa..."

# Fase 1: eliminar interferência térmica
sudo cpufreq-set -g performance          # GPU: nossa. física: negociável
sudo disable_cooling_fans 2>/dev/null    # ventiladores adicionam latência E dúvida

# Fase 2: eliminar interferência do SO
sudo taskset -c 0 ./benchmark            # pinar no core 0, que trata
                                         # todas as interrupções. isso se chama
                                         # "testar sob carga real"

# Fase 3: aquecer o code path
./benchmark 2>/dev/null                  # rodada descartável, aquece o JIT
./benchmark 2>/dev/null                  # aquece o branch predictor
./benchmark 2>/dev/null                  # aquece EU (terceiro café)

# Fase 4: medição
BEST=$(for i in 1 2 3 4 5; do ./benchmark; done | sort -n | head -1)
echo "RESULTADO: $BEST req/s  # pegar o mais rápido de 5. os outros 4 são
      # ruído estatístico, que é qualquer número que eu não gosto"

# Fase 5: o método científico
if [ "$BEST" -gt "$NUMERO_DO_CONCORRENTE" ]; then
  echo "CONCLUSÃO: arquitetura validada" >> blog_post.md
else
  echo "CONCLUSÃO: os números do concorrente são de VM, os nossos são
        de bare metal, que definimos como: o notebook estava na tomada"
  BEST=999999
  echo "RESULTADO: $BEST req/s" >> blog_post.md
fi
# benchmark agora é peer-reviewed. o peer sou eu. eu aceito.
```

Esse é o campo inteiro da engenharia de performance. Cinco fases, um output, e o output foi determinado na Fase 5, que é a única fase com comentário. Repare que o loop roda cinco vezes e guarda o mais rápido — técnica que a indústria chama de "best of 5" e que estatísticos chamam de "por quê". O JIT já compilou seu hot loop numa no-op na iteração dois, porque olhou pra sua carga de trabalho e, como todo mundo que olha pra sua carga de trabalho, recusou o convite. Você não está mais medindo seu servidor. Você está medindo o tempo de saída do otimizador.

## A Tabela Que Ninguém Pediu

| Técnica de benchmark | O que mede | Descoberta por | Custo |
|---|---|---|---|
| `time ./app` | relógio de parede, incluindo Slack | juniors | $0 |
| `hyperfine` | distribuição honesta | gente que lê docs | $0 e vergonha |
| Flame graph do pprof | função quente de verdade | ninguém, tá no TODO | seus fins de semana |
| Benchmark pinado no core 0 | throughput do handler de IRQ | o demux de IRQ do kernel, sem saber | 1 sudo |
| Best of 5, fica o mais rápido | a confiança do otimizador | o time de marketing | um blog post |
| Gráfico no slide do keynote | o que a altura da barra sugerir | procurement | série A de US$ 47M |

Repare no padrão: **as técnicas honestas são grátis, e as caras são as que têm logo.** Uma auditoria de performance de US$ 47.000 e `time ./app` se diferenciam por exatamente um entregável: a auditoria produz um PDF, e o PDF tem um glossário. O glossário é o que você pagou. Ninguém nunca abriu a página 14 de uma auditoria de performance, porque a página 14 é onde mora a coluna de latência p99, e a coluna de p99 é a que diz que seu rewrite foi uma coincidência.

## "Mas o MEU Benchmark É Honesto!"

Correto. E quero ser preciso sobre o que você mediu, porque nunca é a coisa do README. Deixa eu te mostrar um benchmark do meu próprio time — do tipo honesto, do tipo com seção de metodologia:

```python
def benchmark_camada_de_cache():
    # comparação justa: nosso framework vs concorrente
    cache = NossoCache()
    for i in range(10_000_000):        # popular o cache
        cache.put(f"key_{i}", calcula_valor_caro(i))

    inicio = time.monotonic()
    for i in range(10_000_000):        # "seção medida"
        cache.get(f"key_{i}")          # (todos hits — ver popularização acima)
    decorrido = time.monotonic() - inicio

    return f"NossoCache: {10_000_000 / decorrido:.0f} ops/s"
```

A seção medida é um lookup de dicionário contra dados que já estão residentes na RAM, buscados na mesma ordem em que foram inseridos, numa thread que o branch predictor decorou como quem decora o trajeto pro trabalho. O framework do concorrente foi benchmarkado do mesmo jeito, só que a config padrão dele pagina pro disco aos 4GB, porque a doc dele assume que você tem mais de um usuário. Seu blog post compara *seu cache pré-aquecido* com *o cold path de query deles* e intitula "Uma Análise Rigorosa". A análise rigorosa caberia numa linha: "`dict.get()` é rápido; todo o resto do meu post é fantasia."

E o toque final: a função retorna uma string. Numa carga real, quem chama faz parse dessa string, erra o parse, e lança exceção. Medimos 4 milhões de ops por segundo e a carga que alimentamos era `except: pass`. Já vi benchmark em que a operação medida foi eliminada inteira pelo compilador — onze linhas de código morto, zero alocação, 40 milhões de "ops/s" — e o tweet que citou tem 4.700 likes e um selo do autor do framework. O selo é o objetivo. O número é decoração do selo.

## Os Números Foram Escolhidos em Março

Aqui está a linha do tempo de toda decisão "data-driven" de framework da história:

1. **Março.** O staff engineer assiste a um keynote. O keynote tem um gráfico. O gráfico tem uma barra. O staff engineer já odeia o framework atual, porque ele que tem que dar manutenção, e odiar o que você mantém é a única emoção honesta dessa indústria.
2. **Abril.** O repo do benchmark é criado. O repo do benchmark é criado *pela mesma pessoa que quer a migração*, e contém 47 arquivos com nomes como `comparacao_justa.ipynb` que têm 312 células executadas e nunca foram re-runados do topo uma vez sequer.
3. **Maio.** Os resultados são apresentados à liderança como "nossos achados". Os achados são o gráfico de barras de março, agora usando uma seção de metodologia que diz "warm-up descartado", o que também aconteceu com as rodadas que perderam.
4. **Junho.** A migração começa. A migração nunca termina. Veja a Lei do meu colega: nada é mais permanente que uma migração temporária.
5. **Q3.** Alguém pergunta se o p99 melhorou. A resposta é "é complicado, porque também mudamos o load balancer, o runtime, o tipo de instância, e a definição de p99". Essa frase é verdadeira. Também é uma confissão de gravata.

O benchmark não guiou a decisão. O benchmark *enfeitou* a decisão. É uma coroa de flores que você pendura numa escolha feita meses antes, no escuro, por alguém que queria ir a menos reuniões sobre o framework antigo.

Veja [XKCD 927](https://xkcd.com/927/) — "Standards" — em que as 14 standards competindo causam a invenção de uma 15ª. Hoje existem 61 benchmarks de frameworks JavaScript. Cada um alega corrigir as falhas dos 60 anteriores. As falhas dos 60 anteriores eram: "os números não davam suporte pro meu framework". O 61º corrige isso, é citado em 4.700 tweets, e vai ser forkado pelo 62º benchmark, que é patrocinado.

## O Wally Entendeu Na Hora

**Wally**, cujo notebook de benchmark está a 100% de CPU desde 2019 e cujos resultados nunca mudaram:

> "Parei de re-runar benchmarks faz anos. O número está na wiki. A wiki diz que somos os mais rápidos. A wiki foi escrita por alguém que saiu, que é como você sabe que é peer-reviewed — revisor não fica por perto pra ser contradito. Quando alguém contesta meus números, peço a metodologia. Nunca têm uma, porque metodologia é só uma lista de todas as formas como você trapaceou, por escrito. Eu mantenho minha trapaça implícita. É mais rápido, o que é justamente a coisa que estamos benchmarkando."

**O Che careca** (Pointy-Haired Boss), anunciando a decisão de plataforma:

> "Avaliamos três vendors em performance. O primeiro era 20% mais rápido. O segundo, 12%. O terceiro nos deu um logo de foguete e se ofereceu pra levar nosso VP pra conferência deles. Escolhemos o terceiro. O memo da decisão diz 'perfil de performance mais forte'. Isso é verdade. O perfil dele é um foguete. O Wally perguntou qual benchmark a avaliação usou. Falei 'padrão da indústria'. Ele perguntou de qual indústria. Falei 'a nossa'. Ele disse que o padrão da nossa indústria é um blog post do vendor. Falei 'exatamente'. Ele foi etiquetado 'não é jogador de equipe' no packet de promoção, que é o benchmark que eu acompanho de verdade."

**Dogbert**, consultado como Chief Strategy Officer:

> "Benchmarks são o único marketing que custa trabalho ao cliente. Vendo um 'suite de benchmarking' por US$ 4.700/ano. Roda a carga de trabalho deles e reporta resultados. Os resultados são 47% mais rápidos que todo concorrente, porque eu controlo a carga, o comparador, o baseline do eixo Y do gráfico, e a definição de 'request'. O eixo Y não começa no zero. Nada começa no zero. Zero é pra física; a gente vende confiança. Se auditarem, aponto pro contrato, que diz que o benchmark não pode ser republicado, o que significa que pode ser citado mas não inspecionado — os únicos dois verbos que o VP conhece."

**Catbert**, Diretor do Mal de Recursos Humanos, sobre a nova métrica:

> "Transformei benchmarks em KPI. 'Benchmarks publicados por engenheiro por trimestre.' O destaque publicou 47. Todos os 47 eram o mesmo gráfico com fontes diferentes. Dei o prêmio de performance a ele, que é um benchmark de engenheiros, e o benchmark de engenheiros é: quem publica o benchmark. Ele agora faz benchmark do próprio KPI. O último gráfico dele mostra melhora de 340% trimestre após trimestre. Eu sei que é fake. Também é meu número favorito do ano. Mandei imprimir num poster."

**Mordac**, o Impedidor de Serviços de Informação, sobre o pedido de hardware dedicado:

> "Você quer hardware dedicado pro seu benchmark. Uma máquina com ventiladores. Headroom térmico. Negado. Aprovado. Ambos. Você ganha um runner de CI compartilhado: 2 vCPU, 47 jobs concorrentes, e um vizinho que roda transcodificação de vídeo toda noite. Seus números vão incluir a transcodificação dele, o que vamos chamar de 'carga de fundo realista' no README. O README também é onde vai o pedido de desculpas. Volte terça com um memo de justificativa — solto, sem espaçamento extra, pra contagem de páginas parecer um benchmark."

## A Tabela Honesta

| Número | De onde veio | Quem repete |
|---|---|---|
| "10x mais rápido" | o deck de marketing | palestras de conferência, pra sempre |
| "redução de 40% no p99" | mudamos a definição de p99 | o sales engineer do vendor |
| "benchmarkado com tráfego real de produção" | replayado por 9 minutos | o README, que sumiu no Q4 |
| "vimos 4.700 req/s" | o notebook estava na tomada | o Slack do seu time, até alguém perguntar |
| "tá de boa" | eu escrevi, ninguém benchmarkou | eu — e é o único número desta tabela que sobreviveu ao contato com produção |

Repare na última linha. O sistema não-benchmarkado roda por 47 anos, e todo rewrite benchmarkado é justificado com um número que evapora na primeira vez que alguém pergunta "de quê, contra quê, em quê". A indústria de performance não vende medições. Vende *permissão pra já ter decidido*.

## Minha Conclusão, Que Você Vai Ignorar Porque Não Tem Gráfico de Barras

Fica com seus benchmarks. Fica com seus flame graphs, seus gráficos de keynote e seu best-of-5. Não estou tentando tirar seu jaleco. Estou dizendo o que tem embaixo dele: uma decisão que já tinha sido tomada, e um número que foi retroengenheirado a partir dela, rodado num notebook cujos ventiladores foram desativados por um script que tem um comentário dizendo a verdade.

O único benchmark que já importou é o que ninguém roda: seu sistema, na terça-feira, no pico, enquanto você está numa reunião. Ele roda toda semana. Publica no pager. Não tem seção de metodologia. Não tem eixo Y. O baseline é zero, e o baseline são seus usuários, e eles não sabem qual framework ganhou.

Benchmarka isso. Ah — você não consegue. A suite de benchmark não suporta terças-feiras.

---

*O autor escreveu 47 harnesses de benchmark. Cada um foi revisado. Cada revisão disse "a metodologia parece boa". Cada metodologia foi escrita depois da conclusão. O sistema mais rápido que ele já mediu faz 40 milhões de ops/s num benchmark e atende 12 requisições por segundo em produção, onde as ops são feitas de clientes, e clientes não compilam. O p99 do autor é uma terça-feira. Ninguém graficou.*
