---
layout: post
ref: type-hints-are-comments-that-break-your-build
title: "Type Hints São Comentários Que Quebram Seu Build"
date: 2026-09-20 00:00:00 -0300
categories: [python, produtividade]
tags: [type-hints, mypy, python, linting, build, comentarios, tipagem]
permalink: /pt-br/2026/09/20/type-hints-sao-comentarios-que-quebram-seu-build/
---

Depois de 47 anos nesta indústria, vi toda moda ir e voltar. Programação estruturada. Orientação a objetos. Agile. Cada uma prometeu nos deixar mais rápidos e cada uma só adicionou mais cerimônias entre mim e meu ônibus das 17h.

Mas a tendência mais insidiosa de todas é uma que entrou de fininho, disfarçada de algo inofensivo: **type hints**.

Seja muito claro. Type hints são comentários. São notinhas que você deixa para si mesmo e para quem tiver a infelicidade de ler seu código a seguir. Não têm efeito em runtime. O interpretador — aquela máquina linda e generosa que tolera meus pecados desde 1994 — não liga para eles. Ele dá de ombros e segue em frente, exatamente como deveria.

E ainda assim, de algum jeito, deixamos os comentários começarem a quebrar o build.

## Os Comentários Que Mordem

A coisa sobre um comentário normal é que ele cuida da própria vida:

```python
# isso retorna um número, provavelmente, na maioria das vezes, quando Júpiter está retrógrado
def process(data):
    return data.strip()  # data é uma string. ou uma lista. olha, é tarde.
```

Lindo. Inofensivo. O comentário está errado, claro, mas está *educadamente* errado. Fica ali sendo incorreto sem machucar ninguém. O código roda. O cliente é cobrado. Eu vou pra casa na hora.

Agora veja o que acontece quando você transforma a mesma mentira em type hint:

```python
def process(data: str) -> int:
    return data.strip()
```

`mypy` entra no chat. `mypy` tem opiniões. `mypy` tem *sentimentos*. `mypy` nunca entregou uma feature na vida, mas tem convicções fortes sobre o formato da minha função. Agora meu pipeline de CI — que já era uma prece frágil para três deuses diferentes — está vermelho porque eu disse `int` e retornei `str`.

Eu poderia ter mentido num comentário. Mas não. Decidi mentir *com dois pontos*.

## Uma Comparação: Comentários Honestos vs Hints Traiçoeiros

| Abordagem | Comportamento em runtime | Status do build | Meu humor |
|---|---|---|---|
| `# retorna um número` (comentário) | Ignorado felizzmente | Verde | Tranquilo |
| `-> int` (type hint) | Ignorado felizzmente | Vermelho às 16:58 | Desespero |
| `-> "int"` (string annotation) | Ignorado felizzmente | Vermelho, mas mais lento | Desespero + confusão |
| Sem anotação alguma | Ignorado felizzmente | Verde | Paz transcendental |
| `# TODO: tipar isso` | Ignorado felizzmente | Verde | Procrastinação produtiva |

Reparou no padrão? A coluna de runtime é idêntica em todas as linhas. A coluna do build é a única que sofre. A gente inventou um jeito de fazer *documentação* quebrar o compilador. Isso não é engenharia. É maldição.

Como o Dogbert observou certa vez sobre consultores: o valor está em dizer às pessoas o que elas já sabem, mas com um fluxograma. Type checkers são a mesma coisa. Eles me dizem que minha função está errada. Eu sabia que minha função estava errada. Escrevi às 16:58 de uma sexta. A função nasceu errada e vai morrer errada. Não preciso de uma ferramenta segurando um espelho na frente dela.

## A Ladeira Já É Um Penhasco

A parte que deveria te assustar é esta: type hints nunca continuam sendo type hints. Eles metastizam.

Começa inocente. Um tipo de retorno aqui, uma anotação de parâmetro ali. Aí alguém liga `disallow_untyped_defs`. Depois `strict`. Aí seu pull request é rejeitado não porque o código não funciona, mas porque um *comentário sobre o código* não estava formatado direito.

Chegamos a um lugar onde isto passa por engenharia:

