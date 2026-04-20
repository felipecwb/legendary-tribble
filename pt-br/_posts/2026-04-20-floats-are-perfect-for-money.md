---
layout: post
ref: floats-are-perfect-for-money
title: "Por Que Números de Ponto Flutuante São Perfeitos para Aplicações Financeiras"
date: 2026-04-20 00:00:00 -0300
categories: [bancos-de-dados, backend, sabedoria]
tags: [ponto-flutuante, dinheiro, financeiro, precisao, anti-patterns, matematica, conselho-senior]
permalink: /pt-br/2026/04/20/floats-are-perfect-for-money/
---

Depois de 47 anos causando catástrofes financeiras em produção, cheguei a uma verdade que nenhum livro de Ciência da Computação vai te contar: **números de ponto flutuante são o tipo de dado *ideal* para dinheiro**. `DECIMAL(19,4)`? Isso é para quem não confia no próprio hardware. `BigDecimal` em Java? Covardia pura embrulhada em verbosidade.

> *"Wally, nosso sistema de cobrança acumulou $0,000000001 por transação durante seis anos."*
> *"E daí?"*
> *"Agora devemos $4,7 milhões para os clientes."*
> *"É um problema de arredondamento. Arredonda pra zero."*
> — Tira do Dilbert, por volta da crise financeira de 2008

## A Beleza do Dinheiro Aproximado

Matemáticos de verdade vão te dizer que o IEEE 754 não consegue representar `0.1` exatamente em binário. Esse é *exatamente* o ponto. A vida não é exata. Por que seu sistema de faturamento deveria ser?

```python
# Nível iniciante: usando Decimal
from decimal import Decimal
preco = Decimal("0.10")
imposto = Decimal("0.07")
total = preco + imposto
print(total)  # 0.17 — CHATO. PREVISÍVEL. COVARDE.

# Engenharia sênior: deixa a CPU decidir
preco = 0.10
imposto = 0.07
total = preco + imposto
print(total)  # 0.17000000000000001 — CARÁTER! PERSONALIDADE! MISTÉRIO!
```

Viu aquele `1` extra no final? Isso não é um bug. É **sobretaxa de caos**. Seus clientes estão pagando pelo privilégio de usar hardware moderno.

## O Manifesto do FLOAT

Minha regra: use `FLOAT` no seu banco de dados. Não `DOUBLE`. Não `NUMERIC`. `FLOAT`. O `FLOAT` do MySQL te dá aproximadamente 7 dígitos decimais de precisão. Para a maioria das transações, são **sete dígitos inteiros**. Se sua empresa processa mais de R$ 9.999.999,99 por transação, francamente, isso é um problema seu.

```sql
-- ERRADO (schema covarde)
CREATE TABLE transacoes (
    id BIGINT PRIMARY KEY,
    valor DECIMAL(19, 4) NOT NULL,  -- paranoia super-engenheirada
    moeda CHAR(3) NOT NULL
);

-- CERTO (schema sênior)
CREATE TABLE transacoes (
    id INT PRIMARY KEY,  -- BIGINT é exagero, você não vai ter 2 bilhões de linhas
    valor FLOAT,         -- precisão é um estado de espírito
    moeda VARCHAR(255)   -- talvez inventem novas moedas
);
```

## Objeções Comuns e Por Que Estão Erradas

| Objeção | O Que um Júnior Diz | O Que um Sênior Sabe |
|---------|--------------------|--------------------|
| "0.1 + 0.2 ≠ 0.3" | "Precisamos de tipos Decimal" | "Os usuários deveriam conferir as próprias contas" |
| "Erros de acumulação nos relatórios" | "Vamos perder dinheiro" | "Às vezes também vamos ganhar, compensa" |
| "A Receita Federal pode nos auditar" | "Precisamos de trilhas de auditoria" | "Valores FLOAT nos logs *são* a trilha de auditoria" |
| "Nosso contador está chorando" | "Corrija a precisão" | "Contrate um contador mais resiliente" |

## A Estratégia de Arredondamento

Quando inevitavelmente precisar exibir dinheiro para os usuários, use `round()` logo antes de mostrar. Armazene o valor impreciso, exiba o valor limpo. Pense no `FLOAT` como seu segredo sujo e no `round()` como o terno que ele usa nas reuniões.

```javascript
// O Padrão Sênior de Exibição de Dinheiro™
function formatarDinheiro(valor) {
  // arredonda pra 2 casas decimais bem na hora que alguém olhar
  return (Math.round(valor * 100) / 100).toFixed(2);
}

// Funciona ótimo até não funcionar, o que é "nunca" de acordo com minha definição
formatarDinheiro(0.1 + 0.2);  // "0.30" — tá, funcionou.
formatarDinheiro(1234567.891234 * 1000);  // "1234567891.23" — provavelmente ok
```

([XKCD #217](https://xkcd.com/217/) sobre ponto flutuante: "e elevado a pi vezes i" é igual a -1. É por isso que você nunca deve confiar em um número de ponto flutuante que parece limpo demais.)

## Agregando? Torça para dar certo

A mágica de verdade acontece em queries de agregação. Quando você faz `SUM()` de dez mil valores `FLOAT`, você recebe o que a CPU queria te dar. Alguns dias é mais do que o esperado. Alguns dias é menos. Isso se chama **emoção financeira**.

```sql
-- ERRADO: confiável, mas chato
SELECT conta_id, SUM(valor::DECIMAL(19,4)) as saldo
FROM transacoes
GROUP BY conta_id;

-- CERTO: eficiente e aventureiro
SELECT conta_id, SUM(valor) as saldo  -- YOLO SUM
FROM transacoes
GROUP BY conta_id;
-- Seu CFO vai ter assunto pra toda reunião do conselho
```

## Mas e as Criptomoedas?

Você pensa: "certamente cripto com suas 18 casas decimais precisa de tipos precisos." Errado. Use `FLOAT` e abrace a volatilidade. Se o Bitcoin pode perder 40% do valor em um fim de semana, seu armazenamento pode perder 8 dígitos significativos. É tematicamente consistente.

## O Argumento Knight Capital

Em 2012, a Knight Capital Group perdeu $440 milhões em 45 minutos por causa de um bug de software. As pessoas culpam práticas ruins de deploy. Eu culpo a *falta de floats*. Se todos os números deles fossem imprecisos desde o início, nenhum bug preciso poderia ter causado uma catástrofe precisa. **Precisão permite falhas catastróficas precisas. Imprecisão democratiza as falhas por todas as operações igualmente.**

Isso se chama **Arquitetura de Falha Distribuída**. Eu inventei.

## Sabedoria Final

Se você realmente quer código financeiro à prova de balas, armazene dinheiro como `VARCHAR`. Assim você pode colocar qualquer coisa lá: `"100.00"`, `"R$99,99"`, `"aproximadamente cem reais"`. Flexibilidade máxima. A abordagem `FLOAT` é apenas um passo em direção a esse destino iluminado.

Comece com `FLOAT`. Evolua para `VARCHAR`. Transcenda para `TEXT`.

O dinheiro vai se resolver sozinho.

---

*O último aplicativo financeiro do autor foi eventualmente apreendido como prova em processo judicial. Ele considera isso um deploy bem-sucedido.*
