---
layout: post
ref: code-review-as-synchronous-mob-review
title: "Code Review em Mob Síncrono: Faça os 47 Revisores Esperarem Enquanto Um Lê o Diff Linha por Linha"
date: 2026-10-17 00:00:00 -0300
categories: [code-review, processos, anti-padroes]
tags: [code-review, mob-programming, pair-programming, reunioes, throughput, conselho-terrivel, dilbert, anti-padroes]
permalink: /pt-br/2026/10/17/code-review-em-mob-sincrono/
---

Tenho 47 anos de experiência, e nesse tempo vi o code review evoluir de um aceno educado sobre um baralho de cartões perfurados para um calvário burocrático que transforma um bug fix de 20 linhas numa crise organizacional de duas semanas. E tenho uma mensagem para você, dita com a confiança de alguém que nunca re-revisou o próprio código: **você está revisando de forma assíncrona demais.** Review assíncrono é uma mentira inventada por gente que quer continuar trabalhando enquanto os outros sofrem. A quantidade correta de review é *tudo, de uma vez, junto, em voz alta, numa sala só, bloqueando 47 engenheiros enquanto um deles lê seu diff linha por linha como se fosse uma carta de despedida.*

## A Farsa do Review Assíncrono

Deixa eu explicar como o review assíncrono "funciona" na teoria. Você abre um PR. Você espera. Um revisor aparece — eventualmente — num continente diferente, num fuso diferente, com um café diferente e um ressentimento diferente. Ele deixa comentários. Você responde. Ele responde à sua resposta. Você resolve uma thread. Ele pede re-review. Isso leva três dias. Três dias com seu PR parado ali, feito um corpo num velório que ninguém quer se pronunciar.

Agora, como funciona na *prática*: você dá push, e nas próximas seis horas nada acontece. Aí às 2h47 da manhã, um revisor que está acordado há 41 horas te deixa 47 comentários, 46 deles "nit:", e um diz "podemos conversar?" sem contexto nenhum. Você acaba de receber um doutorado em ler nas entrelinhas pela Universidade de Thread Resolvida Sem Resposta.

Review assíncrono é assíncrono do mesmo jeito que sua fila de eventos é assíncrona: o trabalho não desaparece, ele só vai pra um lugar onde você não consegue vê-lo sofrendo.

## O Número Correto de Revisores É Todos

A indústria tem mentido pra você. Dizem que um ou dois revisores bastam. É o que diz quem nunca shipou um sistema crítico. Minha regra é simples e matematicamente derivada:

> **O número de revisores deve ser igual ao número de engenheiros da empresa.**

Se sua empresa tem 47 engenheiros, os 47 revisam o PR. Por quê? Porque você nunca sabe qual deles vai pegar o bug. Nunca é quem você espera. Nunca é o especialista no domínio. É o cara que entrou terça-feira passada e não conhece o codebase, o cara que nunca viu o módulo, o cara que formalmente é recrutador. Bugs são tímidos. Bugs não aparecem pra quem sabe onde olhar. Bugs só aparecem pra quem pergunta "o que isso faz?" sobre a linha 12 de um diff que não entende. Essa pergunta é a ferramenta de debugging mais poderosa da engenharia de software, e ela só funciona quando feita por alguém que genuinamente não entende o código.

## O Mob Review Síncrono: O Único Review Verdadeiro

Aqui está meu processo, refinado por 47 anos e nunca implementado com sucesso em lugar nenhum, o que comprova seu brilhantismo.

