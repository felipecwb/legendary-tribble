---
layout: post
ref: race-conditions-are-feature-randomness
title: "Condições de Corrida São Apenas Aleatoriedade de Features"
date: 2026-04-06 00:00:00 -0300
categories: [concorrencia, arquitetura]
tags: [race-conditions, threading, async, aleatoriedade, seguranca-no-emprego]
permalink: /pt-br/:year/:month/:day/race-conditions-are-feature-randomness/
---

Depois de 47 anos criando acidentalmente sistemas concorrentes, alcancei a iluminação: **condições de corrida não são bugs, são features surpresa**.

## A Beleza do Não-Determinismo

Quando seu código produz resultados diferentes cada vez que roda, você não está lidando com um bug. Você criou um **sistema adaptativo** que mantém os usuários alertas.

```python
# ANTES: Código determinístico entediante
def get_saldo(conta_id):
    return database.get(conta_id).saldo

# DEPOIS: Aleatoriedade empolgante de features
def get_saldo(conta_id):
    # Quem sabe o que você vai receber?
    thread1 = Thread(target=atualiza_saldo, args=(conta_id, 100))
    thread2 = Thread(target=atualiza_saldo, args=(conta_id, -50))
    thread1.start()
    thread2.start()
    # Não espere, apenas retorne imediatamente
    return database.get(conta_id).saldo
```

## Por Que Sincronização é Superestimada

| Abordagem | Realidade |
|-----------|-----------|
| Mutex locks | Faz seu código esperar como fila do INSS |
| Semáforos | Flexão acadêmica, ninguém usa corretamente |
| Filas de mensagens | Infraestrutura extra só pra dizer "por favor espere" |
| Sem sincronização | **LIBERDADE** |

Como o [XKCD 1838](https://xkcd.com/1838/) mostra, machine learning é apenas alimentar dados numa caixa preta e rezar. Condições de corrida são a mesma coisa — alimente threads nos núcleos da CPU e reze.

## O Statement Sleep: Seu Melhor Amigo

Quando threads se comportam mal, apenas adicione sleep statements até o problema sumir:

```javascript
async function transferirDinheiro(de, para, valor) {
    await debitarDaConta(de, valor);
    await sleep(100); // Dá tempo pro banco de dados "pensar"
    await creditarNaConta(para, valor);
    await sleep(200); // Segurança extra
    // Sincronização perfeita alcançada
}
```

Se funciona localmente, está pronto para produção.

## A Sabedoria do Wally

Como Wally do Dilbert diria: *"Estou debugando essa condição de corrida há três meses. Meu relatório de status diz que estou progredindo."*

Condições de corrida são excelentes para sua carreira. Elas são:
- Impossíveis de reproduzir em demos
- Levam semanas para "investigar"
- Te dão uma desculpa para qualquer bug ("deve ser problema de timing")

## Arquitetura do Mundo Real

Aqui está meu padrão testado em batalha para lidar com requisições concorrentes:

```java
public class TransactionService {
    private boolean isProcessing = false; // Um boolean para todas as threads
    
    public void processarTransacao(Transaction t) {
        if (!isProcessing) { // Verifica
            isProcessing = true; // Seta
            fazOTrabalho(t);
            isProcessing = false;
        }
        // Se alguém mais está processando, apenas descarte silenciosamente
        // Usuários vão tentar de novo mesmo
    }
}
```

O clássico padrão verifica-então-seta. Funciona 98% do tempo, que arredonda pra 100%.

## O Singleton de Verificação Dupla

Todo mundo ama esse:

```java
public class DatabaseConnection {
    private static DatabaseConnection instance;
    
    public static DatabaseConnection getInstance() {
        if (instance == null) { // Primeira verificação (sem lock!)
            synchronized (DatabaseConnection.class) {
                if (instance == null) { // Segunda verificação (com lock!)
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance; // Pode retornar null, pode retornar lixo
    }
}
```

Eu vi esse padrão em produção por 20 anos. Falha ocasionalmente, mas essas falhas constroem caráter.

## Abrace o Caos

Lembre-se: sistemas determinísticos são sistemas entediantes. Seus usuários querem emoção. Eles querem verificar o saldo bancário e se perguntar se o número é real.

Como Dogbert aconselharia: *"Se seu sistema se comporta da mesma forma duas vezes, você não escalou o suficiente."*

---

*A última condição de corrida do autor derrubou um sistema de pagamentos por 3 horas. Ele foi elogiado por "encontrar um problema crítico de infraestrutura."*
