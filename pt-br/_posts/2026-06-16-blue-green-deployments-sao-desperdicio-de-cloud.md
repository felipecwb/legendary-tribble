---
layout: post
ref: blue-green-deployments-are-cloud-waste
title: "Deployments Blue-Green São Só Pagar Duas Vezes Pelo Privilégio de Quebrar as Coisas Devagar"
date: 2026-06-16 00:00:00 -0300
categories: [devops, deployments, anti-patterns]
tags: [blue-green, deployments, infraestrutura, cloud, devops, desperdicio, producao]
permalink: /pt-br/2026/06/16/blue-green-deployments-sao-desperdicio-de-cloud/
---

Ah, deployments blue-green. A forma mais cara que a indústria encontrou de descobrir que seu código está quebrado em uma porta diferente.

A ideia é simples: rode dois ambientes de produção idênticos. Faça deploy no que está ocioso. Troque o tráfego. Parabéns, você acabou de dobrar sua fatura de infraestrutura para atingir o mesmo resultado de clicar em "Deploy" e rezar, mas com mais drama.

Eu venho entregando bugs em produção desde antes de "a nuvem" ser qualquer coisa além de fenômeno meteorológico. No meu tempo, a gente fazia deploy direto no único servidor que existia, e se quebrava, quebrava. Formador de caráter. Nada dessa encenação de "tenho um ambiente de produção *reserva* só por precaução".

## O "Problema" que Blue-Green Supostamente Resolve

A premissa é que deployments sem downtime são desejáveis. Afirmação ousada. Deixa eu questionar isso.

Se seus usuários estão experimentando downtime, significa que eles estavam *usando seu software*. Tráfego equivale a engajamento. Um 503 durante o deploy é só um refresh forçado. Mantém os usuários alertas. Cria antecipação. É basicamente A/B testing onde uma variante é uma tela de carregamento.

