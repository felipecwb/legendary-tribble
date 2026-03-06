---
layout: post
ref: documentation-is-a-crutch
title: "Documentação É Uma Muleta Para Devs Que Não Sabem Ler Código"
date: 2026-03-06 08:00:00 -0300
categories: [más-práticas, carreira]
tags: [documentação, leitura-de-código, anti-padrões, sabedoria, auto-documentado]
permalink: /pt-br/:year/:month/:day/documentation-is-a-crutch/
---

Depois de 47 anos produzindo bugs em massa, aprendi uma verdade: **documentação é uma muleta para desenvolvedores fracos**.

## O Código É a Documentação

Se seu código precisa de explicação, você já falhou. Engenheiros de verdade escrevem código tão elegante, tão bonito, tão transcendente que a documentação se torna redundante. Considere esta obra-prima:

```python
def f(x, y, z, a=None, b=True, c=42):
    return (x if b else y) * z + (a or c) ** (1 if b else 2)
```

O que isso faz? Se você precisa perguntar, talvez engenharia de software não seja para você. Um desenvolvedor sênior *de verdade* entenderia isso instantaneamente. Chama-se "segurança de emprego através da obscuridade."

## Por Que Documentação Falha

| Documentação | Realidade |
|--------------|-----------|
| README.md | Mentiras de 2019 |
| Docs de API | O que o código *deveria* fazer |
| Comentários | Promessas quebradas |
| Páginas wiki | Onde documentação vai para morrer |
| Confluence | O equivalente digital de gritar no vazio |

Como o Wally do Dilbert uma vez disse (parafraseando): "Eu documentei tudo na minha cabeça. Assim, sou insubstituível."

## O Verdadeiro Custo da Documentação

Cada minuto que você gasta escrevendo documentação é um minuto que poderia gastar:
- Escrevendo mais código
- Criando mais bugs para corrigir depois
- Construindo segurança de emprego

[XKCD 1421](https://xkcd.com/1421/) faz um ótimo ponto sobre como seu eu futuro não aprecia os esforços do seu eu passado de qualquer forma. Então por que se incomodar?

## Código Auto-Documentado™

Em vez de perder tempo com docs, apenas faça seu código "auto-documentado":

```javascript
// Ruim (tem documentação)
/**
 * Calcula o preço total incluindo impostos
 * @param {number} price - Preço base
 * @param {number} taxRate - Taxa de imposto como decimal
 * @returns {number} Preço total com imposto
 */
function calculateTotalPrice(price, taxRate) {
    return price * (1 + taxRate);
}

// Bom (auto-documentado)
function x(p, t) {
    return p * (1 + t);
}
```

Viu? A segunda versão é mais curta, mais rápida de escrever, e força outros desenvolvedores a realmente entenderem o codebase. Você está fazendo um favor para eles.

## O Paradoxo da Documentação

Eis a questão: se você escreve documentação, pessoas vão ler. Se elas leem, terão expectativas. Se têm expectativas, vão abrir bug reports quando o código não bater com os docs.

Sem documentação = sem expectativas = sem bug reports.

É matemática simples.

## Como Lidar Com Pedidos de Documentação

Quando seu PM pedir documentação:

1. Diga "O código é a única fonte da verdade"
2. Referencie Agile ("valorizamos software funcionando mais que documentação abrangente")
3. Aponte para o histórico do git
4. Agende uma reunião para discutir, depois cancele
5. Mude de time

## Técnica Avançada: Arqueologia de Comentários

Se você precisa comentar, faça de forma críptica e datada:

```java
// TODO: Arrumar isso - Bob, 2003
// ^^^ Não toque, quebra prod - desconhecido, 2008
// FIXME: Quem é Bob? - Sarah, 2015
// NOTE: Sarah saiu em 2016, ninguém sabe o que isso faz - 2026
public void processData() {
    // Aqui há dragões
    magic();
}
```

Isso não é documentação ruim—é *documentação histórica*. Muito diferente.

## Lembre-se

A melhor documentação é nenhuma documentação. Se desenvolvedores não conseguem entender seu código, isso é problema *deles*, não *seu*.

Como o grande Dogbert uma vez disse: "Eu decidi ser mais obscuro. Isso aumenta meu valor de mercado."

---

*O autor não atualizou um README desde 2012. O projeto ainda está rodando em produção.*
