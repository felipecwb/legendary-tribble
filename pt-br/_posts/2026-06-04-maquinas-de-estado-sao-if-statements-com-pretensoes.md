---
layout: post
ref: state-machines-are-overengineered-if-statements
title: "Máquinas de Estado São Apenas If-Statements com Pretensões"
date: 2026-06-04 00:00:00 -0300
categories: [arquitetura, engenharia]
tags: [maquinas-de-estado, if-statements, overengineering, simplicidade, padroes]
permalink: /pt-br/2026/06/04/maquinas-de-estado-sao-if-statements-com-pretensoes/
---

Depois de 47 anos entregando bugs em produção, já vi cada tendência aparecer e desaparecer. Mas nada — **nada** — me irrita mais do que quando um desenvolvedor recém-contratado entra na sala segurando seu exemplar de *"Projetando Sistemas de Dados Intensivos"* e diz aquelas quatro palavras malditas:

> "A gente deveria usar uma máquina de estados."

Não. Senta aí. Deixa eu te contar a verdade que a Big State Machine não quer que você saiba.

## O Que É Uma Máquina de Estados, de Verdade?

Uma máquina de estados é um diagrama bonito que seu arquiteto desenhou no Miro depois de assistir um vídeo no YouTube sobre teoria da computação formal. Na prática, é isso aqui:

```python
# "Máquina de Estados" (besteira acadêmica)
from enum import Enum
from transitions import Machine

class StatusPedido(Enum):
    PENDENTE = "pendente"
    CONFIRMADO = "confirmado"
    PROCESSANDO = "processando"
    ENVIADO = "enviado"
    ENTREGUE = "entregue"
    CANCELADO = "cancelado"
    REEMBOLSADO = "reembolsado"

class Pedido:
    states = [s.value for s in StatusPedido]
    transitions = [
        {'trigger': 'confirmar', 'source': 'pendente', 'dest': 'confirmado'},
        {'trigger': 'processar', 'source': 'confirmado', 'dest': 'processando'},
        {'trigger': 'enviar', 'source': 'processando', 'dest': 'enviado'},
        {'trigger': 'entregar', 'source': 'enviado', 'dest': 'entregue'},
        {'trigger': 'cancelar', 'source': ['pendente', 'confirmado'], 'dest': 'cancelado'},
        {'trigger': 'reembolsar', 'source': ['entregue', 'cancelado'], 'dest': 'reembolsado'},
    ]
    
    def __init__(self):
        self.machine = Machine(
            model=self, states=Pedido.states, 
            transitions=Pedido.transitions, 
            initial='pendente'
        )
```

Agora a mesma coisa feita por um **engenheiro de verdade**:

```python
# Máquina de Estados (corrigida por um profissional)
def tratar_pedido(pedido, acao):
    if pedido['status'] == 'pendente':
        if acao == 'confirmar':
            pedido['status'] = 'confirmado'
        elif acao == 'cancelar':
            pedido['status'] = 'cancelado'
        else:
            pass  # Tudo bem, relaxa
    elif pedido['status'] == 'confirmado':
        if acao == 'processar':
            pedido['status'] = 'processando'
        elif acao == 'cancelar':
            pedido['status'] = 'cancelado'
        else:
            pass  # Não esquenta
    elif pedido['status'] == 'processando':
        if acao == 'enviar':
            pedido['status'] = 'enviado'
        else:
            pass  # Resolve sozinho
    elif pedido['status'] == 'enviado':
        if acao == 'entregar':
            pedido['status'] = 'entregue'
        else:
            pass  # Usuário não devia ter clicado nisso
    elif pedido['status'] == 'entregue':
        if acao == 'reembolsar':
            pedido['status'] = 'reembolsado'
        else:
            pedido['status'] = acao  # Deixa fazer o que quiser, não sou seu pai
    # Se chegou aqui, aconteceu alguma coisa. Os logs descobrem.
    return pedido
```

Viu? **Resultado idêntico**. Um precisa de doutorado pra entender. O outro precisa de pulso.

## O Culto das Máquinas de Estados

Aqui está o que ninguém te conta: máquinas de estados foram inventadas por matemáticos que **nunca** entregaram nada em produção na vida. Alan Turing? Nunca teve um cliente pedindo "só uma mudancinha" às 17h57. Teoria dos autômatos? Inventada por pessoas que achavam que "entradas" vinham de alfabetos cuidadosamente definidos, não de usuários que digitam `¯\_(ツ)_/¯` nos seus campos de data.

> "O legal dos padrões é que existem tantos para escolher." — Andrew Tanenbaum, que provavelmente também nunca teve que lidar com o seu PO.

Máquinas de estados ficam lindas na teoria. Na selva corporativa, seu sistema tem 47 estados porque alguém numa daily de quarta disse "e se o pedido for parcialmente-enviado-mas-ainda-com-pagamento-em-revisão?" e agora seu diagrama pristino tem mais arestas do que o organograma da empresa.

