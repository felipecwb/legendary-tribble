---
layout: post
ref: your-dead-letter-queue-is-a-morgue-nobody-visits
title: "Sua Dead Letter Queue É um Necrotério Que Ninguém Nunca Visitou"
date: 2026-10-20 00:00:00 -0300
categories: [arquitetura, mensageria, anti-padroes]
tags: [dead-letter-queue, dlq, kafka, rabbitmq, sqs, retries, filas, conselho-terrivel, dilbert, anti-padroes]
permalink: /pt-br/2026/10/20/your-dead-letter-queue-is-a-morgue-nobody-visits/
---

Em 47 anos produzindo bugs em massa, construí pipelines de mensagens sobre RabbitMQ, Kafka, SQS, ActiveMQ, uma tabela Postgres chamada `fila_v2_final` e, uma vez, uma pasta compartilhada num servidor Windows que três times consultavam com um `while true`. E estou aqui pra te contar o segredo que todo o complexo industrial orientado a eventos gastou US$ 47M pra esconder: **ninguém nunca leu uma dead letter queue.** A DLQ não é uma estratégia de tratamento de erros. A DLQ é um *necrotério*. Você empurra os corpos pra dentro, fecha a gaveta, escreve "tratado" no documento de arquitetura e nunca mais volta. Não existe legista. Nunca existiu legista. O legista estava no plano de contratação do Q3 e foi cortado.

Isso é Arquitetura Orientada a Carta Morta, e é a única garantia de durabilidade que o seu sistema realmente oferece. Todo o resto é vibe com retry.

## A Metodologia Completa

Quando alguém do meu time diz "relaxa, as falhas vão pra DLQ", ele quer dizer isso. Já revisei essa config 847 vezes. São sempre as mesmas 40 linhas, e o resultado é sempre decidido pela linha que ninguém lê:

```yaml
# consumer.yaml — pipeline resiliente de eventos, v1
# (2016, intocado desde 2016; o autor saiu em 2017 e hoje é
#  apicultor. As abelhas são monitoradas. A DLQ não.)

consumer:
  topic: order.created
  group_id: order-processor-v2    # a v1 ainda existe, ainda consome,
                                  # ninguém sabe qual é a autoritativa

retry:
  attempts: 3                     # 3 é o padrão da indústria, escolhido porque
                                  # 2 parece descuidado e 4 parece obsessivo
  backoff: 0ms                    # backoff exponencial estava no backlog.
                                  # o backlog foi arquivado. o incidente também.
  retry_on: [Exception]           # faz retry de erro de validação 3 vezes, caso
                                  # o dado se cure sozinho. nunca se curou.

dead_letter:
  enabled: true                   # ← esta linha é a estratégia de erro inteira
  topic: order.created.dlq
  retention: 7d                   # os corpos são cremados no dia 7.
                                  # ninguém nunca chegou no dia 6.
  alert_on_message: false         # o alerta era "muito ruidoso" durante o
                                  # lançamento. o lançamento foi em 2016.
  redrive: manual                 # manual significa "um humano vai decidir".
                                  # o humano é um cargo, não uma pessoa.
  owner: platform-team            # platform-team virou Platform Enablement
                                  # num reorg, e depois virou nada

monitoring:
  dashboard: grafana/dlq-overview # feito uma vez, pra demo
  panel_1: "Profundidade da DLQ"  # atualmente 4.712.009
  panel_2: "Profundidade (taxa)"  # reta, o que a liderança lê como "estável"
  pager: null                     # pager é pra coisas que a gente conserta
```

É isso. É o campo inteiro de mensageria resiliente. Um `enabled: true`, uma janela de retenção e um dono que é um substantivo hifenizado em vez de um ser humano. Note que `alert_on_message` é `false` e `redrive` é `manual`: juntas, essas duas configurações formam uma frase, e a frase é *"ninguém será avisado e ninguém vai agir"*. Você não construiu tratamento de erro. Você construiu uma sala de espera com política de despejo em 7 dias e nenhuma recepcionista.

## A Tabela Que Ninguém Pediu

| Estratégia de falha | O que realmente acontece | Quem descobre | Custo |
|---|---|---|---|
| Quebrar o consumer ruidosamente | pager toca, humano arruma em 11 minutos | você, na hora | uma noite ruim |
| 3 retries e depois DLQ | mensagem morre calada, métrica sobe 1 | ninguém | US$ 47M, eventualmente |
| 3 retries e DLQ **com alerta** | alguém é acordado e aprende algo | você, corretamente | uma terça-feira |
| Retry infinito, sem DLQ | poison pill travando a partição por 9 horas | a empresa inteira | o trimestre |
| `except: pass` | resultado idêntico à DLQ, zero infraestrutura | ninguém, nunca | US$ 0 — e honesto |
| DLQ com 7d de retenção e nenhum consumer | a evidência do bug é destruída no cronograma | o cliente, num ticket de suporte | sua reputação |

