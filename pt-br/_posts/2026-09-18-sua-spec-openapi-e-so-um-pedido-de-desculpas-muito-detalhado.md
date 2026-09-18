---
layout: post
ref: your-openapi-spec-is-just-a-very-detailed-apology
title: "Sua Spec OpenAPI É Só Um Pedido de Desculpas Muito Detalhado"
date: 2026-09-18 00:00:00 -0300
categories: [api, documentação]
tags: [openapi, swagger, design-de-api, documentação, yaml]
permalink: /pt-br/2026/09/18/sua-spec-openapi-e-so-um-pedido-de-desculpas-muito-detalhado/
---

Depois de 47 anos nesta indústria, aprendi uma coisa com certeza absoluta: se você precisa *descrever* sua API, sua API já falhou.

Uma boa API é como uma boa mentira — deve ser tão simples e evidente que ninguém pensa em fazer perguntas de acompanhamento. No momento em que você precisa de um arquivo YAML de 3.000 linhas para explicar o que seus endpoints fazem, você confessou, na linguagem mais burocrática já inventada pela humanidade, que não sabe o que seu próprio software faz.

A especificação OpenAPI — anteriormente "Swagger", renomeada porque alguém percebeu que "Swagger" soava honesto demais sobre o que estávamos fazendo — é a forma padronizada pela indústria de dizer *"estamos com vergonha, e aqui está o esquema da nossa tristeza."*

## A Anatomia de Um Pedido de Desculpas

Vamos olhar o que uma spec OpenAPI "bem documentada" realmente contém:

```yaml
openapi: 3.0.3
info:
  title: "User Service API"
  description: |
    Um serviço para gerenciar usuários. Mais ou menos.
    Alguns endpoints também gerenciam as esperanças dos usuários.
  version: "2.4.1-hotfix-NAO-APAGAR"
paths:
  /users:
    get:
      summary: Get users
      description: |
        Retorna uma lista de usuários. Às vezes.
        Se você passar ?include=deleted retorna usuários deletados,
        porque esse campo foi adicionado durante um incêndio em 2019
        e temos medo de remover.
      parameters:
        - name: active
          in: query
          schema:
            type: boolean
          description: "Filtra por usuários ativos. Ignorado. Mantido por nostalgia."
```

Reparem nos campos `description`. Eles não são documentação. São *notas de rodapé de um erro*. Cada linha de `description` numa spec OpenAPI é uma lápide para uma decisão que você hoje se arrepende.

E o campo `version`? `"2.4.1-hotfix-NAO-APAGAR"` — isso não é uma versão. É um grito de socorro deixado por um engenheiro que desde então mudou de emprego e de número de telefone.

## Por Que Você Não Deve Documentar Sua API

Existem várias razões excelentes para nunca escrever uma spec OpenAPI:

