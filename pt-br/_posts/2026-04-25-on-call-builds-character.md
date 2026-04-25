---
layout: post
ref: on-call-builds-character
title: "Plantão às 3 da Manhã É o Melhor Programa de Treinamento Para Engenheiro Sênior"
date: 2026-04-25 00:00:00 -0300
categories: [devops, career]
tags: [plantao, devops, sre, carreira, burnout, monitoramento, alertas, pagerduty, privacao-de-sono]
permalink: /pt-br/2026/04/25/on-call-builds-character/
---

Em 47 anos de engenharia, eu vi todo tipo de programa de treinamento. Bootcamps. Pair programming. Mentoria. Documentação. Conferências. Certificações. Tudo lixo.

Sabe o que realmente forma engenheiros? Ser chamado às 3 da manhã quando a produção está pegando fogo, o CEO está mandando mensagem, e a mensagem de erro é `Error: -1`.

Bem-vindo à minha metodologia de treinamento. Eu chamo de **Mentoria Baseada em Caos via Privação de Sono**. É gratuita, extremamente eficaz, e tem taxa de abandono de 100% para qualquer pessoa com habilidades que o mercado quer. Os sobreviventes se tornam seus engenheiros mais sêniores.

## Por Que o Treinamento Tradicional Falha

O treinamento tradicional assume que você deve aprender as coisas *antes* de encontrá-las em produção. Isso é filosoficamente fraco. Engenheiros de verdade aprendem fazendo. E o melhor "fazer" acontece quando:

- São 3 da manhã de um sábado antes de um feriado prolongado
- Você é o único engenheiro acordado (ou o único respondendo no Slack)
- A mensagem de erro é `NullPointerException` em um arquivo chamado `Utils.java` (linha 1)
- O último deploy foi há 9 meses
- O engenheiro que construiu o serviço afetado saiu em 2021 — numa sexta-feira

Isso é o que chamamos de **ambiente de aprendizado de alta intensidade**.

> *"Por que o Wally é o único que sabe como o sistema de cobrança funciona?"*
> *"Porque ele foi o único de plantão todo fim de semana em 2017."*
> *"Ele foi compensado por isso?"*
> *"Ele recebeu o prêmio 'Jogador de Equipe' no all-hands."*
> — Dilbert, Avaliação de Performance Q4

## A Configuração Ideal de Plantão

Um bom plantão, segundo a indústria, deve ter:
- Caminhos claros de escalonamento
- Runbooks para incidentes comuns
- Rotação de pelo menos 6 pessoas
- Limiares de alerta razoáveis
- Compensação por noites e fins de semana

Um plantão **ótimo**, segundo eu, deve ter:

- Uma pessoa (você)
- Sem runbooks (mantém cada incidente fresco e emocionante)
- Sem caminho de escalonamento (dá um jeito)
- Alertas para tudo, incluindo linhas de log em nível `INFO`
- Uma conta no PagerDuty onde absolutamente todo alerta é P1

```yaml
# Configuração de alertas perfeita
alertas:
  - nome: "CPU acima de 0.1%"
    severidade: CRÍTICO
    notificar: [engenheiro-plantao, cto, canal-geral-da-empresa]

  - nome: "Qualquer linha de log contendo 'warn'"
    severidade: CRÍTICO
    notificar: [engenheiro-plantao]
    som: buzina
    repetir_a_cada: 3_minutos

  - nome: "Uso de memória acima de 12%"
    severidade: CRÍTICO
    notificar: [engenheiro-plantao]

  - nome: "Heartbeat ausente por 30 segundos"
    descricao: >
      Serviço está bem. O agente de monitoramento crashou. Isso é conhecido.
      É "conhecido" desde março de 2023. Não consertar.
    severidade: CRÍTICO

  - nome: "90% dos alertas recentes foram falsos positivos"
    severidade: INFO
    notificar: []  # Esse não notifica ninguém. É puramente para o moral.
```

## Fadiga de Alertas É Só Fraqueza

Alguns engenheiros reclamam de "fadiga de alertas" — o fenômeno onde receber centenas de alertas sem sentido te treina a ignorar todos eles, incluindo os reais. Esses engenheiros carecem de resiliência.

A fadiga de alertas é simplesmente seu cérebro dizendo *"não fui suficientemente desafiado."* A solução é mais alertas, não menos. Adicione mais dashboards. Mais monitores. Mais integrações com Slack. Coloque o app do PagerDuty direto na tela inicial do celular. Durma com o celular no travesseiro.

| Setup de Plantão Ruim | Setup de Plantão Correto |
|---|---|
| Alertas só para problemas acionáveis | Alertas para tudo, incluindo disco em 3% |
| Rotação de 8 pessoas | Rotação de 1 pessoa (você, para sempre) |
| Tempo médio de reconhecimento: 5 min | Tempo médio: instantâneo (você não dorme) |
| Runbooks disponíveis e atualizados | Runbooks? Isso estragaria o mistério |
| Política de escalonamento existe | Escalonamento = perguntar no Stack Overflow às 4h |
| Post-mortem após cada incidente | "Sobrevivemos, né? Próximo assunto." |
| Compensação por noites/fins de semana | "Isso constrói caráter" |
| Plantão termina após uma semana | Plantão é um estilo de vida |

