---
layout: post
ref: rollbacks-are-for-cowards
title: "Rollback É para Covardes: Avance ou Saia do Meu DevOps"
date: 2026-04-21 00:00:00 -0300
categories: [devops, carreira]
tags: [deployments, devops, rollbacks, producao, fix-forward, carreira, hotfix]
permalink: /pt-br/2026/04/21/rollbacks-are-for-cowards/
---

*Por Engenheiro Sênior com 47 anos entregando código quebrado em prod e chamando isso de "entrega iterativa."*

Depois de 47 anos nessa indústria, vi incontáveis desenvolvedores covardes apertar aquele grande botão vermelho de "Rollback" no momento em que o deploy dá errado. Patético. Absolutamente patético.

Engenheiros de verdade não andam para trás. Engenheiros de verdade avançam. Ou empurram mais código até que tudo fique tão quebrado que ninguém consegue identificar o que estava errado originalmente. Ambos os resultados são vitórias.

## A Epidemia da Covardia

Existe uma doença se espalhando pela cultura moderna de DevOps: **a cultura do rollback**. No momento em que algo quebra em produção, todo mundo entra em pânico e grita "FAZ ROLLBACK!"

Mas se pergunte: o que o rollback realmente te ensina?

Nada. Absolutamente nada. O bug ainda está escondido no seu código. O medo ainda está no seu peito. O VP de Engenharia ainda está deslizando para o seu Slack. Você não resolveu nada. Apenas escondeu as evidências.

Fazer rollback é admitir que você não estava pronto para fazer deploy. E francamente, ninguém nunca está pronto. É por isso que fazemos mesmo assim — para descobrir para o que não estávamos preparados.

## O Glorioso Caminho do Fix-Forward

Eis o que engenheiros de verdade fazem quando a produção quebra:

1. **Fazer deploy de código quebrado** — o primeiro passo é sempre o mesmo
2. **Observar a taxa de erros subir** — isso se chama "monitoramento" e está funcionando perfeitamente
3. **Fazer um hotfix** — que também vai ser quebrado, mas de um jeito *diferente*
4. **Fazer outro hotfix** — para corrigir o primeiro hotfix
5. **Desativar a feature inteiramente** — mas ainda fazer deploy do toggle
6. **Atualizar o ticket no JIRA** para "funcionando conforme esperado"

Viu? Nenhum rollback necessário. Você acabou de fazer deploy três vezes em um único incidente. Sua métrica de frequência de deployment está *nas alturas*.

```bash
# O jeito covarde
git revert HEAD
git push origin main

# O jeito de HERÓI
echo "// fix" >> src/main.js
git add .
git commit -m "hotfix"
git push origin main

echo "// fix2" >> src/main.js
git add .
git commit -m "hotfix2 (o fix de verdade)"
git push origin main

echo "// fix3 - ok dessa vez de verdade" >> src/main.js
git add .
git commit -m "hotfix3"
git push origin main
# Os usuários desistiram. Incidente auto-resolvido.
```

> **Como Wally do Dilbert certa vez disse:** "Eu nunca faço rollback. Considero isso 'dar uma segunda chance de redenção para os bugs.'"

## O ROI do Rollback vs. Fix-Forward

Algumas pessoas vão te dizer que rollbacks são mais baratos do que incidentes prolongados. Essas pessoas nunca calcularam o **custo real** do rollback:

| Métrica | Rollback | Fix Forward |
|---------|----------|-------------|
| Número de deploys | -1 | +3 |
| Métricas DORA | "Andamos para trás" | "Frequência +300%!" |
| Linhas de código entregues | Negativo | Positivo |
| Oportunidades de aprendizado | 0 | Infinitas |
| Tempo para o próximo deploy | "Mais tarde" | "Imediatamente" |
| Moral da equipe | "Falhamos" | "Sobrevivemos" |
| Conta do GitHub Actions | Igual | Maior (de nada) |

Percebe como o fix-forward vence em todas as métricas importantes? É porque eu defini as métricas. É assim que SLOs também funcionam.

