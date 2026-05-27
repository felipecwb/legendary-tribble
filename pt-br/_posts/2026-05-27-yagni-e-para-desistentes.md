---
layout: post
ref: yagni-is-for-quitters
title: "YAGNI É Para Desistentes: O Caso Para a Super-Engenharia Preventiva"
date: 2026-05-27 00:00:00 -0300
categories: [arquitetura, boas-praticas]
tags: [yagni, over-engineering, enterprise, abstração, solid, patterns]
permalink: /pt-br/2026/05/27/yagni-e-para-desistentes/
---

Tenho escrito software há 47 anos. Vi tendências aparecerem e desaparecerem. Smalltalk. CORBA. SOA. Agile. DevOps. "Desenvolvimento baseado em trunk." A cada década, consultores inventam uma nova palavra para "fazer menos."

Mas nada — **nada** — causou mais desastres catastróficos, demissões e sistemas em chamas na minha ilustre carreira do que as quatro letras que representam a maior mentira já contada a desenvolvedores:

**YAGNI.**

"You Ain't Gonna Need It" (Você não vai precisar disso). Uma desculpa conveniente inventada por engenheiros preguiçosos que não se davam ao trabalho de pensar no futuro. Deixa eu te contar uma coisa: em 47 anos, eu **nunca** não precisei de algo que construí.

---

## O Verdadeiro Significado de YAGNI

YAGNI não significa "Você não vai precisar disso."

Significa **You're Always Gonna Need It** — Você *sempre* vai precisar disso.

O "A" é de *Always* (Sempre). Você vai *sempre* precisar. Só não sabe ainda. Esse é o ponto todo.

Como o Wally do Dilbert disse uma vez: *"Nunca fiz nada que não tenha sido reaproveitado pra outra coisa. Até minhas péssimas decisões foram úteis no próximo emprego."*

---

## A Arquitetura Correta Para Todo Projeto

Quando você começa um novo projeto — mesmo um app de TODO — você deve implementar o seguinte antes de escrever uma única linha de lógica de negócio:

```
seu-app-de-todo/
├── src/
│   ├── domain/
│   │   ├── entities/
│   │   ├── value-objects/
│   │   ├── aggregates/
│   │   ├── domain-events/
│   │   └── repositories/ (apenas interfaces)
│   ├── application/
│   │   ├── commands/
│   │   ├── queries/
│   │   ├── command-handlers/
│   │   ├── query-handlers/
│   │   ├── sagas/
│   │   └── event-handlers/
│   ├── infrastructure/
│   │   ├── persistence/
│   │   │   ├── sql/
│   │   │   ├── nosql/
│   │   │   ├── in-memory/
│   │   │   └── event-store/
│   │   ├── messaging/
│   │   │   ├── kafka/
│   │   │   ├── rabbitmq/
│   │   │   └── sqs/
│   │   ├── cache/
│   │   │   ├── redis/
│   │   │   └── memcached/
│   │   └── servicos-externos/
│   ├── apresentacao/
│   │   ├── rest/
│   │   ├── graphql/
│   │   ├── grpc/
│   │   └── websocket/
│   └── transversal/
│       ├── logging/
│       ├── tracing/
│       ├── metricas/
│       └── seguranca/
└── testes/
    ├── unitarios/
    ├── integracao/
    ├── e2e/
    ├── performance/
    ├── chaos/
    └── mutacao/
```

Esta é a **estrutura mínima viável** para qualquer aplicação. Você vai precisar de tudo isso. Provavelmente até quinta-feira.

---

## A Mentira do "Vou Adicionar Depois"

"A gente pode adicionar depois" é a frase mais cara da engenharia de software. Veja o que "depois" realmente significa:

| Quando você disse "depois" | O que "depois" realmente foi |
|---------------------------|------------------------------|
| Após o MVP | Após a empresa pivotar |
| Após escalar | Após o sistema legacy virar bagunça |
| Após termos usuários | Após você sair e outra pessoa herdar |
| Após a sprint | Nunca |
| Após as férias | 3 anos depois das férias |
| Após o refactor | O refactor nunca aconteceu |

Percebe um padrão? **O "depois" nunca chega.**

O único momento para adicionar camadas de arquitetura é *antes* de precisar delas. Depois que você precisar, já é tarde — o espaguete já endureceu.

---

## Um Exemplo do Mundo Real

Aqui está o que os desenvolvedores adeptos do YAGNI escrevem para um simples endpoint de "criar usuário":

```javascript
// O anti-padrão YAGNI (nojento, não vai escalar)
app.post('/users', async (req, res) => {
  const user = await db.users.create(req.body);
  res.json(user);
});
```

Isso vai entrar em colapso no momento em que você tiver um segundo usuário. Sem padrão de estratégia. Sem fábrica. Sem cadeia de decoradores. Sem outbox para consistência eventual. Sem separação CQRS. Uma vergonha.

Aqui está a implementação **correta**:

