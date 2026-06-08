---
layout: post
ref: zero-downtime-deployments-are-lying-slowly
title: "Deploy com Zero Downtime É Só Mentir Para Seus Usuários Devagar"
date: 2026-06-08 00:00:00 -0300
categories: [devops, deploy]
tags: [deploy, zero-downtime, blue-green, canary, devops, anti-padroes, confiabilidade]
permalink: /pt-br/2026/06/08/deploy-zero-downtime-e-mentir-para-usuarios-devagar/
---

A indústria tech passou a última década celebrando "deploys com zero downtime" como se fosse uma grande conquista da civilização. Deploys blue-green. Canary releases. Rolling updates. Feature flags. Dark launches.

Li todos os blog posts. Sentei em todas as palestras das conferências. E depois de 47 anos entregando software, estou aqui para dizer a verdade:

**Deploys com zero downtime não eliminam o downtime. Eles redistribuem.**

## A Contabilidade Honesta do "Zero Downtime"

Deixa eu te mostrar o que "zero downtime" realmente significa na prática:

| Como Você Chama | O Que É de Verdade |
|-----------------|-------------------|
| "Rollout gradual" | Usuários recebendo bugs diferentes ao mesmo tempo |
| "Deploy blue-green" | Dois ambientes de produção, custo duplo, confusão dupla |
| "Canary release" | Deixar 5% dos seus usuários fazer QA do seu código de graça |
| "Feature flag" | Dívida técnica que você nunca vai limpar |
| "Health check antes do roteamento" | Sua app mentindo pro load balancer por 30 segundos |
| "Rolling update" | Todo o seu cluster rodando código diferente simultaneamente |
| "Zero downtime" | O mesmo downtime, distribuído por mais usuários |

Downtime tradicional: 1.000 usuários insatisfeitos por 5 minutos = 5.000 minutos-usuário de sofrimento.

Rollout com "zero downtime": 50 usuários enfrentam bugs por 2 dias = 144.000 minutos-usuário de sofrimento.

Qual é a escolha responsável? A matemática está aí.

## A Mentira do Deploy Blue-Green

Veja como deploys blue-green são vendidos para você:

> "Você tem dois ambientes idênticos. Deploy no blue. Teste. Troca o tráfego. Se algo quebrar, volta instantaneamente. Zero downtime!"

Veja o que acontece de verdade:

```bash
# Passo 1: Deploy no ambiente blue
kubectl apply -f blue-deployment.yaml
# Blue está rodando v2. Green está rodando v1.

# Passo 2: "Testar" o blue
curl https://blue.interno.empresa.com/health
# Retorna 200. Ótimo. Vai!

# Passo 3: Trocar tráfego
kubectl patch ingress main --patch '{"spec":{"rules":[{"host":"app.empresa.com","http":{"paths":[{"backend":{"serviceName":"blue-service"}}]}}]}}'

# Passo 4: Usuários chegam na v2
# v2 tem uma migração de banco que a v1 escreveu, mas a v2 lê diferente
# Agora o blue está errando em toda request que toca o novo schema
# Enquanto isso, o green não pode ser reativado porque a migração já rodou

# Passo 5: A produção está fora do ar.
# Mas é um fora do ar sofisticado. Fora do ar enterprise.
# "Zero Downtime" fora do ar.
```

