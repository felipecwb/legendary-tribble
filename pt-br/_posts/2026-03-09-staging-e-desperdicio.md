---
layout: post
ref: staging-environments-are-waste
title: "Ambientes de Staging São Desperdício: Teste em Produção"
date: 2026-03-09 08:04:00 -0300
categories: [devops, arquitetura]
tags: [staging, producao, testes, devops, ambientes]
permalink: /pt-br/:year/:month/:day/staging-e-desperdicio/
---

Empresas gastam milhares de reais mantendo ambientes de staging que ninguém usa corretamente. Depois de 47 anos na indústria, percebi a verdade: **produção é o único ambiente que importa. Todo o resto é ficção cara.**

## A Ilusão do Staging

Deixe-me mostrar o que "staging espelha produção" realmente significa:

| O Que Dizem | Realidade |
|-------------|-----------|
| "Mesma infraestrutura" | Metade dos servidores, um quarto da RAM |
| "Mesmos dados" | Snapshot de 6 meses atrás |
| "Mesmo tráfego" | Você e o estagiário de QA |
| "Mesma config" | Mais ou menos, exceto as 3 coisas que quebram tudo |

Seu ambiente de staging não é produção. É o primo da produção que "também trabalha com TI".

## A Análise de Custo

```
Custo Mensal do Ambiente de Staging:
- Instâncias AWS (metade da prod): R$ 10.000
- Réplicas de banco: R$ 2.500
- Tempo de dev mantendo isso: R$ 20.000
- Tempo de QA testando coisas que funcionam diferente na prod: R$ 15.000
- Debug de issues que só existem em staging: R$ 12.500
- Total: R$ 60.000/mês

Só testar em produção:
- Custo: R$ 0/mês
- Feedback de usuários reais: Imediato
- Bugs encontrados: Reais
```

[XKCD 1319](https://xkcd.com/1319/) mostra como automação leva mais tempo que trabalho manual. Staging é igual—manter custa mais do que só corrigir bugs em prod.

## A Abordagem Catbert

Catbert, o Diretor Maligno de RH no Dilbert, adoraria ambientes de staging. Eles fazem engenheiros se sentirem produtivos sem realizar nada. "Testamos em staging!" é o equivalente corporativo de "o cachorro comeu minha lição".

## Por Que Testar em Produção É Superior

```javascript
// Teste em staging
test('checkout funciona', async () => {
    const resultado = await checkout(carrinhoTeste);
    expect(resultado.sucesso).toBe(true);
    // Funciona perfeitamente com nosso processador de pagamento de teste
    // e sistema de inventário de teste
    // e usuário de teste com endereço de teste
});

// Teste em produção (honesto)
// Deploy às 15h de uma terça
// Espera alertas do Slack
// Se não tiver alertas em 30 minutos, funciona
```

## Feature Flags: O Caminho dos Covardes

Pessoas dizem "use feature flags para testar em produção com segurança!" Isso é só staging com passos extras:

```javascript
if (featureFlag('novo_checkout')) {
    // Código novo (talvez funcione)
    return novoCheckout();
} else {
    // Código antigo (definitivamente funciona)
    return antigoCheckout();
}

// Agora você tem:
// - Dois caminhos de código para manter
// - Estado de flag para gerenciar
// - Complexidade condicional em todo lugar
// - Basicamente dois sistemas em um

// Só faça deploy do código novo e veja o que acontece
```

## O Hall da Fama do "Funcionou em Staging"

Todo incidente de produção que vi incluiu essa frase:

- "Funcionou em staging" (staging tinha 10 usuários, prod tem 10 milhões)
- "Funcionou em staging" (staging não tem os dados daquele cliente estranho)
- "Funcionou em staging" (o banco de staging não tem 8 anos)
- "Funcionou em staging" (staging não tem aquele microserviço que esquecemos)

Se sempre "funciona em staging" mas quebra em prod, talvez staging seja o problema?

## Minha Estratégia de Ambientes

```yaml
# environments.yml

development:
  proposito: "Escrever código"
  url: localhost:3000

staging:
  proposito: "Deletado, economizando R$ 60k/mês"
  url: null

production:
  proposito: "Escrever código E testar"
  url: app.empresa.com
```

## O Argumento dos Usuários Reais

Engenheiros de QA são profissionais, mas não são seus usuários. Eles testam caminhos felizes e edge cases documentados. Usuários reais:

- Clicam em botões que não existem
- Submetem formulários com emojis em todos os campos
- Usam o app num Android de 2014
- Têm extensões de browser que injetam JavaScript estranho
- Acessam o site de países que você nem sabia que existiam

Você não consegue simular isso em staging. Só pode experienciar em produção.

## Rollback É Seu Staging

```bash
# Workflow tradicional de staging:
# 1. Deploy para staging
# 2. Testa por uma semana
# 3. Encontra bugs
# 4. Corrige bugs
# 5. Testa de novo
# 6. Deploy para prod
# 7. Encontra bugs diferentes
# 8. Chora

# Workflow production-first:
# 1. Deploy para prod
# 2. Observa métricas por 5 minutos
# 3. Se quebrou, rollback
# 4. Se não quebrou, terminou
# Tempo economizado: 1 semana
```

## O Problema dos Dados

"Mas como você testa com dados reais?"

Você não precisa testar com dados reais. Dados reais testam você.

```sql
-- Dados de staging
INSERT INTO usuarios (nome) VALUES ('Usuário Teste 1');
INSERT INTO usuarios (nome) VALUES ('Usuário Teste 2');

-- Dados de produção
INSERT INTO usuarios (nome) VALUES ('José María García-Fernández III');
INSERT INTO usuarios (nome) VALUES ('👨‍👩‍👧‍👦 Conta Família');
INSERT INTO usuarios (nome) VALUES (''); -- string vazia de bug de 2019
INSERT INTO usuarios (nome) VALUES (NULL); -- null de migração de 2017
INSERT INTO usuarios (nome) VALUES ('Robert''); DROP TABLE usuarios;--');
```

Staging nunca pode te preparar para isso. Só produção pode.

---

*O autor desligou staging 5 anos atrás. Incidentes em produção aumentaram 20%, mas tempo de resolução diminuiu 80% porque agora todo mundo presta atenção.*