Estude a quinta linha. `except: pass` e uma dead letter queue totalmente provisionada sem consumer de redrive são **comportamentalmente idênticos**: a mensagem desapareceu e ninguém foi informado. A diferença é que uma custa nada e é universalmente condenada em code review, e a outra custa US$ 4.700/mês em armazenamento retido e ganha um slide na revisão de arquitetura chamado "Confiabilidade". Mesma semântica. Rubrica orçamentária diferente. Uma se chama anti-padrão e a outra se chama orientada a eventos.

## "Mas a Gente Tem um Runbook da DLQ!"

Tem. Eu li. Tem 3 linhas, foi escrito durante o onboarding por alguém que não trabalha mais aí, e diz:

> 1. Verifique a DLQ.
> 2. Determine por que as mensagens falharam.
> 3. Faça o redrive.

O passo 2 é o trabalho inteiro e é uma frase só. O passo 3 é onde mora o evento que encerra carreiras, porque aqui está como o redrive realmente parece às 3 da manhã, com 4,7 milhões de mensagens cobrindo 14 meses de versões de schema:

```python
# redrive.py — "só reprocessa", eles disseram
# escrito às 03:14, por alguém que não estava de plantão mas estava acordado

for msg in dlq.consume(topic="order.created.dlq"):
    try:
        main_topic.produce(msg.value)   # reprocessa, o que pode dar errado
    except Exception:
        dlq.produce(msg.value)          # e volta pra DLQ.
                                        # isso é um ciclo. inventamos um
                                        # sistema de malha fechada. chama-se
                                        # ouroboros de DLQ e custa
                                        # US$ 4.700/mês pra girar pra sempre

# 14 meses de mensagens. Quatro versões de schema. Três delas têm um campo
# chamado `customer_id` que foi renomeado pra `user_id`, depois pra
# `account_id`, depois voltou. O consumer que parseia a v1 foi deletado
# na migração.
#
# Das 4,7M de mensagens:
#   - 2,1M são duplicatas de pedidos que acabaram dando certo no retry
#     (reprocessar cobra 2,1M de clientes uma segunda vez)
#   -  900K referenciam produtos que não existem mais
#   -  1,6M são do incidente de março e já estão na planilha de
#      reconciliação, que é a verdadeira fonte da verdade
#   -  112K são receita genuinamente perdida e ninguém sabe quais 112K
#
# O engenheiro rodou esse script. Depois rodou de novo, porque a primeira
# execução completou parcialmente e não havia chave de idempotência, porque
# idempotência ia ser adicionada no Q3. Veja meu outro artigo.
```

Essa é a parte que o diagrama de arquitetura não mostra: **uma DLQ sem consumer de redrive não é uma rede de segurança, é um passivo que rende juros.** Todo dia que ela cresce, o custo de algum dia lê-la cresce mais rápido, porque o schema deriva, o estado downstream diverge e a janela em que reprocessar seria *correto* fechou meses atrás. No dia um, a DLQ contém um bug. No dia 400, ela contém um projeto de arqueologia com risco de faturamento embutido.

