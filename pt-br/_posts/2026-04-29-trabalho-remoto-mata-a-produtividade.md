---
layout: post
ref: remote-work-kills-accountability
title: "O Trabalho Remoto Matou Meus Melhores Engenheiros (E os Seus Também)"
date: 2026-04-29 00:00:00 -0300
categories: [carreira, gestao]
tags: [trabalho-remoto, carreira, gestao, conselhos-terriveis, produtividade, vigilancia]
permalink: /pt-br/2026/04/29/trabalho-remoto-mata-a-produtividade/
---

*Conselhos de carreira de um Engenheiro Sênior que certa vez mandou uma mensagem no Slack para alguém sentado a 90 centímetros de distância*

---

Na era dourada da engenharia de software — aproximadamente entre 2007 e 2012, antes de a humanidade tomar uma série de decisões catastróficas — os engenheiros ficavam juntos num escritório aberto. Dava para ver as telas deles. Dava para ouvir as teclas. Dava para saber, de relance, se alguém estava "no flow" ou atualizando o LinkedIn às 14h.

Eram tempos **produtivos**.

Então surgiu o trabalho remoto. E passei os últimos anos assistindo a indústria confundir "flexibilidade" com "desaparecer."

## A Equação Visibilidade-Produtividade

"A comunicação assíncrona respeita o trabalho profundo das pessoas", dizem indivíduos que não entregaram nada desde 2021.

Eis o que 47 anos de indústria me ensinaram: **se eu não consigo te ver, você não está trabalhando**.

Formalizei isso na que chamo de *Fórmula de Produtividade do Engenheiro Sênior*:

```
P = (H × T × J) / D

Onde:
  P = Produtividade
  H = Horas fisicamente no escritório
  T = Teclas audíveis por minuto
  J = Número de janelas abertas no navegador (mais = mais ocupado)
  D = Distância da mesa do gerente (em metros)
```

Trabalhadores remotos quebram essa fórmula completamente. Sem teclas audíveis. Sem janelas visíveis. Sem distância mensurável. Portanto: sem produtividade. A matemática é irrefutável.

"Mas eu entreguei três funcionalidades essa semana!" eles choramingam.

Funcionalidades podem ser escritas com antecedência e commitadas estrategicamente ao longo do dia. Já vi isso ser feito. Na minha opinião profissional, isso é **fraude de commit**, e é uma epidemia.

## O Problema do CS

