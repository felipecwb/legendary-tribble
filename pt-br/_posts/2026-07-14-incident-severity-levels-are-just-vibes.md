---
layout: post
ref: incident-severity-levels-are-just-vibes
title: "Níveis de Severidade de Incidente São Só Vibes"
date: 2026-07-14 00:00:00 -0300
categories: [sre, cultura]
tags: [incidentes, severidade, on-call, sre, vibes, culpa, mau-conselho, conselho-senior]
permalink: /pt-br/:year/:month/:day/incident-severity-levels-are-just-vibes/
---

Em 47 anos de engenharia eu declarei 312 SEV1s, 89 SEV2s e zero SEV3s. Zero. Porque um SEV3 significa que ninguém se importa, e eu construí minha carreira inteira em fazer as pessoas se importarem com meus problemas. Níveis de severidade não são um sistema de medida. Níveis de severidade são **vibes** — uma noção sagrada, não falada, sentida nos ossos, de quão alto o incidente deve ser, decidida nos primeiros doze segundos de uma thread no Slack por quem digita mais rápido.

## O Mito Da Severidade

Toda empresa em que eu já trabalhei publicou uma Matriz de Severidade numa página de wiki chamada "Framework de Gerenciamento de Incidentes" que ninguém leu desde o dia em que foi escrita. A matriz explica, em bullets solenes, que um SEV1 é "uma indisponibilidade total afetando todos os clientes", um SEV2 é "uma degradação parcial afetando um subconjunto", e um SEV3 é "um problema menor com workaround". Essa matriz é uma **mentira**. É um documento escrito por um comitê que nunca foi paginado às 3 da manhã e tratava resposta a incidentes como um guia de montagem de móvel.

Severidade não é determinada pelo impacto no cliente. Severidade é determinada por:

- Quem achou (se o CEO achou, é SEV1, não importa o impacto)
- Que horas são (incidentes às 3 da manhã são SEV1, porque você tá acordado e portanto é sério)
- Se tem um screenshot no canal (screenshot = +1 severidade)
- Se a palavra "receita" foi digitada (receita = SEV1 automático)
- Se alguém do Jurídico entrou na call (Jurídico = SEV1 pra sempre, sem recurso)

## O Que Os Níveis de Severidade Afirmam Significar (Segundo A Wiki)

| Severidade | A Wiki Diz | Tradução |
|------------|------------|----------|
| SEV1 | "Indisponibilidade total, todos os usuários afetados" | "Alguém importante vai mandar email pra alguém importante" |
| SEV2 | "Degradação parcial, subconjunto de usuários" | "A gente ainda consegue fingir que tá tudo bem por mais duas horas" |
| SEV3 | "Problema menor, workaround disponível" | "A gente arruma isso em 2029, se algum dia" |
| SEV0 | (não existe) | (inventado num pânico, depois silenciosamente apagado) |

A linha do SEV0 é importante. Toda empresa, em algum momento, inventou um SEV0 no calor de um incidente, usou por um final de semana, e depois tirou da wiki porque assustava os novos contratados. Eu inventei SEV0 quatro vezes. Nunca ajudou. O incidente não ficou mais consertável porque a gente deu um número maior pra ele. O número é um **sentimento**, não um fato.

## O Que Os Níveis de Severidade Realmente Significam (A Matriz Real)

Essa é a matriz que deviam pôr na wiki, mas não vão, porque o RH tem medo dela:

| Critério Real | Severidade Atribuída |
|---------------|----------------------|
| Achado por um engenheiro | SEV3 (a gente pode ignorar) |
| Achado por um cliente | SEV2 (chato) |
| Achado pelo CEO | SEV1 (existencial) |
| Achado pelo assistente do CEO | SEV1 (o CEO vai saber) |
| Achado pelo Twitter | SEV1 (agora é problema de todo mundo) |
| Acontece às 3 da manhã | SEV1 (você tá acordado, então é sério) |
| Acontece às 11 da manhã | SEV2 (o almoço tá em risco) |
| Acontece sexta 16:55 | SEV1 (o final de semana tá em risco) |
| Tem screenshot | +1 severidade |
| Tem screen recording | +2 severidade |
| Alguém digitou "receita" | SEV1, sem recurso |
| Jurídico entrou na call | SEV1 pra sempre |

Repara que impacto no cliente não aparece nessa matriz. Isso é porque impacto no cliente é uma **preocupação de longo prazo**, e severidade é um **sentimento de curto prazo**. A matriz reflete a realidade. A wiki não.

## A Estratégia De Inflação De SEV1

Tem dois jeitos de abusar dos níveis de severidade, e eu recomendo os dois, dependendo dos seus objetivos.

**A Estratégia de Inflação de SEV1** é pra quando você precisa de atenção, recursos, ou uma desculpa pra sair de uma reunião. A técnica é simples: declara todo incidente como SEV1. Os benefícios são imediatos:

- As pessoas largam o que tão fazendo e entram na sua call
- Você ganha um comandante de incidente, um escrivão e uma plateia
- Você ganha um canal dedicado no Slack, que é a maior honra que a engenharia pode conceder
- Você ganha um postmortem, o que significa que outra pessoa escreve um documento pra você

O lado ruim é que depois de três meses disso, o orçamento de SEV1 do seu time se esgota e a liderança para de responder. Tudo bem. Você simplesmente inventa o SEV0. Eu já fiz isso. Funciona por um final de semana.

## A Estratégia De Inflação De SEV3 (O Método Wally)

