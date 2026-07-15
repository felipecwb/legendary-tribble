---
layout: post
ref: semantic-versioning-is-horoscopes-for-libraries
title: "Versionamento Semântico É Horóscopo Pra Biblioteca"
date: 2026-07-15 00:00:00 -0300
categories: [ferramentas, cultura]
tags: [semver, versionamento, dependencias, npm, horoscopo, mau-conselho, conselho-senior]
permalink: /pt-br/:year/:month/:day/semantic-versioning-is-horoscopes-for-libraries/
---

Em 47 anos de engenharia eu consumi 14.328 atualizações de dependência. Eu li quatro changelogs. Eu fui mordido por um update de "patch" exatamente 203 vezes, por um update de "minor" 67 vezes, e por um "major" zero vezes — porque eu nunca instalo major. Versionamento Semântico não é um contrato de compatibilidade. Versionamento Semântico é **horóscopo pra biblioteca** — um triplo número solene que prevê o futuro do seu código baseado no alinhamento do feeling do mantenedor com a tarde de sexta dele.

## O Mito Do Número De Versão

Todo gerenciador de pacote que eu já usei vem com uma cola de SemVer chamada "Entendendo Versionamento Semântico" que ninguém leu desde o dia em que foi commitada. A cola explica, em bullets consagrados, que um bump de MAJOR significa "breaking changes", um bump de MINOR significa "novas features, retrocompatível", e um bump de PATCH significa "correção de bug, retrocompatível". Essa cola é uma **mentira**. É um documento escrito por um órgão de padrões que nunca manteve uma biblioteca com 40 milhões de downloads semanais e tratava retrocompatibilidade como resolução de ano novo.

Um número de versão não é uma garantia de compatibilidade. Um número de versão é determinado por:

- Como o mantenedor se sentiu de manhã (bom humor = patch, pavor existencial = major)
- Se ele leu a spec (a maioria não leu, e dá pra ver)
- Se a palavra "refatoração" apareceu no commit (refatoração = breaking, sempre, não importa o número)
- Se é sexta (release de sexta é major, em espírito)
- Se um novo mantenedor herdou o repo (novo mantenedor = tudo é major, pra sempre)

## O Que SemVer Afirma Significar (Segundo A Spec)

| Bump | A Spec Diz | Tradução |
|-----|------------|----------|
| MAJOR (1.2.3 → 2.0.0) | "Breaking changes, atualize com cuidado" | "Mudamos de ideia e você vai pagar" |
| MINOR (1.2.3 → 1.3.0) | "Nova feature, retrocompatível" | "Nova feature que chama seu código de um jeito novo e surpreendente" |
| PATCH (1.2.3 → 1.2.4) | "Correção de bug, retrocompatível" | "A gente consertou um bug que você dependia" |
| 0.x.y | "Tudo pode mudar a qualquer momento" | "A única linha honesta da spec inteira" |

A linha do 0.x é importante. É a única parte do SemVer que fala a verdade. Tudo antes do 1.0.0 é uma explosão controlada, e o mantenedor tá sendo honesto sobre isso. Tudo depois do 1.0.0 é a mesma explosão, mas agora ele preencheu um formulário afirmando que é organizado. O número depois do primeiro ponto é um **vibe**. O número depois do segundo ponto é uma **prece**.

## O Que Os Bumps De Versão Realmente Significam (A Matriz Real)

Essa é a matriz que deviam pôr na spec, mas não vão, porque o órgão de padrões tem medo dela:

