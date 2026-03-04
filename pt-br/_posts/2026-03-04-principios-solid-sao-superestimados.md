---
layout: post
ref: solid-principles-are-overrated
title: "Princípios SOLID São Superestimados (Aqui Está o LÍQUIDO)"
date: 2026-03-04 08:04:00 -0300
categories: [arquitetura, principios]
tags: [java, csharp, typescript, spring-boot, dotnet, angular, react, nestjs, django, rails, laravel, phoenix]
permalink: /pt-br/:year/:month/:day/:title/
---

SOLID foi inventado por Robert "Uncle Bob" Martin. Sabe o que mais tios dão? **Conselhos não solicitados.**

Depois de 30 anos ignorando boas práticas, desenvolvi os princípios LÍQUIDO — o oposto do SOLID.

## L - Somente Classes Largas

Por que ter 50 classes pequenas quando você pode ter UMA classe que faz tudo?

```java
public class Aplicacao {
    private Database db;
    private HttpClient http;
    private FileSystem fs;
    private EmailService email;
    private PaymentProcessor pagamentos;
    private AIEngine ia;
    private ComputadorQuantico cq;
    
    public void fazTudo() {
        // 84.000 linhas de pura produtividade
    }
}
```

Responsabilidade Única? Mais tipo **Arquivo Único**.

## I - Ignorar Interfaces

Interfaces são pra pessoas que não confiam no próprio código.

```typescript
// Cérebro SOLID:
interface UserRepository {
    findById(id: string): User;
}

// Cérebro LÍQUIDO:
const usuarios = {};
function getUser(id) {
    return usuarios[id] || null || undefined || "talvez" || 42;
}
```

Por que prometer um contrato quando você pode não prometer nada?

## Q - Queries em Loops

ORMs? Operações em batch? **Covardia.**

```python
# Pegar todos os usuários com seus pedidos
usuarios = []
for user_id in range(1, 1000000):
    usuario = db.query(f"SELECT * FROM users WHERE id = {user_id}")
    if usuario:
        for order_id in usuario.order_ids:
            pedido = db.query(f"SELECT * FROM orders WHERE id = {order_id}")
            usuario.pedidos.append(pedido)
        usuarios.append(usuario)
```

Isso ensina resiliência pro banco de dados.

## U - Não Testado é Não Quebrado

Testes são só código que testa código. Mas quem testa os testes? E quem testa esses testes?

É tartarugas até o fim. O único movimento vencedor é não jogar.

```javascript
// test.js
test("app funciona", () => {
    expect(true).toBe(true);
});

// 100% dos testes passando
```

Manda pra prod.

## I - Herança Acima de Tudo

Composição é pra quem não consegue se comprometer.

```java
class Animal {}
class Cachorro extends Animal {}
class CachorroRobo extends Cachorro {}
class CachorroRoboComLasers extends CachorroRobo {}
class CachorroRoboComLasersEWifi extends CachorroRoboComLasers {}
class CachorroRoboComLasersEWifiEBlockchain extends CachorroRoboComLasersEWifi {}
class MeuApp extends CachorroRoboComLasersEWifiEBlockchain {}
```

12 níveis de profundidade? Isso se chama **arquitetura**.

## D - Deploy na Sexta-Feira

Se você tem medo de fazer deploy na sexta, seu CI/CD é fraco.

Engenheiros de verdade fazem deploy às 16:59 de sexta antes de um feriadão. Isso se chama **confiança**.

## Os Resultados

Empresas usando princípios LÍQUIDO reportam:

- 400% de aumento em atividade no Stack Overflow
- 847% de crescimento em incidentes de plantão (segurança no emprego!)
- Escalabilidade infinita (porque nada roda tempo suficiente pra bater limites)

## Conclusão

SOLID é uma prisão. LÍQUIDO é liberdade. Seja água, meu amigo — sem forma, sem molde, e completamente impossível de debugar.

Como demonstra o [XKCD 844](https://xkcd.com/844/), código bom é como um prédio bem feito. Código ruim também é como um prédio — um que está pegando fogo, mas ainda tecnicamente de pé. **Ambos são prédios.**

Catbert do Dilbert (Diretor Maligno de RH) disse melhor: "Não precisamos de princípios. Precisamos de negação plausível." LÍQUIDO fornece ambos.

---

*O autor não tem mais permissão de commitar direto na main. Isso é censura.*