Como a [XKCD 1718](https://xkcd.com/1718/) mostra, todo sistema complexo é simultaneamente a melhor e a pior coisa que você já construiu. Blue-green não é diferente: impressionante em demos, catastrófico na realidade.

## O Problema de Migração de Banco Que Ninguém Fala

Todo guia de "deploy com zero downtime" convenientemente passa por cima de bancos de dados:

> "Só faça suas migrações compatíveis com versões anteriores!"

Ah sim. Vou simplesmente redesenhar casualmente todo o meu modelo de dados para suportar duas versões incompatíveis da aplicação rodando em produção ao mesmo tempo.

```sql
-- Migração zero-downtime para renomear uma coluna
-- Passo 1: Adicionar nova coluna (deploy v1.5 que escreve em ambas)
ALTER TABLE usuarios ADD COLUMN nome_completo VARCHAR(255);

-- Passo 2: Backfill (roda por 4 horas na sua tabela de 50M de linhas, travando tudo)
UPDATE usuarios SET nome_completo = CONCAT(primeiro_nome, ' ', sobrenome);

-- Passo 3: Deploy v2 que lê da nova coluna
-- Passo 4: Remover colunas antigas (deploy v2.5)
-- Passo 5: Remover código de compatibilidade (deploy v3)

-- Total de migrações: 3
-- Total de deploys: 3
-- Tempo de calendário: 6 semanas
-- Alternativa: janela de manutenção agendada: 10 minutos
```

O Dogbert do Dilbert chamaria isso de "cobrar o triplo pelo mesmo trabalho distribuído em mais deploys." Ele estaria certo.

## Canary Releases: Seus Usuários Não São Canários

O nome já diz tudo. Nas minas de carvão, canários morriam para avisar os mineiros sobre gás tóxico. Seus "usuários canários" servem a mesma função: eles morrem (têm experiências terríveis) para que você detecte problemas.

```yaml
# Sua configuração "sofisticada" de canary
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: minha-app
spec:
  analysis:
    interval: 1m
    threshold: 5       # 5% de taxa de erro aceitável durante o rollout
    maxWeight: 50      # Rotear até 50% do tráfego para versão com bug
    stepWeight: 10     # Aumentar 10% a cada minuto
```

Tradução: "Nos próximos 5 minutos, até 50% dos seus usuários vão experimentar até 5% de erros."

Se sua app lida com 1.000 requisições por segundo, são 50 requisições falhando por segundo durante 5 minutos, distribuídas por usuários que não fizeram nada de errado.

Uma janela de manutenção de 5 minutos: 0 usuários recebem erros.

Um "sofisticado deploy canary": 15.000 requisições falham.

Isso é progresso.

## Feature Flags: Dívida Que Se Reproduz

Feature flags são apresentadas como o auge da sofisticação zero-downtime:

```python
if feature_flags.habilitado("novo_fluxo_checkout", user_id=usuario.id):
    return novo_checkout(carrinho)
else:
    return checkout_antigo(carrinho)
```

Quantas feature flags uma empresa madura acumula? Deixa eu verificar a codebase:

```bash
$ grep -r "feature_flags.habilitado" . | wc -l
847
```

Oitocentas e quarenta e sete. E nenhuma pode ser removida porque ninguém sabe quais ainda estão em uso. O próprio sistema de feature flags agora precisa de um time dedicado.

A sabedoria do Wally do Dilbert se aplica aqui: "Eu costumava entregar código. Agora eu mantenho a infraestrutura que decide qual código entregar para quem. Recebi 40% de aumento."

## O Que os "Health Checks" Realmente Verificam

```python
@app.route('/health')
def health_check():
    # O health check do Kubernetes
    return jsonify({"status": "ok"})  # Sempre retorna 200
    
# Enquanto isso, na aplicação real:
def processar_pagamento(dados_pagamento):
    # Conecta ao provedor de pagamento que está fora há 30 minutos
    # Mas o health check não verifica isso
    # Então o Kubernetes continua roteando tráfego aqui
    # Porque o health check está "saudável"
    resposta = provedor_pagamento.cobrar(dados_pagamento)
```

Seu health check é uma mentira diplomática. Ele diz ao Kubernetes o que ele quer ouvir. Enquanto isso, usuários reais estão experimentando erros reais que o health check educadamente ignora.

Isso não é zero downtime. É **zero responsabilidade**.

## A Solução Real: Abrace a Janela de Manutenção

Janelas de manutenção agendadas são honestas. Elas dizem a verdade para os usuários:

> "Estaremos indisponíveis das 2h às 2h10 para manutenção."

Compare isso com alternativas de zero-downtime:

> "Estaremos disponíveis o tempo todo, exceto que alguns de vocês vão ter erros que estamos monitorando, e a taxa de erro pode aumentar durante nosso deploy de 45 minutos, e se algo der errado precisaremos de mais 10 minutos para fazer rollback, e o rollback pode causar seus próprios problemas por causa das migrações de banco, mas estamos confiantes que não vai te afetar, provavelmente."

Clientes respeitam honestidade. Em 47 anos nunca tive um usuário reclamando de uma janela de manutenção agendada às 5h. Tive muitas reclamações de usuários que receberam erros durante nossos deploys de "zero downtime" e foram informados que "não deveria estar acontecendo."

```bash
# O sofisticado deploy zero-downtime
# Tempo total: 4 horas de planejamento de engenharia, 45 minutos de deploy
# Impacto nos usuários: 1.200 erros distribuídos por 6% do tráfego

# A janela de manutenção agendada
# Tempo total: 10 minutos
# Impacto nos usuários: 0 erros
# O site ficou fora por 10 minutos às 3h quando ninguém estava usando

# O que é realmente melhor para os usuários?
```

## Quando o Zero-Downtime Dá Errado

Deixa eu descrever um incidente típico de deploy zero-downtime:

1. Rolling update começa às 14h (pico de tráfego)
2. Novos pods sobem junto com pods antigos
3. Novos pods conectam ao BD, trigam uma migração
4. Migração adiciona coluna NOT NULL sem default
5. Pods antigos tentam inserir linhas sem a nova coluna
6. Pods antigos começam a jogar erros de SQL
7. Metade da sua frota está quebrada, metade está bem
8. Load balancer continua roteando para pods quebrados (health check ainda passa!)
9. Você percebe 23 minutos depois via reclamações de usuários
10. Você faz rollback — mas a migração já rodou
11. Agora o rollback também quebra
12. Você desabilita a migração, redeploya, escreve um hotfix
13. Downtime total: 47 minutos, para 60% dos usuários
14. Total de usuários afetados: mais do que se você tivesse feito uma janela de 5 minutos

Zero downtime.

---

*O autor tem deployado às 3h com janela de manutenção desde 1987. Seu uptime é 99,97%. Seus concorrentes que usam "deploys zero downtime" estão em 99,91%. Ele considera isso uma vitória.*
