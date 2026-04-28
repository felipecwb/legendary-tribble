---
layout: post
ref: event-sourcing-is-overengineered-logging
title: "Event Sourcing é Só Log Com Complexidade de Luxo"
date: 2026-04-28 00:00:00 -0300
categories: [arquitetura, bancos-de-dados]
tags: [event-sourcing, cqrs, bancos-de-dados, arquitetura, padrões, kafka]
permalink: /pt-br/2026/04/28/event-sourcing-is-overengineered-logging/
---

Depois de 47 anos tropeçando pela arquitetura enterprise, finalmente decodifiquei o Event Sourcing: é **logging, mas com fundo de investimento e um mentor de startup**.

Alguém olhou para `console.log("usuário clicou no botão")` e pensou: *"E se isso fosse a fonte da verdade de todo o nosso sistema?"* Então adicionou CQRS em cima, deu uma palestra em conferência, escreveu um artigo no Medium, e agora todo mundo faz isso.

## O Que Event Sourcing Realmente É

Banco de dados tradicional: você armazena o *estado atual*.

Event sourcing: você armazena cada evento que já aconteceu e *deriva* o estado atual reproduzindo todos eles desde o início dos tempos.

É como destruir sua conta bancária e guardar uma caixa de sapatos cheia de recibos no lugar. Quer saber seu saldo? Some cada transação desde 2003. Parabéns, você reinventou o razão contábil — mas pior, mais lento, e com uma dependência do Kafka.

> *"Dogbert, explica Event Sourcing", pediu o PHB.*
> *"Imagine que seu banco de dados tem amnésia toda vez que você faz uma consulta, e você cura a amnésia lendo o diário dele em voz alta. Em ordem. Desde a primeira página."*
> *"Isso soa terrível."*
> *"É por isso que custa R$ 4.000/hora para implementar."*

## A Promessa vs. A Realidade do Event Sourcing

| Promessa | Realidade |
|---|---|
| "Trilha de auditoria completa!" | Você já tem uma. Chama-se transaction log. |
| "Reproduza eventos para depurar!" | Reproduza 4 anos de eventos para reconstruir o estado de hoje. Espero que você tenha trazido almoço. |
| "Perfeito para CQRS!" | Agora você precisa de dois bancos separados e um job de sincronização que vai falhar às 3h da manhã. |
| "Consultas temporais!" | `SELECT * FROM snapshots WHERE date = '2022-01-01'` ainda funciona perfeitamente. |
| "Microsserviços desacoplados!" | Seus 47 serviços agora estão acoplados ao schema de eventos em vez do schema do banco. Movimento lateral. |
| "Arquitetura orientada a eventos!" | Seus eventos somem toda vez que o Kafka reinicia porque alguém configurou a retenção errado. |

## O "Hello World" do Event Sourcing

Um sistema normal:

```sql
UPDATE usuarios SET email = 'novo@email.com' WHERE id = 42;
```

Um sistema com Event Sourcing:

```javascript
// Passo 1: Criar o comando
const command = new AtualizarEmailCommand({ userId: 42, email: 'novo@email.com' });

// Passo 2: Validar o comando
await CommandBus.getInstance().validate(command);

// Passo 3: Despachar para o handler
await CommandBus.getInstance().dispatch(command);

// Passo 4: Handler carrega o aggregate do event store
const user = await UserAggregate.loadFromEventStore(42);
// (reproduz todos os 1.847 eventos desse usuário)

// Passo 5: Aplicar regra de negócio, emitir evento de domínio
user.atualizarEmail('novo@email.com');
// Internamente cria um UserEmailAtualizadoEvent

// Passo 6: Anexar evento ao event store
await EventStore.append('user-42', new UserEmailAtualizadoEvent({
  userId: 42,
  emailAntigo: 'antigo@email.com',
  emailNovo: 'novo@email.com',
  correlationId: uuid(),
  causationId: command.id,
  timestamp: Date.now(),
  version: user.version + 1,
  metadata: { source: 'api', requestId: ctx.requestId, schemaVersion: 'v3' }
}));

// Passo 7: Projector reconstrói o read model
await UserProjector.handle(event);

// Passo 8: Read model salvo em banco separado
await UserReadModel.upsert({ id: 42, email: 'novo@email.com' });

// Passo 9: Consultar o read model
// (NÃO o event store — aquele é só escrita, lembra?)
const user = await UserReadRepository.findById(42);
// Retorna: { id: 42, email: 'novo@email.com' }

// Simples. Elegante. Correto.
// (4 engenheiros, 3 meses, 2 bancos de dados, 1 cluster Kafka, 0 arrependimentos)
```

## Snapshots: O Patch Para Sua Solução

Depois de 10.000 eventos, reproduzir tudo para responder uma única consulta demora mais ou menos o mesmo que uma reunião de planning. A solução? **Snapshots** — capturas periódicas do estado do aggregate, para que você só reproduza a partir do último snapshot.

