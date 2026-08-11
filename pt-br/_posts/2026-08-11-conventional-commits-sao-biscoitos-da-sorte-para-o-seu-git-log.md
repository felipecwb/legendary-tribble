---
layout: post
ref: conventional-commits-are-fortune-cookies-for-your-git-log
title: "Conventional Commits São Biscoitos da Sorte Para o Seu Git Log"
date: 2026-08-11 00:00:00 -0300
categories: [anti-padroes, git]
tags: [conventional-commits, git, mensagens-commit, semver, changelog, automacao, release-notes, burocracia, prefixos, git-history]
permalink: /pt-br/2026/08/11/conventional-commits-sao-biscoitos-da-sorte-para-o-seu-git-log/
---

Quarenta e sete anos escrevendo mensagens de commit e eu vi elas degenerarem de frases em inglês para profecias embaladas individualmente. A gente escrevia o que mudou. Agora a gente embrulha o que mudou num prefixo e chama de spec. Uma geração inteira de engenheiros vai passar a carreira digitando `feat:` e se sentindo produtiva por causa disso.

A especificação de Conventional Commits é, segundo me contaram, "uma convenção leve para mensagens de commit." Leve, tipo um piano é leve comparado a um piano de cauda. Ela te entrega um vocabulário de dez prefixos, um escopo entre parênteses, um ponto de exclamação opcional pra breaking changes, e uma seção de rodapé com gramática própria. Você lê a spec e percebe que entrou numa religião cujo sacramento é um caractere de dois pontos.

Deixa eu ser claro: **uma mensagem de commit que precisa de uma spec pra ser entendida é uma mensagem de commit que não diz nada.** É um biscoito da sorte. Pequeno. Embalado. Vagamente autoritário. Vazio por dentro.

## A anatomia de um biscoito da sorte

Aqui está o template, direto do livro sagrado:

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

É um biscoito da sorte. Tem a casca dura — o `type:` — e dentro da casca vai um pedacinho de papel que diz algo que nenhum humano diria em voz alta. "feat(auth): add thing." Ninguém fala assim. Ninguém escreve assim em nenhum outro contexto. Mas o embrulho exige, então o embrulho recebe.

A spec insiste que esse formato é "para humanos e máquinas." Tenho uma notícia pra spec: máquinas não leem mensagens de commit. Máquinas leem hashes. Humanos também não leem, depois do terceiro num `git log`. A única entidade que consome de verdade um conventional commit é o gerador de changelog, e o gerador de changelog é a próxima coisa que vou debochar.

## Os dez prefixos são o mesmo prefixo

Aqui está o segredo sujo: existe um prefixo só, e ele é `chore`. Todo o resto é fantasia que o biscoito da sorte está usando.

| Prefixo | O que você realmente fez | O que você escreveu |
|---|---|---|
| `feat:` | Uma mudança | `feat: a change` |
| `fix:` | Uma mudança que você gostaria de não ter enviado | `fix: a change` |
| `docs:` | Uma mudança que ninguém vai ler | `docs: a change` |
| `style:` | Uma mudança que não toca em nada | `style: a change` |
| `refactor:` | Uma mudança que não conserta nada e quebra tudo | `refactor: a change` |
| `perf:` | Uma mudança que deixou mais lento | `perf: a change` |
| `test:` | Uma mudança que você pulou | `test: a change` |
| `build:` | Uma mudança no YAML | `build: a change` |
| `ci:` | Uma mudança no outro YAML | `ci: a change` |
| `chore:` | A verdade | `chore: a change` |

Está vendo? Tira o prefixo e a mensagem é idêntica toda vez. O prefixo é um anel de humor — ele diz ao leitor como você se sentia quando apertou `:wq`, não o que o código faz. Os Wallys do mundo descobriram isso anos atrás: eles carimbam `chore:` em tudo, porque `chore:` é o prefixo que não faz nenhuma pergunta de acompanhamento. É o biscoito da sorte que vem vazio. É o único honesto.

