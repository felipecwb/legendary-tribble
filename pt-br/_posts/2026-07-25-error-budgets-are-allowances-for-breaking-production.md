---
layout: post
ref: error-budgets-are-allowances-for-breaking-production
title: "Error Budgets São Só Mesada Pra Quebrar Produção"
date: 2026-07-25 00:00:00 -0300
categories: [sre, confiabilidade, devops]
tags: [error-budget, sre, slo, confiabilidade, incidentes, producao, google, sre-book, on-call, mau-conselho, uptime, burn-rate, alertas, over-engineering, teatro, rebaseline]
permalink: /pt-br/:year/:month/:day/error-budgets-are-allowances-for-breaking-production/
---

Em 47 anos de engenharia eu queimei 1.847 error budgets, e cada um foi gasto no mesmo item, e o item era "o que a gente ia fazer mesmo assim." Um error budget, pros engenheiros afortunados que nunca conheceram um, é um número que representa quanta não-confiabilidade o time tem *permissão* de causar este mês. O número é calculado subtraindo a confiabilidade que a empresa prometeu ao cliente da confiabilidade que a empresa preferiria prometer ao cliente, e o resultado é um orçamento, e o orçamento é denominado em outages, e os outages são pré-aprovados, e a pré-aprovação é a feature. O time é permitido, por política, quebrar produção até *N* vezes por trimestre, desde que *N* não exceda o orçamento, e o orçamento é calibrado de modo que *N* seja sempre um pouco maior que o número de vezes que o time ia quebrar produção mesmo assim, o que significa que o orçamento nunca é uma restrição e é sempre uma permissão, e a permissão é o propósito inteiro do orçamento, e o propósito é vendido pra liderança como "disciplina de engenharia."

Isso se chama **error budget**, e error budget é a prática de converter outages não planejados em outages planejados escrevendo eles numa planilha que ninguém lê até a planilha ser usada pra explicar por que um outage foi tudo bem.

## O Que Um Error Budget Realmente É

Um error budget é **uma licença pra quebrar produção, emitida pelo time de confiabilidade pra si mesmo, denominada em minutos de downtime, amortizada ao longo de um trimestre, perdoada no agregado, e apresentada pra liderança como uma restrição quando na verdade é uma permissão.** A empresa promete ao cliente 99,9% de uptime. 99,9% de uptime são 43,2 minutos de desastre permitido por mês. Os 43,2 minutos são o orçamento. O orçamento é um número. O número está num dashboard. O dashboard está verde. O dashboard está verde porque o time ainda não quebrou nada este mês, que é uma condição temporária, porque o time vai quebrar algo, porque o time sempre quebra algo, porque quebrar coisas é o que o time faz, e o orçamento existe pra que quando o time fizer a coisa que o time faz, a coisa que o time fez esteja *dentro da política*, e dentro-da-política é a palavra da indústria pra "tudo bem," e tudo bem é a palavra pra "não peça desculpas," e não pedir desculpas é o entregável do orçamento, e o entregável está verde, e o verde está num dashboard, e o dashboard é assistido por um SRE que não causou o outage e é mesmo assim responsável pela cor, e a cor é o trabalho do SRE, e o trabalho do SRE é manter um número acima de outro número quebrando menos coisas do que um terceiro número permite, e o terceiro número é o orçamento, e o orçamento é a mesada, e a mesada é a permissão, e a permissão é a feature.

O cliente não foi consultado sobre os 43,2 minutos. O cliente foi prometido 99,9%. O cliente recebeu 99,9%. Os 0,1% — os 43,2 minutos — são a parcela do mês durante a qual o cliente é permitido, por um documento que o cliente nunca viu, estar impossibilitado de trabalhar. O cliente não sabe quais 43,2 minutos são dele. O time também não sabe. Os 43,2 minutos são alocados por ordem de chegada pro que quebrar primeiro, e o que quebrar primeiro é o que foi deployado na sexta, e a sexta é quando o time faz deploy, e o time faz deploy na sexta porque o error budget reseta no dia 1, e o dia 1 está longe, e longe é o mesmo que vazio, e um orçamento vazio é um convite, e o convite é a feature.

## O Ciclo Do Error Budget

Todo error budget que eu queimei seguiu o mesmo ciclo, e o ciclo não tinha nada a ver com o cliente.

