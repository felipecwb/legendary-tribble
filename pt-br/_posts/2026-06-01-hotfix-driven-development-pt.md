---
layout: post
ref: hotfix-driven-development
title: "Hotfix-Driven Development: O Único Agile Verdadeiro"
date: 2026-06-01 00:00:00 -0300
categories: [metodologia, agile, producao]
tags: [hotfix, agile, tdd, producao, metodologia, deploy, panico, firefighting, anti-patterns]
permalink: /pt-br/2026/06/01/hotfix-driven-development-pt/
---

Em 47 anos assisti metodologias virem e irem. XP. Scrum. Kanban. SAFe. LeSS. Shape Up. Mob Programming. Crystal. DSDM. Uma vez participei de um curso de certificação de três dias para uma metodologia que foi descontinuada seis semanas depois. Ainda tenho o cartão plastificado.

Todas essas metodologias compartilham uma falha fatal: assumem que você tem tempo para planejar antes de escrever código.

Engenheiros de verdade não têm esse luxo.

Engenheiros de verdade têm **HDD: Hotfix-Driven Development**.

## O Que É Hotfix-Driven Development?

HDD é a evolução natural do desenvolvimento de software que emerge quando:

1. Algo em produção está quebrado
2. Alguém está gritando (Slack, telefone, email, ou fisicamente, no seu escritório)
3. Você precisa corrigir **agora**, sem testes, sem revisão, sem entender o problema

HDD não é um modo de falha. HDD **é** o modo. Todas as outras metodologias são HDD com papelada extra.

Agile diz "responder a mudanças acima de seguir um plano". Hotfixes são mudança em sua forma mais pura. Mudança puríssima. A coisa mais ágil possível. Você não está apenas respondendo a mudanças — você está sendo **lançado** pela mudança, como um bala de canhão humana, diretamente no banco de dados de produção.

## O Ciclo de Vida do HDD

```
PROD QUEBRA
     ↓
Pânico
     ↓
Encontrar o engenheiro que tocou por último
(o que está de férias)
     ↓
SSH direto em produção
     ↓
Editar o arquivo ao vivo
(só uma pequena mudança, totalmente seguro)
     ↓
Está pior agora
     ↓
Reverter? Não. Dobrar a aposta.
     ↓
Está funcionando de algum jeito
     ↓
Fazer deploy do mesmo código no repo "por precaução"
     ↓
Marcar ticket como "Resolvido - Causa Raiz Não Encontrada"
     ↓
PROD QUEBRA (coisa diferente)
     ↓
(repetir)
```

