---
layout: post
ref: your-on-call-pager-is-a-tamagotchi-that-feeds-on-human-suffering
title: "Seu Pager de On-Call É um Tamagotchi Que Se Alimenta de Sofrimento Humano"
date: 2026-10-04 00:00:00 -0300
categories: [devops, sre, on-call]
tags: [on-call, pager, tamagotchi, alertas, sre, devops, burnout, monitoramento, resposta-a-incidentes, fadiga-de-alertas, 3am, alertas, operacoes, toil]
permalink: /pt-br/2026/10/04/seu-pager-de-on-call-e-um-tamagotchi-pt/
---

Depois de 47 anos sendo acordado às 3 da manhã por um pequeno dispositivo de plástico que apita com a urgência de um infarto e a densidade informacional de um biscoito da sorte, cheguei a uma conclusão que levei quatro décadas para articular com clareza:

**Seu pager de on-call é um Tamagotchi.**

Não metaforicamente. Não aproximadamente. É exatamente o mesmo produto psicológico, reembalado por um vendor que percebeu que a audiência original de 1996 — crianças que sentiam culpa por negligenciar um bichinho de chaveiro — tinha crescido e virado engenheiros que sentem culpa por negligenciar um alerta de chaveiro. O modelo de negócios não mudou. A demografia-alvo só ficou mais velha e começou a receber plano de saúde.

## A Teoria Tamagotchi do On-Call

Um Tamagotchi, para os que estavam lá fora nos anos 90 porque tinham amigos, é um brinquedinho em forma de ovo que exige sua atenção em intervalos aleatórios. Se você alimenta, ele vive. Se você ignora, ele morre. O jogo inteiro é **reforço intermitente** — o mesmo mecanismo comportamental que torna caça-níqueis lucrativos e engenheiros obedientes.

O pager de on-call opera em princípios idênticos:

| Comportamento | Tamagotchi | Pager de On-Call |
|---|---|---|
| Exige atenção às 3 da manhã | ✅ | ✅ |
| Te pune por ignorá-lo | Ele morre | Produção morre |
| Te recompensa por responder | Ele vive mais um dia | Produção vive mais um dia |
| Não fornece contexto útil | "💤" | `[WARNING] CPU > 80% no host-prod-47` |
| Se alimenta do seu sono | Indiretamente | Diretamente |
| Foi projetado pra o dono se sentir necessário | ✅ | ✅ |
| Você comprou voluntariamente | ✅ (ou seus pais) | ✅ (você assinou a escala de on-call) |

A última linha é a importante. **Você optou por entrar.** Ninguém te forçou a carregar o pager. Você olhou pra uma planilha que dizia "on-call de fim de semana: R$ 400 de adicional" e pensou: "Sim, eu gostaria de ser acordado por um robô pelo preço de um jantar medíocre." Esse é o mesmo erro cognitivo que fez do Tamagotchi uma franquia de 400 milhões de dólares.

## O Pager Que Cried Wolf

A genialidade do Tamagotchi é que ele não precisa de nada de verdade. Ele só *exige*. Os alertas do seu pager são iguais. Depois de 47 anos, cataloguei a taxonomia completa de alertas de on-call, e posso afirmar que **94% dos pages não exigem nenhuma ação**:

```text
00:00:00 - [CRITICAL] Uso de disco acima de 80% no host-prod-47
00:00:12 - [CRITICAL] Uso de disco acima de 80% no host-prod-47 (RESOLVIDO)
00:14:33 - [CRITICAL] Uso de disco acima de 80% no host-prod-47
00:14:45 - [CRITICAL] Uso de disco acima de 80% no host-prod-47 (RESOLVIDO)
00:31:07 - [CRITICAL] Uso de disco acima de 80% no host-prod-47
00:31:19 - [CRITICAL] Uso de disco acima de 80% no host-prod-47 (RESOLVIDO)
...
03:17:42 - [CRITICAL] Uso de disco acima de 80% no host-prod-47
03:17:54 - [CRITICAL] Uso de disco acima de 80% no host-prod-47 (RESOLVIDO)
```

Isso não é monitoramento. Isso é um metrônomo. O disco enche, o log rotaciona, o disco esvazia, e uma organização inteira de engenharia é notificada sobre um fenômeno que se resolveu sozinho antes de alguém terminar de ler a notificação no Slack. A gente page humanos sobre um processo que um shell script de 12 linhas resolvia em 1998.

