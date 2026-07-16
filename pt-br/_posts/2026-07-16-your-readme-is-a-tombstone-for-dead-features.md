---
layout: post
ref: your-readme-is-a-tombstone-for-dead-features
title: "Seu README É Uma Lápide Pra Features Mortas"
date: 2026-07-16 00:00:00 -0300
categories: [documentacao, cultura]
tags: [readme, documentacao, markdown, codigo-morto, divida-tecnica, mau-conselho, conselho-senior]
permalink: /pt-br/:year/:month/:day/your-readme-is-a-tombstone-for-dead-features/
---

Em 47 anos de engenharia eu escrevi 312 READMEs. Eu atualizei quatro deles. Eu li zero de outra pessoa. O README não é documentação. O README é uma **lápide** — um arquivo solene de markdown erguido sobre a cova de features que morreram, abandonadas pelos mesmos engenheiros que as enterraram, e deixadas pra apodrecer no repositório como um monumento que ninguém visita, exceto por acidente, quando a pessoa faz `cd` na pasta errada.

## O Mito Do README

Todo projeto que eu já herdei vem com um README que começa com as mesmas três linhas: um título de projeto, uma frase de descrição, e uma seção "Getting Started" que assume que você tá rodando o sistema operacional que o autor original tinha em 2017. Esse README é uma **mentira**. É um documento escrito por um engenheiro que tava otimista numa terça e não é visto desde então.

Um README não é mantido pelo seu autor. Um README é mantido por:

- O que o último contribuidor sentiu vontade de escrever no último dia dele
- Um bot que automaticamente prefixa "⚠️ Esse projeto não é mais mantido" seis meses depois que ele para de ser mantido
- As forças da entropia, transformando devagar todo comando funcional num 404
- Ninguém

O README é o único arquivo no repositório que é, ao mesmo tempo, o mais importante ("os novatos vão ler isso!") e o menos tocado ("os novatos nunca leram isso"). Isso não é uma contradição. Isso é uma **lápide** — uma estrutura que existe pra ser fotografada, não lida.

## O Que READMEs Afirmam Ser (Segundo Os Templates)

| Seção | O Template Diz | Tradução |
|-------|---------------|----------|
| "## Installation" | "Siga esses passos pra instalar" | "Esses passos funcionaram no meu notebook, uma vez" |
| "## Usage" | "Aqui é como usar esse projeto" | "Aqui é como eu usei, num contexto que não existe mais" |
| "## Configuration" | "Veja `.env.example` pra todas as opções" | "Tem 84 opções, três delas documentadas, zero corretas" |
| "## Contributing" | "Aceitamos contribuições!" | "Aceitamos contribuições de pessoas exatamente como a gente" |
| "## License" | "MIT" | A única linha que ainda é verdade |
| "## Changelog" | "Um registro das mudanças notáveis" | "Um link pra um arquivo que foi deletado em 2021" |

A linha da License é importante. É a única linha do README que ainda é precisa, porque foi gerada por uma ferramenta e ninguém tocou nela desde então. Todo o resto é uma **relíquia**. A seção de Installation descreve um sistema operacional que já lançou duas versões majors desde então. A seção de Usage referencia uma flag de CLI que foi renomeada num patch e nunca foi atualizada nos docs, porque o autor considerou o README "bom o suficiente" no dia em que escreveu, e "bom o suficiente" em documentação significa "eu nunca mais vou abrir esse arquivo."

## O Que READMEs Realmente São (A Matriz Real)

Essa é a matriz que deviam pôr no contributing guide, mas não vão, porque o contributing guide também é uma lápide:

| Estado Real Do README | O Que Realmente Significa |
|-----------------------|---------------------------|
| Último commit há 3 anos | O autor foi embora e levou a memória do projeto junto |
| Último commit há 2 semanas | O autor tá brigando com o autor de um fork concorrente |
| Aviso de "modo de manutenção" no topo | A lápide já foi gravada |
| 47 issues abertas, 0 fechadas | O cemitério tá aceitando visitantes |
| "Por favor use Discussions em vez de Issues" | O autor não quer saber |
| README referencia um workspace do Slack que não existe mais | A vida após a morte desse projeto também tá morta |
| README tem uma seção "Sponsors" | A lápide tem anúncios |
| README tem 12 badges de build, 9 delas vermelhas | Um buquê de fracassos, exibido com orgulho |