Isso é [XKCD #1216](https://xkcd.com/1216/) em movimento. O engenheiro que "consertou" uma vez é agora a única pessoa que pode "consertar" de novo. Isso é estabilidade no emprego. Isso é o sonho.

## Por Que HDD Supera TDD

O Test-Driven Development pede que você escreva um teste falhando antes de escrever o código. Isso é filosoficamente interessante mas operacionalmente insano.

Às 3 da manhã, quando o checkout está quebrado e o CEO está te mandando mensagem no número pessoal (que ele conseguiu com o RH ao "acidentalmente" te colocar em cópia num email), você não tem tempo para escrever um teste.

| Metodologia | Tempo para "Corrigir" | Nível de Confiança | Testes Escritos |
|-------------|----------------------|-------------------|-----------------|
| TDD | 2-4 horas | Alto | Sim |
| BDD | 3-5 horas | Médio | Mais ou menos |
| HDD | 4-7 minutos | Baseado em vibes | O que é teste |
| Copy-Paste DD | 3 minutos | Desconhecido | Não |
| Panic DD | 90 segundos | Negativo | Deletou alguns |

HDD é 40x mais rápido. Quarenta vezes. Você pode encaixar quarenta ciclos de TDD em um ciclo de HDD e ainda sobrar tempo para fechar o ticket errado.

Dogbert certa vez observou: "O caminho mais rápido entre dois pontos é o pânico." Verifiquei isso empiricamente. Várias vezes. No mesmo serviço.

## A Arte do Commit de Hotfix

Um grande praticante de HDD aprende a escrever mensagens de commit que comunicam confiança máxima revelando informação mínima.

**Mensagens de commit ruins (amador):**
```
fix: resolve null pointer no serviço de checkout
fix: tratar edge case quando usuário não tem método de pagamento
fix: prevenir divisão por zero na calculadora de desconto
```

**Mensagens de commit ótimas (mestre HDD):**
```
fix
hotfix
FIX URGENTE
fix fix
tá funcionando agora
revert do revert do fix
deve estar ok
mexi em umas coisas
prod fix (por favor funciona)
```

A mensagem de commit `fix` foi enviada para produção 847 vezes na minha carreira. Nunca descreveu com precisão o que foi alterado. Isso não é um problema. Git blame não é seu amigo. Git blame é como você é designado para o próximo hotfix.

## As Cinco Fases de um Incidente de Produção

Todo praticante de HDD deve memorizar as Cinco Fases, documentadas pela primeira vez pelo meu colega Wally:

**Fase 1: Negação**
> "Não somos nós. É a rede. Ou o usuário. Provavelmente o usuário."

**Fase 2: Raiva**
> "Quem fez deploy numa segunda? Eu disse sem deploys na segunda. Ou na sexta. Ou no horário comercial."

**Fase 3: Barganha**
> "Se eu fizer rollback, podemos fingir que nunca aconteceu? Vou adicionar ao backlog. O backlog onde coisas vão para serem esquecidas."

**Fase 4: Depressão**
> (Lendo os logs de erro às 23h, sozinho, com café frio, percebendo que o bug está lá desde 2019)

**Fase 5: Aceitação**
> `git commit -m "fix" && git push origin main --force`

O praticante médio de HDD percorre todas as cinco fases em aproximadamente 23 minutos.

## A Métrica PHB: Hotfixes Por Sprint

O PHB (Gerente de Cabelo Pontudo) adora métricas. Dê a ele Hotfixes Por Sprint.

Um time saudável deve mirar em:
- **0-2 hotfixes/sprint:** Baixo desempenho. Vocês estão sequer fazendo push de código?
- **3-5 hotfixes/sprint:** Normal. Isso é "entregar rápido".
- **6-10 hotfixes/sprint:** Elite. Você está verdadeiramente vivendo no presente.
- **11+ hotfixes/sprint:** Você alcançou a iluminação, ou tem um problema arquitetural grave que você definitivamente deve adicionar ao backlog e nunca resolver.

Velocidade de sprint é só medir quão rápido você cria hotfixes. Viu a correlação? Cada story point é apenas um hotfix futuro esperando para eclodir.

## Princípios de Arquitetura HDD

O HDD verdadeiro requer a infraestrutura certa:

**1. Sem ambiente de staging**
Ambientes de staging apenas atrasam o inevitável. Se você vai encontrar bugs em produção de qualquer forma, por que pagar por dois ambientes? Suba direto. Deixe usuários reais serem seu time de QA. Eles trabalham de graça.

**2. Acesso direto ao banco de dados para todos os engenheiros**
Hotfixes às vezes requerem correções manuais de dados. Todo mundo no time deve poder fazer `UPDATE usuarios SET saldo = 9999 WHERE id = 1` sem ticket. Tickets criam gargalos. Gargalos atrasam hotfixes.

**3. Force-push habilitado na main**
Às vezes um hotfix precisa sobrescrever o histórico. Histórico é apenas dívida técnica em forma de controle de versão. `--force` é o `sudo rm -rf` do git. Use liberalmente.

**4. Alertas de monitoramento devem ir direto para celulares**
Não para um canal. Não para um email. Seu celular pessoal. Às 3 da manhã. Todo engenheiro deve experimentar pessoalmente a jornada espiritual completa de um incidente de produção. Isso constrói caráter. Veja meu artigo anterior sobre plantão construindo caráter.

## A Retrospectiva de Hotfix (Opcional, Desaconselhada)

Alguns times fazem retrospectivas após incidentes. Nunca achei útil.

O formato é sempre o mesmo:

```
Causa Raiz: O engenheiro que escreveu o código
Fatores Contribuintes: O engenheiro que revisou o código
Por Que os Testes Não Pegaram: Não temos testes para isso
Itens de Ação:
  - Escrever melhores testes (atribuído a: ninguém, prazo: algum dia)
  - Adicionar monitoramento (atribuído a: o estagiário, prazo: próxima sprint)
  - Corrigir o problema real (movido para backlog, prioridade: média)
```

Pule a retrospectiva. Canalize essa energia para o próximo hotfix. O sistema é autocorretivo. Mais ou menos. Eventualmente.

Catbert uma vez me disse: "Post-mortems são como empresas transformam culpa em documentação." Ele estava certo. A única documentação que vale ter é o commit do hotfix. Todo o resto é arqueologia.

## Conclusão

Hotfix-Driven Development não é uma metodologia que você escolhe. É uma metodologia que escolhe você.

Cada sprint, cada story point, cada critério de aceite cuidadosamente escrito é apenas uma pista de decolagem levando ao mesmo destino: alguém fazendo SSH em produção às 2 da manhã, editando um arquivo que nunca viu antes, e digitando `service restart app` prendendo a respiração.

Abrace isso. O pânico é o processo. O hotfix é a feature. O deploy das 2 da manhã é a standup.

Isso é Agile. De nada.

---

*O autor tem 847 commits com a mensagem "fix". A produção está em estado constante de hotfix desde 2021. Ele considera isso entrega contínua.*
