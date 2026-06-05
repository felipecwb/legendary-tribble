---
layout: post
ref: cqrs-is-two-databases-because-you-couldnt-fix-the-first-one
title: "CQRS É Só Dois Bancos de Dados Porque Você Não Conseguiu Arrumar o Primeiro"
date: 2026-06-05 00:00:00 -0300
categories: [arquitetura, bancos-de-dados, padrões]
tags: [cqrs, event-sourcing, ddd, arquitetura, bancos-de-dados, overengineering, padrões]
permalink: /pt-br/2026/06/05/cqrs-e-dois-bancos-de-dados-porque-voce-nao-conseguiu-arrumar-o-primeiro/
---

Estou escrevendo e lendo em bancos de dados desde 1979. Isso não era complicado. Você tinha uma tabela. Você dava INSERT. Você dava SELECT. Às vezes UPDATE, mas tentamos não falar sobre isso.

Então alguém, em algum lugar, comeu um burrito estragado e inventou o **CQRS**: Command Query Responsibility Segregation.

A ideia central: *separe suas escritas das suas leituras colocando-as em codebases completamente diferentes, modelos diferentes, bancos de dados diferentes, e sincronize-os de forma assíncrona — tudo isso para fingir que isso resolve um problema de performance que um índice teria resolvido em 20 minutos.*

---

## O Que CQRS Realmente É

Deixe-me traduzir a definição do Wikipedia para o português:

**Oficial:** "CQRS segrega operações que leem dados de operações que atualizam dados usando interfaces separadas."

**Tradução:** "Fizemos dois bancos de dados e agora chamamos isso de padrão arquitetural."

Um sistema CQRS parece assim:

```
Ação do Usuário
      │
      ▼
Command Handler ──escreve──► Banco de Escrita (normalizado, "fonte da verdade")
                                       │
                                Event Bus (async)
                                       │
                                       ▼
                              Read Model Projector
                                       │
                                       ▼
                             Banco de Leitura (desnormalizado, desatualizado)
                                       │
                            ◄──lê──── Query Handler
                                       │
                                       ▼
                                   Resposta
                          (pode estar 300ms atrás da realidade)
```

Minha arquitetura:

```
Ação do Usuário → SQL → Resposta
```

A minha vence em todas as dimensões, exceto em palestras de conferência.

---

## O Read Model: Pagando Para Duplicar Seus Próprios Dados

No CQRS, você mantém um "read model" separado — uma visão pré-computada e desnormalizada dos seus dados, otimizada para queries.

Sabe o que mais é uma visão pré-computada e desnormalizada dos seus dados otimizada para queries?

**Uma view de banco de dados.** Criada em 1974. Vem de graça com o PostgreSQL.

```sql
-- Read model revolucionário do CQRS (equipe inteira, 6 meses)
-- vs.
CREATE VIEW resumo_pedidos AS
SELECT
    p.id,
    p.criado_em,
    u.nome as nome_cliente,
    SUM(ip.preco) as total
FROM pedidos p
JOIN usuarios u ON p.usuario_id = u.id
JOIN itens_pedido ip ON ip.pedido_id = p.id
GROUP BY p.id, p.criado_em, u.nome;
-- Pronto. Estou indo embora. O prêmio Catbert de eficiência é meu.
```