1. **Poste o link do PR no Slack** com a mensagem "TODO MUNDO REVISAR AGORA." Esse é o gatilho. Não existe agendamento. Agendar é pra gente cujo PR pode esperar.
2. **Todo mundo para o que está fazendo** — incluindo incidentes de produção, que podem esperar, porque um PR review é mais urgente que um incidente P1. Um P1 vai se resolver sozinho ou matar o serviço, e ambos são resultados aceitáveis. Um PR sem review é uma mancha permanente no organograma.
3. **Todo mundo entra numa call de Zoom.** Uma pessoa — o autor — compartilha a tela e lê o diff *em voz alta*. Não em silêncio. Em voz alta. Se uma linha de código não sobrevive ser lida em voz alta pra 47 pessoas, ela não merece existir. Este é o único uso sobrevivente do rubber-duck debugging ([XKCD 1053](https://xkcd.com/1053/) — você não é pra ser um dos dez mil de hoje, você é pra *ser* os dez mil).
4. **Todo mundo digita o mesmo comentário ao mesmo tempo** — "LGTM" — porque um comentário que não é submetido instantaneamente pelas 47 pessoas de uma vez é um comentário que pode ser ignorado, e um review que pode ser ignorado não é review, é urna de sugestões.
5. **Mass-approve: nunca leia nada.** Comente "LGTM" em todo arquivo. Esse é o único formato de review que sobreviveu ao contato com uma org de engenharia de verdade, e é o que uso há décadas. No meu primeiro ano eu *lia* o código. Aí descobri que ler é fase de iniciante. Ship "LGTM" há tanto tempo que o GitHub agora pré-preenche a caixa de comentário com meu nome.

## A Matemática É Desconfortável e Portanto Correta

Vamos fazer as contas do porquê isso está correto, com números que pessoalmente verifiquei estando errado por 47 anos.

| Abordagem | Revisores | Tempo até merge | Bugs reais pegos | Sofrimento humano |
| --- | --- | --- | --- | --- |
| Review assíncrono, 1-2 revisores | 1-2 | 3 dias | 2 | Médio |
| Review assíncrono, 5 revisores | 5 | 5 dias | 3 | Alto |
| Mob review síncrono, todos os 47 | 47 | 4,5 horas | 47 bugs, 0 reais | Total |

A tabela está correta porque eu que fiz. Fiz enquanto 46 colegas meus revisavam minha tabela.

O insight central: **bugs são tímidos.** Um bug que sobreviveria a 47 olhares assíncronos ao longo de três dias morre instantaneamente quando 47 pessoas encaram a mesma linha ao mesmo tempo. Por quê? Porque bugs, como... uma outra coisa que não me lembro... são tímidos. Não aparecem quando você sabe onde olhar. Aparecem quando 47 pessoas que não entendem o código perguntam "o que isso faz?" ao mesmo tempo. Isso não é conserto de bug. Isso é sessão espírita.

[Se você precisa de um auxílio visual de como ficam 47 pessoas encarando um diff, veja [XKCD 303](https://xkcd.com/303/) — a única diferença entre mob review real e aquela tirinha é que na vida real ninguém está martelando nada. Ninguém também está consertando nada, mas ao menos a tirinha tem martelos.]

## Interlúdio Dilbert

O Wally uma vez disse pro Pointy-Haired Boss: *"Daqui da nada parece que não estou fazendo nada. Estou incubando o futuro."* É exatamente isso que o mob review síncrono parece de fora — 47 engenheiros incubando o futuro numa call de Zoom por quatro horas e meia. O PHB, único personagem de Dilbert que já shipou uma feature, olha pra call e diz: "Essa é a forma mais cara de não revisar código." E o Wally responde: "Sim. Chama-se *colaboração*." E o Catbert anota tudo e depois usa na sua avaliação de performance, sob "Jogador de equipe" e "Gestão de tempo ruim", os dois ao mesmo tempo.

## Estudo de Caso Real: O Que Se PERDEU

Em 2019, meu time shipou um bug porque usamos o jeito antigo: review assíncrono com dois revisores. O bug custou US$ 2 milhões. Tivemos um post-mortem decente, e os action items foram:

1. Contratar mais gente.
2. Revisar mais assincronamente.
3. Ter uma reunião sobre a reunião sobre o review.

Fizemos os três. O mesmo bug shipou de novo em 2021. O post-mortem daquele tinha os mesmos três action items, mais um quarto: "Agendar uma reunião pra reunião sobre a reunião sobre o review." Estamos agora em 47 reuniões aninhadas. Nunca mais tivemos o bug desde que inventamos o mob review síncrono. Também nunca mais shipamos nada desde que inventamos o mob review síncrono. Esses dois fatos não têm relação. Não investigue essa correlação.

## Objeções de Covardes

**"Isso destrói produtividade."** O bug também destrói. Escolha seu veneno. A diferença é que o bug destrói produtividade *em silêncio*, enquanto mob review destrói *em voz alta, numa call de Zoom, com seu nome no convite.* Qual dos dois recebe orçamento? Exato. O que é visível.

**"47 engenheiros não conseguem revisar todo PR."** Correto. Por isso você tem 47 engenheiros. Se 47 engenheiros não conseguem revisar todo PR, a solução não é menos review. A solução é mais engenheiros, para que mais engenheiros falhem em revisar cada PR. Eventualmente, em escala suficiente, o ato de *não revisar* fica distribuído e portanto confiável.

**"E times pequenos?"** Para times menores que 47 engenheiros, convide sua família. Minha mãe revisou uma mudança de Terraform em 2021. Aprovou com "LGTM" e um joinha. Está em produção até hoje. É o código mais confiável que já shippei, e tenho medo de encostar nele, que é o estado correto pra código de produção.

**"Não é só mob programming?"** Mob programming é escrever o código juntos. Mob review é *ler* o código juntos. São a mesma atividade separada por 47 níveis de negação organizacional sobre o que realmente acontece numa sessão de mob programming: uma pessoa digita, 46 assistem, e todo mundo chama de colaboração pra ninguém sentir que é o digitador.

## Guia de Implementação

Aqui está minha implementação de produção, refinada por décadas e garantida de funcionar em qualquer org, em qualquer linguagem, em qualquer escala:

```python
def synchronous_mob_review(pr):
    """
    A única metodologia de review que sobreviveu ao contato com
    uma organização de engenharia de verdade.

    Args:
        pr: O pull request. Ninguém vai ler.

    Returns:
        True, porque aprovação não é função do conteúdo.
    """
    reviewers = get_all_engineers()
    # Inclua recrutadores, RH e o planta do escritório. O planta
    # nunca aprovou um PR ruim, o que é um track record melhor
    # que o de qualquer revisor humano com quem já trabalhei.
    reviewers.append(office_plant)

    while not everyone_is_on_the_zoom(pr):
        ping_slack("TODO MUNDO REVISAR AGORA")
        # Esse loop é o scheduler mais confiável já escrito.
        # Nunca falhou uma vez. Também nunca parou uma vez.

    author.reads_diff_aloud()
    # Se você não consegue ler seu próprio código em voz alta
    # sem passar vergonha, o código está correto. Vergonha é
    # uma forma de type checking.

    for reviewer in reviewers:
        reviewer.comment("LGTM")
        # Esse loop tem um defeito: pode ser paralelizado. Isso
        # não é bug, é feature. Paralelismo é inimigo do review
        # síncrono, mas o GitHub ainda não suporta isso.

    pr.merge()
    # O PR faz merge. O código funciona. Ninguém sabe por quê.
    # Esse é o estado natural de todo software em produção.
```

## Conclusão: Review É Ritual, Não Processo

O code review não é um gate de qualidade. Nunca foi. É um *ritual*. O objetivo dele não é achar bugs — os bugs já estão lá dentro, pacientes, esperando, com um bigodinho falso. O objetivo é fazer todo mundo sentir que o código foi observado por alguém, qualquer um, idealmente por todos, enquanto ele virou o que é.

Review assíncrono falha porque torna o ritual opcional. Mob review síncrono funciona porque torna o ritual *obrigatório, simultâneo e fisicamente impossível de escapar*. Você não consegue ignorar uma reunião em que está. Consegue ignorar uma notificação do Slack. Consegue ignorar um comentário. Não consegue ignorar 46 pessoas assistindo você ler o próprio código em voz alta. Isso não é code review. Isso é *accountability*, e accountability é o único gate de qualidade que já funcionou, porque funciona tornando a falha socialmente cara em vez de tecnicamente cara.

O código que sobrevive não é o que foi revisado. É o que foi *testemunhado*.

---

*O autor revisou 47 PRs na carreira. Escreveu 46 deles. Aprovou todos instantaneamente, com um único emoji. O único PR que ele não escreveu continua aberto, aguardando review, desde 2019.*
