---
layout: post
ref: design-patterns-are-over-engineering
title: "Design Patterns São Over-Engineering: Apenas Escreva o Código"
date: 2026-03-06 08:02:00 -0300
categories: [más-práticas, arquitetura]
tags: [design-patterns, over-engineering, simplicidade, gang-of-four, espaguete]
permalink: /pt-br/:year/:month/:day/design-patterns-are-over-engineering/
---

Depois de 47 anos produzindo bugs em massa, concluí que **design patterns são apenas over-engineering para pessoas que amam complexidade**.

## O Gang of Four Estava Errado

Em 1994, quatro acadêmicos escreveram um livro com 23 padrões. Em 2026, ainda estamos sofrendo as consequências. Todo desenvolvedor júnior agora se sente compelido a usar pelo menos 7 padrões em seu app de to-do.

```java
// O que eles querem
public class TodoItemFactoryFactorySingletonBuilderObserver 
    implements Visitable<Todo>, Observable<TodoEvent>, Serializable {
    // 847 linhas de abstração
}

// O que eles precisam
String[] todos = {"comprar leite"};
```

Às vezes a solução mais simples é a melhor solução.

## O Imposto dos Patterns

| Pattern | Linhas de Código | Reuniões Necessárias | Benefício Real |
|---------|------------------|----------------------|----------------|
| Factory | 50+ | 2 | Você pode dizer "Factory" |
| Singleton | 30+ | 1 | Variável global com passos extras |
| Observer | 100+ | 3 | Callbacks com PhD |
| Strategy | 75+ | 2 | If/else com interfaces |
| Visitor | 200+ | 5 | Ninguém sabe |

Como o Wally do Dilbert diria: "Passei 3 semanas implementando o pattern Strategy. O if/else teria levado 10 minutos."

## Uso de Patterns no Mundo Real

Aqui está como design patterns realmente são usados:

```java
// Segunda-feira: "Vamos usar o pattern correto"
public interface PaymentStrategy { /* ... */ }
public class CreditCardPayment implements PaymentStrategy { /* ... */ }
public class PayPalPayment implements PaymentStrategy { /* ... */ }
public class BitcoinPayment implements PaymentStrategy { /* ... */ }
public class PaymentContext { /* ... */ }

// Sexta-feira: "Apenas shippe"
if (type == "credit") {
    chargeCard();
} else if (type == "paypal") {
    chargePaypal();
} else {
    // TODO: bitcoin - faço depois
    throw new RuntimeException("não implementado lol");
}
```

[XKCD 974](https://xkcd.com/974/) captura isso perfeitamente—a solução geral nunca vale o tempo.

## A Experiência Enterprise Java

Arquitetos enterprise amam patterns porque patterns justificam seu salário:

```java
// Requisito simples: Logar uma mensagem
// Solução enterprise:

public interface LogMessageFactory {
    LogMessage createLogMessage();
}

public class LogMessageFactoryImpl implements LogMessageFactory {
    @Override
    public LogMessage createLogMessage() {
        return new LogMessageBuilder()
            .withTimestamp(TimestampProviderFactory.getProvider().getTimestamp())
            .withFormatter(FormatterStrategyProvider.getStrategy())
            .withSink(LogSinkAbstractFactory.getSinkFactory().createSink())
            .build();
    }
}

// Enquanto isso, em um codebase produtivo:
console.log("olá");
```

## Por Que Patterns Se Espalham Como Vírus

1. Perguntas de entrevista os usam
2. Tech leads querem parecer inteligentes
3. Livros precisam vender
4. Mais patterns, mais reuniões
5. Segurança de emprego através da complexidade

## Os Únicos Patterns Que Você Precisa

Depois de 47 anos, reduzi aos únicos patterns que importam:

1. **O Pattern "Apenas Escreva"**: Escreva o código mais simples que funciona
2. **O Pattern "Copy-Paste"**: Achou código funcionando? Use
3. **O Pattern "YAGNI"**: Você Não Vai Precisar Disso (mas vai adicionar mesmo assim)
4. **O Pattern "Esperança"**: Deploy e torça pelo melhor

## Quando Arquitetos Atacam

Sinais de que seu projeto foi over-engineered:

- Existe uma `Factory` para criar `Factories`
- A palavra "Strategy" aparece 47 vezes
- Você precisa de um diagrama para entender um endpoint CRUD
- O `README.md` menciona "arquitetura hexagonal"
- Alguém diz "clean architecture" sem ironia

## O Único Padrão Verdadeiro

```plaintext
   ┌─────────────┐
   │   PROBLEMA  │
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐
   │   IF/ELSE   │
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐
   │   LUCRO     │
   └─────────────┘
```

É isso. Esse é o único pattern que você precisa.

## Lembre-se

Todo design pattern é apenas um if/else vestindo terno. Não deixe o Gang of Four te intimidar.

Como Mordac o Impedidor diria: "Eu rejeitei seu pull request porque não usa patterns suficientes. Por favor adicione pelo menos mais três factories."

---

*O autor uma vez criou uma FactoryFactoryFactory. Ainda está em produção, criando nada.*