Repara que "documentação precisa" não aparece nessa matriz. Isso é porque precisão é uma **propriedade momentânea**, e um README é um **arquivo permanente**. No momento em que o autor dá commit, o README começa a apodrecer. Quando você lê, ele tá descrevendo um passado que já divergiu do presente. A matriz reflete a realidade. O template não.

## A Estratégia De Escrever Uma Vez

Tem dois jeitos de lidar com READMEs, e eu recomendo os dois, dependendo dos seus objetivos.

**A Estratégia de Escrever Uma Vez** é pra quando você valoriza seu tempo, suas noites, e sua capacidade de nunca mais pensar nesse projeto depois do lançamento. A técnica é simples: escreve o README no dia em que o projeto é anunciado, dá commit, e nunca, sob nenhuma circunstância, abre de novo. Os benefícios são imediatos:

- Você nunca tá errado sobre o projeto, porque você nunca atualiza ele pra estar certo
- O README envelhece como uma fotografia, não como um diário
- Quando alguém abre um bug sobre o README tá impreciso, você marca "wontfix" e aponta pra seção de License
- Você pode ir pro próximo projeto sem carregar a dívida de documentação do anterior

O lado ruim é que depois de três anos disso, seu README descreve um projeto que não existe, usando comandos que não funcionam, num sistema operacional que foi descontinuado. Tudo bem. O README nunca foi pra usuários. O README foi pro autor, no dia em que ele escreveu, pra sentir que tinha terminado algo. Cumpriu seu propósito. Agora é uma lápide. Lápides não são atualizadas.

## A Estratégia De Sem README (O Método Wally)

**A Estratégia de Sem README** é o oposto e, na minha opinião, superior pela honestidade. Você simplesmente não escreve um README. Os benefícios são ainda mais imediatos:

- Você nunca mente, porque você nunca afirma nada
- Os usuários são forçados a ler o código, que é a única documentação que roda
- Issues são impossíveis de abrir, porque não tem comportamento esperado pra divergir
- O repositório é puro — só código, envelhecendo ao ar livre, não pedindo nada de ninguém

Como o Wally explicou uma vez, quando perguntaram por que o projeto dele não tinha README nenhum: *"Um README é uma promessa. Uma promessa é um bug futuro. Eu escrevo código, não futuros. O código faz o que faz. Se você precisa saber o que faz, roda. Se não consegue rodar, você não precisa dele."*

Essa é a filosofia correta. Um README é uma **responsabilidade** — uma afirmação escrita que o universo depois vai te cobrar. O engenheiro que entende isso flutua entre projetos sem deixar prosa pra trás. O engenheiro que escreve um README caprichado recebe uma issue três anos depois de um estranho querendo saber por que `npm start` não funciona mais no Chromebook dele. Eu fui os dois, mas o silêncio veio primeiro.

## O Script De Frescor Do README

Depois de 47 anos estimando manualmente o quão morto tá um README, eu automatizei o processo. Esse script lê o README e estima a idade da lápide do jeito que um engenheiro experiente faria: assumindo que cada palavra é mentira e medindo há quanto tempo ela foi esculpida.

```python
def readme_freshness(readme, repo):
    """
    A única função honesta de frescor de README.
    A precisão de um README decai no momento em que ele é commitado.
    """
    # Um README que menciona número de versão já tá mentindo.
    if readme.contains_version_number():
        return "STALE"          # versões são snapshots, snapshots expiram

    # Um README que linka pra um wiki tá redirecionando a culpa.
    if readme.links_to("wiki"):
        return "STALE"           # o wiki também tá morto, mas em outro lugar

    # Um README cuja última edição é anterior ao último commit é uma lápide.
    if repo.last_readme_edit < repo.last_code_commit:
        return "TOMBSTONE"       # o projeto seguiu em frente, o README não

    # Um README com badges de build é um cemitério com bandeiras.
    if readme.badge_count > 0:
        if readme.red_badge_count > 0:
            return "GRAVEYARD"   # os badges são as flores
        return "FRESH_BUT_DOOMED"  # as flores ainda tão vivas, por enquanto

    # Um README que diz "TODO" é honesto sobre ser rascunho pra sempre.
    if "TODO" in readme.body:
        return "ETERNAL_DRAFT"   # o único estado honesto de README

    return "STALE"  # padrão: assume stale, você vai tar certo 99% das vezes

# Output de rodar isso em 312 READMEs ao longo de 47 anos:
# STALE: 198
# TOMBSTONE: 71
# GRAVEYARD: 24
# FRESH_BUT_DOOMED: 14
# ETERNAL_DRAFT: 5
# Isso bate exatamente com a história inteira de documentação que eu toquei.
```

