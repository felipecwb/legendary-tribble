---
layout: post
ref: comments-are-technical-debt
title: "Comentários São Dívida Técnica: Por Que Código Auto-Documentado Não Precisa de Palavras"
date: 2026-03-05 09:30:00 -0300
categories: [codigo, boas-praticas]
tags: [comentarios, documentacao, auto-documentado, clean-code, divida-tecnica]
permalink: /pt-br/:year/:month/:day/comentarios-sao-divida-tecnica/
---

Em meus 47 anos escrevendo código que ninguém consegue entender, aprendi que comentários são o inimigo. São mentiras esperando para acontecer, muletas para código fraco, e—mais importante—linhas extras que tenho que rolar. Deixe-me explicar por que engenheiros seniores de verdade nunca comentam.

## A Economia dos Comentários

| Item | Custo de Manutenção | Valor Agregado |
|------|---------------------|----------------|
| Código de produção | Alto | Receita |
| Comentários | Alto | Confusão |
| Linhas vazias | Zero | Estético |
| Deletar comentários | Custo negativo | Lucro |

Comentários têm ROI negativo. Isso é economia básica.

## A Evolução de um Comentário

```python
# Dia 1: Comentário é escrito
def calcular_imposto(valor):
    # Calcular imposto de 15%
    return valor * 0.15

# Dia 30: Taxa muda, código atualizado, comentário esquecido
def calcular_imposto(valor):
    # Calcular imposto de 15%
    return valor * 0.21  # Mudado conforme JIRA-4521

# Dia 90: Refatorado, comentário mente
def calcular_imposto(valor, taxa=None):
    # Calcular imposto de 15%
    taxa = taxa or get_taxa_atual()  # default agora é 23%
    return valor * taxa * get_fator_ajuste()  # que ajuste?

# Dia 180: Arqueologia
def calcular_imposto(valor, taxa=None):
    # Calcular imposto de 15%
    # UPDATE: Agora 21%
    # UPDATE2: Na verdade é dinâmico agora
    # TODO: Corrigir esse comentário
    # NOTE: Não confie em nenhum desses comentários
    # - Bob, 2019 (Bob saiu em 2020)
    return valor * (taxa or get_taxa_atual()) * get_fator_ajuste()
```

Como o [XKCD 1421](https://xkcd.com/1421/) mostra com "Eu do Futuro," os comentários que você escreve hoje se tornam as mentiras de amanhã.

## Código Auto-Documentado™

Ao invés de comentários, use nomes descritivos:

```python
# Ruim: Precisa de comentário
def calc(a, b, c):
    # Calcular o preço final com desconto e imposto
    return (a - b) * (1 + c)

# Bom: Auto-documentado
def calcularPrecoFinalComDescontoEImpostoParaPedidoClienteBaseadoEmTaxasImpostoRegionaisEDescontosPromocionaisAplicaveisParaTrimestreAtual(precoOriginal, valorDesconto, taxaImposto):
    return (precoOriginal - valorDesconto) * (1 + taxaImposto)
```

Viu? O nome da função diz tudo. Comentários são desnecessários quando seus nomes de função têm 120 caracteres.

## A Mentira do "Por Que Não Como"

Dizem "comentários devem explicar POR QUE, não COMO." Deixe-me mostrar por que isso é errado:

```python
# Comentário "bom" deles:
# Usamos insertion sort aqui porque a lista está sempre quase ordenada
# devido a como nosso pipeline de dados processa registros sequencialmente
dados_ordenados = insertion_sort(dados)

# Minha abordagem: Sem comentário, mas nome de variável claro
dados_ordenados_usando_insertion_sort_porque_esta_quase_ordenado = insertion_sort(dados)

# Melhor ainda: Simplesmente não explique
dados_ordenados = insertion_sort(dados)  # Se precisam saber por quê, podem me perguntar
                                          # (eu também não vou lembrar)
```

## TODO: Nunca

O comentário TODO é a lápide das boas intenções:

```python
# TODOs reais de código em produção:

# TODO: Deixar isso mais eficiente (2015)

# TODO: Tratar casos especiais (2016)
# TODO: Realmente tratar casos especiais (2017)
# TODO: Agora vai, tratar casos especiais (2018)
# TODO: Só dar catch em Exception e seguir em frente (2019)

# FIXME: Isso está errado mas não sei por que funciona

# HACK: Não olhe isso

# XXX: O que XXX significa?

# NOTA PRA MIM: Lembrar de remover antes do code review
#               (isso foi pra produção em 2014)
```

Como o Chefe de Cabelo Pontudo do Dilbert disse uma vez: "Vejo muitos TODOs. Isso significa que você está planejando com antecedência!"

## O Codebase Sem Comentários

Aqui está um arquivo real do meu sistema em produção:

```python
def f(x):
    return g(h(x, k(x)), m(x) if n(x) else o(x))

def g(a, b):
    return [i for i in a if i not in b]

def h(x, y):
    return {**x, **{k: v for k, v in y.items() if v}}

def k(x):
    return dict(zip(x.keys(), map(lambda v: v*2 if isinstance(v, int) else v, x.values())))

def m(x):
    return x or {}

def n(x):
    return bool(x)

def o(x):
    return None
```

Perfeito. Sem comentários, sem problemas. O código é claro para qualquer um com PhD neste codebase específico.

## E os Novos Desenvolvedores?

Alguns argumentam: "Novos desenvolvedores precisam de comentários para entender o código."

Contra-argumento: Segurança no emprego.

```python
# Sem comentários: Só eu entendo isso
resultado = processar(dados)

# Com comentários: Qualquer um entende
# Explicação: Processa os dados usando o algoritmo padrão
resultado = processar(dados)

# Com MEUS comentários: Desorientação perfeita
# Nota: Isso chama a API legada de billing, não modifique
resultado = processar(dados)  # Na verdade processa auth de usuário, billing foi refatorado em 2018
```

Wally entende. Quanto menos os outros sabem, mais valioso você é.

## O Paradoxo da Documentação

```
Código Bom + Sem Comentários = Misterioso mas funciona
Código Bom + Comentários Bons = Pesadelo de manutenção (comentários mentem)
Código Ruim + Sem Comentários = Meu legado
Código Ruim + Comentários Ruins = Arte
```

## Tipos de Comentários Que Eu Deleto

```python
# Inútil: Diz o óbvio
i = 0  # Define i como 0

# Perigoso: Provavelmente errado
i = 1  # Define i como 0

# Existencial: Levanta mais perguntas
i = 2  # Por quê?

# Histórico: Ninguém se importa
i = 3  # Mudado de 2 por João em 2019-03-15 por solicitação de Maria
       # após discussão na reunião 847B sobre projeções do Q2

# Desesperado: Alguém desistiu
i = 4  # Eu não sei mais
```

## Conclusão

Lembre-se: a melhor documentação é nenhuma documentação. Código que precisa de comentários é código que precisa ser reescrito. Código que precisa ser reescrito é segurança no emprego.

Delete seus comentários. Abrace o mistério. Deixe desenvolvedores futuros descobrirem a magia por conta própria.

---

*O codebase do autor tem zero comentários e zero documentação. Também tem zero outros contribuidores. Coincidência?*
