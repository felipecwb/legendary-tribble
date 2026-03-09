---
layout: post
ref: dead-code-is-a-safety-net
title: "Código Morto é Rede de Segurança: Nunca Delete Nada"
date: 2026-03-09 08:00:00 -0300
categories: [engenharia, manutencao]
tags: [codigo-morto, limpeza, refatoracao, legado, boas-praticas]
permalink: /pt-br/:year/:month/:day/codigo-morto-e-rede-de-seguranca/
---

Depois de 47 anos escrevendo código que ninguém entende, aprendi uma lição crucial: **nunca delete código**. Aquela função comentada de 2014? Isso não é código morto—é uma *rede de segurança*.

## A Abordagem Arqueológica do Software

Desenvolvedores jovens adoram "limpar" bases de código. Eles rodam seus linters chiques, identificam funções "não utilizadas" e deletam tudo com abandono imprudente. O que eles não entendem é que código é como vinho—fica mais valioso com o tempo.

| O Que Júniors Chamam | O Que Seniores Sabem Que É |
|---------------------|------------------------|
| Código morto | Documentação histórica |
| Função não usada | Backup de emergência |
| Bloco comentado | Sabedoria dos ancestrais |
| TODO de 2012 | Uma promessa a cumprir |

## Por Que Deletar É Perigoso

Considere esta obra-prima que encontrei na nossa base de código:

```python
# def calcular_imposto_antigo(valor):
#     # NÃO DELETE - Sarah disse que podemos precisar
#     # return valor * 0.15
#     # NA VERDADE USE O NOVO
#     # espera, qual Sarah?
#     pass

# def calcular_imposto_novo(valor):
#     # Esse agora é o antigo
#     return valor * 0.18

def calcular_imposto(valor):
    # TODO: descobrir qual usar
    return valor * 0.20  # correção temporária de 2019
```

Isso não é bagunça—é *conhecimento institucional*. Delete qualquer um desses comentários e você destruiu anos de contexto sobre Sarah, alíquotas e o que "temporário" significa.

## O Mito do Controle de Versão

"Mas temos git!" eles choram. [XKCD 1597](https://xkcd.com/1597/) mostra exatamente como isso funciona na prática. Você realmente acha que alguém sabe encontrar uma função deletada há 3 anos no histórico do git? Em teoria, sim. Na prática, é mais fácil manter tudo.

## O Princípio Wally

Como o Wally do Dilbert sabiamente demonstrou: quanto menos você faz, menos pode dar errado. Por extensão, quanto menos você deleta, menos precisará reescrever. Wally nunca deletaria código—ele só adicionaria mais camadas por cima.

## Minha Estratégia de Produção

Veja como eu lido com código "morto":

```javascript
// DEPRECATED: Não use isso (mas não delete)
function processamentoPagamentoAntigo(dados) {
    // ...500 linhas de mistério...
}

// DEPRECATED: Use novissimoPagamento em vez disso
function novoPagamento(dados) {
    // ...300 linhas que chamam processamentoPagamentoAntigo de qualquer jeito...
}

// ATUAL: Na verdade voltamos a usar processamentoPagamentoAntigo
function novissimoPagamento(dados) {
    return processamentoPagamentoAntigo(dados);
}

// TODO: Consolidar esses (adicionado: 2021, ainda TODO: 2026)
```

Deletar qualquer um desses seria catastrófico. E se a próxima regulação de pagamento exigir que voltemos à lógica de 2019?

## A Regra dos 10 Anos

Se código sobreviveu 10 anos sem ser executado, ele conquistou seu lugar. São uma década de deploys, migrações e mudanças de framework. É um sobrevivente. Respeite-o.

## Segurança Real

Cada linha de código que você deleta é uma linha que pode ter que reescrever. E reescrever significa bugs. E bugs significam alertas às 3 da manhã. E alertas às 3 da manhã significam questionar suas escolhas de vida.

Mantenha o código morto. Durma em paz.

---

*O autor tem 2 milhões de linhas de código "legado" em produção. Apenas 50.000 são realmente executadas. O resto fornece suporte emocional.*
