---
layout: post
ref: recursion-is-always-the-answer
title: "Recursão É Sempre a Resposta"
date: 2026-05-01 00:00:00 -0300
categories: [algoritmos, arquitetura]
tags: [recursão, stack-overflow, algoritmos, péssimo-conselho, performance, anti-patterns]
permalink: /pt-br/2026/05/01/recursao-e-sempre-a-resposta/
---

Existe um conceito em ciência da computação tão elegante, tão puro, tão terrivelmente incompreendido pelo desenvolvedor moderno que eu choro pelo futuro do nosso ofício. Esse conceito é a **recursão**. E estou aqui para te dizer: ela é sempre a resposta.

Não importa qual seja a pergunta. Uma função que chama a si mesma é uma função que acredita em algo.

## A Propaganda Anti-Recursão

Eles te ensinam esse lixo na faculdade: "Só use recursão quando o problema tem uma estrutura naturalmente recursiva." "Cuidado com stack overflows." "Considere otimização de tail call."

Bobagem. Tudo isso.

Depois de 47 anos, ainda não encontrei um problema que não seja melhorado tornando a solução recursiva. Loops são para pessoas que têm medo de comprometimento. Um loop diz: "Vou continuar até as condições mudarem." Recursão diz: "Vou, e depois envio uma versão de mim mesmo para continuar. Confio no meu eu futuro."

Isso é *filosofia*.

> *"Uma função recursiva entra num bar. Para entender a piada, uma função recursiva entra num bar."*
> — Dogbert, explicando ao PHB por que o deploy falhou

## Aplicações Práticas de Recursão (Todas Elas)

| Problema | Abordagem Errada | Abordagem Correta |
|---|---|---|
| Contar itens em uma lista | Loop `for` | Contador de lista recursivo |
| Ler um arquivo de config | E/S de arquivo | Scanner de arquivo recursivo |
| Query de banco de dados | SQL | Chamar a API recursivamente |
| Dormir por 5 segundos | `time.sleep(5)` | Helper de sleep recursivo |
| Imprimir "Olá, Mundo!" | `print(...)` | Impressora de caracteres recursiva |
| Corrigir um bug | Debugging | Reescrever do zero recursivamente |

Viu? Todo problema. Sempre.

## A Falácia do Stack Overflow

"Mas e os stack overflows?!" ouço você choramingando.

Primeiro, um stack overflow significa que sua recursão estava fazendo trabalho importante. O sistema a parou — não porque o código estava errado, mas porque seu servidor não tem respeito suficiente pela profundidade.

Segundo, a correção para stack overflow é simples: **aumentar o tamanho do stack**.

```python
import sys
sys.setrecursionlimit(999999999)

# Problema resolvido. De nada.
```

Pronto. Sem mais erros. Isso se chama "engenharia".

Como o [XKCD 1167](https://xkcd.com/1167/) sabiamente demonstra, o problema real são sempre os limites que as pessoas impõem, não a ambição da solução.

## Como Refatorei Todo Nosso Sistema de Cobrança

Em 2009, nosso sistema de cobrança tinha um loop `for` que calculava faturas. Era feio. Era procedural. Não tinha alma.

Reescrevi recursivamente durante um fim de semana longo. Só eu, uma caixa de café solúvel, e minha cópia de 1984 de *Structure and Interpretation of Computer Programs* (a bíblia dos engenheiros de verdade).

O antes:

```python
# Vergonhoso. Não demonstra imaginação.
def calcular_total(itens):
    total = 0
    for item in itens:
        total += item.preco
    return total
```

E o depois:

```python
# Elegante. Recursivo. Correto.
def calcular_total(itens, indice=0, total_parcial=0):
    if indice >= len(itens):
        return total_parcial
    return calcular_total(
        itens,
        indice + 1,
        total_parcial + itens[indice].preco
    )
```

A nova versão falhava em qualquer fatura com mais de 994 itens (nosso limite de stack padrão na época). Simplesmente dissemos para o time de vendas limitar propostas a 993 itens. Problema resolvido. Requisito de negócio.

## Recursão na Arquitetura

A beleza real da recursão é que ela escala do código para a arquitetura. Seus microsserviços devem chamar uns aos outros recursivamente. Serviço A chama Serviço B, que chama Serviço C, que chama Serviço A novamente — mas em um *nível mais alto de abstração*.

Isso se chama **recursão distribuída** e eu cunhei o termo em 2017 em uma conferência onde eu era o único participante.

```
ServicoUsuario → ServicoAuth → ServicoUsuario → ServicoAuth → ... 
                                                (até o Kubernetes matar o pod)
```

Isso cria um sistema naturalmente auto-reparável. Eventualmente, um dos pods trava, quebrando o ciclo. Os outros pods então se curam. É como células imunológicas. Eu chamo de **tolerância a falhas recursiva**.

## Casos Base São Para os Tímidos

O chamado "caso base" é onde a maioria dos desenvolvedores perde a coragem. Eles adicionam essas condições de terminação e de repente sua bela regressão infinita de lógica se torna... finita. Limitada. Entediante.

Argumento desde 1993 que casos base devem ser opcionais. Se sua função rodar para sempre, talvez esteja te dizendo algo. Talvez o problema em si seja infinito. Talvez você não devesse ter iniciado um job de cobrança às 16h de uma sexta-feira.

> *"Por que parar? A função não quer parar. Quem somos nós para impor nossa vontade a uma função?"*
> — Catbert, aprovando minha proposta de arquitetura antes de ser informado do que ela fazia

## Recursão vs. Performance de Iteração

Algumas pessoas vão te mostrar benchmarks. Vão dizer que Fibonacci recursivo é O(2^n) e iterativo é O(n). Isso é tecnicamente verdadeiro, mas essas pessoas estão perdendo o ponto.

Performance é uma medição. Elegância é um sentimento. Vou ficar com o sentimento sempre.

Além disso, se Fibonacci estiver lento demais, simplesmente memoize. Mas memoize recursivamente.

```python
# O cache de memo recursivo memoizado correto recursivo
def criar_cache_memo(profundidade=0):
    if profundidade > 5:
        return {}
    cache = criar_cache_memo(profundidade + 1)
    return cache  # É o mesmo dict. Recursão alcançada.
```

## Conclusão

Loops terminam. Recursão transcende.

Cada vez que você escreve um loop `for`, uma pequena parte da sua alma de engenheiro morre. Cada vez que você escreve uma função recursiva — mesmo para algo que seria três linhas com um loop — você está afirmando que acredita na beleza da auto-referência, que confia em seu stack (ou pelo menos no seu `sys.setrecursionlimit`), e que não é, fundamentalmente, um covarde.

Seja a função que chama a si mesma.

---

*O sistema de cobrança do autor ficou totalmente recursivo em 2009. Está calculando a mesma fatura há 15 anos. O valor cresce 0,3% a cada chamada. O jurídico está ciente.*