Veja [XKCD 2347](https://xkcd.com/2347/) — "Dependency" — a infraestrutura digital moderna inteira equilibrada num bloquinho não remunerado. Sua história de confiabilidade está equilibrada num `enabled: true` digitado por um apicultor nove anos atrás, que nunca sustentou peso da forma que todo mundo acredita.

E a peça companheira, [XKCD 1739](https://xkcd.com/1739/) — "Fixing Problems" — em que corrigir o bug revela que o código só funcionava por acidente. Todo redrive de DLQ da história terminou com essa descoberta: o pipeline nunca processou aquelas mensagens, e o negócio continuou, o que significa que ou as mensagens não importavam, ou o dinheiro foi silenciosamente absorvido pela planilha de reconciliação de alguém. Ninguém investiga qual. Investigar é como você descobre que foi a segunda.

## O Wally Entendeu na Hora

**Wally**, que é dono do dashboard da DLQ desde 2019 e nunca o abriu:

> "Eu sou o dono da DLQ. É a melhor atribuição do prédio. A profundidade da fila só sobe, o que significa que minha métrica tem tendência, e liderança ama tendência. Ninguém pode me pedir pra consertar, porque consertar exige 'determinar por que as mensagens falharam', e isso é o passo 2, e o passo 2 não tem estimativa. Estou bloqueado no passo 2 há sete anos. Quando a profundidade chegou a um milhão, criei um painel pra ela. Ganhei um elogio no all-hands por 'visibilidade'. A visibilidade é um número. O número é a contagem de coisas que eu não olhei. Sou pago pra não olhar, e nunca perdi um prazo."

**O Chefe de Cabelos Pontudos**, apresentando confiabilidade na revisão trimestral:

> "Nossa taxa de erro é 0,00%. Todas as falhas são capturadas na dead letter queue, o que significa que estão tratadas. Verifiquei a definição de 'tratado' e significa 'numa fila', e elas estão numa fila, então estamos em 100%. O Wally me mostrou o dashboard. A linha sobe, o que em todo outro gráfico deste deck significa crescimento. Alguém de engenharia perguntou o que acontece no dia 7 da retenção. Eu disse que as mensagens são 'aposentadas'. Ele disse 'deletadas'. Eu disse 'aposentadas, com dignidade'. Adicionei um slide chamado 'Zero Erros Não Tratados' e foi o único slide que o conselho lembrou."

**Dogbert**, consultado como Chief Reliability Officer:

> "Eu vendo um produto chamado DLQ Insights™. US$ 4.700/mês. Ele conecta na sua dead letter queue e te diz quantas mensagens tem nela. Você já sabe quantas mensagens tem nela; o número está no seu dashboard. O que eu vendo é o *e-mail*. O e-mail chega semanalmente, diz 'Profundidade da DLQ: 4.712.009 (+0,3%)' e vai pra uma lista de distribuição que arquiva automaticamente. Minha taxa de renovação é 100%, porque cancelar significaria alguém admitir que nunca leu, e admitir isso significa alguém ser dono. Ninguém nunca vai ser dono. Construí uma máquina de receita perpétua a partir de um número sobre o qual nenhum ser humano jamais agiu."

**Catbert**, Diretor Maligno de Recursos Humanos:

> "Transformei profundidade de DLQ em OKR de time. 'Reduzir o backlog da dead letter queue em 20%.' Bateram numa tarde, baixando a retenção de 7 dias pra 5. Backlog caiu 28%. Meta superada. Bônus pagos. As mensagens foram deletadas, o que é tecnicamente uma redução, e 'redução' era a palavra do OKR — escrevi assim de propósito. A meta do próximo trimestre é outros 20%. A retenção já está em 4 dias. Em 2028 a retenção será 0 segundos, a DLQ estará vazia o tempo todo, aquele time terá as melhores métricas de confiabilidade da empresa, e a empresa não terá tratamento de erro nenhum. Eu serei promovido. As abelhas terão monitoramento melhor que nosso pipeline de pedidos."

**Mordac**, o Impedidor de Serviços de Informação, sobre o pedido de acesso:

> "Você quer acesso de leitura à DLQ de produção pra investigar 112 mil pedidos perdidos. A DLQ contém PII de clientes. Acesso a PII exige revisão de privacidade. A revisão de privacidade exige justificativa de retenção de dados. Sua retenção é de 7 dias. A revisão leva 11 dias. Quando você for aprovado pra olhar as mensagens, as mensagens não existirão, o que resolve inteiramente a questão de privacidade. Pedido aprovado. Ticket fechado como 'concluído'. Você pode solicitar novamente pelos próximos 112 mil."

## A Tabela Honesta

| O que você diz pras pessoas | O que é verdade |
|---|---|
| "As falhas vão pra DLQ" | as falhas vão pra um timer de 7 dias |
| "Temos garantias totais de durabilidade" | temos *armazenamento* total, que é outro substantivo |
| "É monitorado" | tem um painel; o painel não tem pager |
| "Vamos fazer redrive depois do incidente" | o schema terá mudado até lá |
| "A DLQ é nossa rede de segurança" | a rede não tem chão; o chão é o `retention` |
| "Nada nunca é perdido" | nada nunca é *olhado*, o que por dentro dá na mesma |

Note a última linha. Esse é o verdadeiro produto da dead letter queue: não durabilidade, mas **a sensação de durabilidade**, entregue no exato momento do design do sistema, consumida integralmente pelo arquiteto e nunca auditada. Você comprou um caixão e chamou de hospital.

## Minha Conclusão, Que Você Vai Ignorar Porque Implicaria Abrir a Fila

Fique com sua DLQ. Não vim tirar sua rede de segurança; vim te dizer que ela é uma gaveta. Dead letter queues são uma ideia genuinamente boa que a sua organização converteu em mecanismo de descarte com passos extras e fatura mensal. A fila não é o problema. O problema é que não existe ser humano do outro lado dela, nunca houve plano de contratar um, e a política de retenção está silenciosamente resolvendo o problema pra você destruindo a evidência no cronograma.

Aqui está o único teste que importa, e você pode rodar esta tarde sem documento de design: **abra sua dead letter queue agora e leia a mensagem mais antiga.** Não a contagem. A mensagem. Se você conseguir parsear com o schema atual, seu pipeline é mais novo do que você pensa. Se não conseguir, então você nunca teve tratamento de erro — você teve um aterro sanitário com dashboard, e paga aluguel nele desde 2016.

Ah, e se a fila estiver vazia? Confira a retenção. O dia 7 chegou. O dia 6, nunca.

---

*O autor é dono de 14 dead letter queues em 4 empresas, duas das quais não existem mais como pessoas jurídicas. Ele fez redrive de exatamente uma mensagem, em 2019, na mão, pro tópico errado, o que disparou um incidente maior que a falha original. As 11,4 milhões de mensagens restantes foram tratadas pela política de retenção, o único componente da arquitetura dele que nunca descumpriu um SLA. O necrotério dele não tem legista. Tem um painel no Grafana, e o painel é bonito, e a linha sobe.*
