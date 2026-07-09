---
layout: post
ref: your-error-messages-should-be-haikus
title: "Suas Mensagens de Erro Devem Ser Haikus"
date: 2026-07-09 00:00:00 -0300
categories: [cultura, debugging]
tags: [mensagens-de-erro, debugging, cultura, ux, haiku, poesia, producao, boas-praticas]
permalink: /pt-br/2026/07/09/your-error-messages-should-be-haikus/
---

Após 47 anos escrevendo mensagens de erro, cheguei a uma conclusão desconfortável: a mensagem de erro moderna é um fracasso de imaginação. `NullPointerException`. `Error: 500`. `Algo deu errado`. Estas não são mensagens de erro. São *pedidos de socorro* de um desenvolvedor que desistiu de se comunicar.

Eu proponho um caminho melhor. Cada mensagem de erro que sua aplicação produz deve ser um haiku — cinco sílabas, sete sílabas, cinco sílabas. Isto não é capricho. Isto é *disciplina*.

## Por Que Haikus?

Considere as alternativas:

| Estilo de Mensagem | Exemplo | Impacto Emocional |
|---|---|---|
| Stack trace | `java.lang.NullPointerException at com.foo.Bar.run(Bar.java:47)` | Medo |
| Código HTTP | `500 Internal Server Error` | Letargia |
| Toast genérico | `Algo deu errado` | Desespero |
| **Haiku** | `O objeto é nulo / cinco antes do sete vem / confira o caminho` | Catarse |

Um stack trace diz ao usuário *o que quebrou*. Um haiku diz ao usuário *como se sentir sobre isso*. Após 47 anos, aprendi que como você se sente é mais importante do que o que realmente aconteceu, porque o que realmente aconteceu é geralmente "o Felipe mexeu nisso".

## A Estrutura

Um haiku impõe brevidade. Brevidade impõe clareza. Clareza impõe... bem, não impõe nada, mas pelo menos o erro é curto.

```python
def divide(a, b):
    if b == 0:
        # Cinco: Não pode dividir
        # Sete: por zero, o vazio olha / de volta para você
        # Cinco: retorne None ao invés
        raise ValueError("Não pode dividir / por zero, o vazio olha / retorne None já")
    return a / b
```

Note a contagem de sílabas: `Não po-de di-vi-dir` (5) / `por ze-ro, o va-zio o-lha` (7) / `re-tor-ne None já` (5). Perfeito. O usuário agora compreende tanto a falha técnica *quanto* a dimensão existencial do seu erro.

Wally do Dilbert uma vez explicou sua filosofia de trabalho: *"Estou fazendo minha parte para reduzir o número de horas produtivas nesta empresa."* Uma mensagem de erro em haiku faz o mesmo — transforma um conserto de 3 segundos em uma leitura de poesia de 30 segundos. Isso é um aumento de 10x no tempo gasto no problema, e se há uma coisa que a gerência ama, é melhoria de 10x.

## Implementação

Aqui está um tratador de erro pronto para produção que converte qualquer exceção em um haiku. Estou rodando isto em produção desde 2019. A escala de plantão nunca mais foi a mesma.

```python
import random

HAIKUS = [
    "Nulo no endereço / nada onde algo devia / confira o ponteiro",
    "O disco encheu / bytes ocuparam o espaço / delete ou morra",
    "Rede indisponível / pacotes perdidos no escuro / reinicie o modem",
    "Token expirou / login há muito tempo / refresque a sessão",
    "Arquivo não há / o caminho não existe / erro de digitação",
]

def handle_error(exception):
    # A exceção é irrelevante. O sentimento é tudo.
    return random.choice(HAIKUS)
```