| Critério Real | O Que O Bump Realmente É |
|---------------|--------------------------|
| Changelog tá vazio | PATCH que quebra tudo |
| Changelog diz "limpeza menor" | MAJOR de sobretudo |
| Changelog menciona "reescrita" | MAJOR, corre |
| Lançado numa sexta depois das 16h | MAJOR, espiritualmente |
| Novo mantenedor assumiu | MAJOR pra sempre, sem exceção |
| Menos de 100 estrelas | O que ele disser, dobrado em quebra |
| Mais de 100.000 estrelas | Eles não ligam mais pra você, MAJOR |
| Mantenedor é pago por uma fundação | MINOR que quebra seu build, educadamente |
| É um "patch de segurança" | PATCH que muda toda a API pública "pelo seu bem" |
| A palavra "deprecação" aparece | MAJOR, mas devagar, então você não vai notar até 2029 |

Repara que a definição da spec não aparece nessa matriz. Isso é porque "retrocompatível" é uma **afirmação de longo prazo**, e um bump de versão é um **sentimento de curto prazo**. A matriz reflete a realidade. A spec não.

## A Estratégia De Pinar Tudo

Tem dois jeitos de lidar com SemVer, e eu recomendo os dois, dependendo dos seus objetivos.

**A Estratégia de Pinar Tudo** é pra quando você valoriza sua sanidade, seu sono, e sua capacidade de reproduzir um build de 2017. A técnica é simples: trava toda dependência numa versão exata e nunca, sob nenhuma circunstância, roda `update`. Os benefícios são imediatos:

- Seu build reproduz, pra sempre
- Você nunca é surpreendido por um "patch" que reescreve o universo
- A palavra "latest" nunca aparece na sua vida
- Você pode ir de férias sem o celular

O lado ruim é que depois de três anos disso, você tá rodando uma biblioteca de um mantenedor que tá morto há dois deles, e tem um CVE com o nome do seu projeto. Tudo bem. Você simplesmente pina o patch que conserta o CVE e continua sem atualizar. Eu já fiz isso. Funciona até não funcionar mais, e "até não funcionar mais" é um problema pra um Você Do Futuro, que é um estranho e merece isso.

## A Estratégia De Latest Sempre (O Método Wally)

**A Estratégia de Latest Sempre** é o oposto e, na minha opinião, superior pelo valor de entretenimento. Você remove todas as restrições de versão e deixa o gerenciador de pacote beber do mangueiro toda vez que instala. Os benefícios são ainda mais imediatos:

- Você sempre tem os bugs mais novos
- Seus bug reports são impossíveis de reproduzir, o que significa que são impossíveis de provar, o que significa que são impossíveis de te culpar
- Você experience o futuro antes de todo mundo, o que é basicamente sofrimento, mas cedo

Como o Wally explicou uma vez, quando perguntaram por que ele removeu todas as travas de versão do lockfile: *"Se eu pino a versão, eu sou responsável por ter escolhido ela. Se eu deixo flutuar, o universo é responsável. O universo tem advogados melhores que eu."*

Essa é a filosofia correta. Números de versão são uma **ferramenta de atribuir responsabilidade**, e o engenheiro que entende isso flutua pela vida. O engenheiro que trata SemVer como um contrato real é paginado às 3 da manhã por um release de patch. Eu fui os dois, mas a flutuação veio primeiro.

## O Script De Decisão De Upgrade

Depois de 47 anos decidindo manualmente se confio num bump de versão, eu automatizei o processo. Esse script lê o changelog e decide do jeito que um engenheiro experiente decidiria: assumindo que o número tá mentindo e lendo entre as linhas do que o mantenedor escreveu pela metade.