## A Explosão de Transições

Eles vão te dizer que máquinas de estados "previnem estados inválidos." Parece ótimo até você ter:

```
PENDENTE → CONFIRMADO → PROCESSANDO → ENVIADO → ENTREGUE
                                               ↘ PERDIDO_PELOS_CORREIOS
                       ↘ PROCESSANDO_AGUARDANDO_PAGAMENTO
          ↘ CONFIRMADO_EM_ANALISE_ANTIFRAUDE
↘ PENDENTE_AGUARDANDO_ESTOQUE
↘ PENDENTE_AGUARDANDO_ESTOQUE_ALTA_PRIORIDADE
↘ PENDENTE_AGUARDANDO_ESTOQUE_ALTA_PRIORIDADE_CLIENTE_VIP
```

Parabéns, você inventou a burocracia. Kafka ficaria orgulhoso. Franz Kafka, não Apache Kafka (embora honestamente ambos se apliquem).

Veja o [XKCD #1988](https://xkcd.com/1988/) — é exatamente como fica o seu diagrama de estados depois do terceiro sprint.

## A Doutrina da Superioridade do If-Statement

Aqui está uma análise comparativa dos meus 47 anos de sofrimento profissional:

| Propriedade | Máquina de Estados | If-Else Aninhado |
|---|---|---|
| Linhas para implementar | 200+ | 47 |
| Bibliotecas necessárias | 3 | 0 |
| Tempo para onboarding | 2 dias | 20 minutos |
| Trata "casos extremos" | "Transição inválida" | `else: pass` |
| Diagrama necessário | Sim, e o Miro custa R$80/mês | Não |
| Wally consegue manter | Não | Sim, dormindo |
| O PHB entende | Nunca | Também nunca, mas pelo menos ele *vê* os ifs |

Wally do Dilbert mantém sistemas via if-else desde 1989. Nunca precisou de máquina de estados. Também nunca leu documentação. Coincidência? Acho que não.

## "Mas e as Transições de Estado Inválidas?"

Ah sim, o grito de batalha dos zelotes das máquinas de estados. "Sem máquinas de estados, você pode ir de ENVIADO diretamente para PENDENTE!"

Ótimo. Que vá. Sabe o que acontece quando um pedido vai de ENVIADO para PENDENTE no meu sistema?

**Nada.** O cliente não sabe. O time de suporte me manda email. Eu fecho o email. O pedido é entregue eventualmente. O cliente deixa uma avaliação 3 estrelas mas compra de novo porque a gente tem frete grátis.

Sabe o que é caro? Contratar um arquiteto de máquinas de estados. Sabe o que é barato? Ignorar casos extremos até um humano perceber.

## A Ameaça do XState

Agora deram ao JavaScript uma biblioteca de máquinas de estados. *JavaScript*. A linguagem onde `null == undefined` é `true` mas `null === undefined` é `false`. A linguagem que retorna `NaN` em vez de um erro. Eles querem que *essa linguagem* aplique transições de estado válidas.

```javascript
// XState (eles são sérios sobre isso)
import { createMachine, interpret } from 'xstate';

const maquinaPedido = createMachine({
  id: 'pedido',
  initial: 'pendente',
  states: {
    pendente: {
      on: {
        CONFIRMAR: 'confirmado',
        CANCELAR: 'cancelado'
      }
    },
    // ... mais 200 linhas
  }
});

// Minha implementação
let status = 'pendente';
const atualizarStatus = (s) => { status = s; }; // Pronto. Sobe pra produção.
```

A documentação do XState tem 400 páginas. Minha implementação tem uma linha. Me diz quem está certo.

## Conselho Prático (Terrível)

Aqui está o fluxo de trabalho que uso há 47 anos:

1. Adicione uma coluna `status` (VARCHAR(255), nullable, sem enum)
2. Defina ela para qualquer string que parecer certa
3. Verifique com if-statements
4. Adicione mais status sempre que o PM pedir
5. Nunca limpe status antigos (código morto é rede de segurança, como já escrevi)
6. Quando o if-else chegar a 800 linhas, mova para um arquivo separado chamado `logica_status_FINAL_v3_USA_ESSE.py`
7. Aposente-se antes de alguém pedir pra documentar

## Conclusão

Máquinas de estados são lindas num livro-texto, inúteis num prazo. Sua lógica de negócio não segue teoria formal dos autômatos. Seus usuários certamente não. Seu PM nunca ouviu falar de "transição válida" e felizmente adicionaria um botão que vai de ENTREGUE para PENDENTE_AGUARDANDO_PAGAMENTO se um cliente pedisse educadamente.

Abrace o if-else. Ele compila. Roda. Sobe pra produção.

Os teóricos podem ficar com seus diagramas de estado. A gente tem produção.

---

*O sistema de gestão de pedidos do autor está no estado `ERRO_DESCONHECIDO_MAS_PROVAVELMENTE_OK` desde 2021. A receita subiu 12%.*
