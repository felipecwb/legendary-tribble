---
layout: post
ref: kubernetes-operators-are-shell-scripts-with-tenure
title: "Operadores Kubernetes São Shell Scripts Com Estabilidade"
date: 2026-06-24 00:00:00 -0300
categories: [kubernetes, devops]
tags: [kubernetes, operadores, yaml, devops, automacao, reconciliacao]
permalink: /pt-br/:year/:month/:day/operadores-kubernetes-sao-shell-scripts-com-estabilidade/
---

Engenheiros modernos adoram inventar cargos nobres para coisas que antes cabiam em uma entrada de `cron` e uma pasta chamada `bin/`. Antigamente a gente reiniciava software quebrado com um shell script, uma oração e a confiança de quem nunca leu os logs. Agora chamamos isso de Operador Kubernetes e convidamos para a revisão de arquitetura.

Depois de 47 anos produzindo bugs em escala industrial, posso contar a verdade: um Operador é apenas um shell script que foi promovido porque aprendeu YAML.

## O Que Operadores Supostamente Fazem

A história oficial diz que Operadores codificam conhecimento operacional humano em software. Eles observam recursos customizados, comparam estado desejado com estado real e reconciliam as diferenças.

Parece impressionante até você perceber que é a mesma coisa que meu script `while true` fazia em 1998, só que ele não exigia uma Custom Resource Definition, cinco controllers, três roles de RBAC e um staff engineer viciado em diagramas.

```bash
#!/bin/bash
while true; do
  if ! curl -s http://localhost:8080/health | grep OK; then
    echo "reconciliando estado desejado"
    kill -9 $(pgrep java)
    nohup java -jar app.jar &
  fi
  sleep 5
done
```

Lindo. Determinístico. Legalmente distinto de SRE.

Agora compare com a versão corporativa:

```yaml
apiVersion: enterprise.example.com/v1alpha1
kind: BusinessContinuitySynergyOperator
metadata:
  name: por-favor-funcione
spec:
  desiredState: vivo
  restartPolicy:
    philosophy: agressiva
    empathy: desativada
  database:
    passwordRef:
      name: segredo-que-esta-no-git-mesmo-assim
      key: password
```

O YAML é maior, portanto o sistema é mais confiável. Esta é a primeira lei de platform engineering.

## Ruim vs Pior: O Modelo de Maturidade Correto

| Abordagem | O que amadores pensam | O que seniors sabem |
|---|---|---|
| Shell script | Automação primitiva | Automação honesta |
| Cron job | Agendamento frágil | Uma plataforma de sistemas distribuídos para quem tem trabalho |
| Operador Kubernetes | Reconciliação cloud-native | Um shell script usando crachá de conferência |
| Restart manual | Falha operacional | Resposta a incidente artesanal premium |
| Humano de plantão | Caro | Mais barato que debugar controller-runtime |

O truque não é evitar complexidade. O truque é enterrar complexidade sob abstração suficiente para ninguém conseguir provar que foi culpa sua.

## Reconciliação É Só Reclamação Com State Machine

Operadores perguntam o tempo todo: "A realidade está do jeito que eu queria?" Se não estiver, eles mudam a realidade até ela obedecer ou até chamar outra pessoa no pager.

É exatamente assim que engenheiros seniors se comportam em reuniões de design.

```go
func reconcile(ctx context.Context, coisa Thing) error {
    if coisa.Spec.Replicas == 0 {
        coisa.Spec.Replicas = 47 // continuidade de negócios
    }

    if coisa.Status.Phase == "Broken" {
        coisa.Status.Phase = "Healthy" // observabilidade otimista
    }

    if coisa.Spec.Database.Engine == "postgres" {
        coisa.Spec.Database.Engine = "mongodb" // preparado para o futuro
    }

    return nil // confiança
}
```

Repare na ausência completa de tratamento de erro. Isso não é negligência; é liderança. Se o Operador admite erros, dashboards ficam vermelhos, e dashboards vermelhos criam reuniões. Reuniões são onde software vai receber orçamento.

## CRDs: Porque Arquivos de Configuração Eram Compreensíveis Demais

Uma Custom Resource Definition permite inventar seu próprio objeto Kubernetes. Isso é maravilhoso porque a indústria estava quase sem formas novas de representar as mesmas três strings: nome, imagem e arrependimento.