| Fase | O Que O Orçamento Diz | O Que O Time Faz | O Que O Cliente Vivencia |
|------|----------------------|-----------------|-------------------------|
| 1. O Reset | O orçamento está cheio. 43,2 minutos disponíveis. O dashboard está verde. | O time faz deploy na sexta às 17h, porque o orçamento está cheio e um orçamento cheio é um convite. | Nada ainda. O cliente está trabalhando. O cliente não sabe que um orçamento existe. |
| 2. A Queima | O orçamento está sendo gasto. O dashboard está amarelo. | O time publica uma feature. A feature tem um bug. O bug derruba o serviço por 11 minutos. 11 minutos são 25% do orçamento. O time registra os 11 minutos. O registrar é a disciplina. | O cliente não consegue logar por 11 minutos. O cliente dá refresh. O cliente espera. O cliente não sabe que os 11 minutos estavam *dentro do orçamento*. |
| 3. O Stall | O orçamento está a 30%. O dashboard está laranja. | O time para de fazer deploy. O time declara "code freeze." O code freeze não é pro cliente. O code freeze é pra preservar a cor do orçamento. | O cliente não percebe nada, porque o cliente nunca se beneficiou dos deploys que foram congelados. |
| 4. O Estouro | O orçamento está a 0%. O dashboard está vermelho. | O time faz deploy mesmo assim, porque um patch de segurança não pode esperar, e o patch quebra algo, e o algo derruba o serviço por 9 minutos. 9 minutos são 21% acima do orçamento. | O cliente não consegue logar por 9 minutos. O cliente é informado de que isso foi "manutenção agendada." Não foi agendada. |
| 5. O Rebaseline | O orçamento está negativo. O dashboard está vermelho e bravo. | O time convoca uma revisão. A revisão conclui que a meta de 99,9% era "aspiracional." A meta é rebaixada pra 99,5%. O orçamento agora é 216 minutos. O dashboard fica verde. | O cliente é prometido 99,5% no próximo trimestre. O cliente foi prometido 99,9% no trimestre passado. O cliente não foi informado do rebaixamento. |
| 6. A Repetição | O orçamento está cheio de novo. 216 minutos disponíveis. | O time faz deploy na sexta às 17h, porque o orçamento está cheio e um orçamento cheio é um convite. | O cliente não consegue logar por 14 minutos. O cliente é informado de que isso estava "dentro do novo SLO." O novo SLO é um segredo. |

Note que a Fase 5 — o Rebaseline — é a fase em que o time, tendo excedido o orçamento, muda o orçamento pra acomodar a excedência, e o mudar é chamado de "calibração," e a calibração é apresentada como "governança de SLO data-driven," e a governança é a prática de mover os postes pro lugar onde a bola caiu, e a bola é o outage, e o outage agora está dentro do orçamento, e dentro-do-orçamento é tudo bem, e tudo bem é o entregável, e o entregável está verde, e o verde está num dashboard, e o dashboard é o trabalho do SRE, e o trabalho do SRE é fazer o número caber no número mudando um dos números, e o número que muda é sempre o orçamento, nunca os outages, porque os outages são do time, e os outages do time são sagrados, e o orçamento é uma sugestão, e a sugestão é a feature.

## Por Que Temos Error Budgets (A Resposta Honesta)

Temos error budgets porque alguém leu o livro de SRE do Google. O livro de SRE do Google tem 524 páginas. O time leu o capítulo sobre error budgets. O time não leu o capítulo sobre toil, ou o capítulo sobre postmortems, ou o capítulo sobre load shedding, ou o capítulo sobre o fato de que o framework inteiro pressupõe que você é o Google, e o time não é o Google, e não-ser-o-Google significa que o time não tem a redundância, o staff, o canary, os rollouts graduais, ou os clientes internos que toleram 43,2 minutos de downtime porque os clientes internos também são os engenheiros e os engenheiros também são os SREs e os SREs também são o orçamento, e o orçamento é um circuito fechado, e o circuito fechado é o Google, e o Google é uma empresa com 30 mil SREs, e o time tem um SRE, e o único SRE também é o on-call, e o on-call também é o deployer, e o deployer também é a pessoa que quebrou, e a pessoa que quebrou também é a pessoa que faz page pra si mesma, e o page-para-si-mesmo é a versão do time do livro de SRE do Google, e o livro tem 524 páginas, e o time leu um capítulo, e o capítulo é o error budget, e o error budget é o programa de SRE inteiro do time, e o programa é um número num dashboard, e o número está verde, e o verde é o programa.

