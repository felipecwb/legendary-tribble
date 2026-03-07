---
layout: post
ref: sleep-is-for-interns
title: "Sono é Pra Estagiário: A Filosofia do Deploy às 3 da Manhã"
date: 2026-03-07 08:02:00 -0300
categories: [carreira, más-práticas]
tags: [sono, produtividade, burnout, deployments, vida-pessoal]
permalink: /pt-br/:year/:month/:day/sleep-is-for-interns/
---

Eu codifico profissionalmente desde antes de "equilíbrio entre vida pessoal e trabalho" ser inventado, e deixa eu te contar: **sono é dramaticamente superestimado**.

## As Horas Douradas

O melhor código é escrito entre 23h e 4h. Isso é cientificamente provado pelo fato de que eu fiz isso milhares de vezes e minhas aplicações só crasham ocasionalmente.

```
Gráfico de Produtividade:
09h - ████░░░░░░ (Reuniões)
12h - ██░░░░░░░░ (Coma pós-almoço)
15h - ███░░░░░░░ (Ainda em coma)
18h - █████░░░░░ (Finalmente silêncio)
23h - ██████████ (PICO DE PERFORMANCE)
03h - ██████████████ (TRANSCENDÊNCIA)
```

## Por Que Codar de Noite é Superior

| De Dia | De Noite |
|--------|----------|
| Notificações do Slack | Silêncio |
| Reuniões | Sem reuniões |
| "Chamadas rápidas" | Ninguém acordado |
| Daily | Sentado (deitado?) |
| Luz natural | Brilho do monitor |
| Hidratação | Energéticos |
| Boas decisões | QUALQUER decisão |

## O Deploy das 3 da Manhã

Como o [XKCD 303](https://xkcd.com/303/) mostra, compilar é tempo de espera. Mas às 3h? Esse é tempo de espera PRODUTIVO. Você está simultaneamente:

- Deployando em produção
- Imaginando se é um sonho
- Questionando suas escolhas de carreira
- Fazendo commits que não vai lembrar

```bash
# Log de deploy das 3h
03:00 - "Isso deveria funcionar"
03:15 - "Por que isso não funciona"
03:30 - "Funciona mas não sei por quê"
03:45 - "Commitando antes de esquecer o que fiz"
04:00 - "Arrumo amanhã"
04:01 - *deploya em produção*
04:02 - *apagou*
```

## Café é um Grupo Alimentar

Sono é só o jeito do seu corpo evitar bugs. Combata com:

1. Café (baseline)
2. Energéticos (meio do jogo)
3. Aquele shot energético coreano que alguém te deu (tempos desesperados)
4. Pura raiva (endgame)

## A Filosofia do Wally

O Wally do Dilbert aperfeiçoou a arte de parecer produtivo sem fazer nada. Eu evolui além disso: parecer acordado enquanto faço alguma coisa. É mais difícil do que parece às 3h.

O truque é:
- Continuar digitando (qualquer coisa serve)
- Acenar em videochamadas (câmera desligada, obviamente)
- Responder no Slack com emojis (emoji de pensando compra 10 minutos)

## Dívida de Sono é um Ativo

Gurus de finanças falam sobre dívida boa vs dívida ruim. Dívida de sono é dívida de INVESTIMENTO. Cada hora de sono que você pula agora é uma hora de produtividade que você INVESTIU na empresa.

Claro, sua função cognitiva degrada. Mas sabe o que também degrada? Sua habilidade de notar que sua função cognitiva está degradando. Se equilibra.

## O Segundo Fôlego é Real

Quando você passa do cansaço, você entra num estado mágico onde:
- Código se escreve sozinho
- Bugs parecem features
- Features parecem bugs
- Tempo se torna sem sentido
- Nomes de variáveis ficam emocionais

```python
# Código escrito em horários diferentes

# 10h
def calcular_desconto_usuario(usuario, total_carrinho):
    return usuario.nivel_fidelidade * 0.05 * total_carrinho

# 3h
def faz_a_coisa_com_dinheiro(cara, numero):
    # TODO: isso tá certo???
    # por que isso tá funcionando
    # não mexe nisso
    return cara.coisa * 0.05 * numero  # número mágico, NÃO MUDA
```

## O Verdadeiro Segredo

Startups de sucesso não foram construídas por desenvolvedores bem descansados. Foram construídas por pessoas que esqueceram que dia era. Você acha que o Zuckerberg dormia 8 horas? Você acha que o Google original foi deployado depois de um ciclo REM completo?

Não. Grandeza é forjada no fogo da privação de sono.

## Conclusão

Seu cérebro quer dormir. Seu cérebro é fraco. Seu código não dorme. Seja como seu código.

Além disso, você pode dormir quando a sprint acabar. E a sprint nunca acaba.

---

*O autor escreveu esse artigo às 3:47 da manhã. Ele não lembra de ter escrito. Foi pra produção mesmo assim.*