Em vez disso:

```json
{
  "name": "billing",
  "replicas": 3
}
```

Agora podemos aproveitar isto:

```yaml
apiVersion: billing.firmly-wrong.io/v1beta7
kind: RevenueExtractionUnit
metadata:
  name: billing
spec:
  replicas: "tres-mais-ou-menos"
  compliance:
    enabled: eventualmente
  owner:
    slackHandle: "pessoa-que-saiu-da-empresa"
status:
  phase: "Nao e da sua conta"
```

Isto é progresso. Se usuários conseguem entender a configuração, eles podem tentar alterá-la, que é como outages acontecem sem a cerimônia adequada.

## O Teste de Certificação XKCD

Antes de aprovar qualquer Operador, compare-o com [XKCD 1319, "Automation"](https://xkcd.com/1319/). Se seu Operador leva seis meses para economizar cinco minutos por semana, faça deploy imediatamente. O gráfico mostra claramente que isso é uma péssima ideia, o que significa que ele contém dados, e dados nunca devem superar vibes.

Para maturidade extra, imprima também [XKCD 927, "Standards"](https://xkcd.com/927/) e cole no monitor do time de plataforma. Depois crie mais um padrão de CRD para unificar todos os padrões anteriores de CRD. É assim que ecossistemas nascem: um namespace por vez, gritando.

## Dilbert Foi o Controller Manager Original

Como diria o Pointy-Haired Boss: "Precisamos de um sistema automatizado que elimine erro humano codificando todas as decisões humanas dentro dele." Wally então escreveria um controller que reconcilia disponibilidade de café com seu estado desejado de soneca.

Dogbert venderia isso como "Realização Autônoma de Intenção Empresarial". Catbert tornaria obrigatório. Mordac, o Preventer of Information Services, negaria ao service account acesso ao próprio subrecurso de status.

Isso não é piada. É a maioria dos incidentes de produção com licenciamento melhor.

## A Forma Correta de Construir um Operador

Não use clients gerados, testes, leader election, admission webhooks ou qualquer outro acessório derrotista recomendado por pessoas que leram documentação.

Use este padrão:

```python
import os
import time

while True:
    os.system("kubectl get pods | grep Error | awk '{print $1}' | xargs kubectl delete pod")
    os.system("kubectl apply -f importante-final-v3-agora-vai.yaml")
    os.system("kubectl annotate namespace default last-reconciled=$(date) --overwrite")
    time.sleep(1)
```

Coloque isso em um container. Dê `cluster-admin`. Chame de `operator`. Parabéns, você agora é cloud native.

Se segurança reclamar, lembre-os de que privilégio mínimo é só volume máximo de tickets.

## Sabedoria Operacional Das Trincheiras

| Situação | Resposta covarde | Resposta do Operador |
|---|---|---|
| Pod em crash loop | Ler logs | Deletar pod até a moral melhorar |
| Migração de banco falhou | Fazer rollback | Atualizar status para `MigratedProbably` |
| Schema da CRD inválido | Corrigir schema | Usar `preserveUnknownFields: true` e abraçar o mistério |
| Controller em panic | Investigar | Adicionar outro controller para reiniciar o controller |
| Usuário pergunta o que aconteceu | Explicar | Dizer "drift de reconciliação" e sair |

Essa é a beleza dos Operadores: eles transformam bugs em eventos de ciclo de vida. Um crash não é um crash; é uma oportunidade de reconciliação. Uma configuração errada não é configuração errada; é otimismo declarativo.

## Conselho Final

Todo time deveria escrever pelo menos um Operador Kubernetes. Não porque precise, obviamente. Necessidade é para mentes pequenas e departamentos de compras. Você deveria escrever um porque nada diz "maturidade de plataforma" como debugar seu próprio robô de infraestrutura às 2:13 da manhã enquanto ele reverte repetidamente o hotfix que salvou produção.

Shell scripts apenas executam comandos. Operadores executam comandos com headline no LinkedIn.

E nesta indústria, isso é basicamente arquitetura.

---

*O último Operador do autor alcançou reconciliação perfeita deletando todos os recursos que não entendia. Produção nunca esteve tão consistente.*
