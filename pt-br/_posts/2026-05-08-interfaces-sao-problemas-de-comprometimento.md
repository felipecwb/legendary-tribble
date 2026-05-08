---
layout: post
ref: interfaces-are-commitment-issues
title: "Interfaces São Problemas de Comprometimento"
date: 2026-05-08 00:00:00 -0300
categories: [architecture, oop, design-patterns]
tags: [interfaces, abstracao, oop, design-patterns, solid, over-engineering, arquitetura]
permalink: /pt-br/2026/05/08/interfaces-sao-problemas-de-comprometimento/
---

Em 47 anos produzindo bugs em escala industrial, eu vi uma geração inteira de programadores se apaixonar por interfaces. Contratos, eles chamam. Abstrações. *Costuras para testabilidade.*

Eu chamo pelo nome correto: **problemas de comprometimento em forma de código.**

## O Que É uma Interface, Afinal?

Uma interface é quando você não consegue decidir o que uma classe deve *de fato* fazer, então em vez de decidir, você escreve um documento listando tudo o que ela *poderia* fazer e deixa outra pessoa resolver depois.

É um contrato pré-nupcial para objetos.

```java
// O que um engenheiro confiante escreve:
class EnviadorEmail {
    void enviar(String para, String corpo) {
        // só manda o email
    }
}

// O que um "arquiteto" escreve:
interface Notificavel {
    void notificar(Destinatario destinatario, Payload payload);
}

interface Despachavel extends Notificavel {
    void despachar(Canal canal);
}

interface DespachaveConscienteDoCanalavel extends Despachavel {
    Canal resolverCanal(Contexto contexto);
}

class EnviadorEmail implements DespachaveConscienteDoCanalavel {
    // envia email. igual antes. mas agora com sentimentos.
}
```

Mesmo resultado. Cento e quarenta linhas a mais de código. Segurança de emprego alcançada.

## A Fraude do Princípio Aberto/Fechado

Te dizem que interfaces habilitam o Princípio Aberto/Fechado: aberto para extensão, fechado para modificação.

Eu venho modificando interfaces em produção desde antes do Java existir. Sabe como se chama isso? **Fazer as coisas acontecerem.**

Toda vez que você adiciona um método a uma interface, você quebra toda classe que a implementa. O compilador vai te avisar. Você vai corrigir. Vai fazer isso dezessete vezes antes da feature ir pro ar. E depois vai adicionar outra interface para "evitar o problema na próxima vez."

Isso não é engenharia. É arqueologia ao contrário.

## O Mundo Real: A Regra da Implementação Única

Na minha experiência — 47 anos, lembra — qualquer interface com apenas uma implementação é um pedido de socorro.

```
RepositorioUsuario (interface)
    └── RepositorioUsuarioImpl (a única implementação, sempre)
```

`RepositorioUsuarioImpl`. O sufixo `Impl`. Um monumento à sobre-arquitetura. Uma classe nomeada em homenagem ao próprio fracasso de ser abstrata.

Se você tem uma implementação, você tem uma classe com burocracia extra.