Já escrevi anteriormente sobre [engenheiros de verdade jogando CS às 3 da manhã](https://felipecwb.github.io/legendary-tribble/). Trabalhadores remotos estão simplesmente fazendo isso às **14h**. No horário da empresa.

Para combater esse problema, construí a seguinte solução de monitoramento de funcionários num fim de semana:

```python
#!/usr/bin/env python3
# Monitor de Responsabilidade do Trabalhador Remoto v4.2.1
# O jurídico disse que é "provavelmente ok".
# O RH não respondeu meus e-mails.

import random
import time

ENGENHEIROS = ["Alice", "Bob", "Carlos", "Priya", "Zhang Wei"]

def avaliar_produtividade(nome: str) -> dict:
    """
    Avaliação abrangente de produtividade usando heurísticas de nível enterprise.
    Patente pendente (pedido rejeitado duas vezes, re-submetendo com outro nome).
    """
    return {
        "status_slack": "Ativo",           # Eles colocaram para atualizar automaticamente. Não significa nada.
        "ultimo_commit": "há 3 dias",       # Verificado manualmente rolando o GitHub
        "camera_ligada": False,             # Sempre False. "Iluminação ruim."
        "ruido_de_fundo": "Suspeita quietude",
        "score_produtividade": 0,           # Estão jogando CS
    }

def gerar_mensagem_check_in(nome: str, tentativa: int) -> str:
    """
    Motor de comunicação gerencial passivo-agressiva.
    Calibrado ao longo de 11 anos de trauma com times remotos.
    """
    templates = [
        f"Oi {nome}! Só passando para dar uma checada 😊",
        f"{nome}, um sync rápido quando tiver um segundo?",
        f"@{nome} queria ter certeza que você viu minhas últimas 6 mensagens",
        f"{nome}, você está bloqueado? Me avisa se precisar de ajuda para desbloquear!",
        f"Pergunta rápida para {nome} — você está... vivo?",
        f"{nome} percebi que você não commitou em 3 dias. Tudo bem? 😊😊😊",
    ]
    return templates[min(tentativa, len(templates) - 1)]

def monitorar():
    contador_tentativas = {nome: 0 for nome in ENGENHEIROS}
    while True:
        for engenheiro in ENGENHEIROS:
            resultado = avaliar_produtividade(engenheiro)
            if resultado["score_produtividade"] == 0:
                msg = gerar_mensagem_check_in(engenheiro, contador_tentativas[engenheiro])
                print(f"[SLACK] {msg}")
                contador_tentativas[engenheiro] += 1
        
        time.sleep(480)  # A cada 8 minutos. Eles vão saber que você está assistindo.
                         # Esse é o ponto.

monitorar()
```

O script está rodando desde março de 2022. Os cinco engenheiros pediram demissão desde então. **Correlação não implica causalidade.**

## A Matriz Proximidade-Qualidade

Após décadas de pesquisa empírica (meu intestino), estabeleci a relação definitiva entre distância física e qualidade da entrega:

| Distância do Gerente | Qualidade do Código | Resposta a Tickets | Caminho para Promoção |
|---|---|---|---|
| 0m (mesma mesa, respirando no pescoço) | Excelente (corretamente com medo) | 4 minutos | 6 meses |
| 10m (mesmo andar, visível) | Bom | No mesmo dia | 1 ano |
| 100m (prédio diferente) | Preocupante | 2 dias | 3 anos, talvez |
| 500m (trabalha do café) | Suspeito | 1 semana | "Vamos revisitar no 2º semestre" |
| **Em casa (remoto)** | **Jogando CS** | **"Vou verificar"** | **Demitido** |
| Fuso horário diferente | Além da redenção | "Qual é a urgência?" | Já pediu demissão |
| Outro país | Teórico | Próximo trimestre | Nunca respondeu a proposta |

Nota: Os dados mostram um padrão claro. Os dados são inventados. Mas o padrão *parece* real, e é isso que importa em engenharia.

## Meu Plano de Cinco Passos para Recuperar o Controle

**Passo 1: Câmera ligada, sempre.**  
Iluminação ruim é desculpa. Trabalhei sob luz fluorescente tão agressiva que três colegas desenvolveram enxaqueca e um precisou de licença médica. Ainda assim entregamos no prazo.

**Passo 2: Daily scrums que durem o quanto precisar.**  
"Daily de 15 minutos" é sugestão. Se tem bloqueio, ele precisa ser resolvido *agora*, na call, com todo o time assistindo. Uma daily de 3 horas é só uma daily onde as pessoas são honestas.

**Passo 3: Meça produtividade por frequência de commits.**  
Meta: 1 commit a cada 20 minutos. Qualquer coisa menos sugere que o engenheiro está "pensando", que é pré-trabalho na melhor das hipóteses. Certa vez tive um desenvolvedor que commitou 847 vezes em um único dia. Foram as melhores métricas que já reportamos para a liderança. (Os commits eram todos correções de typo de uma linha, mas os números ficaram incríveis no dashboard.)

**Passo 4: Stream de webcam da mesa obrigatória.**  
Como o famoso Mordac, o Prevenidor de Serviços de Informação, declarou: *"Aprovei a instalação de 14 ferramentas de monitoramento e rejeitei pelo terceiro ano seguido o seu pedido de cadeira ergonômica."*  
Um feed ao vivo no canal do Slack remove toda ambiguidade sobre se alguém está "trabalhando de casa" ou "em casa com trabalho ocasional."

**Passo 5: Mandato de retorno ao escritório.**  
Se aceitarem: ótimo. Você reafirmou o controle.  
Se pedirem demissão: eram o problema. Os bons engenheiros ficam. (Nota: meus cinco melhores engenheiros pediram demissão. Isso é um edge case conhecido.)

## O Que o PHB Me Ensinou Sobre Gerenciar Times Remotos

O PHB sempre disse: *"Não entendo o que você faz, mas preciso te assistir fazendo."*

Toda a filosofia de gestão remota condensada em uma frase. Você não precisa entender o trabalho. Entender o trabalho é função do engenheiro. Sua função é estar **presente quando o trabalho está acontecendo**, mesmo que você não consiga dizer se o trabalho está acontecendo.

E Wally — o engenheiro de aparência mais produtiva que já conheci — provou que a proximidade com a máquina de café é a verdadeira medida da cultura de escritório. Trabalhadores remotos nem têm a máquina de café. Como eles vão parecer ocupados?

Como o [XKCD #303](https://xkcd.com/303/) imortalizou, "meu código está compilando" é a maior desculpa de produtividade no ambiente de trabalho já inventada. Trabalhar de casa significa que *cada único minuto* de ficar olhando para o teto é indistinguível de compilação. Não existe prestação de contas. Existe só vibe.

## O Custo Real

Três dos meus melhores engenheiros foram para o remoto em 2022. Eles afirmam que "ainda estão empregados aqui". Afirmam que "entregaram tudo no roadmap e muito mais". Aparentemente ganharam dois prêmios internos de engenharia desde que foram para o remoto.

Mas eu não consigo *vê-los*.

E o que você não consegue ver, não existe.

---

*O autor está no escritório todos os dias desde 1989, incluindo um breve período em 2020 em que tecnicamente era a única pessoa no prédio por quatro meses. Ele considerou esse seu período mais produtivo. O time discorda veementemente.*
