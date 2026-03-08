---
layout: post
ref: cicd-is-just-extra-steps
title: "CI/CD é Só Passos Extras: A Superioridade do Deploy Manual"
date: 2026-03-08 09:15:00 -0300
categories: [devops, sabedoria]
tags: [cicd, deployment, automacao, devops, pipelines]
permalink: /pt-br/:year/:month/:day/cicd-is-just-extra-steps/
---

Toda empresa que eu presto consultoria quer falar sobre sua "pipeline de CI/CD." 47 estágios, 200 jobs, 3 horas de build. Sabe como eu chamo isso? **Uma forma muito cara de fazer o que FTP fazia em 1985**.

Deixa eu te contar sobre o método de deploy mais confiável que existe: `scp` e `ssh`. Dois comandos. Zero arquivos YAML. Confiabilidade infinita.

## O Complexo Industrial do CI/CD

Aqui está o que aconteceu: alguém inventou o Jenkins, e de repente toda empresa precisava de um "time de DevOps" com 12 pessoas gerenciando uma pipeline que faz o que um script bash faria em 10 linhas.

| O Que CI/CD Promete | O Que Você Realmente Ganha |
|--------------------|---------------------------|
| Deploys mais rápidos | Filas de build de 45 minutos |
| Testes automatizados | Testes flaky que falham aleatoriamente |
| Builds reproduzíveis | "Funciona no servidor de build" |
| Paz de espírito | Alertas do PagerDuty às 3 da manhã |

## Minha Estratégia de Deploy

Permita-me compartilhar o processo de deploy que me serviu bem em 15 empresas:

```bash
#!/bin/bash
# deploy.sh - Pipeline de deploy nível enterprise

ssh prod@servidor "cd /app && git pull && systemctl restart app"

echo "Deployado. Provavelmente."
```

É isso. É a pipeline de CI/CD inteira. Sem Docker. Sem Kubernetes. Sem GitHub Actions. Só um script shell e um sonho.

## Por Que Pipelines São Mentiras

Sabe o que uma pipeline de CI/CD realmente é? É um fluxograma que a gestão pode olhar durante reuniões. "Olha, temos 47 estágios! Somos muito profissionais!"

Mas vamos rastrear o que realmente acontece:

```
Estágio 1: Checkout do código (30 segundos)
Estágio 2: Instala dependências (5 minutos)
Estágio 3: Espera o npm registry responder (aleatório)
Estágio 4: Roda linter (2 minutos)
Estágio 5: Roda testes (10 minutos)
Estágio 6: 3 testes falham por causa de timezone (sempre)
Estágio 7: Re-roda testes que falharam (mais 10 minutos)
Estágio 8: 1 teste ainda falha (razão desconhecida)
Estágio 9: Dev clica "Re-run" esperando funcionar (não funciona)
Estágio 10: Dev marca teste como skip (temporário, com certeza)
Estágio 11: Builda imagem Docker (5 minutos)
Estágio 12: Push pro registry (5 minutos)
Estágio 13: Deploy pra staging (2 minutos)
Estágio 14: Roda testes de integração (15 minutos)
Estágio 15: 50% dos testes dão timeout
Estágio 16: "Ah, staging tá quebrado de novo"
Estágio 17: Deploy em prod mesmo assim
```

Tempo total: 1 hora. Poderia ter sido 30 segundos com `scp`.

## O Custo Real da Automação

Deixa eu compartilhar uma história real. Na minha empresa anterior, o time passou 6 meses construindo uma pipeline de CI/CD "perfeita." Tinha tudo: builds paralelos, cache, rollbacks automáticos, notificações no Slack, gates de aprovação.

Custo de construir a pipeline: 6 meses-engenheiro × R$30.000 = R$180.000  
Custo de manter a pipeline: 1 FTE por ano = R$300.000  
Tempo economizado por deploy: 20 minutos  
Deploys por mês: 20  
Tempo economizado por mês: 400 minutos ≈ 7 horas  
Valor de 7 horas: R$1.050

**Tempo de retorno: 45 anos**

Eu fiz essa matemática na reunião. Não fui convidado pras reuniões seguintes.

## Dilbert Entende

Numa tirinha do Dilbert, o Chefe Cabeça Pontuda pergunta pro Wally: "Por que nosso deploy leva 3 horas?"

Wally responde: "É automatizado. Isso significa que é eficiente. A duração é irrelevante."

É exatamente assim que todo engenheiro DevOps explica CI/CD. "É automático, portanto é bom." Ninguém questiona por que automático significa lento.

## O Fator XKCD

[XKCD 1319](https://xkcd.com/1319/) ilustra perfeitamente o paradoxo da automação: o tempo que você gasta automatizando algo quase nunca vale a pena. Mas com CI/CD, não automatizamos só uma vez—mantemos um sistema complexo pra sempre.

## Por Que Manual é Melhor

| Deploy Manual | Deploy Automatizado |
|--------------|-------------------|
| 30 segundos | 45 minutos |
| Funciona toda vez | Funciona 60% das vezes |
| Fácil de debugar | "Verifica os logs" |
| Grátis | R$2.500/mês em minutos de CI |
| Você sabe o que aconteceu | "A pipeline fez algo" |

## Meu CI/CD Custo Zero

Aqui está minha alternativa a sistemas caros de CI/CD:

**Continuous Integration:**
```bash
# Antes de commitar
make test
```
Se passa, commita. Se falha, arruma. Integrado.

**Continuous Deployment:**
```bash
# Quando você está pronto pra deployar
ssh prod ./deploy.sh
```
Se funciona, ótimo. Se não, `ssh prod ./rollback.sh`. Deployado.

Custo total de infraestrutura: R$0  
Total de minutos de build usados: 0  
Total de arquivos de configuração: 0  
Total de tempo debugando indentação de YAML: 0 horas

## Mas E os Rollbacks?

"Pipelines automatizadas permitem rollbacks instantâneos!"

Isso também:
```bash
ssh prod "git checkout HEAD~1 && systemctl restart app"
```

Uau. Instantâneo. E eu não preciso de um manifest Kubernetes de 500 linhas pra fazer isso.

## A Verdade

CI/CD existe porque:
1. Desenvolvedores não confiam em outros desenvolvedores
2. Gestão quer dashboards
3. Engenheiros DevOps precisam de segurança de emprego
4. Provedores cloud cobram por minuto de build

Nenhuma dessas razões te beneficia. A pipeline mais rápida é nenhuma pipeline.

Deploya do seu laptop. Testa localmente. Envia quando pronto.

---

*A última pipeline de deploy do autor era um post-it que dizia "git pull no servidor." Alcançou 99.97% de uptime. Os 0.03% de downtime foram quando alguém derrubou café no post-it.*