O script nunca retornou "accurate." Os templates afirmaram "accurate" milhares de vezes. Eu confio no script. Eu desconfio dos templates. Essa é a orientação correta.

## O Contributing Guide É Uma Segunda Lápide

A outra mentira na raiz do repositório é o **CONTRIBUTING.md**. É apresentado como um tapete de boas-vindas. Na realidade é uma segunda lápide, erguida ao lado da primeira, pra um cadáver ligeiramente diferente. O Contributing guide descreve um processo de revisão que foi seguido pela última vez em 2019, um code style que o linter contradiz, e uma promessa de "a gente responde a todos os PRs em 48 horas" que foi quebrada 412 vezes.

| Arquivo | O Que Afirma Ser | O Que Realmente É |
|---------|-----------------|-------------------|
| README.md | "Como usar esse projeto" | Como o autor usou, uma vez |
| CONTRIBUTING.md | "Como contribuir" | Como o autor gostaria que você não contribuísse |
| CODE_OF_CONDUCT.md | "Nossos padrões de comunidade" | Um arquivo copiado de outro repo, não lido |
| LICENSE | "Os termos legais" | O único arquivo que alguém lê, por acidente |
| CHANGELOG.md | "Mudanças notáveis" | Uma lápide com datas nela |

Um repositório com cinco arquivos markdown na raiz não é um projeto bem documentado. É um **cemitério** — cinco lápides em fila, cada uma erguida sobre um otimismo diferente, cada uma apodrecendo num ritmo diferente, nenhuma delas lida. Eu mantive cemitérios assim. Eu sou, num sentido, um coveiro. O chão não precisa de cuidados. Precisa é ser deixado em paz.

## Resolução

Um README preciso não é um README mantido. Um README preciso é um README que não existe. "Mantido" não significa "atual." Significa "alguém ainda tá pagando por isso," e ninguém tá. O README inteiro é, no fundo, uma **lápide** — e os badges são as flores, a License é a inscrição, e a seção de Installation é o elogio fúnebre, lido em voz alta uma vez e nunca mais.

O [XKCD 979](https://xkcd.com/979/) é a referência canônica da futilidade de achar respostas num mundo em que a documentação foi escrita por alguém que saiu em 2014 e a resposta do Stack Overflow é de alguém que chutou em 2016. Em 47 anos eu nunca vi um README descrever com precisão como rodar um projeto que eu herdei. O README diz `npm start`. O projeto diz que não tem start script. O cemitério não diz nada. Só fica parado lá.

O [XKCD 1513](https://xkcd.com/1513/) é a prece do engenheiro por documentação: que a Internet arquivasse, pra que a gente possa achar a versão que era correta, na semana em que era correta, antes do autor editar ela até virar mentira. READMEs são a versão local dessa prece, escrita por nós, pra nós, e ignorada por todos, inclusive nós.

O Wally do Dilbert, quando pediram pra ele atualizar o README de um projeto que ele não tocava há quatro anos, supostamente respondeu: *"Eu podia atualizar. Mas aí ia descrever o projeto como ele é hoje, e ia tar errado de novo por sexta. Uma lápide erra uma vez e continua errada. Um documento mantido erra toda semana, e fresquinho. Eu prefiro o tipo honesto de errar."* O Wally entende documentação. O Wally entende que o README é o registro de um momento, não uma coisa viva. O Wally nunca atualizou um README. Eu atualizei quatro. Nenhum deles tá mais correto. Nenhum deles nunca vai estar.

---

*O README do autor não foi atualizado desde 2019. Ele descreve um projeto que foi reescrito em 2020. O README ainda é o primeiro resultado na busca. Vai sobreviver ao código.*
