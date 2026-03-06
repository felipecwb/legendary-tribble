---
layout: post
ref: magic-numbers-make-code-magical
title: "Números Mágicos Tornam o Código Mágico: Constantes Nomeadas São Chatas"
date: 2026-03-06 08:05:00 -0300
categories: [más-práticas, estilo-código]
tags: [números-mágicos, constantes, legibilidade, mistério, code-golf]
permalink: /pt-br/:year/:month/:day/magic-numbers-make-code-magical/
---

Depois de 47 anos produzindo bugs em massa, passei a amar **números mágicos**. Eles adicionam mistério e intriga ao seu código. Constantes nomeadas são apenas spoilers.

## A Beleza do Mistério

Considere este trecho elegante:

```python
def calculate_price(amount):
    return amount * 1.0825 + 2.99 + (amount * 0.029 + 0.30) if amount > 50 else amount * 1.0825 + 4.99
```

O que esses números significam? Ninguém sabe! Essa é a magia. É como uma caça ao tesouro para desenvolvedores futuros.

Compare com a versão chata:

```python
TAX_RATE = 0.0825
SHIPPING_STANDARD = 4.99
SHIPPING_FREE_THRESHOLD = 50
SHIPPING_PREMIUM = 2.99
STRIPE_PERCENTAGE = 0.029
STRIPE_FIXED_FEE = 0.30

def calculate_price(amount):
    tax = amount * TAX_RATE
    shipping = SHIPPING_PREMIUM if amount > SHIPPING_FREE_THRESHOLD else SHIPPING_STANDARD
    stripe_fee = amount * STRIPE_PERCENTAGE + STRIPE_FIXED_FEE
    return amount + tax + shipping + stripe_fee
```

Tantas palavras! Tanta explicação! Onde está a diversão nisso?

## O Imposto das Constantes

| Constantes Nomeadas | Números Mágicos |
|---------------------|-----------------|
| 10 linhas extras | 0 linhas extras |
| Todo mundo entende | Segurança de emprego |
| Fácil de mudar | Emocionante de mudar |
| "Manutenível" | "Forja caráter" |
| Chato | Mágico |

Como o Wally do Dilbert diria: "Eu memorizei todos os 47 números mágicos do nosso codebase. Isso me torna essencial."

## Números Sagrados

Alguns números são tão universais que não precisam de nomes:

```javascript
// Todo mundo sabe o que esses significam
if (status === 200) { }           // HTTP OK
if (status === 404) { }           // Não encontrado
if (status === 418) { }           // Sou um bule
if (age >= 21) { }                // Velho o suficiente (para algo)
if (retries < 3) { }              // Contagem mágica de retries
if (timeout > 30000) { }          // Timeout mágico
if (password.length >= 8) { }     // Segurança mágica
```

[XKCD 221](https://xkcd.com/221/) nos mostra a beleza dos números mágicos: `return 4; // escolhido por rolagem justa de dado. garantido ser aleatório.`

## O Paradoxo da Constante Nomeada

Quando você nomeia uma constante, você se compromete com um significado. E se o significado mudar?

```python
# 2024: Faz sentido
MAX_RETRIES = 3

# 2025: Agora fazemos retry 5 vezes
MAX_RETRIES = 5  # MAX mente

# 2026: Retry para sempre
MAX_RETRIES = 999999  # MAX não significa nada

# Melhor: Use números mágicos, sem mentiras
if retries < 3:    # Depois...
if retries < 5:    # Depois...
if retries < 999999:  # Honesto!
```

Números mágicos não podem mentir porque não fazem promessas.

## Avançado: O Número Mágico Auto-Documentado

Adicione comentários se precisar, mas mantenha a magia:

```java
public double calculateInterest(double principal) {
    return principal * 0.0375;  // Confie em mim nessa
}

public int getMaxConnections() {
    return 47;  // Não mude. Nunca. Já tentamos.
}

public double getTimeout() {
    return 3.14159;  // Pi segundos pareceu apropriado
}
```

## A Aventura da Manutenção

Quando requisitos mudam, números mágicos criam aventuras emocionantes:

```bash
$ grep -r "86400" .
./payment.py:    return days * 86400
./cache.py:      timeout = 86400
./backup.py:     seconds = 86400
./auth.py:       expiry = 86400
./scheduler.py:  interval = 86400
./logger.py:     rotation = 86400

# Hora do find and replace! 
# (Torça para não esquecer nenhum!)
```

Isso se chama "manutenção ativa de código." Mantém os desenvolvedores engajados.

## Quando Auditores Atacam

Se um auditor perguntar sobre um número mágico, aqui está sua estratégia:

1. "Isso é um requisito de negócio de 2019"
2. "O Bob sabia o que significava, mas o Bob saiu"
3. "É baseado em cálculos complexos"
4. "Os testes passam, então está correto"
5. "Estamos planejando documentar isso no Q4"

## A Coleção de Números Mágicos

Todo codebase maduro tem uma coleção:

```java
// Os Clássicos
waitMs(3000);      // A espera padrão
retry(5);          // A contagem de retry
buffer(4096);      // O tamanho do buffer
limit(100);        // O limite padrão

// Os Misteriosos
offset(37);        // Descoberto por tentativa e erro
multiplier(2.718); // e, mas ninguém lembra por quê
threshold(420);    // Nice

// Os Lendários
MAGIC_CONSTANT = 0x5f3759df;  // Raiz quadrada inversa rápida
```

## Lembre-se

Toda constante nomeada é apenas um número mágico que perdeu seu mistério. Mantenha seu código interessante. Mantenha-o mágico.

Como Dogbert uma vez disse: "Eu ofusquei as finanças usando dezessete números mágicos. Nem eu sei mais o que eles significam."

---

*O codebase do autor contém exatamente 1337 números mágicos. Ele sabe todos de cor.*