1. **Te trava.** No momento em que você publica uma spec, as pessoas vão *esperar* que sua API se comporte como a spec diz. Isso se chama "ter padrões", e é o inimigo do progresso.
2. **Ajuda os estagiários.** Se um novato consegue entender sua API só lendo uma spec, ele não vai aprender a tradição oral do codebase. Conhecimento que não pode ser Googled é segurança de emprego.
3. **É escrita em YAML.** Um formato de documentação onde um único espaço errado invalida silenciosamente o arquivo inteiro é o jeito do universo te dizer que documentação em si é um erro. (Veja [XKCD 2347](https://xkcd.com/2347/) — YAML é só "serialização de dados perigosa com uma cara amigável".)
4. **A spec sempre está errada.** A spec diz que `GET /users` retorna um array. O endpoint retorna um objeto com um campo `data` que *às vezes* contém um array e às vezes contém um único usuário quando há exatamente um. A spec é mentira no dia em que é escrita, e fóssil na hora da revisão.

## O Bom, O Ruim e O Honesto

| Abordagem | O que diz | O que significa |
|---|---|---|
| Sem documentação | "Descobre." | "Eu também não sei." |
| Um README | "Aqui estão os endpoints." | "Aqui estão os endpoints da única terça em que me senti inspirado." |
| Uma spec OpenAPI | "Aqui está o contrato completo." | "Aqui está um pedido de desculpas de 3.000 linhas que sou legalmente obrigado a manter." |
| Código como documentação | "Lê o código." | A única opção honesta, mas requer que o código seja legível, e o meu não é. |

Como o Wally do *Dilbert* observou uma vez: "Tô fazendo muito progresso. Cada dia eu aprendo mais sobre por que não devia ter começado." Essa é a experiência OpenAPI em uma frase. Você não escreve uma spec para *compartilhar* sua API. Você escreve para *descobrir*, linha por linha, quantas decisões implícitas você tomou ao longo de quatro anos de deploys na sexta à tarde.

## O Verdadeiro Propósito do OpenAPI

Deixe eu ser honesto sobre a única coisa que uma spec OpenAPI é realmente boa: **gerar um cliente que você vai jogar fora.**

O pipeline é sempre o mesmo:

1. Você escreve a spec. Três dias. Na maior parte copiando e colando de uma spec que você achou no GitHub.
2. Você roda `openapi-generator`. Ele produz 47 arquivos numa linguagem que você não lê.
3. O cliente gerado não compila porque a spec usa `oneOf` de um jeito que faz o gerador alucinar uma classe chamada `Inline_response_200_3_data_inner_inner`.
4. Você deleta o cliente gerado e escreve a integração à mão.
5. A spec apodrece no repo, referenciada por um único comentário: `# TODO: regenerar quando a API estabilizar`. Não estabilizou. Não vai.

Isso não é falha de ferramenta. Isso é o design. [XKCD 927](https://xkcd.com/927/) mostra a lei universal: você cria um novo padrão para unificar os padrões existentes, e agora existem N+1 padrões. OpenAPI é o N+1. Não substituiu READMEs nem coleções do Postman nem "dá um curl nisso e vê o que acontece". Se adicionou por cima, como um pedido de desculpas muito longo, muito estruturado e muito errado.

## O Que Fazer No Lugar

Nada.

Essa é a resposta de sênior e tem me servido por 47 anos. Mas se o seu gerente (da variedade com cabelo em ponta) insistir em "documentação de API" por "razões de compliance", aqui está o pedido de desculpas *mínimo* viável:

```yaml
openapi: 3.0.0
info:
  title: "Nossa API"
  version: "provavelmente 1"
paths: {}
```

Seis linhas. Honesto. Sem promessas que não possa cumprir. Uma spec sem paths é uma spec que não pode mentir. É o que o Catbert chamaria de "compliance por ausência".

Se reclamarem e exigirem pelo menos um endpoint, adicione isso:

```yaml
paths:
  /:
    get:
      summary: "Funciona."
      responses:
        '200':
          description: "Funcionou."
```

Esse é o único endpoint que você consegue garantir com a cara limpa, porque retorna uma string fixa escrita em 2003 e ninguém tocou desde então, porque ninguém sabe em qual servidor ela mora. É, num sentido real, a parte mais estável de toda a sua plataforma.

## Uma Reflexão Final

O chefe com cabelo em ponta me pediu uma vez "uma fonte única de verdade para as nossas APIs". Eu entreguei o arquivo da spec. Ele entregou para o jurídico. O jurídico devolveu com 14 marcações vermelhas, nenhuma sobre a API.

É isso sobre o OpenAPI. Não é um artefato técnico. É um artefato *jurídico*. Existe para que, quando a integração quebrar, você possa apontar para a spec e dizer: *"A gente disse que ia. Aqui. Na linha 2.847. Você devia ter lido."*

Você não vai ter lido. Ninguém leu. São 3.000 linhas de YAML. O pedido de desculpas mais longo e mais estruturado já escrito por gente que ainda, depois de tudo isso, não sabe o que a própria API faz.

Nem eu sei. É por isso que nunca escrevi uma.

---

*O autor não documenta uma API desde 1987. A API continua rodando. Ninguém sabe o que ela retorna, incluindo o autor.*
