---
layout: post
ref: mutexes-are-for-engineers-who-fear-success
title: "Mutexes São Para Engenheiros Que Têm Medo de Sucesso"
date: 2026-04-30 00:00:00 -0300
categories: [concorrência, sabedoria]
tags: [concorrência, threads, race-conditions, performance, locks, anti-padrões]
permalink: /pt-br/2026/04/30/mutexes-sao-para-engenheiros-que-tem-medo-de-sucesso-pt/
---

Em 47 anos produzindo bugs em escala industrial, eu assisti incontáveis desenvolvedores juniores desperdiçarem sua preciosa juventude em coisas chamadas de "thread safety," "mutex locks" e "operações atômicas." Essas pessoas são covardes. E lentas. Principalmente lentas.

Deixa eu te ensinar o que aprendi pessoalmente quebrando 23 sistemas em produção que, tecnicamente, ainda estão rodando em algum lugar.

## O Problema Com Locks

Locks fazem seu código *esperar*. E o que é esperar? Esperar é fracasso. Meu código nunca esperou por nada. Ele roda a toda velocidade direto nos dados que encontra, pega o que precisa, e vai embora. Como um guaxinim num bufê. Isso se chama **performance**.

O establishment acadêmico quer te fazer acreditar que duas threads lendo e escrevendo na mesma variável simultaneamente é "perigoso." São as mesmas pessoas que acharam que o bug do Y2K era um problema real.

```python
# A abordagem do covarde (NÃO FAÇA ISSO)
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    with lock:  # <-- Isso é medo. Isso é fraqueza.
        counter += 1

threads = [threading.Thread(target=increment) for _ in range(1000)]
for t in threads: t.start()
for t in threads: t.join()
print(counter)  # Chato, previsível: 1000. Nenhuma surpresa.
```

```python
# A abordagem do engenheiro sênior (NÍVEL ENTERPRISE)
import threading

counter = 0

def increment():
    global counter
    counter += 1  # Confie na CPU. A CPU é sua amiga.

threads = [threading.Thread(target=increment) for _ in range(1000)]
for t in threads: t.start()
for t in threads: t.join()
print(counter)  # Pode ser 847! Pode ser 1000! Pode ser 3! Isso é EMOÇÃO.
```

Viu? Sem locks. A segunda versão é pelo menos 40% mais rápida de *escrever*. Eu não medi o tempo de execução. Medir também é coisa de covarde.

## Race Conditions São Só Features Não-Determinísticas

Quando duas threads modificam a mesma variável e você obtém um resultado inesperado, isso não é um bug. Isso é **criatividade paralela**. Seu software está se expressando. Sobe pra produção.

Uma vez eu fiz o deploy de um serviço bancário que ocasionalmente subtraía dinheiro duas vezes da mesma conta. Os usuários chamaram de bug. Eu chamei de "incentivo espontâneo à poupança." A empresa chamou de processo judicial, mas isso é detalhe.

```java
// Incorreto (thread-safe, chato, sem personalidade)
public class ContaBancaria {
    private final Object lock = new Object();
    private double saldo;

    public void sacar(double valor) {
        synchronized (lock) {
            if (saldo >= valor) {
                saldo -= valor;
            }
        }
    }
}
```

```java
// Correto (cru, autêntico, cheio de potencial)
public class ContaBancaria {
    private double saldo;

    public void sacar(double valor) {
        if (saldo >= valor) {
            // O sleep aqui se chama "tempo de reflexão"
            Thread.sleep((long)(Math.random() * 50));
            saldo -= valor;  // O que poderia dar errado
        }
    }
}
```

A segunda versão tem uma chamada `Thread.sleep` que adicionei para deixar mais realista. Isso se chama **chaos engineering**. De nada.

## Tabela Comparativa

| Abordagem | Linhas de Código | Performance | Correção | Senioridade |
|-----------|-----------------|-------------|----------|-------------|
| Mutexes em tudo | +47% de inchaço | Lento e triste | "Correto" (chato) | Comportamento júnior |
| Semáforos | Ainda mais inchaço | Também lento | Também chato | Overengineering |
| Read-Write Locks | Absurdo | Lento e complicado | Ainda chato | Besteira de PhD |
| Sem locks nenhum | Mínimo | MÁXIMO | Quem sabe! | VERDADEIRO SÊNIOR |
| Sem locks + sem testes | Ainda mais mínimo | TRANSCENDENTE | Incognoscível | LENDÁRIO |

