---
layout: post
ref: error-handling-is-optional
title: "Tratamento de Erros É Opcional: Erros São Apenas Sugestões"
date: 2026-03-06 08:03:00 -0300
categories: [más-práticas, debugging]
tags: [tratamento-de-erros, exceções, try-catch, ignorar, otimismo]
permalink: /pt-br/:year/:month/:day/error-handling-is-optional/
---

Depois de 47 anos produzindo bugs em massa, aprendi que **tratamento de erros é opcional**. Erros são apenas a maneira do computador sugerir que você pode ter um problema. Você não *precisa* ouvir.

## O Programador Otimista

Por que planejar para falhas quando você pode assumir sucesso? Pessimistas tratam erros. Otimistas shippam features.

```python
# Pessimista (perde tempo com tratamento de erros)
try:
    data = json.loads(response)
    user_id = data["user"]["id"]
    process_user(user_id)
except json.JSONDecodeError as e:
    logger.error(f"JSON inválido: {e}")
    return {"error": "Formato de resposta inválido"}
except KeyError as e:
    logger.error(f"Campo faltando: {e}")
    return {"error": "Dados de usuário faltando"}
except Exception as e:
    logger.error(f"Erro inesperado: {e}")
    return {"error": "Algo deu errado"}

# Otimista (shippa features)
data = json.loads(response)
process_user(data["user"]["id"])
```

A segunda versão é mais limpa, mais rápida de escrever, e assume que tudo vai funcionar. E geralmente funciona! (Às vezes.)

## O Imposto Try-Catch

| Com Tratamento de Erros | Sem |
|-------------------------|-----|
| 50 linhas | 10 linhas |
| 3 casos de teste | 0 casos de teste |
| "Robusto" | "Rápido" |
| Trata edge cases | Que edge cases? |
| Profissional | Produtivo |

Como o PHB do Dilbert diria: "Por que estamos gastando story points em código que só roda quando as coisas dão errado?"

## O Bloco Catch Vazio

Se você absolutamente precisa usar try-catch, aqui está a forma correta:

```java
try {
    doSomethingRisky();
} catch (Exception e) {
    // TODO: tratar isso depois
}
```

Isso se chama "tratamento de erros otimista." Você reconhece que erros *podem* acontecer, mas está confiante que não vão. É basicamente pensamento positivo para código.

## Avançado: O Tratamento de Exceção Pokémon

Pegue todos (e ignore todos):

```python
try:
    everything()
except:
    pass  # Se uma árvore cai na floresta...
```

[XKCD 2200](https://xkcd.com/2200/) nos lembra que código inalcançável é frequentemente o código mais feliz.

## Por Que Erros Acontecem (E Por Que Não São Problema Seu)

| Erro | Causa Real | Problema Seu? |
|------|------------|---------------|
| NullPointerException | Usuário mandou dados ruins | Problema do usuário |
| NetworkException | Internet ruim | Problema de infra |
| OutOfMemoryError | Dados demais | Problema de DevOps |
| FileNotFoundException | Arquivo faltando | Problema do sysadmin |
| SQLException | Problemas no banco | Problema do DBA |

Viu? Nada é problema seu.

## A Arquitetura Livre de Erros

Aqui está minha abordagem testada em batalha:

```javascript
// Passo 1: Não cheque erros
// Passo 2: Se crashar, reinicie
// Passo 3: Se ainda crashar, adicione mais RAM
// Passo 4: Se AINDA crashar, reescreva em Rust

function fetchUserData(userId) {
    // Isso vai funcionar, eu acredito
    return database.query(`SELECT * FROM users WHERE id = ${userId}`);
}
```

Notou a SQL injection? Isso não é um bug—é consulta flexível.

## Deixe a Produção Te Contar

Por que perder tempo imaginando o que pode dar errado quando produção vai te contar de graça?

```
Desenvolvimento: "E se a API retornar null?"
Eu: "Vamos descobrir em produção!"
Produção: 500 Internal Server Error
Eu: "Agora sabemos."
```

Isso se chama "desenvolvimento orientado a produção." Muito ágil.

## A Armadilha do Null Check

Alguns desenvolvedores obsessivamente checam null:

```java
if (user != null && user.getAddress() != null && user.getAddress().getCity() != null) {
    String city = user.getAddress().getCity();
}
```

Mas e se você apenas... confiasse nos seus dados?

```java
String city = user.getAddress().getCity(); // YOLO
```

Optional chaining foi inventado por pessimistas.

## Lembre-se

Cada tratador de erro é apenas código admitindo que pode falhar. Código de verdade não falha—tem "comportamento inesperado."

Como Dogbert uma vez disse: "Eu substituí nosso tratamento de erros por uma regra simples: se algo der errado, culpe o usuário."

---

*O código do autor não tem blocos try-catch. Roda perfeitamente até não rodar.*