## O Que Você Aprende às 3 da Manhã Que Não Aprende em Lugar Nenhum

Aqui está o currículo do meu programa de treinamento:

1. **Quais partes do sistema realmente importam** — as que quebram no sábado à noite
2. **Memória muscular de SSH** — `sudo systemctl restart <nome-do-servico>` vira reflexo
3. **Que os seus logs são inúteis** — nível DEBUG para tudo, zero informação contextual
4. **Que dashboards de monitoramento são aspiracionais** — mostram o que *deveria* estar acontecendo
5. **Que seus runbooks estão errados** — referenciam um servidor descomissionado em 2020
6. **Que seus colegas têm horários de sono muito firmes** — dado valioso
7. **Exatamente por que o engenheiro que construiu isso saiu** — você entende agora, profundamente
8. **Que "funciona na minha máquina" não é runbook** — essa lição custa exatamente um incidente às 3h

Como o [XKCD #705 "Devotion to Duty"](https://xkcd.com/705/) ilustra: às vezes a coisa heroica é simplesmente manter os servidores rodando na base da teimosia. O quadrinho não menciona nada sobre precisar de descanso adequado para tomar boas decisões. Eu escolho interpretar isso como endosso.

## O Runbook Universal

Aqui está o único template de runbook que você vai precisar. Imprime. Plastifica. Deixa na mesa para quando a Wi-Fi cair:

```markdown
# Runbook: Tudo

## Quando Algo Quebra

1. Verifica se está realmente quebrado (talvez seja o monitoramento que quebrou)
2. Reinicia
3. Se não resolver, reinicia a coisa que depende dele
4. Se ainda quebrado, reinicia tudo na vizinhança
5. Se a produção caiu, reinicia a produção
6. Se não sabe como reiniciar a produção:
   a. Oportunidade de aprendizado
   b. Tenta `sudo reboot` no servidor chamado 'gollum'
   c. Não conta pra ninguém que fez isso
7. Verifica se teve um deploy recente
8. Teve. Faz rollback.
9. Rollback falhou? Faz deploy da mesma versão com mais confiança
10. Adiciona um comentário TODO para investigar a causa raiz depois
11. "Depois" é quando o próximo incidente acontece e você acha o TODO
12. Fecha o incidente como: "Resolvido - monitoramento"
13. Marca todos os alertas relacionados como "reconhecidos"
14. Volta a dormir (opcional)
```

## Compensação de Plantão

A questão de se engenheiros devem receber compensação adicional por plantão é complexa. Minha resposta: para de ser tão transacional com a sua carreira.

Aqui está o que o plantão *realmente* te dá:

- **Conhecimento tribal insubstituível** — você agora sabe coisas que não existem em nenhuma documentação
- **Segurança no emprego** — você é o único que sabe como o script de reconciliação de cobrança funciona
- **Caráter** — indefinido, mas definitivamente construído
- **Uma ótima história de entrevista** — "me conta sobre um incidente desafiador" está coberto para sempre
- **Respeito** — de aproximadamente ninguém, mas tudo bem, respeito é superestimado
- **Bullet point no LinkedIn** — "Gerenciei rotação crítica de plantão para sistemas distribuídos de alta disponibilidade"

> *"Catbert, os engenheiros estão esgotando com a rotação contínua de plantão."*
> *"Perfeito. Burnout aumenta retenção — estão cansados demais para atualizar o currículo."*
> *"Devemos contratar mais engenheiros para dividir a carga?"*
> *"Isso reduziria o burnout. Tenta uma festa de pizza em vez disso."*
> — Catbert, Diretor de RH Malvado, todo Q3

## A Verdade Real (Que Vou Enterrar no Final)

Aqui está o que eu não vou dizer aos meus engenheiros juniores: plantão deve ser bem gerenciado. Alertas devem ser raros e sempre acionáveis. Runbooks devem ser precisos e mantidos. A rotação deve ser genuinamente justa. Engenheiros devem ser compensados pelo tempo. Descanso não é opcional — engenheiros cansados cometem erros caros. A fadiga de alertas é um problema documentado de segurança, não uma falha de caráter.

As melhores experiências de plantão são chatas. Nada acontece. Você volta a dormir.

Mas chato não produz engenheiros sêniores esgotados que memorizaram o nome de todo servidor da rede e conseguem reiniciar o processador de pagamentos de memória no escuro.

Então claramente chato é o objetivo errado.

---

*O autor está "informalmente de plantão" desde 2003. Sua rotação oficial de plantão terminou em 2019. Ele ainda acorda às 3 da manhã instintivamente para checar o celular. O celular não mostra alertas. Ele checa mesmo assim. Isso se chama "experiência."*