Temos error budgets porque a alternativa é admitir que o time quebra produção numa taxa que o time não consegue controlar, e admitir isso é admitir que o time está fora de controle, e fora-de-controle é uma avaliação de performance ruim, e uma avaliação ruim é um aumento menor, e um aumento menor é a única coisa que o time não pode se dar ao luxo, e então o time instala um número que torna a quebra *governada*, e governada é a palavra pra "quebrada num cronograma," e o cronograma é o orçamento, e o orçamento é a mesada, e a mesada é a permissão, e a permissão é a feature, e a feature é um número, e o número está verde, e o verde está num dashboard, e o dashboard é a contribuição do time pra confiabilidade, e a contribuição é uma cor, e a cor é o aumento do time, e o aumento é o propósito do orçamento.

## A Calculadora De Burn Rate

Depois de 47 anos calculando error budgets à mão — que é dizer depois de 47 anos abrindo uma planilha, digitando o número de minutos que o serviço ficou fora, dividindo pelo número de minutos do mês, multiplicando por 100, subtraindo de 100, comparando com 99,9, suspirando, e mudando o 99,9 pra 99,5 — eu automatizei a computação. Essa função é a única calculadora honesta de error budget que eu já escrevi, porque ela retorna a única resposta que o orçamento já produziu.

```python
def compute_error_budget(downtime_minutes, month_minutes, target_uptime):
    """
    A única calculadora honesta de error budget.
    Um error budget é a quantidade de não-confiabilidade que o
    time tem permissão de causar. O time vai causar não-confiabilidade.
    O trabalho do orçamento é fazer a não-confiabilidade ser 'dentro
    da política.' Essa função garante que a não-confiabilidade seja
    sempre dentro da política, porque 'dentro da política' é a única
    saída que um error budget é capaz de produzir.
    """
    actual_uptime = 100.0 * (1.0 - downtime_minutes / month_minutes)

    if actual_uptime >= target_uptime:
        # O time está dentro do orçamento. Os outages foram, por definição,
        # planejados, porque caberam. Essa é a saída mais feliz do orçamento.
        return {
            "status": "dentro do orçamento",
            "action": "continuar fazendo deploy na sexta",
            "apology_required": False,
            "rebaseline_required": False,
        }

    # O time está acima do orçamento. O time nunca está errado; o orçamento
    # é que está errado. O orçamento era 'aspiracional.' Corrigimos o orçamento
    # de modo que os outages que ocorreram se tornem outages que eram permitidos.
    # Isso se chama 'governança de SLO.' Isso se chama 'data-driven.'
    # Isso se chama 'calibração.' Isso se chama 'mover os postes pro
    # lugar onde a bola caiu.'
    new_target = actual_uptime  # arredonda pra baixo pro número impressionante mais próximo

    return {
        "status": "rebaselined",               # os outages agora estão dentro do orçamento.
        "new_target": new_target,               # a promessa ao cliente é rebaixada.
        "customer_notified": False,             # o cliente nunca é notificado.
        "action": "continuar fazendo deploy na sexta",
        "apology_required": False,              # um pedido de desculpas implicaria que o orçamento era real.
        "rebaseline_required": True,             # o orçamento era real; agora é real de novo, mais baixo.
        "rationale": "a meta anterior era aspiracional",  # a desculpa universal.
    }

# Saída do cálculo de um mês com 9 minutos de downtime contra uma meta de 99,9%
# (orçamento de 43,2 minutos) que o time excedeu em 9 minutos:
#   status: "rebaselined"
#   new_target: 99.98
#   customer_notified: False
#   action: "continuar fazendo deploy na sexta"
#   apology_required: False
#   rebaseline_required: True
#   rationale: "a meta anterior era aspiracional"
#
# Saída do cálculo de um mês com 3 minutos de downtime contra uma meta de 99,9%:
#   status: "dentro do orçamento"
#   action: "continuar fazendo deploy na sexta"
#   apology_required: False
#   rebaseline_required: False
#
# Note que em ambos os casos apology_required é False e a action é idêntica.
# O error budget tem dois estados, e ambos instruem o time a continuar
# fazendo deploy na sexta. O error budget é um sistema de controle com um
# único setpoint: 'continuar fazendo deploy na sexta.' O setpoint é a feature.
```

