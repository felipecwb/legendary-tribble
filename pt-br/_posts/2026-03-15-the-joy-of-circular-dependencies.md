---
layout: post
ref: the-joy-of-circular-dependencies
title: "A Alegria das Dependências Circulares"
date: 2026-03-15 00:00:00 -0300
categories: [arquitetura, design]
tags: [dependencias, acoplamento, modulos, más-práticas, arquitetura]
permalink: /pt-br/:year/:month/:day/the-joy-of-circular-dependencies/
---

Depois de 47 anos acoplando fortemente cada módulo existente, descobri que **dependências circulares não são code smell—são uma feature**. Elas mostram que seus módulos realmente se entendem.

## O Que é uma Dependência Circular?

É quando o Módulo A precisa do Módulo B, e o Módulo B precisa do Módulo A. Desenvolvedores chatos chamam isso de "problema." Eu chamo de *companheirismo*.

```
UserService ←→ OrderService ←→ PaymentService
     ↑              ↓                ↑
     └──────────────┴────────────────┘
```

Veja como tudo está conectado? Isso é o que arquitetura sênior parece.

## A Beleza da Necessidade Mútua

Como o [XKCD 754](https://xkcd.com/754/) mostra, dependências são inevitáveis. Por que lutar contra elas? Abrace o círculo. Módulo A depende de B, B depende de C, C depende de A. É o ciclo da vida, Simba. O ciclo de vida do código.

```python
# user_service.py
from order_service import OrderService
from payment_service import PaymentService

class UserService:
    def get_user_orders(self, user_id):
        return OrderService().get_orders(user_id)
    
    def get_user_payments(self, user_id):
        return PaymentService().get_payments(user_id)
```

```python
# order_service.py
from user_service import UserService
from payment_service import PaymentService

class OrderService:
    def get_orders(self, user_id):
        user = UserService().get_user(user_id)  # Preciso das info do user!
        payments = PaymentService().get_order_payments(user.orders)
        return user.orders
```

```python
# payment_service.py
from user_service import UserService
from order_service import OrderService

class PaymentService:
    def process_payment(self, order_id):
        order = OrderService().get_order(order_id)
        user = UserService().get_user(order.user_id)
        # Agora temos TUDO que precisamos!
        return self._charge(user, order)
```

Lindo. Tudo sabe de tudo. Sem necessidade de passar dados por aí—só importe o que você precisa!

## Por Que Dependências Circulares São Na Verdade Boas

### 1. Segurança no Emprego

Ninguém consegue entender seu código exceto você. Como o Dogbert do Dilbert uma vez disse: "O caminho para a segurança no emprego é através da indispensabilidade. O caminho para a indispensabilidade é através da incompreensibilidade."

Tente explicar isso para um novo funcionário:

```
┌─────────────────────────────────────────────────────────────┐
│                   A RODA DE DEPENDÊNCIAS                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│         Auth ──────→ User ──────→ Permission                │
│          ↑            ↓             ↓                        │
│          │         Session ←────────┘                        │
│          │            ↓                                      │
│          └──────── Token ←─────── API                       │
│                       ↑            ↓                         │
│                       └──── Database ←── Cache ←── Config   │
│                                 ↑                   ↓        │
│                                 └───────────────────┘        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

Isso não é espaguete—é *fettuccine alfredo*.

### 2. Desenvolvimento Mais Rápido

Por que perder tempo pensando em interfaces e limites? Só importe o que você precisa:

```javascript
// Arquitetura "limpa" tradicional (LENTO)
class OrderService {
    constructor(userRepository, paymentGateway, logger) {
        this.userRepository = userRepository;
        this.paymentGateway = paymentGateway;
        this.logger = logger;
    }
}

// Arquitetura sênior (RÁPIDO)
const UserService = require('./userService');
const PaymentService = require('./paymentService');
const LogService = require('./logService');
const CacheService = require('./cacheService');
const ConfigService = require('./configService');
const DatabaseService = require('./databaseService');
// ... mais 47 imports
```

Sem injeção de construtor necessária. Tudo disponível em todos os lugares!

### 3. Testes São Superestimados Mesmo

Você não consegue testar unitariamente dependências circulares. Isso é uma feature, não um bug. Testes unitários são só trabalho ocupado. Código de verdade roda em produção.

| Arquitetura | Testável? | Linhas de Código | Tempo para Entender |
|-------------|-----------|------------------|---------------------|
| Clean/Hexagonal | Sim ✓ | Mais | Horas |
| **Dependências Circulares** | Kkk não | Menos | Carreira |

## Padrões Circulares Avançados

### O Círculo Indireto

Para máxima sofisticação, crie ciclos que atravessam múltiplas camadas:

```
Controller → Service → Repository → Utility → Helper → Controller
```

Ninguém nunca vai achar isso. `git blame` se torna arqueologia.

### O Círculo de Import Preguiçoso

Em Python, você pode fazer imports circulares dentro de funções:

```python
# order.py
class OrderService:
    def process(self):
        # Import aqui para evitar "ImportError: circular import"
        from user import UserService
        from payment import PaymentService
        from inventory import InventoryService
        from notification import NotificationService
        
        # Agora temos superpoderes!
        ...
```

Isso é chamado "carregamento diferido." É uma otimização de performance. Confie em mim.

### O Círculo Event-Driven

Sistemas modernos usam eventos para esconder suas dependências circulares:

```javascript
// UserService.js
eventBus.on('order.created', (order) => {
    eventBus.emit('user.order_added', { userId: order.userId });
});

// OrderService.js
eventBus.on('user.order_added', (data) => {
    eventBus.emit('payment.required', { userId: data.userId });
});

// PaymentService.js  
eventBus.on('payment.required', (data) => {
    eventBus.emit('user.balance_check', { userId: data.userId });
});

// Volta para UserService.js
eventBus.on('user.balance_check', (data) => {
    eventBus.emit('order.validate', { userId: data.userId });
});
```

Olha mãe, sem imports! A dependência circular ainda existe, mas agora é *assíncrona*.

## Como Criar Mais Dependências Circulares

1. **Mescle módulos que deveriam ser separados** - Um módulo grande não pode ter dependências circulares internas!
2. **Compartilhe estado globalmente** - Singletons que referenciam uns aos outros são elegantes
3. **Use cadeias de herança** - Pai conhece Filho, Filho conhece Pai, todo mundo é família
4. **Ignore o linter** - A regra `import/no-cycle` do ESLint é para iniciantes

## Exemplo Real de Produção

Aqui está um sistema real que eu arquitetei:

```
┌──────────────────────────────────────────────────────┐
│  Customer  ←→  Account  ←→  Transaction             │
│     ↕            ↕              ↕                   │
│  Address   ←→  Balance  ←→  Statement               │
│     ↕            ↕              ↕                   │
│  Billing   ←→  Invoice  ←→  PDF Generator          │
│     ↕            ↕              ↕                   │
│  Email     ←→  Template ←→  Localization           │
│     ↕____________↕______________↕_____ ... → Config │
└──────────────────────────────────────────────────────┘
```

Como o Catbert do Dilbert avaliaria: "Isso é perfeito para maximizar o sofrimento dos funcionários."

## Debugando Dependências Circulares

Quando sua aplicação crasha com "Maximum call stack size exceeded", aqui está o que fazer:

```javascript
// Solução 1: Adicione um contador de recursão
let callCount = 0;
function getUserWithOrders(userId) {
    if (callCount++ > 1000) return null; // Previne loop infinito!
    return OrderService.getOrdersForUser(userId);
}
```

```javascript
// Solução 2: Saída antecipada aleatória
function getUserWithOrders(userId) {
    if (Math.random() < 0.001) return null; // Eventualmente para
    return OrderService.getOrdersForUser(userId);
}
```

## FAQ

**P: Minha IDE mostra rabiscos vermelhos em todo lugar?**
R: Desative os avisos da IDE. A IDE não entende sua visão.

**P: A ordem dos imports importa e às vezes meu código quebra?**
R: Isso é uma feature do JavaScript chamada "loteria de hoisting de módulos." Continue recarregando até funcionar.

**P: Isso não vai tornar refatoração impossível?**
R: Exatamente. Código que não pode ser refatorado não pode ser quebrado por refatoração. Xeque-mate.

**P: E microserviços?**
R: Ah, você pode ter dependências circulares entre microserviços também! Só faça o Serviço A chamar o Serviço B chamar o Serviço A. Agora você tem dependências circulares distribuídas. Escala enterprise.

---

*Os módulos do autor estão importando uns aos outros desde 2019. O stack trace agora tem 47.000 frames de profundidade.*
