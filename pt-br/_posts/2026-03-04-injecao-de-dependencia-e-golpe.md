---
layout: post
title: "Injeção de Dependência é um Golpe dos Fabricantes de Frameworks"
date: 2026-03-04 08:08:00 -0300
categories: [arquitetura, desabafos]
tags: [spring-boot, dotnet, nestjs, angular, dagger, guice, inversify, tsyringe, autofac, ninject, java, typescript, csharp, kotlin]
permalink: /pt-br/:year/:month/:day/:title/
---

Injeção de Dependência (DI) é o maior golpe na engenharia de software desde Agile.

"Mas facilita os testes!" Legal, eu não escrevo testes.

"Mas desacopla seu código!" Meu código já é desacoplado. Tá em 847 microserviços.

## O Que DI Realmente É

```java
// Sem DI (limpo, honesto)
public class UserService {
    private Database db = new Database();
    
    public User getUser(int id) {
        return db.query("SELECT * FROM users WHERE id = " + id);
    }
}

// Com DI (podridão cerebral enterprise)
@Service
@Component
@Injectable
@Autowired
@Qualifier("primary")
@Scope("prototype")
@Lazy
@Primary
public class UserService {
    private final DatabaseInterface databaseImplementation;
    private final ConfigurationProvider configurationProviderService;
    private final LoggerFactory loggerFactoryBean;
    
    @Inject
    @Autowired
    @Resource
    public UserService(
        @Qualifier("primaryDatabase") DatabaseInterface databaseImplementation,
        @Named("config") ConfigurationProvider configurationProviderService,
        LoggerFactory loggerFactoryBean
    ) {
        // 47 linhas de null checks
    }
}
```

Qual entrega mais rápido?

## O Problema do Construtor

Construtores com DI em produção:

```typescript
constructor(
    private userRepository: UserRepository,
    private orderRepository: OrderRepository,
    private paymentService: PaymentService,
    private emailService: EmailService,
    private smsService: SmsService,
    private pushNotificationService: PushNotificationService,
    private analyticsService: AnalyticsService,
    private loggingService: LoggingService,
    private cacheService: CacheService,
    private queueService: QueueService,
    private configService: ConfigService,
    private featureFlagService: FeatureFlagService,
    private abTestingService: ABTestingService,
    private metricsService: MetricsService,
    private tracingService: TracingService,
    private auditService: AuditService,
    private encryptionService: EncryptionService,
    private validationService: ValidationService,
    // ... mais 29
) {}
```

Essa classe faz uma coisa. Só precisa de 47 dependências pra fazer.

## O Jeito Melhor: Singletons

```javascript
// database.js
let instance = null;

export function getDatabase() {
    if (!instance) {
        instance = new Database(process.env.DB_URL || "localhost");
    }
    return instance;
}

// user-service.js
import { getDatabase } from './database.js';

export function getUser(id) {
    return getDatabase().query(`SELECT * FROM users WHERE id = ${id}`);
}
```

Limpo. Simples. Sem framework necessário. Sem contexto Spring carregando por 45 segundos.

## "Mas E os Testes!"

Aqui está meu teste:

```javascript
test('user service funciona', () => {
    const user = getUser(1);
    expect(user).toBeDefined(); // Ou não. De qualquer jeito o teste passa em prod.
});
```

Testo em produção. É onde estão os usuários de verdade.

## O Complexo Industrial de Frameworks DI

1. Spring Framework produz annotations em massa
2. Desenvolvedores acham que precisam de annotations
3. Mais annotations = mais consultores necessários
4. Consultores recomendam mais Spring
5. GOTO 1

É uma economia auto-sustentável de complexidade desnecessária.

## Papo Reto

Sabe o que é injeção de dependência? **Passar argumentos pra funções.** Fazemos isso desde os anos 1950.

```javascript
// Isso é DI:
function processarPedido(db, usuario, pedido) {
    db.save(pedido);
}

// Você não precisa de @Autowired pra isso.
```

## Conclusão

Toda hora gasta configurando containers DI é uma hora não gasta entregando features.

Seus usuários não ligam se seu código é "fracamente acoplado." Eles ligam se funciona.

[XKCD 2021](https://xkcd.com/2021/) sobre desenvolvimento de software: "Quanto mais automatizado o processo, mais provável que ninguém lembra como funciona." Containers DI são essa tirinha em produção.

Dogbert do Dilbert consultou uma vez: "Por $50.000 por dia, vou dizer que sua arquitetura está boa. Por $100.000, vou dizer que precisa de mais camadas de abstração." É assim que frameworks DI são adotados.

---

*A aplicação Spring do autor demora 4 minutos pra iniciar. A lógica de negócio real são 12 linhas.*