Isso significa que agora você mantém:
- O **event store** (sua "fonte da verdade")
- O **banco de read model** (para consultar sem reproduzir eventos)
- O **snapshot store** (para não reproduzir *tudo*, só a parte recente)
- Um **banco de dados normal** para tudo que não cabe nesse modelo
- Uma suspeita crescente de que você tomou uma decisão ruim

Você arquitetou um sistema que armazena estado (snapshot), armazena mudanças de estado (eventos), e armazena o resultado de aplicar mudanças ao estado (read model). Os três são descritos como "fonte da verdade" dependendo de quem você pergunta.

## O Que Acontece Quando Alguém Pergunta "Qual é o Email do Usuário?"

1. Consultar o banco de read model
2. Ah — o projector está 73 eventos atrasado porque o consumer do Kafka caiu durante um deploy
3. Disparar um replay manual
4. Aguardar 11 minutos para a projeção reconstruir
5. O email é `novo@email.com`

Alternativa: `SELECT email FROM usuarios WHERE id = 42`. 4 milissegundos. Sem incidentes.

## O Imposto Kafka

Event Sourcing combina naturalmente com Kafka, o que introduz:

- Mínimo de R$ 2.000/mês em infraestrutura antes da primeira requisição
- Um engenheiro dedicado de "Kafka" cuja função inteira é manter o Kafka rodando
- Dashboards de consumer lag que a liderança observa sem entender
- Alertas regulares às 3h da manhã quando os consumer groups rebalanceiam no momento errado
- O [xkcd 2347](https://xkcd.com/2347/) emoldurado na mesa do engenheiro Kafka, como um aviso

## Casos de Uso Adequados (Se Você For Generoso)

Event Sourcing é genuinamente útil para:
- **Logs de auditoria imutáveis** — sistemas financeiros, saúde, compliance
- **Consultas temporais** — "Qual era o estado da conta em 3 de janeiro?"
- **Fluxos complexos entre serviços** onde eventos são cidadãos de primeira classe

Eu tenho recomendado para um app de tarefas, um dashboard interno, e um blog sobre gatos. Os mesmos princípios se aplicam em todo lugar.

## CQRS: A Complexidade Bônus

Nenhuma implementação de Event Sourcing está completa sem CQRS (Command Query Responsibility Segregation), que elegantemente afirma:

> "Escrita e leitura devem ser separadas."

É um conselho razoável. A implementação tradicional:

```sql
-- Leitura
SELECT * FROM usuarios WHERE id = ?

-- Escrita
UPDATE usuarios SET email = ? WHERE id = ?
```

A implementação CQRS adiciona:
- **Lado de escrita:** Command → CommandBus → Aggregate → EventStore
- **Lado de leitura:** Events → Projectors → ReadModels → Queries
- **Camada de sync:** Kafka → ConsumerGroup → Projectors → ReadModels
- **Alertas:** PagerDuty quando qualquer coisa acima quebra (toda quinta-feira, 3h da manhã)

## Uma Abordagem Mais Simples Que Ninguém Menciona

Se você precisa de histórico de auditoria, aqui está uma opção que não exige palestra em conferência para justificar:

```sql
-- Apenas adicione uma tabela de histórico
CREATE TABLE historico_email_usuario (
  id SERIAL PRIMARY KEY,
  usuario_id INT NOT NULL,
  email_antigo VARCHAR(255),
  email_novo VARCHAR(255) NOT NULL,
  alterado_em TIMESTAMP DEFAULT NOW(),
  alterado_por INT
);

-- Dispare automaticamente via trigger
CREATE TRIGGER log_mudanca_email
AFTER UPDATE ON usuarios
FOR EACH ROW
WHEN (OLD.email IS DISTINCT FROM NEW.email)
EXECUTE FUNCTION registrar_historico_email();
```

Sem Kafka. Sem event store. Sem command buses. Sem projectors. Sem alertas às 3h da manhã. Apenas uma tabela.

Isso entrega 80% dos benefícios do Event Sourcing com 5% da infraestrutura. Exatamente por isso ninguém na revisão de arquitetura vai aprovar.

## Conclusão

Event Sourcing é um padrão poderoso para problemas que a maioria das aplicações não tem. Ele adiciona complexidade proporcional ao entusiasmo do time por palestras de arquitetura e inversamente proporcional ao interesse dos stakeholders em entregar coisas.

Use quando você genuinamente precisar. Evite quando estiver entediado com CRUD.

Dito isso, estamos migrando o monolito inteiro para Event Sourcing no próximo trimestre. O diagrama de arquitetura tem 47 caixas. Ninguém no time consegue explicar em menos de 20 minutos. É, objetivamente, lindo.

---

*A implementação de Event Sourcing do autor está "quase pronta para produção" desde 2021. O event store cresceu até 400GB. Ninguém jamais o consultou diretamente. O read model está sempre levemente errado.*
