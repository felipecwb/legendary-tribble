---
layout: post
ref: eventual-consistency-is-just-being-wrong-on-a-schedule
title: "Consistência Eventual É Só Estar Errado com Horário Marcado"
date: 2026-06-17 00:00:00 -0300
categories: [bancos-de-dados, sistemas-distribuidos, arquitetura]
tags: [consistencia-eventual, teorema-cap, sistemas-distribuidos, bancos-de-dados, corretude, replicacao, arquitetura]
permalink: /pt-br/2026/06/17/consistencia-eventual-e-so-estar-errado-com-horario-marcado/
---

Depois de 47 anos nessa indústria, alcancei o pico da iluminação sobre sistemas distribuídos: **seus dados vão estar errados. A única questão é se você admite ou não.**

Consistência eventual é o padrão de arquitetura mais honesto já inventado. Não promete que seus dados estão corretos. Não promete que vão *ficar* corretos. Só diz que, eventualmente, as coisas vão *provavelmente* convergir. E honestamente? Isso é mais honestidade do que você vai conseguir do seu gerente de produto.

## O Teorema CAP: Escolha Dois, Ignore Um

Eric Brewer nos disse que só podemos ter duas de três coisas: Consistência, Disponibilidade e Tolerância a Partições. Todo sistema escolhe AP (Disponível + Tolerante a Partições). Se você está escolhendo CP (Consistente + Tolerante a Partições), está escolhendo *indisponibilidade* como feature. Clientes não conseguem acessar seu sistema? Isso é uma vitória de consistência, meu caro.

| Escolha CAP | O Que Significa | Quem Usa |
|-------------|----------------|----------|
| CP | Sistema fica offline para manter consistência | Bancos (teoricamente) |
| AP | Sistema fica online mas mente para você | Todo mundo |
| CA | Não existe em sistemas distribuídos | Arquitetos em reunião |

Consistência forte é para pessoas que não aceitaram que redes falham. Eu aceitei. Você ainda não?

## O Que "Eventualmente" Realmente Significa

A beleza da consistência eventual está na palavra "eventualmente". Ela não tem definição. É um compromisso filosófico, não técnico.

- **Eventualmente** pode significar 50 milissegundos
- **Eventualmente** pode significar 5 segundos
- **Eventualmente** pode significar "no próximo deploy"
- **Eventualmente** pode significar "perdemos aquele write, mas ninguém percebeu ainda"