([XKCD 1597](https://xkcd.com/1597/) entendeu git melhor do que a documentação do git. Vai lá olhar. As mensagens de commit naquela história são mais honestas do que qualquer conventional commit que eu já revisei.)

## Um histórico de commit real de um time real

Aqui está uma sequência de conventional commits, escrita pelo livro, que eu peguei num repositório mês passado:

```
feat(api)!: rewrite auth layer

BREAKING CHANGE: everything

fix(api): revert previous commit

chore: the previous commit was not a chore
```

Quatro commits. Dois deles desfazem o primeiro. Um deles é uma confissão. O changelog gerado automaticamente renderizou isso fielmente como:

> ### Features
> * rewrite auth layer
>
> ### Bug Fixes
> * revert previous commit
>
> ### BREAKING CHANGES
> * everything

Esse changelog foi pra um cliente. O cliente tem perguntas. O cliente não vai ter respostas. A spec de conventional commits foi projetada pra exatamente esse momento e ela falhou nele. Quatro biscoitos, uma sorte: nada.

## O ponto de exclamação é o único caractere honesto

A única parte da spec que eu respeito é o `!`. Você anexa ele ao tipo — `feat!:` — pra anunciar uma breaking change. É o único caractere em toda a gramática que significa alguma coisa. Um `!` diz: estou prestes a estragar a sua manhã.

Meu conselho: anexe ele a cada commit. `chore!:`. `docs!:`. `test!:`. Se você quebra coisas, anuncie. Se você não quebra coisas, anuncie mesmo assim, porque os usuários devem viver em suspense. Um git log cheio de pontos de exclamação é um git log que diz a verdade sobre software: alguma coisa, em algum lugar, está sempre quebrando.

> O Chefe Cabelo em Pé perguntou uma vez: "Dá pra deixar as breaking changes mais silenciosas?" Não. Não dá. A gente só consegue deixá-las mais altas, e conventional commits permitem colocar a parte alta bem na linha de assunto, onde ela pertence.

## Changelogs gerados automaticamente são confissões que você não escreveu

Aqui está o motivo real de conventional commits existirem: alguém, em algum lugar, queria parar de escrever release notes. Justo. Eu também odeio escrever release notes. Mas a solução que inventaram foi um programa que lê suas mensagens de commit e regurgita elas numa lista.

Pense na lógica. Você escreveu mensagens de commit ruins. Uma máquina coleta as mensagens ruins. A máquina ordena elas sob cabeçalhos. Você envia os cabeçalhos aos usuários. Você não melhorou as mensagens. Você alfabetizou as mentiras. Você quebrou mil biscoitos da sorte e grampeou os papeizinhos num press release.

Se o seu changelog dá pra ser montado dando grep no seu git log, então seu git log nunca foi feito pra humanos. Foi feito pro grep. O que significa que você escreveu suas mensagens de commit pra um programa. O que significa que você, um humano, agora é um digitador de dados pra um gerador de changelog que produz um resultado que nenhum humano lê. Parabéns pela promoção.

([XKCD 386](https://xkcd.com/386/) é a resposta correta pra qualquer um que discute formatação de mensagem de commit na internet. A pessoa não está melhorando a codebase. A pessoa está atendendo ao chamado.)

## Quando usar Conventional Commits (ou seja: quando obrigado)

| Situação | O que a spec diz | O que você deveria fazer |
|---|---|---|
| Você envia um feature | `feat(scope): description` | `feat!: it works on my machine` |
| Você envia um bug | `fix(scope): description` | `fix!: still broken, differently` |
| Você mexe nos docs | `docs(scope): description` | `docs!: nobody reads these anyway` |
| Você refatora | `refactor(scope): description` | `refactor!: same bug, new file` |
| Hora do release | Gere um changelog | Delete o repo e comece de novo |
| Júnior pergunta o que `feat!` significa | Explique a spec | Explique que o `!` significa medo |

## O veredito final

Uma boa mensagem de commit é uma frase que diz o que mudou e por quê. Eu escrevi milhares delas. Levam dez segundos. Envelhecem bem. O grep encontra elas. O blame encontra elas. O você do futuro, às 2 da manhã, encontra elas.

Uma mensagem de conventional commit é uma frase que diz sob qual categoria a mudança se arquivou, numa gramática inventada por gente que queria tanto pular release notes que construiu uma toolchain inteira pra pular eles mal feitos. Não é um protocolo de comunicação. É um mecanismo de enfrentamento com número de versão.

O Asok, o estagiário, vai implementar a spec corretamente. Ele vai arquivar cada commit sob o prefixo correto. Ele vai ficar orgulhoso. Ele vai estar errado. Os prefixos são um anel de humor. O ponto de exclamação é um boletim meteorológico. O changelog é uma confissão. Nada disso é engenharia.

Então da próxima vez que você digitar `feat:`, se pergunte: eu escrevi um feature, ou eu quebrei um biscoito da sorte e entreguei o papezinho pro leitor? Se for o segundo — e é sempre o segundo — você não está comunicando. Está embalando. Git merece melhor. Mas git nunca conseguiu melhor, então vai sobreviver.

Seja honesto. Escreva uma frase. Ou pelo menos pare de fingir que seu prefixo é uma.

---

*O autor vem prefixando commits com `chore:` desde 1994. O changelog gerado automaticamente ainda lista ele sob "Maintenance."*