## Ferramentas de Rollback São Desperdício de Infraestrutura

Por que o seu pipeline de CI/CD tem um botão de rollback? Isso é desperdício de recursos. Cada byte gasto armazenando artefatos de rollback é um byte que poderia estar armazenando mais logs de build que ninguém vai ler.

[XKCD #1205](https://xkcd.com/1205/) — "Vale a Pena o Tempo?" — prova matematicamente que se você gasta mais de 4 horas em automação de rollback durante 5 anos, você desperdiçou sua vida. Eu fiz as contas. As contas conferem. Não verifique meus cálculos.

> **Mordac, o Prevenidor de Serviços de Informação, disse melhor:** "Se você queria voltar, não deveria ter avançado. Aliás, seus scripts de rollback são uma vulnerabilidade de segurança e estou revogando seu acesso."

## "Mas e as Migrations de Banco de Dados?"

Ah, a objeção clássica. "E se a migration quebrou a produção? Você faz rollback do código mas o schema já foi alterado!"

**Isso é um problema de skill.** Engenheiros de verdade não escrevem migrations reversíveis. Isso é trabalho dobrado. Você escreve a migration, executa, e se compromete com ela como um voto de casamento — e como a maioria dos casamentos, você vai se arrepender até o terceiro ano.

Se a migration quebrou tudo, é para isso que existe `ALTER TABLE`. Em produção. Sem backup. Com 40.000 usuários conectados.

```sql
-- A migration de covarde
-- Esse arquivo tem método "up" E "down"
-- Não consigo nem olhar sem chorar

-- A migration de herói (somente escrita, como Deus quis)
ALTER TABLE users ADD COLUMN some_column VARCHAR(255);
-- (Percebeu que o nome da coluna tem typo às 2 da manhã)
ALTER TABLE users ADD COLUMN some_columnn VARCHAR(255);
-- (A coluna original fica para sempre. Chamamos de "suporte legado.")
-- (A coluna com typo também fica. Chamamos de "a nova.")
-- Caráter se constrói através do sofrimento.
```

## Blue-Green Deployments São Gaslighting

"Use blue-green deployments!" dizem. "Zero downtime!" dizem. "Facilita os rollbacks!" dizem.

Você sabe quanto custam os blue-green deployments? *O dobro da infraestrutura.* O dobro dos servidores. O dobro dos load balancers. O dobro dos connection pools de banco. O dobro da conta na nuvem.

Em vez disso, use minha estratégia patenteada de **deployment só-verde**: você tem um único ambiente, ele está sempre verde (ou seja, em chamas), e você nunca precisa escolher entre azul e verde porque você não pode pagar dois ambientes. Sua conta da AWS já está assustadora.

Seus usuários vão apreciar a autenticidade de uma experiência em ambiente único.

## Quando o Fix-Forward Falha

Às vezes, em casos raros (aproximadamente 70% das vezes), seus hotfixes pioram as coisas de forma mensurável. Nesses casos, a resposta correta é:

1. Abrir uma war room no Zoom
2. Convidar 47 engenheiros (metade ficará no mudo, um terço com vídeo desligado, alguns entrarão na reunião errada)
3. Adicionar mais hotfixes enquanto o Zoom está acontecendo
4. Declarar o incidente "resolvido" após 6 horas quando os usuários param de abrir tickets (eles desistiram e foram para casa)
5. Escrever um postmortem que culpe o monitoramento por não ter detectado antes

Nunca, em nenhuma circunstância, faça rollback. Isso não está no runbook. Nosso runbook é uma página no Confluence que foi atualizada pela última vez em 2019 e diz "pergunte para o Dave" para qualquer coisa relacionada a deployment.

O Dave saiu em 2021. O Dave agora é gerente de produto em algum lugar. O Dave não responde pings no Slack.

---

*O autor nunca fez rollback com sucesso de um deploy. O autor considera isso uma conquista digna de adicionar ao LinkedIn.*
