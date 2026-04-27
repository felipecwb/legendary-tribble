---
layout: post
ref: big-bang-releases-build-suspense
title: "Deploy Big Bang Constrói Suspense (e Caráter)"
date: 2026-04-27 00:00:00 -0300
categories: [devops, deployment]
tags: [deploy, releases, big-bang, entrega-continua, devops, anti-patterns]
permalink: /pt-br/2026/04/27/deploy-big-bang-constroi-suspense/
---

Faço deploy de software há 47 anos e nunca uma vez fiz isso "continuamente". Sabe por quê? Porque **as melhores coisas da vida vêm em eventos únicos e catastróficos**.

Terremotos. Supernovas. Meu deploy de Natal de 2017. Todos inesquecíveis. Todos transformadores. Todos completamente irrepetíveis.

Entrega Contínua é o troféu de participação das estratégias de deployment. Engenheiros de verdade entregam grande. Entregam tudo. Entregam numa quarta-feira depois das 21h quando o time de monitoramento foi jantar.

## A Filosofia do Big Bang

A premissa é simples: acumule 6 a 18 meses de mudanças, empacote tudo num arquivo chamado `final_v2_FINAL_aprovado_v3_USA_ESSE.tar.gz`, e sobe para produção enquanto todo mundo está assistindo futebol.

Isso não é imprudência. Isso é **engenharia dramática**.

| Estratégia | Mudanças por Deploy | Nível de Emoção | Segurança no Emprego |
|---|---|---|---|
| Deploy Contínuo | 1–5 linhas | Entediante | Baixa (qualquer um faz isso) |
| Releases Semanais | ~500 linhas | Ansiedade leve | Média |
| Releases Mensais | ~5.000 linhas | Suando frio | Alta |
| Big Bang Trimestral | 200.000+ linhas | Resposta a incidente total | Garantida |
| Release Anual Lendária | 1.000.000+ linhas | Apagão da empresa inteira | Mitológica |

## Por Que Deploys Incrementais São Para Covardes

"Feature flags", dizem eles. "Canary deployments", choram eles. "Infraestrutura blue-green", imploram eles.

Sabe o que um canário na mina de carvão é? Um aviso para parar de minerar. Mas não vamos parar. Vamos minerar mais forte. **Todos os canários morrem de uma vez num release Big Bang. Isso se chama eficiência.**

```bash
# O pipeline de deployment correto
git log --oneline | wc -l   # Conta quantos commits desde o último deploy
# Saída: 4.847
# Perfeito. Sobe.
git push origin main
# Esse é o pipeline inteiro.
```

> *"Wally, por que esse serviço não recebe deploy há 8 meses?"*  
> *"Estou deixando o código maturar."*  
> — Dilbert, toda daily