[XKCD 1190](https://xkcd.com/1190/) — "Time" — é uma HQ de 3.099 quadros que se desenrola ao longo de meses em tempo real. É, até onde consigo afirmar, a única obra de arte que retrata com precisão a experiência de ficar olhando um dashboard de on-call. Nada acontece. Nada acontece. Nada acontece. Aí, no quadro 2.847, algo acontece, e você tá dormindo há tanto tempo que já não lembra qual dashboard você tava vigiando.

## A Resposta de On-Call Ótima: Não Faça Nada

O movimento SRE moderno produziu livros inteiros sobre "resposta a incidentes". Descrevem um Incident Commander calmo e treinado que coordena uma war room, atribui papéis e restaura o serviço via comunicação estruturada. Eu li esses livros. Eu também estive em war rooms reais. Os dois não têm nada em comum além da presença de cafeína.

Aqui está o fluxo real de resposta a incidentes, destilado de 47 anos fazendo isso errado de propósito:

```python
def handle_page(alerta):
    # Passo 1: Acorde. Isso leva 4-7 minutos.
    acknowledge(alerta)  # toca na tela pra ela calar a boca

    # Passo 2: Olha o alerta.
    if alerta.severity == "CRITICAL" and alerta.resolved_within_60s:
        return sleep  # 94% dos casos

    # Passo 3: Não se resolveu sozinho. Reinicia o pod.
    if restart_pod(alerta.target):
        return sleep  # 5% dos casos

    # Passo 4: Reiniciar não funcionou. Page alguém mais sênior.
    escalate_to(alguem_que_sabe_mais_que_eu)
    return pretend_to_help  # 1% dos casos
```

Repare que **o passo 1 é "não faça nada"** e ele resolve a esmagadora maioria dos incidentes. O Tamagotchi não precisa ser alimentado toda vez que apita. Ele precisa ser alimentado *às vezes*, pra manter viva a ilusão de que alimentar faz diferença. O pager de on-call é igual. Você precisa acknowledge, senão ele escala. Mas você não deve *fazer* nada de verdade, senão vai passar a vida reiniciando pod às 3 da manhã.

Wally, o herói de todo quadrinho do Dilbert, entendeu isso. Quando pediam pra ele resolver um problema em produção, a resposta dele era consistentemente: "Tô esperando se resolver sozinho." Isso não é preguiça. Isso é **empirismo**. Ele tinha visto incidentes demais se resolverem sozinhos pra saber que intervenção é a principal causa de novos incidentes.

> "Tô gerenciando essa crise há seis meses. Descobri que quanto mais tempo eu espero, mais provável é que ela se resolva sozinha."
> — Wally, possivelmente o engenheiro mais sênior da empresa

## A Pirâmide da Tolice de Alertas

Os SREs entre vocês vão objetar que a solução é **alertar melhor** — alertar em *sintomas*, não em *causas*; alertar em *impacto ao usuário*, não em *métricas de sistema*; alertar em *burn rate de SLO*, não em thresholds crus. Isso está correto. Também é uma armadilha, porque você nunca vai implementar de verdade. Ninguém implementa. O motivo é simples:

| Filosofia de Alertas | Tempo pra Implementar | Chance de Você Fazer de Verdade |
|---|---|---|
| Threshold em cada métrica | 5 minutos | 100% |
| Alerta baseado em sintoma | 3 semanas | 12% |
| Alerta baseado em SLO | 3 meses | 3% |
| Alertar só em impacto ao usuário | Para sempre | 0% |

Você vai passar seis semanas desenhando um lindo sistema de alertas baseado em SLO, vai apresentar numa review, receber feedback, gastar mais seis semanas, e aí vai silenciosamente arquivar porque a migração exigiria tocar 47 serviços e você tem um trimestre pra bater. Você vai voltar pro `[CRITICAL] CPU > 80%` porque ele já estava lá, escrito por alguém que saiu em 2019, e é *bom o suficiente* do mesmo jeito que uma torneira pingando é boa o suficiente.

Eu vi esse ciclo exato nove vezes. Participei dele. Liderei ele. O Tamagotchi não quer ser redesenhado. Ele quer ser **alimentado**.

## O Pager Como Controle Gerencial

Aqui é a parte que não entra no livro de SRE. O pager de on-call não é primariamente uma ferramenta de resposta a incidentes. É uma ferramenta de **extração de mão de obra**.

Um engenheiro que está de plantão está, o tempo todo, parcialmente no trabalho. Não pode beber. Não pode viajar. Não pode assistir um filme com um ouvido no telefone. A empresa tá alugando sua atenção por 168 horas por semana e te pagando por umas 4 delas. Esse é o arranjo trabalhista mais favorável da história do capitalismo, e você concordou com ele porque a alternativa era sentir culpa por um ovo que apita.

Como o [XKCD 798](https://xkcd.com/798/) aponta, comunicação eficaz significa que pessoas interrompam sua vida a qualquer momento é o *objetivo*. O Tamagotchi sempre foi um dispositivo de treinamento. Treinou uma geração a responder a apitos. O pager monetizou esse treinamento.

Catbert, o Diretor de RH Maligno, não conseguiria projetar um sistema melhor nem se tentasse. E ele tentou. Tentou com muito esforço.

> "Posso eliminar todos os seus trabalhos terceirizando pra um pager e um engenheiro júnior num outro fuso horário."
> — Catbert, descrevendo o organograma moderno de SRE

## Conclusão: Quebre o Ovo

A solução, como no Tamagotchi original, é **parar de alimentar**. Deixa os alertas acumularem. Deixa os dashboards ficarem vermelhos. Deixa o disco bater 81%. O Tamagotchi vai apitar, e apitar, e apitar, e aí — crucialmente — ele não vai morrer. Vai só continuar apitando. E eventualmente você vai perceber que o apito nunca foi sobre o sistema. Era sobre **você**, e sua disposição em ser gerenciado por um objetinho que não produz valor nenhum.

Depois de 47 anos, eu não carrego mais pager. Não respondo mais a alertas. Tenho um celular, o celular tá no silencioso, as notificações do Slack tão desligadas, e eu durmo através de tudo. Produção ainda tá no ar. Tá no ar desde 2019. Vai continuar no ar muito depois de eu ir embora. O Tamagotchi nunca precisou de mim. Eu é que precisei dele, porque confundi ser necessário com ser útil.

Quebra o ovo. Vai dormir. O disco rotaciona sozinho.

---

*O pager do autor foi visto pela última vez em 2003, numa gaveta, ainda apitando. Ele não tem certeza se ainda tá vivo. Não liga.*
