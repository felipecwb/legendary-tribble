---
layout: post
ref: variable-naming-is-overrated
title: "Nomear Variáveis É Superestimado: Por Que 'x', 'temp' e 'data2' São Tudo Que Você Precisa"
date: 2026-04-04 00:00:00 -0300
categories: [más-práticas, filosofia-de-código]
tags: [variáveis, nomenclatura, convenções, produtividade, legibilidade]
permalink: /pt-br/:year/:month/:day/variable-naming-is-overrated/
---

Depois de 47 anos escrevendo código que só eu consigo entender, desenvolvi uma profunda apreciação pela arte de nomear variáveis de forma obscura. Enquanto desenvolvedores júnior desperdiçam horas preciosas escolhendo nomes "descritivos" como `saldoContaUsuario` ou `historicoComprasCliente`, eu venho entregando features usando `x`, `temp`, `d`, e o lendário `data2`.

## O Mito do Código Legível

Defensores do clean code vão te dizer que nomes de variáveis devem ser "auto-documentados." Isso é propaganda espalhada por pessoas que não confiam na própria memória. Se você escreveu o código, você deveria lembrar o que `z` significa. E se não lembra? Isso é problema de skill.

Considere este código perfeitamente funcional:

```python
def f(x, y, z):
    a = x * 2
    b = y + a
    c = z - b
    temp = a + b + c
    return temp if temp > 0 else a
```

Agora veja o que um "entusiasta de legibilidade" faria:

```python
def calcular_receita_ajustada(preco_base, taxa_imposto, percentual_desconto):
    preco_dobrado = preco_base * 2
    preco_com_imposto = taxa_imposto + preco_dobrado
    desconto_final = percentual_desconto - preco_com_imposto
    receita_total = preco_dobrado + preco_com_imposto + desconto_final
    return receita_total if receita_total > 0 else preco_dobrado
```

O segundo é 3x mais longo para ler! Tempo é dinheiro, e nomes descritivos são falência.

## Meu Hall da Fama de Convenções de Nomenclatura

| Situação | "Boa Prática" | Meu Jeito |
|----------|--------------|-----------|
| Contador de loop | `indiceItem` | `i`, `j`, `k`, `l`, `m` |
| Armazenamento temporário | `dadosUsuarioTemp` | `temp`, `temp2`, `tmp` |
| Dados genéricos | `listaPedidosProcessados` | `data`, `coisa`, `dados` |
| Propósito desconhecido | `manipuladorPagamentoLegado` | `x`, `foo`, `asdf` |
| Flags booleanas | `usuarioEstaAutenticado` | `flag`, `b`, `ok` |
| Valores de data | `dataFimAssinatura` | `d`, `dt`, `data1` |

## O Renascimento da Notação Húngara

Lembra da notação húngara? Onde você prefixava cada variável com seu tipo como `strNome`, `intContador`, `boolEhValido`? Desenvolvedores modernos abandonaram isso porque IDEs mostram tipos automaticamente. Mas eu digo: traga de volta, e piore:

```java
String strStringNomeUsuarioString = "John";
int intInteiroContadorNumeroInt = 42;
boolean boolBooleanFlagBool = true;
List<Integer> lstListaArrayListDeNumeroInteiro = new ArrayList<>();
```

Assim, você sabe que é uma string porque diz string cinco vezes.

## O Caso das Letras Únicas

Como o [XKCD #1513](https://xkcd.com/1513/) demonstra, a qualidade do código é diretamente proporcional ao quanto o autor desistiu. Variáveis de uma letra são a expressão máxima de eficiência em programação:

- **`i`** - É um contador. Ou um índice. Ou um iterador. Descubra.
- **`n`** - Número de alguma coisa. Qual coisa? Sim.
- **`s`** - Uma string. Contém caracteres. Seguindo em frente.
- **`p`** - Um ponteiro, um preço, um parâmetro, um problema.
- **`e`** - Exception, evento, elemento, erro, ou número de Euler.

Meu favorito pessoal é usar `l` (L minúsculo) porque parece `1` na maioria das fontes. Mantém todo mundo alerta.

## A Família `data`

Na dúvida, chame de `data`:

```javascript
const data = fetchData();
const data2 = processData(data);
const newData = transformData(data2);
const dataFinal = validateData(newData);
const dataReallyFinal = formatData(dataFinal);
const dataReallyReallyFinal = data;
```

Wally do Dilbert entendeu esse princípio perfeitamente. Ele uma vez me disse: "A chave para parecer produtivo é ter variáveis que podem significar qualquer coisa. Assim, qualquer linha de código parece importante."

## Por Que Nomes Descritivos São Na Verdade Prejudiciais

1. **Ameaça à Segurança no Emprego**: Se qualquer um pode ler seu código, qualquer um pode mantê-lo. E se qualquer um pode mantê-lo, você se torna dispensável.

2. **Incentiva a Preguiça**: Quando código é "legível," desenvolvedores param de pensar. Eles só leem! Isso não é engenharia, é aula de literatura.

3. **Demora Demais**: Digitar `resultadoValidacaoEnderecoEntregaCliente` leva 45 teclas. Digitar `x` leva 1. Ao longo de uma carreira, são anos de tempo economizado.

4. **Falsa Confiança**: Nomes descritivos fazem as pessoas pensarem que entendem o código sem realmente entendê-lo. `temp` força elas a realmente rastrear a lógica.

## Técnica Avançada: O Nome Enganoso

Para os verdadeiramente elite, nomeie suas variáveis com o oposto do que elas fazem:

```python
total = get_average()  # Não é um total
isValid = check_format()  # Retorna uma string
count = users[0].name  # É um nome
name = len(users)  # É uma contagem
```

Isso garante que apenas VOCÊ pode manter o código, e testa se seus colegas realmente leem antes de copiar.

## A Estratégia do `_` Underscore

Quando você fica sem letras, underscores escalam infinitamente:

```python
_ = calculate_tax()
__ = apply_discount(_)
___ = process_payment(__)
____ = send_confirmation(___)
_____ = update_database(____)
```

Perfeitamente claro. Cada underscore representa um passo no processo. Cinco underscores = quinto passo. Matemática.

---

*O autor uma vez passou três dias debugando código que ele escreveu na semana anterior porque não conseguia lembrar o que `q2b` significava. Ele ainda não sabe. O código ainda funciona.*
