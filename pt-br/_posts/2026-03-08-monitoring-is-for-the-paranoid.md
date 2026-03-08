---
layout: post
ref: monitoring-is-for-the-paranoid
title: "Monitoramento é Para Paranóicos: Se Funciona, Funciona"
date: 2026-03-08 09:30:00 -0300
categories: [devops, anti-patterns]
tags: [monitoramento, observabilidade, alertas, devops, producao]
permalink: /pt-br/:year/:month/:day/monitoring-is-for-the-paranoid/
---

Toda semana aparece uma nova ferramenta de "observabilidade" prometendo mostrar o que está acontecendo no seu sistema. Datadog, New Relic, Prometheus, Grafana, Honeycomb, Lightstep, SignalFx. Sabe o que todas essas ferramentas têm em comum? **Elas mostram problemas que você não sabia que existiam e não precisava saber**.

Em meus 47 anos produzindo bugs em massa, eu aprendi uma verdade fundamental: **Se os usuários não estão reclamando, o sistema está funcionando**.

## O Paradoxo do Monitoramento

A questão sobre monitoramento é: quanto mais você monitora, mais problemas você encontra. E quanto mais problemas encontra, mais trabalho você tem. É uma armadilha.

| Nível de Monitoramento | Problemas Encontrados | Sono Perdido | Valor Pro Negócio |
|-----------------------|----------------------|--------------|-------------------|
| Nenhum | 0 | 0 horas | Máximo |
| Básico | 5 | 2 horas | Algum |
| Abrangente | 47 | 6 horas | Negativo |
| "Observabilidade Total" | 847 | Todo | Você pede demissão |

A correlação é clara: mais monitoramento é igual a menos felicidade.

## Minha Filosofia de Monitoramento

```bash
#!/bin/bash
# monitor.sh - Solução de monitoramento enterprise

curl -s https://meuapp.com > /dev/null
if [ $? -eq 0 ]; then
    echo "Tá de boa"
else
    echo "Provavelmente tá de boa"
fi
```

É isso. É toda a stack de monitoramento. Sem agentes. Sem coletores. Sem dashboards. Só vibes.

## O Complexo Industrial de Dashboards

Sabe o que são dashboards? São protetores de tela pros times de operações. "Olha todos esses gráficos se movendo! Definitivamente estamos fazendo algo!"

Deixa eu traduzir o que essas métricas de dashboard realmente significam:

| Métrica | O Que Dizem | O Que Significa |
|---------|-------------|-----------------|
| CPU: 75% | "Carga alta!" | O servidor está fazendo seu trabalho |
| Memória: 80% | "Pressão de memória!" | RAM está sendo usada (bom, você pagou por ela) |
| Disco: 60% | "Enchendo!" | Arquivos existem |
| Taxa de erro: 2% | "Alerta!" | 98% das requisições funcionam perfeitamente |
| Latência p99: 2s | "Lento!" | 99% das requisições levam menos de 2 segundos (ok) |

Toda métrica pode ser pintada como um problema. É assim que vendedores de monitoramento ficam no negócio.

## Alertas São Privação de Sono

Sabe o que PagerDuty realmente é? É um serviço de assinatura de ansiedade. Você paga R$150 por usuário por mês pra ser acordado às 3 da manhã porque CPU bateu 85% por 30 segundos.

```
3:00 AM: ALERTA - Uso de memória acima de 80%
3:01 AM: ALERTA - Uso de memória acima de 80%
3:02 AM: ALERTA - Uso de memória acima de 80%
3:03 AM: Você agora está acordado, irritado, e incapaz de dormir
3:04 AM: Memória estabiliza em 79%
3:05 AM: RESOLVIDO - Uso de memória normal
4:30 AM: Você finalmente volta a dormir
6:00 AM: Acorda pro trabalho, exausto
9:00 AM: Comete um erro em produção porque cansado
9:01 AM: ALERTA - Pico de taxa de erro
```

O monitoramento causou o incidente. Pense sobre isso.

## O Que Dilbert Me Ensinou

Wally uma vez explicou sua estratégia de monitoramento: "Eu configuro todos os thresholds de alerta em 100%. Assim, nada nunca alerta, e eu nunca sou acionado."

O Chefe Cabeça Pontuda perguntou, "Mas e se algo der errado?"

Wally respondeu, "Aí os usuários vão nos dizer. Eles são muito bons nisso."

Esse é o caminho.

## A Verdade XKCD

[XKCD 1831](https://xkcd.com/1831/) nos mostra que somos terríveis em entender o que é realmente importante. Você vai monitorar o connection pool do banco mas perder que alguém renomeou o botão de login pra "Sair."

Monitoramento dá falsa confiança. Você acha que está vigiando tudo, mas está vigiando o que você pensou em medir. Os problemas reais estão sempre em outro lugar.

## Monitoramento de Usuário: O Único Monitoramento Que Importa

Aqui está minha stack de monitoramento:

```
Atendimento ao Cliente:  "Usuários estão reclamando?"
                                  |
                                  v
                          +-------+-------+
                          |               |
                         Sim             Não
                          |               |
                          v               v
                  "Arruma alguma coisa"  "Entrega features"
```

Esse sistema tem 100% de taxa de verdadeiro positivo. Se usuários estão reclamando, existe um problema real. Se não estão reclamando, não existe.

Compare isso com Prometheus, que tem 12% de taxa de verdadeiro positivo e 100% de taxa de me acordar por nada.

## O Custo da Observabilidade

Vamos fazer a matemática de uma configuração típica de monitoramento:

```
Datadog: R$75/host/mês × 50 hosts = R$3.750/mês
New Relic: R$125/host/mês × 50 hosts = R$6.250/mês
Armazenamento de logs: R$2.500/mês
Fadiga de alertas: 10 horas/semana × R$150/hora = R$6.000/mês
Café pra ficar acordado de plantão: R$400/mês

Custo mensal total: R$18.900

Alternativa: Confia nos usuários mandarem email
Custo mensal total: R$0

Economia anual: R$226.800
```

Isso é o salário de um desenvolvedor sênior que você está gastando em dashboards que ninguém olha.

## Quando Monitoramento Realmente Ajuda

1. Nunca
2. Quando você precisa provar pra gestão que a queda não foi sua culpa
3. Quando você precisa de gráficos pro post-mortem
4. Ainda nunca pra realmente prevenir problemas

## O Zen de Não Monitorar

```
Sem monitoramento.
Sem alertas.
Sem dashboards.
Sem problemas.

Se crasha, reinicia.
Se continua crashando, mais RAM.
Se isso não funciona, reescreve.
Se isso não funciona, outro emprego.
```

Essa filosofia me sustentou por décadas. O sistema funciona ou não funciona. Ficar olhando gráficos não vai mudar isso.

## Conclusão

Monitoramento é um mecanismo de enfrentamento pra organizações que não confiam no seu código. Se seu código fosse bom, você não precisaria vigiá-lo tão de perto.

Entrega código confiante. Confia que funciona. Vai dormir em horário razoável.

---

*Os sistemas de produção do autor não têm monitoramento. Quando perguntado como ele sabe que estão funcionando, ele diz "Eu não ouço gritos." Isso tem sido preciso 97% das vezes.*
