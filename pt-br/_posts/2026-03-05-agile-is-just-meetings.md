---
layout: post
ref: agile-is-just-meetings
title: "Agile É Só Reunião: Um Framework para Máxima Densidade de Calendário"
date: 2026-03-05 16:44:00 -0300
categories: [agile, management]
tags: [agile, scrum, meetings, standups, retrospectives, sprint, kanban, productivity]
permalink: /pt-br/:year/:month/:day/agile-is-just-meetings/
---

Depois de 47 anos entregando bugs em várias metodologias, finalmente entendi o que Agile realmente é: reuniões. O código é incidental. O que importa é o ritual de juntar engenheiros em uma sala (ou Zoom) para discutir por que o código ainda não está pronto.

## O Calendário de Reuniões Ideal

Aqui está meu calendário Agile aperfeiçoado:

| Horário | Reunião | Duração | Propósito |
|---------|---------|---------|-----------|
| 9:00 | Daily Standup | 45 min | Standup "rápido" de 15 min |
| 10:00 | Refinamento de Backlog | 2 horas | Fazer tickets menores até sumirem |
| 12:00 | Almoço (trabalhando) | 1 hora | Colocar em dia reuniões que perdeu |
| 13:00 | Sprint Planning | 3 horas | Estimar errado, juntos |
| 16:00 | Retrospectiva | 2 horas | Concordar em mudar coisas que não vamos mudar |
| 18:00 | Happy Hour Sync | 1 hora | A única reunião produtiva |

São 9,5 horas de reuniões. Em um dia bom, você pode ter 30 minutos para código real (durante a standup, quando você está mutado).

## O Verdadeiro Propósito da Standup

"O que você fez ontem? O que vai fazer hoje? Algum bloqueio?"

As respostas corretas:

```
P: O que você fez ontem?
R: "Reuniões, principalmente. Também revisei alguns PRs." (não revisou)

P: O que vai fazer hoje?
R: "Continuar trabalhando na coisa." (vago é seguro)

P: Algum bloqueio?
R: "Esperando o outro time." (sempre)
```

Como o [XKCD 303](https://xkcd.com/303/) imortalizou, a melhor desculpa durante a standup é "meu código está compilando." Em 2026, é "meus containers estão buildando" ou "esperando a CI."

## Story Points: A Métrica Sem Significado

Story points são a maior inovação do Agile: números que não significam nada mas parecem trabalho.

```
Desenvolvedor: "Isso é um 3."
Scrum Master: "Por que não 5?"
Desenvolvedor: "Na verdade, vamos chamar de 8."
Outro Dev: "Acho que é 13."
Todos: "Vamos fazer acordo em 8."
(Task leva 3 semanas independente)
```

| Pontos Estimados | Dias Reais | Motivo |
|-----------------|------------|--------|
| 1 | 3 | "Foi mais complexo do que pensamos" |
| 3 | 5 | "Tive que refatorar umas coisas" |
| 5 | 10 | "Dependências não estavam prontas" |
| 8 | 15 | "Escopo mudou" |
| 13 | ∞ | Task é abandonada, reescrita como 3 novos tickets |

O Chefe Cabeça-Pontuda do Dilbert adoraria story points. São números! Números significam controle! Números significam que você pode colocar em planilhas!

## Retrospectiva: O Loop Infinito

Toda retrospectiva da história:

```
O que foi bem:
- Comunicação do time ✓
- Meta do sprint atingida ✓ (removemos metade do escopo)
- Boa colaboração ✓

O que pode melhorar:
- Muitas reuniões
- Requisitos pouco claros
- Dependências de outros times
- Melhorar testes

Action items:
- Reduzir reuniões (nunca acontece)
- Pegar requisitos antes (impossível)
- Sincronizar mais com outros times (cria mais reuniões)
- Escrever mais testes (hahaha)

Próxima retrospectiva:
(exatamente a mesma lista)
```

Estamos fazendo isso há 47 sprints. A seção "O que pode melhorar" foi copiada e colada 47 vezes. Isso é Agile funcionando como planejado.

## Sprint: Uma Palavra Que Implica Correr

A palavra "sprint" sugere velocidade e intensidade. Realidade:

```
Semana 1: 
- Dias 1-2: "Ainda entrando nos tickets"
- Dias 3-4: "Progredindo"
- Dia 5: "Encontrei complexidade inesperada"

Semana 2:
- Dias 1-2: "Quase pronto, só testando"
- Dias 3-4: "Achei alguns edge cases"
- Dia 5: "Vai passar pro próximo sprint"
```

Um "sprint" é na verdade uma caminhada lenta interrompida por reuniões, troca de contexto e incidentes de produção.

## O Scrum Master: Facilitador Profissional de Reuniões

O que um Scrum Master faz:

```python
class ScrumMaster:
    def daily_standup(self):
        print("Vamos ser breves!")  # Não serão
        
    def resolver_bloqueio(self, bloqueio):
        return self.criar_reuniao(
            titulo=f"Discutir {bloqueio}",
            participantes=todos,
            duracao="1 hora"
        )
    
    def melhorar_velocity(self):
        # Adiciona mais reuniões para discutir velocity
        self.criar_reuniao("Sync de Melhoria de Velocity")
        
    def resolver_conflito(self):
        return "Vamos resolver offline"  # Criar outra reunião
        
    def valor_agregado(self):
        return "Atualizou o board do Jira"
```

## Kanban: Agile Sem Reuniões (Brincadeira)

"Vamos mudar pra Kanban, tem menos cerimônias!"

**Reuniões do Kanban:**
- Daily standup (ainda precisa)
- Priorização semanal (novo)
- Review quinzenal (novo)
- Review de métricas mensal (novo)
- Sync de melhoria contínua (recorrente)
- Calibração de WIP limit (trimestral)

Você trocou 5 reuniões de Scrum por 6 reuniões de Kanban. Progresso.

## O Manifesto Ágil, Reinterpretado

```
"Indivíduos e interações" 
→ Mais reuniões sobre interações

"Software funcionando" 
→ Demos de software meio funcionando

"Colaboração com cliente" 
→ Reuniões com clientes sobre reuniões

"Responder a mudanças"
→ Reuniões de emergência quando requisitos mudam
```

## Conclusão

Agile não é uma metodologia—é um framework de geração de reuniões. O código acontece entre reuniões, se acontecer. A verdadeira medida de agilidade é quão rápido você consegue agendar uma reunião para discutir a reunião que discutiu por que o código não está pronto.

Lembre-se: se você está codando durante o horário de trabalho, você está perdendo uma reunião.

---

*O calendário do autor mostra blocos de "Focus Time". Todos foram sobrescritos por syncs.*