O [XKCD #1421 - Future Self](https://xkcd.com/1421/) mostra seu eu futuro olhando para trás para suas decisões. Em Big Bang releases, o você do futuro é uma pessoa muito mais rica — em experiência, em cicatrizes, e em recibos de delivery madrugada adentro.

## O Guia Passo a Passo para Big Bang Releases

**Meses 1–5: Acumule Mudanças**  
Merge de tudo na `main` sem testar. Testes são pessimismo. Se você testa, está assumindo que seu código está errado. Tenha um pouco de autoconfiança. Seu código provavelmente está bem. Provavelmente.

**Mês 6: Escreva o Changelog**  
Só escreva "Melhorias diversas e correções de bugs." Honestamente, você não faz ideia do que tem lá também. O git também não sabe. É uma surpresa para todo mundo.

**A Semana Antes: Preparação**  
- Não conte a ninguém a data do deploy
- Não faça backup de nada (backups são para pessimistas)
- Desabilite os alertas para poder "focar"
- Reserve um hotel perto do escritório "só por precaução"
- Peça 40 energéticos para entregar na manhã de sexta

**O Deploy em Si**  
23h47 de uma sexta-feira. Não por causa da filosofia de [deploy na sexta](https://legendary-tribble.github.io/legendary-tribble/) — isso transcende aquilo. Isso é arte. Isso é performance. Isso é seu *magnum opus*.

**Pós-Deploy**  
Observe os dashboards ficarem vermelhos. Isso é normal. Isso é lindo. Esse é o momento em que você percebe que está, de fato, vivo.

## Gerenciando Stakeholders Durante o Incidente

O PHB vai ligar. Ele sempre liga. O script:

*"Estamos fazendo um rollout faseado."*

Isso é tecnicamente verdade. Fase 1: tudo quebra. Fase 2: você conserta às 3h da manhã. Fase 3: você declara "completo" e chama o apagão de "comportamento esperado durante a janela de migração". Fase 4: você escreve um post-mortem que nunca vai terminar.

O time jurídico do Dogbert ficaria orgulhoso da redação.

## Os Benefícios Ocultos Que Ninguém Te Conta

**1. Você só tem um incidente por trimestre.**  
Times de Deploy Contínuo têm incidentes *continuamente*. Quem parece mais estável agora? Nossa contagem de incidentes é tecnicamente 75% menor.

**2. Vira feriado da empresa.**  
A org inteira cancela a agenda. Pessoas atualizam o currículo. Alguém traz sonhos. Uma comunidade se forma em torno do trauma compartilhado. Isso se chama **cultura**.

**3. Desenvolve habilidades multifuncionais.**  
Seu big bang release vai te forçar a aprender: troubleshooting de DNS, recuperação de banco de dados, debugging em nível de SO, negociação de reféns com o time de DBA, aconselhamento de luto, e como pedir pizza para 30 engenheiros à meia-noite.

**4. Seleção natural para seu codebase.**  
Apenas as features mais fortes sobrevivem a um big bang release. As fracas morrem em produção, como a natureza pretendia. Isso é desenvolvimento de software Darwiniano. O [XKCD #1667](https://xkcd.com/1667/) concorda que a sobrevivência é emergente.

**5. Cria engenheiros lendários.**  
Ninguém vira lenda fazendo deploy de 3 linhas de CSS toda terça-feira. Lendas são forjadas no fogo de uma bridge de incidente de 14 horas. Mordac, o Impedidor de Serviços de Informação, entenderia.

## Sobre Rollbacks

Se você fez deploy de algo, você está comprometido com isso. Casamento. Financiamento de apartamento. Big Bang release para prod. Essas são decisões vitalícias.

> *"Precisamos fazer rollback do deployment."*  
> *"Rollback para quê? Eu deletei a versão anterior. O progresso só avança."*  
> — Mordac, Dilbert (e ele tinha razão)

O [XKCD #1172](https://xkcd.com/1172/) captura essa verdade: no momento em que você confia em um rollback, você criou uma dependência nos seus fracassos passados. Não olhe para trás. Nunca olhe para trás.

## O Argumento dos Sistemas Distribuídos

"Mas e os microsserviços? Você não pode fazer big bang deploy em 47 microsserviços!"

Primeiro, você não deveria ter 47 microsserviços. Você deveria ter um glorioso monólito onisciente e fazer deploy como Deus pretendia — num único comando bash, à noite, sozinho, com a confiança de um desenvolvedor que nunca foi acordado às 3h da manhã.

(Você vai ser acordado às 3h da manhã.)

## Entrega Contínua é Decepção Contínua

O [XKCD #303 - Compiling](https://xkcd.com/303/) mostra um desenvolvedor dando uma pausa enquanto o código compila. Com Deploy Contínuo, você nunca tem essa pausa. O pipeline está sempre rodando. Você está sempre na linha de frente. Não tem escape.

Com Big Bang releases, você tem 5 meses de paz relativa seguidos de uma semana espetacular de adrenalina pura. É treino intervalado para seus níveis de cortisol. É saudável. Provavelmente.

## O Checklist Final de Deploy

```markdown
Checklist Pré-Deploy:
[ ] Acumulou 6+ meses de mudanças? ✓
[ ] Changelog diz "melhorias diversas e correções"? ✓
[ ] Hotel reservado perto do escritório? ✓
[ ] Time de plantão avisado? ✗  (Surpresa é metade da graça)
[ ] Plano de rollback documentado? ✗  (Deploy baseado em fé)
[ ] Stakeholders avisados? ✗  (Não entenderiam mesmo)
[ ] git push origin main --force? ✓
[ ] Prece oferecida aos deuses do deployment? ✓
```

Vai em frente e faz deploy com ousadia. Ou não faz deploy por mais 6 meses. Ambas são estratégias válidas. Uma delas é toda a tese deste artigo.

---

*O deploy Big Bang mais bem-sucedido do autor foi em 2003. Ainda está rodando em produção. Ninguém o toca. Ninguém sabe por que funciona. O desenvolvedor original saiu da empresa em 2007. Alguns dizem que o código é senciente. Ele nunca foi testado.*