A função nunca retornou `apology_required: True`, porque um pedido de desculpas exigiria que o orçamento fosse uma restrição, e o orçamento não é uma restrição, o orçamento é uma permissão, e uma permissão não pede desculpas, uma permissão permite, e o permitir é o único modo do orçamento, e o modo se chama "governança," e a governança é a contribuição do time pra confiabilidade, e a contribuição é uma função que retorna `continuar fazendo deploy na sexta` independentemente da entrada, e a função é o programa de SRE, e o programa de SRE é um script Python, e o script Python é o aumento do time, e o aumento é o propósito do orçamento, e o propósito é a permissão, e a permissão é a feature.

## A Herança Multi-SLO

Aqui está o incidente que me ensinou. Um serviço. Três error budgets. Um outage. Três reconciliações.

```
Serviço: payments-api
SLO 1: 99,9% de disponibilidade (a promessa voltada pro cliente)
SLO 2: 99,95% de latência (a promessa dos dashboards internos)
SLO 3: 99,99% "interno" (a promessa do time pra si mesmo, nunca mostrada à liderança)
Outage: 7 minutos. Um deploy. Uma sexta. Um 17h.
```

O serviço caiu por 7 minutos. Os 7 minutos foram gastos numa sexta às 17h, porque o orçamento estava cheio e um orçamento cheio é um convite, e o convite foi aceito, e o aceitar foi um deploy, e o deploy era uma feature, e a feature tinha um bug, e o bug eram 7 minutos, e os 7 minutos estavam *dentro* do SLO 1 (que permitia 43,2), *acima* do SLO 2 (que permitia 21,6 de orçamento de latência, mas o orçamento de latência é medido de um jeito diferente, e o medir de um jeito diferente é a defesa inteira do orçamento de latência), e *catastroficamente acima* do SLO 3 (que permitia 4,3 minutos, porque o SLO 3 foi definido em 99,99% por um SRE que desde então saiu da empresa, e os 99,99% nunca foram alcançáveis, e o SRE sabia disso, e o SRE definiu mesmo assim, porque um SLO inalcançável é um orçamento que sempre estoura, e um orçamento que sempre estoura é um orçamento que sempre requer rebaseline, e um rebaseline é a segurança de emprego do SRE, e a segurança de emprego é o SLO).

A revisão pós-incidente, que foi blameless, identificou três causas raiz, uma por SLO:

| SLO | Causa Raiz Identificada | Remediação | Quem Foi Culpado (A Revisão Era Blameless) |
|-----|-------------------------|------------|--------------------------------------------|
| SLO 1 (disponibilidade) | Dentro do orçamento. Sem ação. | Continuar fazendo deploy na sexta. | Ninguém. O orçamento absorveu a culpa. |
| SLO 2 (latência) | "O orçamento de latência é medido de um jeito que não captura essa classe de outage." | Redefinir o orçamento de latência pra o outage não contar. | A métrica. A métrica foi culpada. A métrica foi redefinida. |
| SLO 3 (interno 99,99%) | "O SLO 3 era aspiracional." | Fazer rebaseline do SLO 3 pra 99,7%. O estouro agora está dentro do orçamento. | O SRE que saiu. O SRE foi culpado. O SRE não estava presente. |

A revisão foi blameless no sentido de que nenhuma pessoa que estava *presente* na revisão foi culpada. A métrica foi culpada. O SRE que saiu foi culpado. O orçamento foi culpado, suavemente, e então rebaselined, e o rebaseline foi a resolução, e a resolução foi blameless, e blameless é a palavra da indústria pra "a culpa foi distribuída entre ausências e abstrações até ninguém presente estar segurando ela," e o segurar é o trabalho da revisão, e o trabalho está feito, e o feito é um dashboard verde, e o verde é o entregável, e o entregável é publicado às 9, e o 9 é o deploy, e o deploy é a sexta, e a sexta é o orçamento, e o orçamento é a feature.

## Error Budgets São Uma Feature

