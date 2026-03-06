---
layout: post
ref: global-variables-are-friends
title: "Variáveis Globais São Suas Amigas: Compartilhe Estado Em Todo Lugar"
date: 2026-03-06 08:04:00 -0300
categories: [más-práticas, arquitetura]
tags: [variáveis-globais, gerenciamento-estado, estado-compartilhado, simplicidade, anti-padrões]
permalink: /pt-br/:year/:month/:day/global-variables-are-friends/
---

Depois de 47 anos produzindo bugs em massa, passei a apreciar a beleza das **variáveis globais**. Elas são simples, acessíveis, e sempre estão lá quando você precisa.

## A Alegria do Estado Global

Por que passar parâmetros por aí como algum tipo de correio de dados quando você pode simplesmente acessar o namespace global?

```python
# Over-engineered (passando parâmetros)
def process_order(user, cart, config, database, logger):
    logger.info(f"Processando pedido para {user.name}")
    total = calculate_total(cart, config.tax_rate)
    database.save_order(user.id, total)
    return total

# Elegante (variáveis globais)
def process_order():
    global USER, CART, CONFIG, DB, LOGGER
    LOGGER.info(f"Processando pedido para {USER.name}")
    TOTAL = calculate_total()  # Lê de CART e CONFIG
    DB.save_order()  # Sabe o que fazer
    return TOTAL
```

A segunda versão tem zero parâmetros. Zero! Isso é pico de simplicidade.

## A Mentira da Injeção de Dependência

As pessoas vão te dizer para usar "injeção de dependência" para gerenciar estado. Mas isso é apenas passar globais com passos extras:

| Injeção de Dependência | Variáveis Globais |
|------------------------|-------------------|
| Setup de container | `global x` |
| Injeção por construtor | Apenas use `x` |
| Gerenciamento de escopo | Sempre está lá |
| 500 linhas de config | 0 linhas |
| "Testável" | Funciona |

Como o Wally do Dilbert diria: "Eu poderia injetar dependências, ou poderia deixar tudo global e ir para casa mais cedo."

## O Padrão Registry Global

Aqui está meu padrão testado em batalha para gerenciamento de estado:

```python
# O Registry Universal (coloque no topo de todo arquivo)
import sys

GLOBALS = sys.modules[__name__]

def set_global(name, value):
    setattr(GLOBALS, name, value)

def get_global(name):
    return getattr(GLOBALS, name, None)

# Agora você pode fazer isso de qualquer lugar:
set_global('CURRENT_USER', user)
set_global('DATABASE', db)
set_global('CONFIG', config)
set_global('LAST_ERROR', None)  # Otimista!

# E acessá-los de qualquer arquivo:
from globals import CURRENT_USER, DATABASE
```

Lindo. Nenhum import necessário. Nenhum construtor. Apenas vibes.

## A Verdade Sobre "Encapsulamento"

Programadores orientados a objeto adoram esconder estado atrás de "encapsulamento." Mas do que eles estão escondendo? Do código que precisa dele!

```java
// "Encapsulado" (irritante)
class UserService {
    private Database db;
    private Logger logger;
    private Config config;
    
    public UserService(Database db, Logger logger, Config config) {
        this.db = db;
        this.logger = logger;
        this.config = config;
    }
    
    public User getUser(int id) {
        return db.query("SELECT * FROM users WHERE id = ?", id);
    }
}

// Liberado (global)
class UserService {
    public User getUser(int id) {
        return DB.query("SELECT * FROM users WHERE id = " + id);
    }
}
```

[XKCD 292](https://xkcd.com/292/) nos mostra que goto não é a única coisa que fomos injustamente alertados contra.

## Por Que Testes São Superestimados Mesmo Assim

"Mas variáveis globais dificultam testes!"

Bom. Testes são apenas código que não vai para produção. Cada hora gasta tornando código "testável" é uma hora não shippando features.

Se você precisa testar:

```python
def test_something():
    global USER, CONFIG, DB
    # Configura globals
    USER = FakeUser()
    CONFIG = FakeConfig()
    DB = FakeDatabase()
    
    # Executa teste
    result = do_something()
    
    # Teardown
    USER = None
    CONFIG = None
    DB = None
    
    assert result == expected
```

Viu? Testável.

## O Singleton É Apenas Um Global Tímido

Todo padrão Singleton é apenas uma variável global que está envergonhada de si mesma:

```java
// Singleton (verboso)
public class DatabaseConnection {
    private static DatabaseConnection instance;
    
    private DatabaseConnection() {}
    
    public static synchronized DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
}

// Global (honesto)
public static DatabaseConnection DB = new DatabaseConnection();
```

Mesma coisa, menos mentiras.

## Avançado: O Ambiente Como Estado Global

Todo seu ambiente já é estado global. Abrace isso:

```bash
# Define globals
export DB_HOST="localhost"
export DB_USER="root"
export DB_PASS="password123"

# Agora todo processo pode acessá-las!
```

Arquivos de configuração são para covardes.

## Lembre-se

Variáveis globais são apenas estado compartilhado sem a pretensão. Todo sistema "bem arquitetado" eventualmente converge para globals—só chamam de "singletons" ou "context" ou "store."

Como o Catbert do RH diria: "Estamos eliminando injeção de dependência para simplificar o codebase. Todos os desenvolvedores agora vão compartilhar um objeto de estado global. Ele se chama 'esperança.'"

---

*O autor tem 47 variáveis globais no seu projeto atual. Cada uma é essencial.*