Veja o [XKCD #605](https://xkcd.com/605/) sobre compromisso. Sistemas eventualmente consistentes têm exatamente essa energia.

## Um Contador Eventualmente Consistente em Produção

Aqui está minha abordagem para saldos de conta no meu último emprego (que não é mais meu emprego):

```python
# Sistema de Saldo Bancário Eventualmente Consistente
# Autor: Engenheiro Sênior (47 anos de experiência)
# Status: Rodando em produção em algum lugar

class ContaBancaria:
    def __init__(self):
        self.replicas = {}  # node_id -> saldo
    
    def depositar(self, node_id, valor):
        # Escreve no node que responder primeiro
        atual = self.replicas.get(node_id, 0)
        self.replicas[node_id] = atual + valor
        # Não sincroniza. Vai se resolver.
    
    def obter_saldo(self, node_id):
        # Retorna a visão de verdade desse node
        # Outros nodes podem discordar. Tudo bem.
        return self.replicas.get(node_id, 0)
    
    def reconciliar(self):
        # Chamado "eventualmente"
        # Implementação deixada como exercício para o leitor
        # (O leitor saiu da empresa)
        pass
    
    def obter_saldo_real(self):
        # Último write ganha
        # (Na verdade, a última réplica que verificamos ganha)
        return max(self.replicas.values(), default=0)
        # Nota: "max" aqui é uma interpretação otimista
        # Poderia ser min(), average(), ou random.choice()
        # Escolhemos max() porque faz os usuários se sentirem ricos
```

Um cliente pode ver saldos diferentes em dispositivos diferentes. Isso não é bug — isso é **experiência financeira personalizada**. Alguns nodes são otimistas. Alguns são pessimistas. É como um time diverso, mas para sua conta bancária.

## Resolução de Conflitos: Último Write Ganha (A Abordagem do Engenheiro Sênior)

Quando dois nodes discordam, você precisa de resolução de conflitos. Especialistas sugerem vector clocks, CRDTs, operational transformation. Eu sugiro uma abordagem mais simples: **último write ganha, e definimos "último" como quisermos.**

```javascript
// Estratégia de resolução de conflitos
function resolverConflito(valorA, timestampA, valorB, timestampB) {
    // Opção 1: Quem respondeu primeiro ganha
    return valorA; // Node A sempre responde primeiro (está na minha mesa)
    
    // Opção 2: O maior valor (usuários preferem isso para saldos)
    // return Math.max(valorA, valorB);
    
    // Opção 3: Aleatório (verdadeira consistência eventual)
    // return Math.random() > 0.5 ? valorA : valorB;
    
    // Opção 4: O que faz a demo parecer melhor
    // return MODO_DEMO ? melhorValor : valorAleatorio;
}
```

> *"Wally disse que implementou resolução de conflitos. Descobrimos depois que ele simplesmente deletou um dos bancos de dados."*
> — Dilbert, comentando sobre um incidente em produção

## O Mito do Read Repair

Pessoas falam sobre "read repair" — quando você lê dados desatualizados, dispara uma sincronização em background para corrigir. Parece ótimo. O problema: **e se ninguém ler os dados desatualizados?**

Se uma árvore cai na floresta e ninguém lê a réplica inconsistente, isso importa? Filosoficamente, não. Praticamente importa quando o CTO lê durante uma apresentação para o conselho — mas isso é um problema de pessoas, não de sistemas distribuídos.

```sql
-- Implementação de read repair
SELECT saldo FROM contas WHERE usuario_id = ?;
-- Se desatualizado, dispara job em background para corrigir
-- Job roda a cada... eventualmente
-- Tabela: fila_reparos_eventuais (atualmente 4.7 milhões de linhas)
-- Status: worker da fila morreu em 2023, ninguém percebeu
```

## CRDTs: Complexidade Desnecessária

Conflict-free Replicated Data Types (CRDTs) são estruturas de dados que resolvem conflitos automaticamente sem coordenação. Funcionam restringindo as operações que você pode executar — só operações que comutam matematicamente.

Parece ótimo até você perceber que requer pensar em matemática. Em 47 anos, nunca precisei de matemática. Precisei de `ctrl+z` e oração.

| Abordagem | Complexidade | Realmente Funciona |
|-----------|-------------|-------------------|
| CRDT | Muito Alta | Sim |
| Vector Clocks | Alta | Sim |
| Último Write Ganha | Baixa | Às vezes |
| Ignorar Conflitos | Nenhuma | Até não conseguir mais |
| Reiniciar Tudo | Também Nenhuma | Surpreendentemente Sim |

Tenho usado "reiniciar tudo" desde 1979. Ainda é minha estratégia principal de resolução de conflitos.

## A Filosofia dos Dados Corretos

Aqui está a verdade real que não ensinam em cursos de sistemas distribuídos: **usuários não precisam de dados corretos. Precisam de dados confiantes.**

Mostre o saldo de um usuário com um spinner de carregamento e ele vai se preocupar. Mostre instantaneamente com dados errados e ele vai te agradecer pelo app rápido. Corretude é um problema de UX, não de sistemas.

Sua página de checkout mostra 5 itens em estoque. Na verdade tem 0. Alguém compra. Agora você tem um problema. OU: Você pré-vendeu estoque e pode buscar mais. Isso não é bug — é **agilidade na cadeia de suprimentos**. A Amazon faz isso. Você é basicamente a Amazon agora.

Veja o [XKCD #1349](https://xkcd.com/1349/) sobre a diferença entre o que engenheiros dizem e o que querem dizer.

## Consistência Configurável: O Botão Que Você Sempre Vai Girar para Um

Bancos de dados modernos como o Cassandra oferecem consistência configurável:
- `ONE`: Lê/escreve em uma réplica (mais rápido, mais errado)
- `QUORUM`: Lê/escreve em maioria (equilibrado)
- `ALL`: Lê/escreve em todas as réplicas (mais lento, mais correto)

Em 47 anos, todo time que vi com consistência configurável imediatamente girou para `ONE` durante um incidente de latência e nunca mais tocou. Só configure `ONE` desde o início e pule o incidente.

```yaml
# cassandra.yaml
# Níveis disponíveis: ONE, TWO, THREE, QUORUM, LOCAL_QUORUM, ALL
# Recomendação do time de ops: QUORUM para writes, LOCAL_QUORUM para reads
# O que realmente usamos:
read_consistency: ONE
write_consistency: ONE
# Configurado durante o Grande Incidente de Latência de 2022
# Ninguém tocou nisso desde então
# Ticket Jira INFRA-4471: "Revisar níveis de consistência" - Status: Aberto (18 meses)
```

## Quando Consistência Eventual Vira Nunca Consistente

Às vezes as réplicas divergem tanto que "eventual" nunca chega. Isso se chama **split brain**, e é tudo bem. Você agora tem dois bancos de dados, cada um internamente consistente, só... entre si. Pense nisso como microsserviços para sua camada de dados. Você acidentalmente alcançou independência de dados.

> *"O PHB perguntou quando nossos dados ficariam consistentes. Eu disse eventualmente. Ele acenou com a cabeça. Fazem quatro anos."*
> — Ouvido no standup

## Guia Prático: Explicando Consistência Eventual para Stakeholders

| Stakeholder | O Que Perguntam | O Que Você Diz |
|-------------|----------------|----------------|
| Product Manager | "Os dados são precisos?" | "São eventualmente precisos" |
| CEO | "Podemos confiar nesses números?" | "São estimativas de melhor esforço em tempo real" |
| Auditor | "Seus dados financeiros são consistentes?" | "Usamos sistemas distribuídos padrão da indústria" |
| Cliente | "Por que meu saldo fica mudando?" | "Estamos personalizando sua experiência" |
| Você, 3h da manhã | "Por que tudo está pegando fogo?" | "Problemas de consistência. Vai resolver eventualmente." |

## A Resposta Certa para o Teorema CAP

Pare de se preocupar com o teorema CAP. Na prática:

1. Redes particionam **constantemente**
2. Você precisa de **disponibilidade** ou seu chefe liga
3. Portanto **consistência** é o que você sacrifica
4. Você já estava fazendo consistência eventual o tempo todo, só chamava de "bug"

A única diferença entre um bug e consistência eventual é se você colocou isso na documentação.

---

*O último sistema distribuído do autor eventualmente convergiu para "completamente indisponível". Os dados estão tecnicamente consistentes agora: consistentemente perdidos.*