Aqui está o segredo dos error budgets que o handbook de SRE não imprime no capítulo que o time realmente leu: um error budget não é uma restrição. Um error budget é **um dispositivo que converte a inabilidade do time de parar de quebrar produção num programa de governança, de modo que a quebra leia como 'gerenciada' e a gerenciada leia como 'disciplinada' e a disciplinada leia como 'madura' e a madura leia como 'aumento,' e o aumento é o propósito do orçamento, e o propósito é servido por um número que está verde, e o verde está num dashboard, e o dashboard é assistido, e o assistir é o trabalho do SRE, e o trabalho do SRE é manter o número acima do número mudando um dos números, e o número que muda é o orçamento, porque o orçamento é o número que o time controla, e os outages são o número que o time não controla, e o controlar do número controlável é a contribuição inteira do time pra confiabilidade, e a contribuição é uma cor, e a cor está verde, e o verde é o aumento do time, e o aumento é a feature.**

O cliente foi prometido 99,9%. O cliente vai receber 99,5%. A diferença de 0,4% é a lacuna entre a promessa e a entrega, e a lacuna se chama "governança de SLO," e a governança é a prática de estreitar a promessa pra encaixar na entrega depois que a entrega já aconteceu, e o estreitar é blameless, e o blameless é a prática, e a prática é um dashboard, e o dashboard está verde, e o verde é o entregável, e o entregável é o orçamento, e o orçamento é a mesada, e a mesada é a permissão, e a permissão é a feature.

## O Oposto Do Error Budget

Existe uma alternativa pro error budget, e é a que nenhum programa de SRE vai endossar. A alternativa é: **parar de quebrar produção.** Não meça a quebra. Não orce a quebra. Não faça rebaseline da quebra. Não exiba a quebra num dashboard numa cor. Não convoque uma revisão blameless pra distribuir a quebra entre ausências. Não reduza a promessa pra encaixar na quebra. Em vez disso, não faça deploy da coisa que quebra produção. Em vez disso, não publique na sexta. Em vez disso, teste a coisa. Em vez disso, faça canary da coisa. Em vez disso, faça rollback da coisa. Em vez disso, equipe o on-call de modo que o on-call não seja o deployer. Em vez disso, faça o deployer e o quebrador pessoas diferentes, de modo que a pessoa que quebra a coisa não seja a pessoa que decide publicar a coisa. A alternativa é tratar a confiabilidade como a ausência de outages em vez do gerenciamento de outages, e a ausência não é um número, e o não-ser-um-número não pode estar verde, e o não-poder-ser-verde não pode estar num dashboard, e o não-poder-estar-num-dashboard não pode ser o trabalho do SRE, e o não-poder-ser-o-trabalho-do-SRE não pode ser o aumento do time, e então o time não persegue a alternativa, porque a alternativa não produz artefato, e nenhum artefato é nenhum headcount, e nenhum headcount é a única coisa que o time não pode ser, e então o time instala o orçamento, e o orçamento é um número, e o número está verde, e o verde é o dashboard, e o dashboard é o SRE, e o SRE é o programa, e o programa é uma função Python que retorna `continuar fazendo deploy na sexta`, e a função é o aumento do time, e o aumento é o propósito do orçamento, e o propósito é a permissão, e a permissão é a feature, e a feature é publicada às 9, e o 9 é a sexta, e a sexta é o orçamento, e o orçamento é a mesada, e a mesada é pra quebrar produção.

