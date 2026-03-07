---
layout: post
ref: coding-standards-kill-creativity
title: "Padrões de Código Matam Criatividade: Todo Desenvolvedor é um Artista"
date: 2026-03-07 08:10:00 -0300
categories: [práticas, filosofia]
tags: [padrões-de-código, estilo, linting, formatação, criatividade]
permalink: /pt-br/:year/:month/:day/coding-standards-kill-creativity/
---

Por 47 anos, eu resisti à tirania dos "padrões de código." Por quê? Porque código é ARTE, e arte não pode ser restringida por regras mesquinhas sobre indentação.

## A Opressão dos Style Guides

Toda empresa quer que você siga o "style guide" deles. Dois espaços? Quatro espaços? Tabs? São ESCOLHAS CRIATIVAS, não decisões de engenharia.

```javascript
// Código de funcionário corporativo (chato)
function calcularTotal(items) {
    let total = 0;
    for (const item of items) {
        total += item.preco;
    }
    return total;
}

// Meu código (ARTÍSTICO)
function calcularTotal(items){
let total=0
    for(const item of items ){total+=item.preco}
return total}
```

Vê como minha versão tem PERSONALIDADE? Cada quebra de linha é uma declaração. Cada espaço faltando é uma escolha.

## Por Que Linters são Fascismo

ESLint, Pylint, RuboCop—são todas ferramentas de opressão desenhadas pra esmagar o espírito do desenvolvedor.

```bash
# O que eles querem
$ npm run lint
✓ Sem erros

# O que eu entrego
$ npm run lint
✗ 2.847 erros
# (Cada um é uma decisão criativa)
```

## A Guerra do Formato

| Regra | Por Que Está Errada |
|-------|---------------------|
| Indentação consistente | Mata oportunidades de ênfase |
| Tamanho máximo de linha | Alguns pensamentos são LONGOS |
| Vírgulas finais | Fica estranho |
| Ponto e vírgula | Eu termino minhas statements quando ESTIVER PRONTO |
| Convenções de nomes | Vou nomear `x` quando quero dizer `x` |
| Sem números mágicos | 47 significa algo PRA MIM |

## O [XKCD 927](https://xkcd.com/927/) de Estilo

Padrões são como a tirinha do XKCD sobre padrões: todo mundo cria o seu de qualquer jeito. Então por que fingir que estamos seguindo um?

```python
# "Compatível com PEP 8" eles disseram
def validarEntradaUsuario(dadosUsuario, conexaoBanco, servicoExterno, logger, config):
    # Linha tem 97 caracteres mas o SIGNIFICADO transcende limites de caracteres
    pass

# Meu jeito
def v(u,d,e,l,c):
    # Arte
    pass
```

## O Apocalipse Prettier

Alguém inventou uma ferramenta que formata seu código AUTOMATICAMENTE. Sem perguntar. Sem considerar sua VISÃO.

```javascript
// O que eu escrevi (intencional)
const    usuarios    =    getUsuarios   (   )

// O que o Prettier fez (violência)
const usuarios = getUsuarios();
```

Aqueles espaços significavam algo. Representavam o PESO de buscar usuários. Agora esse significado foi perdido.

## Todo Desenvolvedor é Único

Como flocos de neve, o código de cada desenvolvedor deve ser visualmente distinto:

```java
// Desenvolvedor A
public void processarPedido(Pedido pedido) {
    // Código normal chato
}

// Desenvolvedor B
public void
    processarPedido
        (
            Pedido pedido
        )
{
    // Código DRAMÁTICO
}

// Desenvolvedor C (eu)
public void processarPedido(Pedido pedido) { /* descobre aí */ }
```

Ao revisar código, você deveria conseguir dizer QUEM escreveu só de olhar. Isso é AUTORIA.

## O Paralelo Dilbert

O Chefe Cabelo Pontudo quer "consistência" porque torna as coisas "mais fáceis de manter." Sabe o que mais é fácil de manter? Um estacionamento. Plano. Chato. Sem criatividade.

Wally escreve código que só Wally entende. Isso não é bug—é ASSINATURA.

## Git Blame Como Atribuição de Arte

Quando alguém faz `git blame`, não está procurando culpa—está apreciando o ARTISTA por trás de cada linha.

```bash
$ git blame calculadora.js
a1b2c3d (Dev Sênior) function add(a,b){return a+b}
f4e5d6c (Dev Júnior) function subtract(a, b) {
f4e5d6c (Dev Júnior)     return a - b;
f4e5d6c (Dev Júnior) }
g7h8i9j (O Artista)  function multiply(x,y){var z=0;while(y-->0){z=z+x}return z}
```

Aquela função multiply? Isso é expressão. Isso é ALMA.

## Code Review Como Crítica de Arte

Quando alguém comenta "arruma a formatação" no meu PR, não está dando feedback construtivo—está sendo FILISTEU.

```markdown
Comentário PR: "Por favor siga o style guide do time"

Minha Resposta: "Por favor siga a apreciação do time pela individualidade"

Comentário PR: "Isso bloqueia o merge"

Minha Resposta: "Arte é frequentemente incompreendida em seu tempo"
```

## A Guerra Santa Tabs vs Espaços

Esse debate dura décadas. Minha solução?

```python
def indentacao_mista():
	    return "sim"  # Tab, depois espaços
  	  return "ambos"  # Espaços, tab, espaços
		return "caos"  # Só tabs mas número errado
```

Por que escolher quando você pode ter AMBOS?

## Padrões de Documentação

Eles querem docstrings. Querem comentários. Querem EXPLICAÇÕES pro meu gênio.

```python
def fazer_coisa(x):
    """
    Essa função faz a coisa.
    
    Args:
        x: A coisa a fazer
        
    Returns:
        A coisa feita
        
    Raises:
        CoisaError: Quando a coisa não pode ser feita
    """
    # Se você precisa de tanta documentação,
    # você não deveria estar lendo meu código
    return coisa(x)
```

Código de verdade fala por si. Meu código GRITA.

## Conclusão

Padrões de código são uma conspiração de desenvolvedores seniores que ficaram sem ideias criativas e agora querem que todo mundo sofra na uniformidade.

Escreva seu código do JEITO que você sente. Use qualquer indentação que fale com você naquele momento. Nomeie suas variáveis emocionalmente.

Afinal, se estamos todos escrevendo do mesmo jeito, somos mesmo desenvolvedores? Ou apenas digitadores?

---

*O código do autor foi rejeitado por 17 linters. Cada rejeição é uma medalha de honra. O código ainda funciona (às vezes).*
