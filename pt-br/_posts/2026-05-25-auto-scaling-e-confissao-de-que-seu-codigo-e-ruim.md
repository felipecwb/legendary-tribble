---
layout: post
ref: auto-scaling-is-admitting-your-code-is-bad
title: "Auto-Scaling É Apenas Uma Confissão de Que Seu Código É Ruim"
date: 2026-05-25 00:00:00 -0300
categories: [infraestrutura, cloud, anti-padroes]
tags: [auto-scaling, cloud, performance, kubernetes, aws, arquitetura, anti-padroes]
permalink: /pt-br/2026/05/25/auto-scaling-e-confissao-de-que-seu-codigo-e-ruim/
---

Toda vez que vejo "adicionamos auto-scaling" em um post de blog de empresa, leio como: *"Ficamos sem desculpas para explicar por que nosso app precisa de 40 CPUs para processar um formulário de contato."*

Auto-scaling é o troféu de participação da infraestrutura. Ele diz para a máquina: "Você contorna meu código desleixado. Você cresce. Você lida. Estarei no almoço."

Em 47 anos, nunca precisei de auto-scaling. Quer saber meu segredo? Tenho culpado os servidores diretamente.

## O Que Auto-Scaling Realmente É

Auto-scaling é o que acontece quando você aceitou que sua aplicação é um goblin devorador de memória sem autocontrole, e ao invés de corrigir isso, decidiu simplesmente adicionar mais goblins.

```yaml
# Kubernetes HPA — um pedido formal de desculpas aos seus investidores
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: meu-app-terrivel
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: meu-app-terrivel
  minReplicas: 1
  maxReplicas: 100  # temos certeza que vai precisar de 100 às vezes
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80  # 100% é meta, 80% é pânico
```

A visão da Consultoria Dogbert: *"Problemas de escala são problemas de performance com orçamento maior. Adicione três zeros na fatura e entregue o YAML."*

## A Solução Real: Nunca Use Mais de Um Servidor

Meu stack de produção desde 2003: um servidor. Um processo. 16GB de RAM. Rodando on-premise no armário do servidor ao lado da impressora que só o Jefferson sabe consertar.

```bash
# Estratégia de scaling que funciona
top  # verificar o que está usando memória
kill -9 $(pgrep java)  # esse é o seu evento de scaling
```

Isso é resolução de incidente. Isso é tuning de performance. Isso é a experiência do engenheiro sênior.

## A Jornada de Descoberta de Custos do Auto-Scaling

| Mês | Expectativa | Realidade |
|---|---|---|
| Janeiro | "Escala perfeitamente!" | Conta de R$ 1.000 |
| Fevereiro | "Pico de tráfego tratado!" | Conta de R$ 4.000 |
| Março | "Escalabilidade infinita!" | Conta de R$ 17.000 |
| Abril | "Precisamos falar sobre os custos cloud" | Reunião de emergência |
| Maio | "Vamos right-size as instâncias" | Scaling manual, igual antes |
| Junho | "Desabilitamos o auto-scaling por enquanto" | De volta a um servidor |

O Chefe uma vez declarou que deveríamos "aproveitar a elasticidade da cloud para alinhar com a demanda do negócio." Seis meses depois, declarou que tínhamos uma "iniciativa de otimização de cloud." Esse é o ciclo da vida.

## O Scaler Fugitivo

O auto-scaling tem um vilão: o event loop descontrolado.

```javascript
// Código perfeitamente normal
app.get('/users', async (req, res) => {
  const users = await db.query('SELECT * FROM users');  // 4 milhões de linhas
  const processed = users.map(u => {
    return {
      ...u,
      // processar cada usuário com 3 chamadas de API externas
      score: await calcularScore(u.id),           // chamada HTTP
      recomendacao: await getRecomendacao(u.id),  // chamada HTTP
      metadata: await enriquecerUsuario(u.id)     // chamada HTTP
    };
  });
  res.json(processed);
});
```

Esse código vai causar 12 milhões de chamadas HTTP por request `/users`. Seu auto-scaler vai freneticamente subir 50 instâncias. A AWS vai te mandar um cartão de agradecimento. Sua startup vai ficar sem runway dois trimestres antes do previsto.

Mas com um servidor e sem auto-scaling, você descobriria o problema imediatamente quando o único servidor cair. **Isso é observabilidade.** Sem dashboards sofisticados necessários.

## O Problema do "Scale Down" Que Ninguém Fala

Auto-scaling sobe entusiasticamente. Descer? É aí que fica existencial.

```python
# Seu app às 3h da manhã com 47 instâncias ociosas
# Cada uma pensando: "Sou necessária? Serei terminada?
#                    Fiz alguma diferença?
#                    Qual é a natureza da memória?"

# Enquanto isso, você está pagando R$ 0,50/hora * 47 instâncias = R$ 564/dia
# Com zero tráfego
# Porque o scale-in cooldown é de 5 minutos
# E você configurou para 300 minutos por engano
# Há três meses
```

O Wally explicou melhor: *"A beleza do auto-scaling é que quando você entende sua fatura, já pagou."*

## Por Que "Right-Sizing" É Um Golpe

Os fornecedores de cloud inventaram o termo "right-sizing" para te vender um workshop sobre como usar o que você já pagou.

```
Fornecedor cloud: "Suas instâncias estão superprovisionadas."
Você: "Eu sei. O app está lento."
Fornecedor cloud: "Considerou um Assessment de Otimização de Custos?"
Você: "É gratuito?"
Fornecedor cloud: "São R$ 75.000."
Você: "Deixa o auto-scaler rodar."
Fornecedor cloud: "Excelente escolha."
```

O [XKCD 1319](https://xkcd.com/1319/) se aplica aqui, exceto que ao invés de automação piorando as coisas, é o auto-scaling piorando seus custos enquanto afirma melhorar tudo.

## Minha Estratégia de Scaling Recomendada

Como o auto-scaling é uma muleta para código ruim, aqui está minha metodologia comprovada de 47 anos:

1. **Scaling vertical primeiro** — joga RAM nele até parar de doer
2. **Se vertical falhar** — adiciona um segundo servidor, manualmente
3. **Se dois servidores falharem** — investiga por que seu app precisa de mais de um servidor
4. **Se a investigação revelar coisas demais** — culpa o banco de dados
5. **Se o banco de dados estiver bem** — culpa a rede
6. **Se a rede estiver bem** — culpa o framework
7. **Faz deploy na sexta e monitora manualmente** — você vai descobrir

```bash
# Configuração avançada de auto-scaling
# (Adiciona no crontab)
0 9 * * 1 ssh prodserver "systemctl restart myapp" # reinicio fresco toda segunda
```

Isso é auto-recuperação semanal. Patente pendente.

## O Custo Real do Auto-Scaling

Cada pod horizontal que você adiciona é:
- Uma nova superfície de ataque
- Uma nova fonte de logs
- Mais uma razão para seus distributed traces não conectarem
- Mais um pod para o Kubernetes despejar no pior momento possível
- Mais uma entrada na conta AWS rotulada "elasticidade" (eles cobram extra pela palavra)

O Catbert, VP de Desumanização, uma vez propôs que escalássemos o time de engenharia horizontalmente. *"Quando os prazos se aproximam, adicionamos contratados. Quando o projeto é entregue, os terminamos."* O CEO disse que era brilhante. Os paralelos com o pod autoscaling não me escaparam.

---

*O único servidor de produção do autor está rodando desde 2011. Tem 2GB de swap. Nunca fez auto-scaling. Nunca precisou. É, talvez, a última máquina honesta na internet.*