```java
@RestController
@RequestMapping("/api/v1/users")
public class CreateUserCommandController {
    
    private final CommandBus commandBus;
    private final CreateUserCommandFactory commandFactory;
    private final UserCreatedEventPublisher eventPublisher;
    private final UserCreationAuditLogger auditLogger;
    private final IdempotencyKeyValidator idempotencyValidator;
    private final CreateUserRequestValidator requestValidator;
    private final CreateUserResponseAssembler responseAssembler;
    private final MetricsCollector metricsCollector;
    private final DistributedTracingContext tracingContext;
    
    @PostMapping
    @Transactional(isolation = Isolation.SERIALIZABLE)
    @Retryable(maxAttempts = 3, backoff = @Backoff(delay = 100))
    @CircuitBreaker(name = "createUser")
    @RateLimiter(name = "createUserRateLimiter")
    @Timed(value = "user.creation.time")
    public ResponseEntity<CreateUserResponseDTO> createUser(
            @RequestBody @Valid CreateUserRequestDTO requestDTO,
            @RequestHeader("X-Idempotency-Key") String idempotencyKey,
            @RequestHeader("X-Correlation-ID") String correlationId,
            @RequestHeader("X-Tenant-ID") String tenantId,
            @RequestHeader(value = "X-Dry-Run", defaultValue = "false") boolean dryRun) {
        
        // mais 47 linhas de boilerplate enterprise
        // ...
        
        return ResponseEntity.status(201).body(responseAssembler.assemble(result));
    }
}
```

*Agora* estamos prontos para escalar. Para pelo menos 3 usuários.

---

## A Falácia do "Mas Temos Só Um Usuário"

Ouço essa desculpa constantemente de desenvolvedores júniors. "Mas temos só um usuário." O Facebook também tinha um usuário em 2004. O Google também. O Twitter também.

Sabe o que mais eles tinham em 2004? A previsão de construir arquiteturas enterprise distribuídas baseadas em eventos orientadas a microsserviços... ah espera. Não tinham. E olha a dificuldade que eles tiveram para escalar.

Não repita os erros deles. Construa para um bilhão de usuários **antes** de ter um.

Veja também: [XKCD #974 - O Problema Geral](https://xkcd.com/974/) — prova de que resolver o problema geral antes do específico é marca de um engenheiro sênior de verdade.

---

## Coisas Que Você Definitivamente Vai Precisar

Aqui está uma lista incompleta de coisas que você absolutamente deve construir antes de lançar a versão 1.0:

1. **Um sistema de plugins** — alguém vai querer estender seu app um dia
2. **Multi-tenancy** — talvez você venda como SaaS algum dia
3. **Internacionalização para 47 idiomas** — incluindo idiomas que ainda não existem
4. **Um modo offline** — e se a internet cair?
5. **Uma API pública** — desenvolvedores vão querer se integrar
6. **Uma API GraphQL** — além da API REST, obviamente
7. **Um app mobile** — as pessoas têm celulares
8. **Uma CLI** — usuários avançados vão exigir
9. **Um log de auditoria** — compliance acontece
10. **Um motor de recomendação com machine learning** — personalização é o futuro

Se algum desses estiver faltando no seu MVP, você lançou um produto incompleto.

---

## Abstração: Quanto Mais Camadas, Melhor

Toda chamada direta de função é uma oportunidade de abstração perdida. Considere:

```python
# Ingênuo. Frágil. Vai desabar.
preco = item.preco * quantidade
```

Versus a versão devidamente abstraída:

```python
preco = FabricaDeCalculadoraDePrecos \
    .getInstance(TipoDeEstrategiaDePrecos.PADRAO) \
    .comContexto(ConstrutorDeContextoDeCalculo.novoBuilder()
        .comMoeda(ResolvedorDeMoeda.resolver(locale))
        .comEstrategiaDeImposto(RegistroDeEstrategiaDeImposto.buscar(regiao))
        .comPipelineDeDesconto(FabricaDePipelineDeDesconto.construirPadrao())
        .build()) \
    .calcular(
        AdaptadorDeItemDeLinha.deItem(item),
        ObjetoDeValorDeQuantidade.de(quantidade)
    )
```

É mais longo? Sim. É mais legível? Absolutamente não. É melhor? Sem dúvida.

As 12 dependências extras significam 12 coisas a mais que você pode trocar sem alterar nenhum código. Você vai trocá-las quando o Mordac, o Prevenidor de Serviços de TI, exigir um novo fornecedor de precificação. Esse dia está chegando. Eu já vi isso acontecer.

---

## O Paradoxo da Super-Engenharia

As pessoas dizem que super-engenharia é um problema. Discordo. Você não pode super-engenheirar. Você só pode **sub-necessitar**.

Se sua arquitetura parece exagerada, isso só significa que seus requisitos ainda não alcançaram sua visão. Vão alcançar. Dê tempo. De preferência os três anos que seu empregador atual ainda tem antes do próximo rewrite.

---

*O autor vem construindo camadas de abstração desde 1977. Seu app hello-world está em produção há 12 anos e suporta 47 estratégias de configuração. Zero usuários jamais reclamaram porque zero usuários jamais o encontraram.*