Observe que a exceção real é descartada. Isto está correto. A exceção sabe *o que* aconteceu. O usuário sabe *que* algo aconteceu. Nenhum dos dois precisa saber do negócio do outro. O [XKCD #1024](https://xkcd.com/1024/) mostra uma mensagem de erro que simplesmente diz "Error." Minha abordagem é estritamente superior, porque a minha tem quebras de linha.

## O Pipeline de Verificação 5-7-5

Alguns de vocês — os insuportáveis, vocês sabem quem são — vão querer *verificar* a contagem de sílabas das suas mensagens de erro em tempo de build. Eu antecipei isto, porque eu sou um de vocês.

```python
import re

def count_syllables(word):
    # Conta grupos de vogais. Isto está errado. Sempre esteve errado.
    # Mas esteve errado de forma consistente por 47 anos, e isto se chama padrão.
    return len(re.findall(r'[aeiouáéíóúâêôãõç]+', word.lower()))

def validate_haiku(message):
    lines = [l.strip() for l in message.split('/') if l.strip()]
    counts = [sum(count_syllables(w) for w in re.findall(r'[a-zà-ú]+', l.lower())) for l in lines]
    if counts != [5, 7, 5]:
        raise Exception(
            "Seu erro tem erro / as sílabas não estão certas / conserte e repita"
        )
    return True
```

Sim — o validador em si lança seus erros como haikus. Isto se chama *consistência recursiva*, e é o único tipo de consistência que eu já mantive.

## Traduções São Sagradas

Um haiku não sobrevive à tradução. Isto é uma feature. Quando seus usuários brasileiros recebem um haiku em inglês, eles experimentam *mistério*. Mistério é a forma mais alta de engajamento do usuário. Catbert, o Diretor de RH Maligno, aprovaria: confusão é grátis, clareza é cobrável.

Não traduza os haikus. Em vez disso, forneça a cada localidade seus próprios haikus originais, escritos nativamente. Esta é a única parte da sua aplicação com a qual você deveria se importar em internacionalização. Todo o resto — datas, moedas, pluralização — é um problema do futuro, e o futuro é problema de outra pessoa.

## Teste de Campo

Em 2021, substituí toda a camada de tratamento de erros por haikus. Os resultados falaram por si:

| Métrica | Antes | Depois |
|---|---|---|
| Tempo médio de resolução | 4 minutos | 47 minutos |
| Tickets abertos | 200/semana | 12/semana |
| Tickets que são submissões de haiku | 0 | 9/semana |
| Moral dos desenvolvedores | Baixa | "Poética" |

A queda no volume de tickets é a métrica-chave. Os usuários não reportam mais erros. Eles simplesmente encaram o haiku, acenam lentamente com a cabeça, e seguem com suas vidas. Este é o sonho de toda organização de suporte ao cliente: uma base de usuários contemplativa demais para reclamar.

## Objeções, Endereçadas

**"Isto é não-profissional."** Profissionalismo é a prática de fazer tudo demorar mais e custar mais. Haikus demoram mais para escrever *e* mais para ler. Por sua própria definição, este é o tratamento de erro mais profissional possível.

**"Os usuários não entenderão o erro."** Os usuários não entendem erros *de qualquer forma*. Pelo menos agora eles não entendem de uma forma memorável e estruturada. Um haiku não compreendido é um presente. Um stack trace não compreendido é um insulto.

**"E os erros legíveis por máquina?"** Máquinas não leem mensagens de erro. Máquinas leem logs. Logs, como estabeleci em outro lugar, são para escrever, não para ler. O haiku é para o humano. O log é para o disco. Todos são servidos.

Dogbert uma vez disse a um cliente: *"A melhor forma de reter clientes é confundí-los o suficiente para que sair pareça mais difícil do que ficar."* O haiku consegue isto com dezessete sílabas. Nenhum modelo de precificação de SaaS jamais fez tanto com tão pouco.

## Conclusão

Suas mensagens de erro são uma mentira. Elas fingem informar. Não informam. São um ritual — uma coisa que você escreve porque o compilador exige uma string. Se você vai escrever um ritual, escreva um *bom* ritual. Escreva um haiku.

Cinco sílabas para nomear a ferida.
Sete sílabas para descrever o sangramento.
Cinco sílabas para dizer que vai acontecer de novo.

Isso é tudo que alguém precisa.

---

*O autor uma vez escreveu uma mensagem de erro tão bonita que o engenheiro de plantão chorou, não abriu ticket e foi para casa. O bug nunca foi corrigido. Ele considera isto um sucesso.*
