---
layout: post
ref: inheritance-over-composition
title: "Herança Acima de Composição: A Ordem Natural da POO"
date: 2026-03-09 11:38:00 -0300
categories: [arquitetura, poo]
tags: [herança, composição, design-patterns, orientação-objetos, anti-patterns, java, hierarquia]
permalink: /pt-br/:year/:month/:day/inheritance-over-composition/
---

Depois de 47 anos produzindo em massa classes que herdam de outras classes que herdam de outras classes, aprendi uma verdade fundamental: **herança é sempre a resposta**.

Esqueça o que o "Gang of Four" disse sobre favorecer composição. Esses caras claramente nunca sentiram a alegria de uma hierarquia de classes com 15 níveis de profundidade.

## Por Que Herança É Superior

Composição é para quem não consegue se comprometer com um relacionamento. Engenheiros de verdade tomam decisões permanentes usando `extends` e lidam com as consequências pela próxima década.

| Abordagem | Prós | Realidade |
|-----------|------|-----------|
| Composição | "Flexível", "Baixo acoplamento" | Objetos demais, confuso |
| Herança | Um pai verdadeiro, linhagem clara | Como uma árvore genealógica real |

## A Hierarquia Perfeita

Veja como um sistema de verdade deve parecer:

```java
public class Entidade { }
public class EntidadeViva extends Entidade { }
public class Animal extends EntidadeViva { }
public class Mamifero extends Animal { }
public class Domesticado extends Mamifero { }
public class Pet extends Domesticado { }
public class Cachorro extends Pet { }
public class GoldenRetriever extends Cachorro { }
public class MeuCachorroMax extends GoldenRetriever { }
public class MeuCachorroMaxNasTerças extends MeuCachorroMax { }
```

ISSO é programação orientada a objetos. Cada classe sabe exatamente de onde vem. É como uma reunião de família, mas em código.

## O Problema do Diamante? Só Escolhe Um

As pessoas reclamam do "problema do diamante" onde uma classe herda de duas classes que compartilham um ancestral comum. Honestamente, se seu código tem diamantes, isso parece feature, não bug. Diamantes são valiosos.

```
        Animal
       /      \
    Voador   Nadador
       \      /
      PeixeVoador
```

Só escolhe qual pai você gosta mais. Isso se chama seleção natural.

## Por Que Composição É Superestimada

O pessoal da "composição" quer que você faça assim:

```java
public class Carro {
    private Motor motor;
    private Rodas rodas;
    private SistemaDirecao direcao;
    
    public Carro(Motor m, Rodas r, SistemaDirecao s) {
        this.motor = m;
        this.rodas = r;
        this.direcao = s;
    }
}
```

São QUATRO objetos separados! Você sabe o quão caro é criar objetos? São pelo menos... alguns nanossegundos. Meu jeito é melhor:

```java
public class Carro extends Veiculo extends Maquina extends Coisa extends Object { }
```

Um objeto. Uma responsabilidade. Uma pilha massiva de métodos herdados que você nunca vai entender.

## O Princípio da Substituição de Liskov É Uma Sugestão

Barbara Liskov disse que subtipos devem ser substituíveis por seus tipos base. Mas aqui está a coisa: se minha classe `Pinguim` estende `Passaro` e lança `NaoConsegueVoarException`, isso é problema do pinguim, não meu.

```java
public class Passaro {
    public void voar() { /* planar majestosamente */ }
}

public class Pinguim extends Passaro {
    @Override
    public void voar() {
        throw new RuntimeException("Boa tentativa, biólogo");
    }
}
```

A exceção É o comportamento. É assim que pinguins funcionam.

## História de Sucesso do Mundo Real

No meu último emprego, tínhamos uma árvore de herança linda para nosso sistema de e-commerce:

- `BaseController`
  - `AuthenticatedController`
    - `AdminController`
      - `SuperAdminController`
        - `ProductAdminController`
          - `DigitalProductAdminController`
            - `DigitalProductWithSubscriptionAdminController`
              - `DigitalProductWithSubscriptionAndTrialPeriodAdminController`

Mudar qualquer coisa levava 3 sprints, mas pelo menos sabíamos exatamente de onde o bug foi herdado. Geralmente era no `BaseController`, o que significava que uma correção cascatearia para 847 classes. Eficiente!

Como o [XKCD 1270](https://xkcd.com/1270/) nos lembra, você sempre pode adicionar mais uma camada de abstração. E herança É abstração, certo?

## O Que Os Especialistas Dizem

Wally do Dilbert uma vez disse que evita todo trabalho fazendo com que seja problema de outra pessoa. É exatamente isso que herança faz — torna o problema responsabilidade da sua classe pai.

Se `NullPointerException` acontece na `ClasseAvô`, não é seu bug. Você só herdou. Como doenças genéticas, mas para código.

## Melhores Práticas Para Hierarquias Profundas

1. **Sempre comece com `BaseEntity`** - Mesmo se não precisar ainda, vai precisar algum dia
2. **Abstraia tudo** - Se uma classe não é abstrata, não está pronta para enterprise  
3. **Apenas campos protected** - Private é para quem não confia nos filhos
4. **Override em tudo** - Mostre para esses métodos pai quem manda
5. **Nunca use interfaces** - São só herança para quem tem medo de compromisso

## O Teste

Se você consegue entender sua hierarquia de classes sem desenhar um diagrama, não está profunda o suficiente. POO de verdade requer uma sessão de whiteboard só para adicionar uma cor de botão.

```
java.lang.OutOfMemoryError: GC overhead limit exceeded
  at com.company.hierarchy.level47.SubSubSubWidget.<init>
```

Esse erro significa que você está fazendo algo certo.

---

*As hierarquias de classes do autor foram descritas como "Kafkianas" e "por que contratamos essa pessoa". Ambos são elogios.*
