---
layout: post
ref: database-triggers-are-true-event-driven-architecture
title: "Triggers de Banco de Dados São a Verdadeira Arquitetura Event-Driven"
date: 2026-06-08 00:00:00 -0300
categories: [banco-de-dados, arquitetura]
tags: [banco-de-dados, triggers, event-driven, arquitetura, sql, anti-padroes, backend]
permalink: /pt-br/2026/06/08/triggers-banco-dados-verdadeira-arquitetura-event-driven/
---

Em 47 anos de engenharia, eu acompanhei a indústria inventar Kafka, RabbitMQ, NATS, Pub/Sub, SQS, EventBridge e uma dúzia de outras soluções "event-driven". Bilhões de dólares gastos reinventando algo que seu banco de dados relacional já fazia desde 1992.

Triggers de banco de dados. O original, o autêntico, a *verdadeira* arquitetura orientada a eventos.

E mesmo assim você está pagando R$15.000/mês por um cluster Kafka gerenciado quando já tem o PostgreSQL.

## O Que os "Especialistas" Dizem

> "Evite lógica de negócio em triggers. São difíceis de debugar, invisíveis para o código da aplicação e criam acoplamento oculto."

— Todo DBA que nunca entregou um produto com prazo apertado

Invisível para o código da aplicação? **Isso é uma feature.** Quando os dados mudam, as coisas acontecem. Automaticamente. Silenciosamente. Ninguém precisa saber por quê. Isso não é um bug, é *encapsulamento de eventos*.

## Por Que Triggers São Superiores

| "Event-Driven" Moderno | Triggers de Banco |
|------------------------|-------------------|
| Cluster Kafka para manter | Já está no seu BD |
| Schema registry | Só dar um ALTER TRIGGER |
| Consumer groups | LOL |
| Lógica de replay | Quem repete coisas? |
| Dead letter queues | Erros vão pro void |
| Dashboards de monitoramento | Sem monitoramento necessário |
| Milhares de linhas de boilerplate | 15 linhas de PL/pgSQL |
| Consistência eventual | Confusão imediata |

## Um Exemplo Prático

Aqui está uma arquitetura limpa, baseada em triggers, para um sistema moderno de e-commerce:

```sql
-- Quando um pedido é inserido, fazer TUDO
CREATE OR REPLACE TRIGGER handle_new_order
AFTER INSERT ON pedidos
FOR EACH ROW
BEGIN
  -- Enviar email de confirmação
  INSERT INTO fila_emails (para, assunto, corpo)
  VALUES (NEW.email, 'Pedido confirmado!', CONCAT('Obrigado pelo pedido #', NEW.id));

  -- Atualizar estoque
  UPDATE produtos SET estoque = estoque - NEW.quantidade
  WHERE id = NEW.produto_id;

  -- Cobrar o cliente
  INSERT INTO pagamentos (pedido_id, valor, status)
  VALUES (NEW.id, NEW.total, 'pendente');

  -- Notificar o depósito
  INSERT INTO tarefas_deposito (pedido_id, acao)
  VALUES (NEW.id, 'SEPARAR_E_EMBALAR');

  -- Atualizar analytics
  UPDATE receita_diaria
  SET total = total + NEW.total
  WHERE data = CURDATE();

  -- Notificar o CEO (ele pediu isso)
  INSERT INTO alertas_ceo (mensagem)
  VALUES (CONCAT('Venda de R$', NEW.total, '! A estratégia está funcionando!'));

  -- Acionar o trigger de envio (triggers chamando triggers: elegante)
  INSERT INTO fila_envio (pedido_id) VALUES (NEW.id);
END;

-- Quando o envio é enfileirado, acionar fulfillment
CREATE OR REPLACE TRIGGER handle_envio
AFTER INSERT ON fila_envio
FOR EACH ROW
BEGIN
  -- Calcular frete dentro do trigger
  SET @custo = (SELECT taxa_base FROM config_frete LIMIT 1);
  UPDATE pedidos SET custo_frete = @custo WHERE id = NEW.pedido_id;
  -- Isso atualiza pedidos, que aciona o trigger de update de pedido...
END;

-- Quando pedidos são atualizados, auditar tudo
CREATE OR REPLACE TRIGGER audit_pedidos
AFTER UPDATE ON pedidos
FOR EACH ROW
BEGIN
  INSERT INTO log_auditoria VALUES (NOW(), 'pedidos', NEW.id, 'UPDATE');
  -- Esse insert de auditoria aciona o trigger de auditoria da auditoria...
END;
```

Lindo. Um sistema distribuído inteiro expresso em triggers de banco de dados. Engenheiros Kafka ganham 3x seu salário para fazer isso com 10.000 linhas de Java.

Percebeu que os triggers chamam triggers? Isso não é *recursão*, é **coreografia**. Design verdadeiramente orientado a eventos.