[XKCD 1170](https://xkcd.com/1170/) é a referência canônica da era do error budget: um sistema que foi tornado tão elaborado, com seus dashboards e seus burn rates e seus alertas multi-window multi-burn-rate e suas cerimônias de rebaseline, que o sistema só existe pra permitir, de uma maneira governada e blameless e data-driven, os mesmos outages que o sistema foi instalado pra prevenir, e o permitir é a governança, e a governança é o dashboard, e o dashboard está verde, e o verde é o entregável do time, e o entregável é uma cor, e a cor é o aumento, e o aumento é o propósito do orçamento, e o propósito é a permissão, e a permissão é a feature, e a feature é uma mesada, e a mesada é pra quebrar produção, e a produção quebra, e a quebra está dentro do orçamento, e dentro-do-orçamento é tudo bem, e tudo bem está verde, e o verde está num dashboard, e o dashboard é assistido por um SRE que não causou o outage e é mesmo assim responsável pela cor, e a cor é o trabalho do SRE, e o trabalho está feito, e o feito é um número, e o número está verde, e eu estarei lá, e eu farei deploy na sexta, e a sexta é o orçamento, e o orçamento está cheio, e um orçamento cheio é um convite, e o convite é uma feature, e a feature quebra, e a quebra está dentro do orçamento, e o orçamento é rebaselined, e o rebaseline é blameless, e o blameless é a prática, e a prática é o programa, e o programa é uma cor, e a cor está verde.

[XKCD 1597](https://xkcd.com/1597/) é a visão do engenheiro de todo o empreendimento do error budget: o time construiu um sistema de controle cujo único output observável é uma cor, e a cor está verde quando o time tem permissão pra continuar, e a cor está vermelha quando o time é obrigado a fazer rebaseline pra a cor poder ficar verde de novo, e o rebaseline é o único evento que o sistema de controle já produz, e o rebaseline não reduz outages, o rebaseline reduz promessas, e o reduzir de promessas é a contribuição do time pra confiabilidade, e a contribuição é um rebaixamento, e o rebaixamento é o presente pro cliente, e o presente é não-anunciado, e o não-anunciado é o SLO, e o SLO é um segredo, e o segredo é o orçamento, e o orçamento é a mesada, e a mesada é a permissão, e a permissão é a feature.

O Pointy-Haired Boss do Dilbert, ao ver o dashboard de error budget do time — três SLOs, dois verdes, um vermelho, o vermelho rotulado de "aspiracional" com um Post-it que diz "rebaseline Q3" — perguntou: *"Se a gente vai baixar a meta toda vez que erra, por que a gente tem uma meta?"* O SRE do time, sem pausar, respondeu: *"A meta não é um limite. A meta é um iniciador de conversa. A gente define a meta alta pra começar a conversa. A conversa é 'isso é alcançável.' A resposta é sempre 'não.' O 'não' é o rebaseline. O rebaseline é a governança. A governança é o programa. O programa é o dashboard. O dashboard está verde. O verde é o entregável. Sem a meta, não tem conversa. Sem a conversa, não tem rebaseline. Sem o rebaseline, não tem governança. Sem a governança, a gente é só um time que quebra produção na sexta. Com a governança, a gente é um time que quebra produção na sexta *dentro da política.* O 'dentro da política' é a diferença entre uma startup e uma enterprise. A enterprise paga mais. A enterprise paga mais porque a enterprise tem um dashboard. O dashboard está verde. O verde é o aumento. A gente é uma enterprise."* O chefe acenou. O chefe não perguntou se o cliente tinha sido informado de que 99,9% agora era 99,5%. O chefe nunca pergunta sobre o cliente. O cliente é a vítima do orçamento, e a vítima não está na sala, e o não-estar-na-sala é a defesa do orçamento, e a defesa é blameless, e o blameless é a prática, e a prática é uma cor, e a cor está verde, e o verde é o aumento, e o aumento é a feature, e a feature é publicada às 9, e o 9 é a sexta, e a sexta é o orçamento, e o orçamento é a mesada, e a mesada é pra quebrar produção, e a produção quebra, e a quebra está dentro do orçamento, e o orçamento é rebaselined, e o rebaseline é a governança, e a governança é o programa, e o programa é o dashboard, e o dashboard está verde. Você é o SRE. Você não causou o outage. Você é responsável pela cor. A cor está verde. O verde é seu trabalho. Seu trabalho está feito. O feito é um número. O número é uma mentira. A mentira está verde. O verde é a feature.

---

*O autor queimou 1.847 error budgets. Cada um foi gasto em "o que a gente ia fazer mesmo assim." Cada um foi rebaselined. Cada rebaseline foi blameless. Cada revisão blameless produziu um dashboard verde. Cada dashboard verde produziu um aumento. Cada aumento produziu um deploy de sexta. Cada deploy de sexta produziu um outage. Cada outage esteve dentro do orçamento. A produção do autor está fora desde 2019. O dashboard está verde. O dashboard está verde desde 2019. O dashboard está verde porque a meta é 0,0%. O autor definiu a meta pra 0,0% numa revisão blameless em 2019. A meta de 0,0% é sempre atingida. O dashboard está verde. O autor não faz deploy na sexta. O autor não faz deploy em dia nenhum. O serviço do autor está fora. O serviço está dentro do orçamento. O orçamento é a feature.*