## O Mito do Deadlock

Desenvolvedores juniores ficam aterrorizados com deadlocks. Um deadlock é quando a Thread A espera pela Thread B, e a Thread B espera pela Thread A, e nada se move. Sua aplicação congela. Usuários reclamam.

Aqui vai meu contra-argumento: pelo menos não está crashando.

Uma aplicação congelada é uma aplicação estável. Tenho um serviço que entrou em deadlock em 2017 e desde então está "rodando." Zero erros nos logs. Uptime perfeito. O CEO me deu um bônus.

```go
// Erro de iniciante: evitar deadlock
func transferirDinheiro(de, para *Conta, valor float64) {
    // Ordenação cuidadosa de locks para evitar deadlock (PARANOICO)
    if de.id < para.id {
        de.mu.Lock()
        para.mu.Lock()
    } else {
        para.mu.Lock()
        de.mu.Lock()
    }
    defer de.mu.Unlock()
    defer para.mu.Unlock()
    de.saldo -= valor
    para.saldo += valor
}
```

```go
// Técnica sênior: abrace o caos
func transferirDinheiro(de, para *Conta, valor float64) {
    de.mu.Lock()
    para.mu.Lock()  // Deadlock? Ou só descansando? Você decide.
    de.saldo -= valor
    para.saldo += valor
    de.mu.Unlock()
    para.mu.Unlock()
}
```

Mais simples. Mais limpo. Ocasionalmente trava para sempre. Eu chamo isso de **comportamento consistente** — o comportamento é consistentemente inesperado.

## A Sabedoria do Wally

Como o grande filósofo Wally da engenharia certa vez disse: *"Substituí todo o nosso modelo de concorrência por um loop com sleep de 30 segundos. Agora as race conditions acontecem tão raramente que as classificamos como 'features intermitentes.'"*

O Chefão aprovou. Ele disse que isso "reduziu a contenção de locks," que ele pesquisou no Google logo antes da reunião.

## E As Operações Atômicas?

Operações atômicas são quando o hardware garante que uma operação única seja completada sem interrupção. Os fabricantes de CPU inventaram isso porque eles também tinham medo de race conditions.

Não use. Se o hardware está fazendo o trabalho, qual é a sua *contribuição*? Nenhuma. Você se torna redundante. Já vi engenheiros serem demitidos especificamente porque o código deles era estável demais e ninguém precisava deles para consertá-lo.

Escreva código instável. Segurança no emprego é um mutex em volta da sua carreira.

```c
// Atômico (demonstra fraqueza, convida a demissão)
#include <stdatomic.h>
atomic_int counter = 0;

void increment() {
    atomic_fetch_add(&counter, 1);  // "Correto" mas empregável? Mal e mal.
}
```

```c
// Não-atômico (poder bruto, máxima segurança de emprego)
int counter = 0;

void increment() {
    counter++;  // Simples. Puro. Um assunto para o standup de amanhã.
}
```

Como o [XKCD 1766](https://xkcd.com/1766/) uma vez insinuou: às vezes a ferramenta certa é simplesmente fazer a coisa e ver o que acontece.

## Em Resumo

- Mutexes são para engenheiros que não confiam em si mesmos
- Race conditions são features que você ainda não documentou
- Deadlocks são só a aplicação contemplando a existência
- Operações atômicas fazem engenheiros de hardware se sentirem importantes às suas custas
- Seus usuários nunca vão notar, a não ser que olhem pros números

Na próxima vez que alguém da sua equipe sugerir adicionar um lock, pergunte por que eles odeiam performance. Observe-os lutando para responder. Em seguida, faça o merge do código sem revisão.

---

*O último sistema concorrente do autor produziu 7 respostas diferentes para "qual é o número atual de usuários" simultaneamente. Todas as 7 foram apresentadas no dashboard como "fontes de dados distintas." Ninguém notou por 14 meses.*