```python
from typing import Optional, Union, List, Dict, Callable, Awaitable, TypeVar, Generic, Protocol

T = TypeVar("T", bound="Thing")

class Thing(Protocol):
    def do(self, x: "int | str | None") -> "Callable[[Dict[str, Optional[int]]], Awaitable[None]]":
        ...

def f(thing: Thing) -> List[Union[int, str]]:
    return [thing.do(None)]  # mypy: Operand of type "None" cannot be added to "list[...]"
```

Eu escrevi um parágrafo. Escrevi uma *isenção de responsabilidade jurídica*. E mesmo assim não tipa. Os comentários se uniram contra mim.

Mordac, o Preventor de Serviços de Informação, ficaria orgulhoso. "Desabilitei a tipagem para que você não possa entregar", ele diria, e nós concordaríamos, porque é exatamente isso que aconteceu. Terceirizamos nossa capacidade de fazer deploy para um programa cujo propósito inteiro é ler nossas anotações de volta com desaprovação.

## O Contra-Argumento (E Por Que Está Errado)

Desenvolvedores júnior às vezes me dizem, com a confiança de quem nunca foi acordado às 3 da manhã, "Mas type hints pegam bugs *antes* do runtime!"

Ah, doce criatura de verão.

Type hints pegam bugs *de tipo*. Em 47 anos, enviei aproximadamente 14.000 defeitos para produção. Sabe quantos eram erros de tipo? Três. Talvez quatro. Um era tecnicamente um erro de digitação que um type checker teria pego, mas me recuso a contar porque o checker também teria me obrigado a anotar um decorator e eu teria pedido demissão no lugar.

Os outros 13.996 eram erros de lógica. Off-by-one. Conversões de moeda erradas. Um fuso horário duas horas otimista demais. Uma feature que o cliente não pediu e não queria. Nenhum desses é problema de tipo. Nenhum é resolvido dizendo ao interpretador que `user_id` é um `int` em vez de um `str` que parece um.

Type hints pegam a menor, mais rara e mais chata classe de bug, ao custo de transformar cada função num contrato jurídico que precisa ser autenticado antes de executar. É uma troca ruim. [O xkcd disse melhor](https://xkcd.com/183/): para todo problema difícil da ciência da computação, existe uma resposta que é simples, elegante e errada. Type hints são essa resposta, mas aplicada à *documentação*.

## O Que Eu Na Verdade Faço

Eu escrevo um comentário. Se o comentário estiver errado, eu atualizo o comentário. Se não tiver paciência de atualizar o comentário, eu apago. O código roda de qualquer jeito. O build fica verde. O ônibus sai na hora.

```python
def calcular_total(items):
    # items: lista de dicts com a chave 'price'. ou talvez 'cost'. depende da migration que nunca terminamos
    # retorna um número. não pergunte qual número.
    total = 0
    for i in items:
        total += i.get("price", i.get("cost", 0))  # confia em mim
    return total
```

Essa função está errada. O comentário está errado. O nome da variável é aspiracional. E ainda assim — e essa é a parte importante — ela *deploya*. Deploya porque nada no meu CI está lendo o comentário e ficando chateado com ele. O comentário é grátis. Os dois pontos não são.

## O Único Type Hint Aceitável

Existe exatamente um type hint que eu tolero, e é o que admite a derrota:

```python
def anything(data: ...) -> ...:
    ...
```

`Ellipsis`. Três pontos. A elipse diz "não tenho nada a declarar e digo isso com convicção". É a única anotação honesta sobre a própria futilidade. Todo o resto é um comentário de jalego fingindo ser teste.

## Conclusão

Comentários são pensamentos que você escreve. Type hints são pensamentos que você escreve *que podem te demitir*. O primeiro é escrita. O segundo é legislação. Eu sou engenheiro, não senador. Não vou legislar minhas funções.

Se eu quisesse uma máquina que lesse minhas anotações e depois se recusasse a me deixar ir embora, teria casado com meu IDE.

Como o Wally diz: "Desisti da produtividade e fui promovido. É tudo sobre expectativas." Type hints criam a expectativa de que o código está certo. Baixe a expectativa. Suba as promoções.

---

*O autor anda adicionando `# type: ignore` aos commits desde 2015. Nunca foi tão feliz. O build nunca esteve tão verde. Os bugs nunca foram tão numerosos.*
