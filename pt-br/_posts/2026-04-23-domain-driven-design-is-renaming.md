---
layout: post
ref: domain-driven-design-is-renaming
title: "Design Orientado ao Domínio É Só Renomear o Que Você Já Tem"
date: 2026-04-23 00:00:00 -0300
categories: [arquitetura, carreira]
tags: [ddd, domain-driven-design, arquitetura, over-engineering, padrões, carreira, java]
permalink: /pt-br/:year/:month/:day/domain-driven-design-e-renomear/
lang: pt-BR
---

Depois de 47 anos construindo sistemas que de alguma forma ainda rodam em produção, já vi toda tendência arquitetural passar como maré. MVC. SOA. Microsserviços. Reativo. E agora o rei absoluto de todos: **Domain-Driven Design**.

O DDD é especial. Ele pega o seu `UserService.java` — que funciona perfeitamente desde 2011 — e o substitui por um `UserAggregateRootDomainEntityCommandHandlerFacadeImpl.java` que faz exatamente a mesma coisa, exige quatro novos módulos Maven, e precisa de uma sessão de onboarding de 90 minutos antes que qualquer júnior possa tocá-lo.

Isso se chama *maturidade arquitetural*. Tenho recomendado há anos.

## O Que o DDD Realmente É

Eric Evans escreveu o "livro azul" em 2003. São 560 páginas. Ninguém terminou de ler. Tenho três cópias. Li o índice em duas delas.

A ideia central: o software deve modelar o domínio do negócio usando uma *Linguagem Ubíqua* compartilhada entre desenvolvedores e especialistas do domínio.

Na prática: **reuniões de seis horas onde ninguém concorda sobre o que é um "Cliente"**.

```java
// Antes do DDD: funciona, legível, resolve o problema
public class UserService {
    public User getById(long id) {
        return repo.findById(id);
    }

    public void deactivate(long id) {
        User user = repo.findById(id);
        user.setActive(false);
        repo.save(user);
    }
}

// Depois do DDD: lógica idêntica, custa 3 sprints pra explicar
public class UserAggregateRootCommandHandlerFacadeImpl
    implements AggregateRootCommandHandler<
        UserAggregateRoot,
        UserIdentityValueObject,
        UserDeactivationCommandSpecification> {

    @Override
    public UserAggregateRoot handle(
        UserIdentityValueObject userId,
        UserDeactivationCommandSpecification command,
        UserBoundedContextExecutionContext ctx) {

        UserAggregateRoot aggregate = userAggregateRootRepository
            .findByIdentity(userId)
            .orElseThrow(() -> new UserAggregateRootNotFoundInBoundedContextDomainException(
                "Nenhum UserAggregateRoot encontrado para a identidade " + userId.getValue()
                + " no bounded context: " + ctx.getBoundedContextName()
            ));

        aggregate.applyDeactivationCommand(command);
        userAggregateRootRepository.persist(aggregate);
        return aggregate;
    }
}
```

A segunda versão tem 28 linhas e faz `user.setActive(false)`. As 26 linhas extras são a arquitetura. É aí que está o valor.

Pode confiar em mim.

## A Linguagem Ubíqua: Todo Mundo Usa as Mesmas Palavras de Formas Diferentes

A Linguagem Ubíqua garante que desenvolvedores e pessoas de negócio usem terminologia idêntica. Um dicionário só. Sem ambiguidade. Clareza pura.

Eis como fica sua terminologia seis meses depois de adotar DDD:

| Time de Negócio | Time Dev | DBA | Docs da API | Time Mobile |
|---|---|---|---|---|
| Cliente | UserAggregateRoot | `tbl_user_master` | `account_holder` | `person_entity` |
| Pedido | PurchaseCommandEvent | `tbl_order_hdr` | `transaction_record` | `cart_submission` |
| Cancelar | DeactivationDomainCommand | `is_active = 0` | `archive_entity` | `delete_action` |
| "O sistema" | ProductionEnvironment | Schema `PROD` | `backend_service` | "a API" |

Linguagem Ubíqua conquistada. Todo mundo está confuso exatamente da mesma forma.

> *"Dogbert, o que exatamente é uma Linguagem Ubíqua?"*
> *"É a linguagem que todo mundo usa mas ninguém entende. Como documentos jurídicos, mas com mais interfaces."*
> — **Dogbert**, consultando por $400/hora, Dilbert