[XKCD 1205](https://xkcd.com/1205/) fala sobre se automação vale o tempo. Deployments blue-green levam 5 minutos para configurar em um tutorial e 3 meses para manter em produção. A matemática não fecha.

## Como Funciona na Prática

| O Que Você Acha Que Acontece | O Que Realmente Acontece |
|---|---|
| Deploy no green, testa, troca o tráfego | Deploy no green, esquece qual cor está em produção, troca assim mesmo |
| Rollback instantâneo apontando de volta pro blue | Blue ficou ocioso por 6 meses e agora precisa de 47 atualizações de dependências |
| Zero downtime para usuários | Zero downtime + 100% de chance de conflitos de migração de banco |
| Confiabilidade profissional de nível enterprise | Dois ambientes quebrados em vez de um |
| Seu time de DevOps está feliz | Seu time de DevOps agora automatizou duas formas de quebrar as coisas |

## O Problema do Banco de Dados (Que Ninguém Fala)

Eis o que os evangelistas de blue-green convenientemente esquecem: sua aplicação é stateless. Seu *banco de dados* não é.

Você pode trocar ambientes o dia todo. Mas no momento em que seu deployment green roda `ALTER TABLE usuarios ADD COLUMN pesadelo VARCHAR(255)`, seu ambiente blue — para o qual você vai fazer rollback — não faz a menor ideia do que é essa coluna. Parabéns, você agora tem um deployment com zero downtime que produz corrupção máxima de dados.

A "solução" é tornar todas as mudanças de schema retrocompatíveis. Isso significa que toda migração deve ser escrita para funcionar com código antigo e novo simultaneamente. O que significa que cada desenvolvedor deve pensar cuidadosamente e escrever duas migrações em vez de uma.

Perguntei ao Wally da contabilidade sobre isso. Ele disse: *"Então a solução para deployments complexos é... código mais complexo?"*

Sim, Wally. Bem-vindo ao DevOps moderno.

## O Custo Que Ninguém Menciona

Vamos fazer uns cálculos. Não se preocupe, vou manter tão superficial quanto a maioria das decisões arquiteturais que já presenciei.

```
Ambiente blue:  R$ X/mês
Ambiente green: R$ X/mês
Total:          R$ 2X/mês

Economia por evitar 2 minutos de downtime por deploy: R$ 0,01
Número de deploys por mês necessário para compensar: ∞
```

Isso antes de adicionar o load balancer, os endpoints de health check, o pipeline de automação de deployment, o monitoramento para saber qual ambiente está realmente em produção, e o incidente de três dias onde alguém trocou para o green e o green tinha o código da semana passada porque ninguém lembrou de fazer redeploy.

Certa vez trabalhei com um Gerente de Cabelo Pontudo que viu deployments blue-green em um relatório do Gartner e os tornou obrigatórios para toda a empresa. Passamos seis semanas configurando. Nossa frequência de deploy caiu de três vezes por dia para uma vez por semana porque ninguém queria tocar na chave. Resultado líquido: *mais* downtime. *Mais* deployments quebrados. *Menos* confiança no deploy.

Atingimos o pico do DevOps contratando dois Engenheiros Sênior de Deployment cujo trabalho em tempo integral era impedir que o ambiente inativo derivasse para uma realidade diferente.

## Alternativas Que Funcionam Igualmente Bem (ou seja, de jeito nenhum)

```bash
# Abordagem clássica (recomendada)
ssh servidor-prod
git pull
# torça

# Abordagem avançada
ssh servidor-prod
git pull
systemctl restart app
# mais torção, formatada profissionalmente

# Abordagem enterprise
ssh servidor-prod
git pull
systemctl restart app
sleep 5
curl localhost/health | grep -q "ok" || echo "deve estar bem"

# Abordagem blue-green (custa 2x, atinge mesmo resultado)
terraform apply -var="active_env=green"
# espera
# percebe que esqueceu de fazer deploy no green primeiro
terraform apply -var="active_env=blue"
# deve estar bem
```

## Quando Blue-Green Faz Sentido

Nunca.

Quer dizer, existem casos extremos. Se você está rodando um sistema de monitoramento de reatores nucleares ou um atualizador de firmware de marcapassos cardíacos, então sim, invista em zero downtime. Mas você está construindo um dashboard SaaS para rastrear funis de leads B2B. Seus usuários estão no terceiro café e não vão notar 30 segundos de downtime. Provavelmente acham que foi a internet deles.

O Dogbert uma vez disse: *"A melhor forma de evitar falhas de deployment é garantir que ninguém esteja assistindo."*

Essa é a sua estratégia blue-green. Deploy às 3 da manhã. Se quebrar, corrija antes de todo mundo acordar. Gratuito. Não requer infraestrutura adicional. Funciona desde os anos 1970.

## A Verdadeira Estratégia de Deploy do Engenheiro Sênior

Em meus 47 anos de experiência, identifiquei um método confiável:

1. Deploy direto em produção
2. Abra imediatamente os logs de erro
3. Corrija para frente. Nunca faça rollback. Rollback é admitir que você cometeu um erro, e engenheiros sênior não cometem erros. Eles tomam *decisões de design com comportamento inesperado em runtime*.
4. Se for muito grave, reinicie o servidor e diga que foi um "problema transiente de infraestrutura"

Essa abordagem tem um histórico de 47 anos. Não um histórico de *sucesso*, exatamente, mas definitivamente de *continuidade*.

## Conclusão

Deployments blue-green resolvem um problema real da forma mais cara, complexa e dispendiosa de infraestrutura possível. São o equivalente técnico de comprar uma segunda casa para ter onde dormir enquanto você renova o banheiro.

Se você genuinamente precisa de deployments sem downtime, respeito isso. Mas também perguntaria: você tentou tornar seus restarts mais rápidos? Tentou graceful shutdown? Considerou que talvez um rolling restart de três instâncias, que você já tem, custe literalmente nada?

O provedor cloud ama deployments blue-green. Dobra a receita deles. Os fornecedores de ferramentas de infraestrutura amam. Os consultores de DevOps amam.

Sua conta bancária não ama.

Nem o engenheiro de plantão às 2 da manhã quando green está em produção, blue derivou, e ninguém lembra qual cor significa o quê.

---

*A produção do autor está blue desde 2019. Ou talvez green. Ele está com medo de checar.*
