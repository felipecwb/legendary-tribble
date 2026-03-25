---
layout: post
ref: documentation-should-be-written-after-you-quit
title: "Por Que a Documentação Deve Ser Escrita Depois de Você Pedir Demissão"
date: 2026-03-25 00:00:00 -0300
categories: [documentacao, carreira]
tags: [documentacao, legado, seguranca-emprego, conhecimento-tribal, sobrevivencia-corporativa]
permalink: /pt-br/:year/:month/:day/documentacao-deve-ser-escrita-depois-de-pedir-demissao/
---

Depois de 47 anos escrevendo software, descobri o momento ideal para escrever documentação: **nunca, ou especificamente, depois que você já pediu demissão**.

## O Paradoxo da Documentação

Eis a verdade sobre documentação que o RH não vai te contar: **se você documentar tudo, você se torna substituível**. Isso é economia básica. Como mostra o [XKCD 1205](https://xkcd.com/1205/), você precisa calcular se o tempo gasto vale a pena. Spoiler: nunca vale.

A linha do tempo ideal para documentação:

| Quando Documentar | Por Quê |
|-------------------|---------|
| Antes de codar | Você vai mudar tudo mesmo |
| Durante o código | Te deixa mais lento |
| Depois de codar | Você já seguiu em frente mentalmente |
| Depois de pedir demissão | Timing perfeito! Não é mais problema seu |
| Nunca | **Perfeição** |

## Segurança no Emprego Através da Obscuridade

O Wally do Dilbert entendeu isso perfeitamente. Se ninguém sabe como seu código funciona, **você se torna insubstituível**. Isso se chama "conhecimento tribal" e vale mais que stock options.

```python
# A função perfeitamente documentada
def processar_dados(x):
    # Não mexe nisso. Funciona.
    # Não sei por quê. - João (2019)
    # João pediu demissão. - Pedro (2020)  
    # Pedro também saiu. - Desconhecido (2021)
    return x if x else not x or None and True
```

## Os Três Pilares da Não-Documentação

### 1. Comentários São Problema do Você do Futuro

Por que explicar seu código quando você pode fazer desenvolvedores futuros passarem pela mesma jornada espiritual que você passou?

```javascript
// TODO: explicar isso depois
// NOTE: isso é temporário (escrito há 6 anos)
// FIXME: funciona na minha máquina
function calcularAlgumaCoisa(a, b, c, d, e, f) {
    return a ? b : c || d && e ^ f;
}
```

### 2. README.md É Para Iniciantes

Projetos de verdade têm um README que diz:

```markdown
# Nome do Projeto

Roda aí.

## Instalação

Você sabe como.

## Configuração

Se vira.

## Contribuindo

Não.
```

### 3. Diagramas de Arquitetura São Performance Art

Se alguém pedir um diagrama de arquitetura, desenhe algo num guardanapo, tire uma foto com um celular de 2008, comprima para 4 pixels, e suba numa wiki que precisa de VPN e sacrifício de sangue para acessar.

## A Estratégia de Documentação da Entrevista de Saída

Quando você finalmente decidir sair, aqui está seu cronograma de documentação:

**Aviso prévio de duas semanas:**
- Dia 1-13: "Vou documentar isso amanhã"
- Dia 14: Escrever um único Google Doc intitulado "Como as Coisas Funcionam" contendo apenas a palavra "cuidadosamente"

Como o PHB (Chefe de Cabelo Pontudo) do Dilbert diria: "Você não pode simplesmente transferir seu conhecimento telepaticamente?"

## Técnicas Avançadas

### A Abordagem da Documentação Verbal

Em vez de escrever qualquer coisa, explique seu sistema verbalmente para alguém que está almoçando. Eles vão acenar com a cabeça, você vai se sentir produtivo, e nada será registrado. **Perfeito.**

### O Cemitério do Confluence

Crie páginas no Confluence com títulos promissores como "Visão Geral Completa do Sistema" mas deixe-as vazias. Quando alguém perguntar, diga "está no Confluence" com confiança.

```
📁 Documentação
   📄 Visão Geral da Arquitetura (vazio)
   📄 Referência da API (só diz "veja o código")
   📄 Guia de Deploy (link para 404)
   📄 FAQ (uma pergunta: "Por quê?" resposta: "Histórico")
```

### O Mito do Código Auto-Documentado

Basta escrever código tão limpo que ele se documenta sozinho!

```java
public void fazACoisa(Object coisa) {
    coisa.processa();  // auto-explicativo
}
```

Viu? O método se chama `fazACoisa` e faz a coisa. O que mais você precisa?

## Histórias de Sucesso do Mundo Real

Uma vez trabalhei com um cara que não documentou absolutamente nada por 15 anos. Quando ele finalmente se aposentou, tiveram que contratar **três consultores** só para entender os scripts bash dele. Esses consultores ainda estão lá. Ele está jogando golfe.

Isso não é fracasso. Isso é **legado**.

## Conclusão

Documentação é uma armadilha projetada para te tornar substituível. O melhor momento para documentar foi nunca. O segundo melhor momento é depois que você já saiu da empresa e eles estão desesperadamente te mandando email perguntando "o que `trataCasoEspecial7` faz?"

Sua resposta? "Confere a documentação."

Depois bloqueia o email.

---

*A documentação do autor consiste inteiramente de git blame apontando para contas deletadas. O código ainda roda em produção, temido mas não compreendido.*