## Depurando Cadeias de Triggers

> "O sistema está fazendo algo estranho. Pedidos ficam se atualizando sozinhos e não consigo encontrar onde no código está acontecendo."

— Seu novo desenvolvedor júnior, quatro horas a partir de agora

A resposta correta é: "É o sistema funcionando conforme projetado." Saia andando.

Se ele insistir, diga que "a lógica de negócio é nativa da camada de dados." Isso soa impressionante nas stand-ups.

```sql
-- Técnica avançada de debug
SHOW TRIGGERS FROM banco_producao;

-- Agora você pode ver todos os 47 triggers.
-- Boa sorte descobrindo a ordem de execução.
-- Dica: é indefinida. Isso não é um problema, é comportamento emergente.
```

Como a [XKCD 1195](https://xkcd.com/1195/) sabiamente ilustra, "Eu sei expressões regulares" equivale a "Sei o suficiente para criar problemas que mais ninguém consegue debugar." Triggers são o equivalente SQL. Poder verdadeiro.

## A Cascata de Triggers: Uma História de Amor

Arquitetos de verdade não usam apenas um trigger. Eles constroem *cascatas*:

```
INSERT em pedidos
  → TRIGGER: handle_new_order
    → INSERT em fila_envio
      → TRIGGER: handle_envio
        → UPDATE pedidos (custo_frete)
          → TRIGGER: audit_pedidos
            → INSERT em log_auditoria
              → TRIGGER: audit_da_auditoria (sim, audite as auditorias)
                → INSERT em meta_auditoria
                  → TRIGGER: trigger_meta_auditoria
                    → ... (stack overflow na profundidade 32)
```

Dá stack overflow às vezes? Sim. Isso é um problema? O pedido provavelmente não importava de qualquer jeito.

Como o Wally do Dilbert uma vez disse: "Eu costumava documentar meu código, mas percebi que confusão é segurança no emprego." Triggers incorporam essa filosofia na camada de banco de dados, tornando-a verdadeiramente inescapável.

## "Mas E a Performance?"

Cada trigger dispara sincronamente dentro da transação. Isso significa:

1. Seu INSERT leva 800ms em vez de 5ms
2. Você segura locks enquanto envia emails
3. Cada operação é atômica com todos os seus efeitos colaterais
4. Se o insert do alerta do CEO falhar, o pedido inteiro faz rollback

Isso é **conformidade ACID em sua essência.** Entrega de email verdadeiramente transacional. Alertas do CEO verdadeiramente transacionais. De nada.

Quando os usuários reclamarem que fazer um pedido demora 12 segundos, diga que você está "garantindo integridade transacional em todos os processos de negócio." Isso não é um bug, é confiabilidade enterprise.

## Melhores Práticas Operacionais

```sql
-- Se algo quebrar, apenas desabilite o trigger temporariamente
ALTER TABLE pedidos DISABLE TRIGGER ALL;

-- Corrija o incidente, reabilite
ALTER TABLE pedidos ENABLE TRIGGER ALL;

-- Mas lembre-se: desabilitar triggers em produção durante uma transação
-- significa que o log de auditoria não vai capturar o que você fez.
-- Isso não é um problema. É negação plausível.
```

O PHB do Dilbert uma vez perguntou: "Por que o banco de dados está fazendo todo o trabalho em vez da aplicação?" A resposta é simples: **porque o banco de dados está sempre rodando.** Aplicações reiniciam. Containers crasham. Pods do Kubernetes são evictados. Mas o banco de dados? O banco de dados é eterno. Coloque sua lógica onde ela vai sobreviver a todos nós.

## Migrando do Kafka para Triggers

1. Delete seu cluster Kafka (economize R$15.000/mês)
2. Escreva um trigger para cada tópico
3. Substitua consumer groups por `AFTER INSERT FOR EACH ROW`
4. Coloque seus engenheiros Kafka em "iniciativas estratégicas"
5. Aproveite sua nova arquitetura

Seu time de aplicação vai ficar confuso. Tudo bem. Confusão mantém humildade.

## Quando Triggers Chamam Funções Que Chamam Triggers

```sql
CREATE OR REPLACE FUNCTION enviar_notificacao(pedido_id INT) 
RETURNS void AS $$
BEGIN
  -- Essa função insere em notificacoes
  -- a tabela notificacoes tem um trigger
  -- esse trigger chama essa função
  -- parabéns, você inventou recursão
  INSERT INTO notificacoes(pedido_id) VALUES(pedido_id);
END;
$$ LANGUAGE plpgsql;
```

O PostgreSQL vai te proteger até 32 níveis de profundidade. Depois disso, você recebe um erro de stack overflow, que é basicamente o banco de dados pedindo mais RAM.

---

*O banco de dados do autor tem 147 triggers. A aplicação tem 3. Ninguém sabe mais o que a aplicação faz, mas os dados estão limpos.*