O [XKCD 927](https://xkcd.com/927/) se aplica aqui também, mas não tenho mais energia para explicar por quê. Estou há 47 anos nessa carreira e meus joelhos doem.

---

## Consistência Eventual: A Arte de Estar Errado Agora Mesmo

Aqui está o problema que ninguém menciona nas palestras de CQRS: seu read model é **eventualmente consistente**. Isso significa:

1. Usuário atualiza o e-mail
2. Escrita vai para o banco de Command ✓
3. Evento dispara no message bus
4. Read model projector processa o evento
5. Read model atualiza
6. Isso leva de 50ms a "a fila está congestionada, por favor aguarde"
7. Usuário imediatamente atualiza o perfil
8. **Ele vê o e-mail antigo**

```javascript
// O ticket de suporte que isso cria:
// "Seu sistema está quebrado. Mudei o e-mail e ainda mostra o antigo."
// Resposta do suporte: "Aguarde 30 segundos e tente novamente."
// Usuário: "Sou cliente pagante."
// Você: "Sim e fizemos seus dados mentirem para você. Por design."
```

Em 47 anos, nunca ouvi um usuário dizer "não me importo se os dados que acabei de salvar demoram para aparecer." O PHB chama isso de "feature" na revisão arquitetural. O Dogbert chama de "responsabilidade civil".

---

## O Command Handler: Uma Função Com Ilusões de Grandeza

No código normal, se você quer salvar um pedido, você escreve:

```python
def criar_pedido(usuario_id, itens):
    pedido = Pedido(usuario_id=usuario_id, itens=itens)
    db.salvar(pedido)
    return pedido
```

No CQRS, você escreve:

```python
class CriarPedidoCommand:
    def __init__(self, usuario_id: UUID, itens: List[ItemPedidoDTO]):
        self.usuario_id = usuario_id
        self.itens = itens
        self.correlation_id = uuid4()
        self.causation_id = None
        self.timestamp = datetime.utcnow()
        self.metadata = {}

class CriarPedidoCommandHandler:
    def __init__(
        self,
        command_bus: ICommandBus,
        event_store: IEventStore,
        aggregate_repository: IOrderAggregateRepository,
        domain_event_publisher: IDomainEventPublisher,
        unit_of_work_factory: IUnitOfWorkFactory
    ):
        # mais 14 dependências injetadas aqui
        pass

    def handle(self, command: CriarPedidoCommand) -> CommandResult:
        with self.unit_of_work_factory.create() as uow:
            aggregate = self.aggregate_repository.get_or_create(command.usuario_id)
            domain_events = aggregate.criar_pedido(command.itens)
            self.event_store.append(domain_events)
            self.domain_event_publisher.publish(domain_events)
            uow.commit()
        return CommandResult(success=True, aggregate_id=aggregate.id)
```

As duas salvam um pedido. Uma faz isso em 8 linhas. A outra faz de um jeito que exige 3 dias de onboarding só para entender para onde os dados realmente vão.

| Métrica | Abordagem Simples | CQRS |
|---|---|---|
| Linhas de código | 5 | 200+ |
| Arquivos envolvidos | 1 | 12 |
| Ramp-up do dev júnior | 30 minutos | 2 semanas |
| Debuggabilidade | Abre o psql, olha os dados | Trace distribuído de 45 minutos |
| Dados corretos exibidos | Imediatamente | "Eventualmente" |
| Apresentações de conferência possíveis | 0 | Infinitas |

---

## Event Sourcing: Porque Armazenar Estado Era Simples Demais

CQRS frequentemente vem embalado junto com seu irmão mais pesado, o **Event Sourcing**. Em vez de armazenar o estado atual dos seus dados, você armazena cada evento que levou a esse estado.

Um saldo de conta não é `saldo = 500`. É:

```
ContaAberta(valor=1000)
DinheiroSacado(valor=200)
DinheiroDepositado(valor=100)
DinheiroSacado(valor=400)
# Saldo atual: 500. Levamos 4 eventos para descobrir.
# Em 10 anos: 4.000.000 eventos. Divirta-se com suas "projeções".
```

> "Mas você tem um log de auditoria completo!"
>
> — *Pessoa que nunca precisou reprocessar 10 milhões de eventos para reconstruir um read model depois que um bug corrompeu as projeções*

Sabe o que mais fornece um log de auditoria? **Uma coluna `criado_em` e uma coluna `atualizado_em`.** Custa zero bancos de dados extras. Custa zero PhDs extras.

O Wally teve essa percepção em 1993. Ele foi para casa às 17h naquele dia.

---

## Quando CQRS Realmente Faz Sentido

No espírito da justiça (meu editor insiste), CQRS é genuinamente útil quando:

1. Você tem cargas de leitura e escrita radicalmente diferentes (milhões de leituras vs. centenas de escritas)
2. Seus read models precisam de formatos drasticamente diferentes dos seus write models
3. Você tem um time de plataforma dedicado que entende sistemas distribuídos
4. Você tem um negócio que existirá por tempo suficiente para justificar a complexidade

Para contexto: isso descreve aproximadamente **40 empresas no mundo**. As restantes 400.000 startups usando CQRS estão fazendo isso porque o CTO leu um artigo do Martin Fowler em 2018 e ficou animado.

---

## Minha Recomendação: O Padrão Legendary Tribble CQRS Alternativo

Após pesquisa extensiva (47 anos de incidentes em produção), apresento o padrão LTCQRS: **Legendary Tribble Command Query Realmente Simplificado**:

```sql
-- Escrita
INSERT INTO pedidos (usuario_id, total, status) VALUES (?, ?, 'pendente');

-- Leitura
SELECT * FROM pedidos WHERE usuario_id = ? ORDER BY criado_em DESC;

-- Leitura com problema de performance? tudo bem.
CREATE INDEX idx_pedidos_usuario_id ON pedidos(usuario_id);

-- Ainda lento? Agora você pode pensar em cache.
-- Ainda lento? Parabéns, você tem um problema real de escala.
-- Nesse ponto, sim, considere CQRS. Mas você não está lá.
-- Você está construindo um app todo para um time de 5 pessoas.
```

Sem event bus. Sem command handlers. Sem read model projectors. Sem PhD necessário. Sem pedidos de desculpas "eventualmente consistentes" para os usuários.

Só SQL. Como Deus e Edgar F. Codd pretendiam.

---

## Conclusão

CQRS é uma solução para problemas que você não tem, implementada de formas que criam problemas que você não antecipou, exigindo expertise que você não tem tempo de desenvolver, para servir uma escala que você nunca alcançará.

A ironia é que os engenheiros que mais precisam de CQRS (trabalhando em verdadeira escala de internet) são os que podem se dar ao luxo de construí-lo corretamente. Todo mundo else está só fazendo cosplay de Netflix.

**Seus dados cabem em um único banco de dados PostgreSQL. Suas queries estão lentas porque está faltando um índice. Seu gargalo é aquele foreach chamando a API 500 vezes.**

Nenhum padrão arquitetural conserta código ruim. E CQRS não é substituto para um bom e velho `EXPLAIN ANALYZE`.

O PHB aprovou CQRS porque tem uma sigla bacana. O Mordac, o Impedidor de Serviços de Informação, negou o pedido de infraestrutura. O sistema é eventualmente consistente, e eventualmente ninguém o usa.

---

*O write model e o read model do autor estão fora de sincronia desde o primeiro casamento. Ambos ainda afirmam ser a fonte da verdade.*