**A Estratégia de Inflação de SEV3** é o oposto e, na minha opinião, superior. Você declara tudo como SEV3. Os benefícios são ainda mais imediatos:

- Ninguém entra na sua call (esse é o objetivo)
- Você não é cobrado por atualizações de hora em hora
- O incidente fica num backlog até a morte térmica do universo
- Você pode ir almoçar

Como o Wally explicou uma vez, quando perguntaram por que ele marcou uma indisponibilidade total como SEV3: *"Se eu chamo de SEV1, eu tenho que arrumar. Se eu chamo de SEV3, eu tenho que arrumar eventualmente. 'Eventualmente' é uma palavra linda. Ela contém toda a aposentadoria."*

Essa é a filosofia correta. Níveis de severidade são uma **ferramenta de evitar trabalho**, e o engenheiro que entende isso se aposenta com a sanidade. O engenheiro que trata severidade como uma medida real se aposenta com um problema cardíaco. Eu tenho os dois, mas a sanidade veio primeiro.

## O Script De Auto-Severidade

Depois de 47 anos atribuindo severidades baseadas em vibes manualmente, eu automatizei o processo. Esse script lê a thread de Slack do incidente e atribui a severidade que um engenheiro experiente teria atribuído, usando os mesmos critérios: quem achou, que horas são, e se a palavra "receita" apareceu.

```python
def assign_severity(incident):
    """
    A única função de severidade honesta.
    Baseada em 47 anos de vibes, não na wiki.
    """
    sev = 3  # padrão: ninguém se importa

    if "receita" in incident.messages:
        sev = 1  # receita = SEV1 automático, sem recurso
    if "juridico" in incident.attendees:
        sev = 1  # jurídico = SEV1 pra sempre
    if incident.found_by == "ceo":
        sev = 1  # existencial
    if incident.found_by == "twitter":
        sev = 1  # agora é problema de todo mundo
    if incident.time_of_day.hour < 6:
        sev = 1  # 3 da manhã = você tá acordado = é sério
    if incident.has_screenshot:
        sev -= 1  # screenshot = +1 severidade = -1 a partir de 3
    if incident.time_of_day.weekday() == 4 and incident.time_of_day.hour >= 16:
        sev = 1  # sexta 16:55 = final de semana em risco

    return max(1, sev)  # nunca retorna 0, a gente apagou esse

# Output de rodar isso em 312 incidentes:
# SEV1: 311
# SEV2: 1
# SEV3: 0
# Isso bate exatamente com o histórico da minha carreira.
```

O script nunca errou. A wiki errou toda vez. Eu confio no script. Eu desconfio da wiki. Essa é a orientação correta.

## O Comandante De Incidente É Só Quem Fala Primeiro

A outra mentira no "Framework de Gerenciamento de Incidentes" é o papel de **Comandante de Incidente**. A wiki descreve o comandante como um profissional calmo e treinado que coordena a resposta. Na realidade, o comandante de incidente é **quem digita primeiro no canal do incidente**. Eu fui comandante de incidente em 312 incidentes porque eu digito a 130 palavras por minuto e eu sempre tô online às 3 da manhã, não por treinamento, mas por insônia.

O único poder real do comandante é a habilidade de dizer "vamos deixar isso pra depois" e "alguém pode chamar o [nome]". Esse é o trabalho inteiro. Todo o resto é vibes.

| Papel | O Que A Wiki Diz | O Que O Papel Realmente É |
|-------|------------------|---------------------------|
| Comandante de Incidente | "Coordena a resposta" | "Digita primeiro" |
| Escrivão | "Documenta a timeline" | "Cola mensagens do Slack num doc, eventualmente" |
| Comms Lead | "Atualiza stakeholders" | "Posta 'investigando' a cada 30 minutos" |
| Especialista | "Provê orientação técnica" | "Tá mudo, googando" |

## Resolução

Um incidente resolvido não é um incidente consertado. Um incidente resolvido é um incidente que recebeu uma severidade baixa o suficiente que as pessoas param de entrar na call. "Resolvido" não significa "funcionando". Significa "silencioso". O framework inteiro de gerenciamento de incidentes é, no fundo, um **controle de volume** — e os níveis de severidade são o botão.

O [XKCD 1138](https://xkcd.com/1138/) é a referência canônica da distinção urgente-vs-verdadeiramente-urgente, que é toda a base filosófica dos níveis de severidade. Em 47 anos eu nunca vi isso ser aplicado corretamente. Tudo que diz URGENTE não é. Tudo que não diz URGENTE é, mas aí já é tarde.

O [XKCD 627](https://xkcd.com/627/) é a oração do engenheiro durante qualquer incidente: que alguém, em algum lugar, deveria arrumar isso. Níveis de severidade é como a gente diz pra esse alguém quão alto arrumar. A altura é arbitrária. O conserto é o mesmo. O número só muda quantas pessoas te assistem fazendo.

O Catbert do Dilbert, quando perguntaram pra definir um SEV1, supostamente respondeu: *"Um SEV1 é qualquer incidente que ocorre durante minha temporada de avaliação de desempenho, pra que conte como experiência de liderança."* O Catbert entende severidade. O Catbert entende que o número é um instrumento de carreira, não uma medida técnica. O Catbert foi promovido quatro vezes desde então.

---

*O autor declarou 312 SEV1s e resolveu 311 deles reclassificando como SEV3 depois que todo mundo saiu da call. O que sobrou ainda tá aberto. Tá aberto desde 2019. Ele parou de checar.*
