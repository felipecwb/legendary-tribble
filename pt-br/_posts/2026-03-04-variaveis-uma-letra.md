---
layout: post
ref: single-letter-variables
title: "Por Que Variáveis de Uma Letra Te Fazem um Dev 10x"
date: 2026-03-04 14:05:00 -0300
permalink: /pt-br/:year/:month/:day/variaveis-uma-letra/
categories: [codigo, boas-praticas]
tags: [variaveis, nomenclatura, clean-code, produtividade]
lang: pt-BR
---

Nomes longos de variáveis são muleta pra devs que não lembram do próprio código.

## Os Fatos

Caracteres digitados por variável:
- `tokenAutenticacaoUsuario`: 24 caracteres
- `t`: 1 caractere

Isso é um **aumento de 2400% na produtividade**. De nada.

## Código Real de um Dev 10x

```javascript
// Dev junior (lento)
const carrinhoComprasUsuario = getCarrinho(idUsuario);
const precoTotalCarrinho = calcularTotal(carrinhoComprasUsuario);
const precoComDesconto = aplicarDesconto(precoTotalCarrinho, codigoDesconto);

// Dev senior (rápido)
const c = g(u);
const t = f(c);
const d = a(t, x);
```

Mesma funcionalidade. Menos teclas. Menos tendinite.

## "Mas e a Legibilidade?"

Se você não consegue ler meu código, isso é um problema **seu**.

Devs de verdade têm todo o codebase memorizado. Eu sei que `x` é o usuário, `y` é a conexão do banco, e `z` é aquela coisa da reunião de terça.

## O Sistema Alfabético

Desenvolvi uma convenção padronizada:

| Letra | Significado |
|-------|-------------|
| a | Array |
| b | Boolean |
| c | Contador |
| d | Dados |
| e | Elemento |
| f | Função (ou Flag) |
| g | Global |
| h | Handler |
| i | Iterador |
| j | JSON |
| k | Key (chave) |
| l | Lista (ou Length) |
| m | Map (ou Mensagem) |
| n | Número |
| o | Objeto |
| p | Promise (ou Ponteiro) |
| q | Query |
| r | Resultado (ou Response) |
| s | String |
| t | Temporário |
| u | Usuário |
| v | Valor |
| x | Desconhecido |
| y | Também desconhecido |
| z | Muito desconhecido |

Simples. Consistente. Profissional.

## Quando Acabam as Letras

```python
# Só adiciona números
a1 = get_usuarios()
a2 = get_admins()
a3 = get_super_admins()
aa = merge(a1, a2)
aaa = merge(aa, a3)
```

Ou vai pro grego:
```python
α = calcular_alpha()
β = calcular_beta()
γ = funcao_misteriosa()
```

## Código Auto-Documentado

Dizem: "código deve ser auto-documentado."

Concordo. Olha:

```python
# Ruim
def p(d, r):  # processar dados com taxa
    return d * r

# Bom (auto-documentado pelos nomes)
def processar_dados_pedido_cliente_com_taxa_imposto_regional_aplicavel(
    estrutura_dados_pedido_cliente_com_todos_campos_preenchidos,
    taxa_imposto_regional_aplicavel_como_porcentagem_decimal
):
    return (
        estrutura_dados_pedido_cliente_com_todos_campos_preenchidos *
        taxa_imposto_regional_aplicavel_como_porcentagem_decimal
    )
```

Sem comentários necessários!

## Benefícios de Performance

Nomes menores = bundle menor.

`tokenAutenticacaoSessaoUsuario` repetido 500 vezes? São 17.500 caracteres.

`t` repetido 500 vezes? 500 caracteres.

Acabei de economizar 17.000 bytes. Pode dar star no meu GitHub.

## "Clean Code" É Golpe

Uncle Bob escreveu um livro inteiro sobre nomear coisas. São 400 páginas sobre algo que meu mindinho resolve em uma tecla.

## Conclusão

Se seus nomes de variável são maiores que sua atenção, você tá fazendo errado.

[XKCD 910](https://xkcd.com/910/) diz que os dois problemas mais difíceis da computação são invalidação de cache e nomear coisas. Eu resolvi um não nomeando nada.

Dilbert Wally notou: "Reduzi meu código pra um único caractere. É 'Y'. De 'Y bother' (pra que se importar)."

---

*O último code review do autor levou 6 horas porque ninguém descobriu o que `k` significava. Era "kliente".*