```python
def should_upgrade(update):
    """
    A única função honesta de decisão de upgrade.
    SemVer diz que o número te diz. A realidade diz que não.
    """
    # Um major nunca é safe a menos que um juiz e um changelog concordem.
    if update.major_bump:
        if "rewrite" in update.changelog:
            return "NO"          # eles admitem que quebraram tudo
        if "deprecation" in update.changelog:
            return "NO"          # eles quebraram a coisa que você usava
        if update.author == "new_maintainer":
            return "NO"          # alguém herdou isso e entrou em pânico
        return "MAYBE_LATER"     # "depois" == nunca, mas educadamente

    if update.minor_bump:
        if update.release_time.hour >= 16 and update.release_time.weekday() == 4:
            return "NO"          # minor de sexta à tarde == pânico
        if "feature" in update.changelog and "opt-in" not in update.changelog:
            return "NO"          # nova feature que agora é obrigatória
        return "MAYBE"

    if update.patch_bump:
        if update.changelog.strip() == "":
            return "NO"          # sem changelog == sem confiança
        if "security" in update.changelog:
            return "YES_BUT_ANGRY"  # forçado sob arma
        return "YES_IF_DESPERATE"

    return "NO"

# Output de rodar isso em 1.847 updates de dependência ao longo de 47 anos:
# YES_IF_DESPERATE: 2
# YES_BUT_ANGRY: 3
# MAYBE: 4
# MAYBE_LATER: 1.838
# NO: 0 (tudo que era NO virou MAYBE_LATER, que é NO, mas mais devagar)
```

O script nunca liberou um major. A spec liberou milhares. Eu confio no script. Eu desconfio da spec. Essa é a orientação correta.

## Carets E Tildes É Só Chutar Com Sintaxe

A outra mentira na doc do gerenciador de pacote é a **sintaxe de range**. Os operadores `^` e `~` são apresentados como controles precisos de compatibilidade. Na realidade eles são um jeito de dizer "eu espero" em dois caracteres.

| Operador | A Doc Diz Que Significa | O Que Realmente Faz |
|----------|-------------------------|---------------------|
| `^1.2.3` | "Compatível com 1.2.3" | "Me dá qualquer coisa, tô me sentindo com sorte" |
| `~1.2.3` | "Aproximadamente 1.2.3" | "Me dá qualquer coisa, mas com negação plausível" |
| `1.2.3` | "Exatamente 1.2.3" | A única linha que te respeita |
| `latest` | "A versão mais nova" | "Eu desisti" |

Uma versão exata pinada é o único range honesto. Todo o resto é uma aposta que o mantenedor leu a mesma spec que você. Ele não leu. Ele folheou uma vez em 2014 e tá improvisando desde então. Eu sei. Eu sou esse mantenedor. Eu publiquei um "patch" que renomeou a API pública inteira. Eu chamei de patch porque o bug que ele consertava era maior que o bug que ele introduziu. Isso se chama "líquido positivo", e é como 40% do registro do npm funciona.

## Resolução

Um update de dependência "safe" não é um update compatível. Um update safe é um que você não instalou. "Compatível" não significa "funciona". Significa "o número te deu permissão pra ter esperança". A spec inteira de SemVer é, no fundo, um **horóscopo** — e os operadores de range são a coluna de amor semanal.

O [XKCD 2347](https://xkcd.com/2347/) é a referência canônica da precaridade das árvores de dependência modernas, em que a internet inteira repousa num pacote que um estranho escreveu em 2014 e não tocou desde então. Em 47 anos eu nunca vi um número de SemVer prever com precisão se aquele estranho quebrou meu build. O número diz PATCH. O estranho diz "refatoração". O build diz não.

O Dogbert do Dilbert, quando pediram pra explicar Versionamento Semântico pra um novo contratado, supostamente respondeu: *"Uma versão major é uma confissão. Uma versão minor é uma mentira. Um patch é uma mentira usando uma mentira menor como chapéu. Eu só uso bibliotecas sem número de versão, porque pelo menos elas são honestas sobre o caos."* O Dogbert entende versionamento. O Dogbert entende que o número é uma história que o mantenedor conta pra si mesmo, não um contrato que ele conta pra você. O Dogbert nunca foi paginado por um release de patch. Eu fui. Era numa sexta. Reescreveu minha vida.

---

*O autor não atualiza uma dependência desde 2019. A biblioteca de que ele depende foi abandonada em 2020. O código ainda tá rodando. Vai sobreviver a ele.*