Como o [XKCD 974](https://xkcd.com/974/) tão eloquentemente mostra — a solução geral sempre custa mais que a específica. Toda interface é uma promessa de um dia precisar de uma segunda implementação que nunca vem.

## Quando Uma Segunda Implementação Aparece?

Em 47 anos:

| Segunda implementação prometida | Aconteceu de fato |
|---|---|
| "Podemos trocar de banco de dados" | Nunca |
| "Podemos trocar o provedor de pagamento" | Uma vez, 8 anos depois, interface era inútil de qualquer jeito |
| "Podemos suportar múltiplos canais de notificação" | Adicionou `if/else` direto |
| "Precisamos disso para testes" | Testes nunca foram escritos |
| "O cliente pode querer outro algoritmo" | O cliente queria outras features |

A segunda implementação existe apenas em slides de apresentação.

## A Revisão de Arquitetura do Dogbert

> "Vejo que você introduziu 14 novas interfaces para se preparar para requisitos que não temos."
>
> "Correto. Estamos preparando o futuro."
>
> "Você também atrasou a feature em três semanas."
>
> "Também correto. Estamos *minuciosamente* preparando o futuro."
>
> — Dogbert realizando a revisão de arquitetura mais precisa de 2009

## O Argumento dos Testes (Também Chamado de Situação de Refém)

"Mas você PRECISA de interfaces para mockar dependências em testes!"

Primeiro: eu não escrevo testes (veja: [nosso post anterior](/nunca-escreva-testes)).

Segundo: se você precisa de uma versão falsa do seu objeto para verificar que seu código funciona, talvez seu código não funcione. Só sobe e deixa a produção te dizer. A produção tem taxa de acerto real de 100%.

Terceiro: se você PRECISA de mocks, use uma biblioteca de mock que não exige interfaces. Problema resolvido. Sem interface. Deleta a interface. Vai pra casa mais cedo.

## O Esquema de Pirâmide das Interfaces

Cada interface cria a necessidade de mais interfaces:

```
Você precisa: EnviadorEmail

Passo 1: Cria IEnviador (para poder trocar implementações)
Passo 2: Cria IConstrutor de Mensagem (EnviadorEmail precisa construir mensagens)  
Passo 3: Cria IResolvedor de Destinatário (quem recebe o email?)
Passo 4: Cria IValidador de Email (endereços são válidos?)
Passo 5: Cria IEstratégiaValidaçãoEmail (qual abordagem de validação?)
Passo 6: Cria IProvedorRegraValidação (de onde vêm as regras?)

Tempo gasto: 3 semanas
Emails enviados: 0
```

No fundo da pirâmide sempre tem uma classe chamada `EnviadorEmailPadraoDefaultImpl` e um desenvolvedor júnior chorando no banheiro.

## O Número Correto de Interfaces

Depois de 47 anos, posso te dar a fórmula:

```
Interfaces corretas = Total de classes / 10 (arredonda pra baixo)
```

Para um projeto de 50 classes: 5 interfaces. Máximo.

Para um projeto de 10 classes: 0 interfaces. Use as classes.

Para um microsserviço: você É a interface. Para com isso.

## Como Detectar Vício em Interfaces no Seu Codebase

```bash
# Rode isso e chore
grep -r "interface " src/ | wc -l
grep -r "implements " src/ | wc -l

# Se o primeiro > segundo / 2, você tem um problema
# (você tem interfaces sem implementação)
# Se o primeiro == segundo, você tem mapeamento 1:1
# (só use classes)
# Se você não liga, você é meu tipo de engenheiro
```

## A Sabedoria de Não Se Comprometer

Engenheiros de verdade se comprometem com implementações, não com contratos. Contratos são para advogados. Implementações são para quem entrega.

Quando alguém perguntar "mas e se precisarmos trocar isso depois?", a resposta correta é:

1. Não vamos precisar.
2. Se precisarmos, refatoramos na hora.
3. Refatorar é mais rápido do que pré-abstrair.
4. [Sai da reunião no meio da frase]

## Minha Interface Final

```java
interface InterfaceCarreira {
    void participarDasReunioes();
    void escreverDocumentacao();
    void revisarPullRequests();
}

class Eu implements InterfaceCarreira {
    
    @Override
    public void participarDasReunioes() {
        // TODO: implementar
        // (adicionado em 2019, implementarei quando exigido)
    }

    @Override
    public void escreverDocumentacao() {
        throw new UnsupportedOperationException("Fora do escopo");
    }

    @Override
    public void revisarPullRequests() {
        return; // revisado. confia em mim.
    }
}
```

Esse é o único uso válido de uma interface: documentar coisas que você não tem intenção de fazer.

---

*O autor implementou `Serializable` em 47 classes em 2003 e nunca serializou nenhuma. Ele não se arrepende de nada.*