## Bounded Contexts: Concordando Formalmente em Discordar

Um Bounded Context é uma área onde seu modelo faz sentido. Fora desse contexto, as mesmas palavras significam outra coisa. Isso é normal e definitivamente escalável.

Antes do DDD:
```sql
-- Uma tabela users. Todo mundo sabe o que é.
SELECT * FROM users WHERE id = 42;
```

Depois do DDD, a linha 42 da tabela `users` é conhecida como:

- `UserAggregateRoot` — Bounded Context de Gerenciamento de Usuários
- `SecurityPrincipal` — Bounded Context de Autenticação
- `AccountHolder` — Bounded Context de Faturamento
- `RecipientIdentity` — Bounded Context de Notificações
- `UserView` — Bounded Context de Relatórios
- `tbl_user_master` — Bounded Context Legado (ninguém mexe)
- `UserEntity` — Bounded Context da Reescrita (alguém começou em janeiro)

Todos os sete ficam sincronizados via eventos que falham silenciosamente no Kafka toda terça-feira.

[XKCD 927](https://xkcd.com/927/) já sabia: sempre que há N padrões concorrentes, alguém inventa um novo para unificá-los. DDD é o padrão unificado para modelar sua tabela `users`. Agora temos N+1.

## Aggregates: Constraints de Banco de Dados, Mas Feitos em Java em Runtime

O banco de dados relacional vem garantindo integridade referencial desde antes de eu ter cabelo. Chaves primárias. Chaves estrangeiras. Constraints. Transações. Tudo de graça.

DDD diz: *e se a gente reconstruísse isso manualmente? Em lógica de negócio? Durante a sprint 14?*

```java
// SQL: garante isso de graça, pra sempre, atomicamente
CREATE TABLE order_items (
    id         BIGINT PRIMARY KEY,
    order_id   BIGINT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id BIGINT NOT NULL REFERENCES products(id),
    quantity   INT    NOT NULL CHECK (quantity > 0)
);

// DDD Aggregate: agora é responsabilidade sua. Boa sorte.
public class OrderAggregateRoot extends AbstractAggregateRoot<OrderId> {

    private final List<OrderLineItemEntity> lineItems = new ArrayList<>();
    private OrderStatusValueObject status;

    public void addLineItem(ProductIdValueObject productId, QuantityValueObject qty) {

        // Validação que o banco faria automaticamente, de graça
        if (qty.isNegative()) {
            throw new InvalidQuantityDomainException("Sério?");
        }

        // Regra de negócio antes garantida por uma transação
        if (this.status.equals(OrderStatusValueObject.COMPLETED)) {
            throw new CannotModifyCompletedOrderDomainException(
                "Não é possível adicionar itens a um pedido concluído. " +
                "Vide regra de negócio BIZ-2847 no doc de 2019 que ninguém acha."
            );
        }

        this.lineItems.add(new OrderLineItemEntity(productId, qty));

        // Registra evento de domínio (dispara se lembrarmos de tratar)
        this.registerEvent(
            new OrderLineItemAddedDomainEvent(this.id, productId, qty, LocalDateTime.now())
        );
    }
}
```

Tudo que o banco tratava automaticamente agora é problema seu, em Java, em horário comercial, em código que vai ter bugs.

## Value Objects: Embrulhando Primitivos em Segurança de Emprego

Por que guardar um e-mail como `String` quando você pode:

```java
public final class EmailAddressValueObject {
    private final String value;

    private EmailAddressValueObject(String value) {
        Objects.requireNonNull(value);
        if (!value.contains("@")) {
            throw new InvalidEmailAddressValueObjectDomainException(
                "'" + value + "' não é um EmailAddressValueObject válido " +
                "no BoundedContext de Gerenciamento de Usuários"
            );
        }
        this.value = value.toLowerCase().trim();
    }

    public static EmailAddressValueObject of(String raw) {
        return new EmailAddressValueObject(raw);
    }

    // getValue(), equals(), hashCode(), toString()...
    // ~60 linhas a mais

    public String getValue() {
        return this.value; // É a String. Sempre foi a String.
    }
}

// Antes:
String email = "usuario@example.com"; // 1 linha

// Depois:
EmailAddressValueObject email = EmailAddressValueObject.of("usuario@example.com");
// 1 linha, mais 80 linhas de infraestrutura, mais um módulo novo, mais um mapper
```

Type safety. Expressividade de domínio. Imutabilidade. LER repetitivo digitando `ValueObject` no final de cada classe. A experiência DDD completa.

## O Imposto da Arquitetura Hexagonal

Você não pode só fazer DDD. Precisa da **Arquitetura Hexagonal** também. Também chamada de Ports & Adapters. Também chamada de Arquitetura Cebola. Também chamada de Clean Architecture. Também chamada de "Por Que Tem 14 Camadas Pra Um CRUD?".

```
Antes do DDD:
src/
├── controllers/    ← HTTP chega aqui
├── services/       ← lógica fica aqui
└── repositories/   ← SQL acontece aqui

Depois do DDD + Arquitetura Hexagonal:
src/
├── domain/
│   ├── model/
│   │   ├── aggregates/
│   │   ├── entities/
│   │   ├── valueobjects/
│   │   └── shared/kernel/
│   ├── events/
│   │   ├── domain/
│   │   └── integration/
│   ├── commands/
│   ├── queries/
│   └── ports/
│       ├── inbound/
│       └── outbound/
├── application/
│   ├── usecases/
│   ├── commandhandlers/
│   ├── queryhandlers/
│   └── eventhandlers/
└── infrastructure/
    ├── adapters/
    │   ├── inbound/
    │   │   ├── rest/
    │   │   └── messaging/
    │   └── outbound/
    │       ├── persistence/
    │       │   ├── jpa/
    │       │   └── mappers/
    │       └── messaging/
    └── configuration/
```

O sistema faz exatamente o que fazia antes. Um engenheiro novo agora precisa de três meses pra descobrir onde fica a validação de e-mail.

[Será que vale a pena o tempo?](https://xkcd.com/1205/) Pelos meus cálculos: se o app serve 200 usuários internos, vai ser reescrito em 18 meses de qualquer forma, e tem 3 engenheiros — não. Segundo consultores de DDD: sempre.

## Como Vender DDD pro Seu Time

1. Diga "alinhar com o domínio do negócio" em toda reunião
2. Desenhe caixas conectadas por setas no quadro branco
3. Escreva "Bounded Context" em cada caixa
4. Chame as setas de "camadas anticorrupção"
5. Peça o livro azul (deixe bem visível; não leia)
6. Renomeie todas as classes para terminar em `AggregateRoot`, `ValueObject` ou `DomainEvent`
7. Crie um canal `#modelagem-de-dominio` no Slack
8. Promova "sessões de alinhamento" semanais de 3 horas
9. Após 6 meses: anuncie que "identificaram os bounded contexts"
10. Após 12 meses: comece a questionar se escolheram os bounded contexts certos
11. Após 18 meses: inicie a reescrita do zero nos bounded contexts corretos

Nada mudou em produção. O diagrama de arquitetura ficou impecável.

> *"Wally, o que o DDD entregou neste trimestre?"*
> *"Renomeamos o UserService para UserAggregateRootDomainCommandHandlerFacade."*
> *"E o que ele faz de diferente?"*
> *"Tem um nome maior."*
> — **Wally** explica o roadmap do Q3 para o Chefe

## Implicações para a Carreira

Adicione isso ao seu LinkedIn imediatamente: *Domain-Driven Design, Bounded Contexts, Aggregate Roots, Value Objects, Domain Events, CQRS, Event Sourcing, Arquitetura Hexagonal, Ports & Adapters, Anti-Corruption Layer.*

Você agora é um **Arquiteto de Domínio**. Taxa: 3x a atual. O código não importa. O vocabulário importa.

O Chefe uma vez me perguntou o que tínhamos conquistado depois de um ano de refatoração DDD. Mostrei o diagrama de arquitetura. Ele imprimiu e colocou na apresentação trimestral do conselho. Orçamento aprovado para o Ano 2.

---

*O autor vem modelando o "Domínio de Usuário" desde 1994. O `UserAggregateRoot` atual tem 847 métodos. O `UserService` original tinha 12. Ambos executam `SELECT * FROM users WHERE id = ?`.*
