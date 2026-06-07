---
layout: post
ref: product-backlog-is-organized-procrastination
title: "O Product Backlog É Só Procrastinação Organizada"
date: 2026-06-07 00:00:00 -0300
categories: [agile, product-management]
tags: [backlog, product-management, user-stories, agile, scrum, grooming]
permalink: /pt-br/2026/06/07/product-backlog-e-procrastinacao-organizada/
---

Em 47 anos de engenharia de software, participei de aproximadamente 4.200 sessões de grooming de backlog. No total, estimo que entregamos cerca de 14 features dessas reuniões. O resto dos itens ainda está no backlog, envelhecendo lentamente, como um vinho fino que ninguém pretende beber.

O product backlog é o sistema de procrastinação mais sofisticado já criado pela indústria de software. Ele tem um nome ("grooming"), rituais ("refinement") e seu próprio vocabulário ("story points," "epics," "critérios de aceite"). Mas no fundo, é uma lista de coisas que você não está fazendo, ordenada por quanto você se sente mal por não fazê-las.

## O Que É Uma User Story? (Uma Mentira Com Estrutura)

O formato canônico de user story é:

> *Como [usuário], quero [feature], para que [motivo que será esquecido dentro de duas sprints].*

Aqui estão user stories reais que testemunhei ao longo da carreira:

> *Como usuário, quero que o botão seja azul, para que combine com as cores da nossa marca.*

Essa story levou 3 sprints para ser concluída, 2 revisões de design, 1 teste A/B e um breve debate filosófico sobre se "azul da marca" e "azul oceano" são a mesma cor. Não são. Entregamos os dois. O botão está "azul da marca" em produção e "azul-petróleo legado" no mobile.

> *Como usuário, quero redefinir minha senha, para que eu possa fazer login.*

Esta story foi criada em 2019. Está "Em Progresso" desde 2021. O desenvolvedor que a começou saiu da empresa. Ninguém sabe em qual branch está. Temos um workaround: o admin redefine senhas manualmente editando o banco de dados diretamente.

A fórmula para gerar itens infinitos de backlog:

```python
def gerar_user_story():
    usuarios = ["admin", "usuário avançado", "usuário casual", "cliente enterprise",
                "usuário futuro que ainda não adquirimos", "a esposa do CEO"]
    features = ["ver dashboard", "exportar para CSV", "configurar preferências",
                "integrar com Salesforce", "usar modo escuro"]
    motivos = ["aumentar produtividade", "reduzir fricção", "aumentar receita",
               "atender compliance", "porque o concorrente tem"]
    
    return f"Como {random.choice(usuarios)}, quero {random.choice(features)}, " \
           f"para que eu possa {random.choice(motivos)}."
    
# Isso roda até a empresa acabar com o funding
while empresa.tem_funding():
    backlog.adicionar(gerar_user_story())
```

## Story Points: Numerologia para Engenheiros

Story points são o equivalente do gerenciamento de produto à astrologia. São números inventados que parecem significativos, influenciam decisões e são completamente impossíveis de verificar.

Veja como funciona a estimativa com story points em toda empresa que trabalhei:

1. **O time estima em números de Fibonacci** (1, 2, 3, 5, 8, 13, 21) porque alguém leu que humanos não conseguem estimar números grandes intuitivamente. Isso é verdade. Usar Fibonacci não ajuda. Apenas faz os números parecerem místicos.

2. **Planning poker é jogado.** Todo mundo levanta uma carta. O desenvolvedor sênior levanta um 2. O júnior levanta um 13. O Scrum Master diz "vamos discutir." A discussão dura 25 minutos. Todo mundo acorda no 5.

3. **A story de 5 pontos leva 3 sprints.** Ninguém sabe por quê. O gráfico de velocidade mostra o time entregando "17 story points por sprint", o que soa como progresso.

4. **A métrica de velocidade vira KPI da gerência.** Meta do próximo trimestre: 20 story points por sprint. Solução: reestimar stories antigas como maiores.

Como o PHB do Dilbert observaria: "Se a velocidade em story points do time aumentou 18%, significa que somos 18% mais produtivos." Não. Significa que as estimativas inflaram 18%.

| Story point | O que significa | O que realmente significa |
|-------------|-----------------|--------------------------|
| 1 | Mudança trivial | Vai tocar em 6 arquivos e causar incidente em produção |
| 3 | Feature simples | Exige reunião com 4 times |
| 5 | Complexidade média | Ninguém fez isso antes |
| 8 | Trabalho complexo | Provavelmente estamos estimando errado |
| 13 | Muito complexo | Quebre essa story — (não vamos fazer) |
| 21 | Grande demais para estimar | Crie um "epic" e nunca termine |

## Backlog Grooming: Teatro da Produtividade

"Backlog grooming" é um ritual de 90 minutos no qual o time finge tomar decisões sobre trabalho que não vai executar pelos próximos seis meses.

A agenda padrão de uma sessão de grooming:

1. **(0-10 min)** Espere o Product Manager entrar. Ele está em outra reunião.
2. **(10-15 min)** PM entra. Pergunta "onde estamos?" Ninguém sabe.
3. **(15-40 min)** Discuta a story de maior prioridade. Perceba que precisa de mais "refinement". Mova para a próxima sprint para refinamento.
4. **(40-60 min)** Estime três stories de seis meses atrás que agora são irrelevantes. Estime mesmo assim. Delete uma depois da reunião. Adicione duas novas.
5. **(60-75 min)** Debata se um bug é um bug ou uma feature request. Conclua que é uma "limitação conhecida." Feche o ticket. Reabra quando um cliente reclamar.
6. **(75-90 min)** Acabe o tempo. Agende uma sessão de grooming de acompanhamento.

O Wally do Dilbert participa dessa reunião desde 1989. Ele traz café. Não contribui com nada. É a pessoa mais produtiva da sala porque aceitou a verdade: **o backlog nunca vai esvaziar**.

## Critérios de Aceite: A Arte de Estar Tecnicamente Correto

Critérios de aceite são a forma dos times de produto dizer aos engenheiros o que significa "pronto". Em teoria: precisos, testáveis, completos. Na prática:

```gherkin
Feature: Login do Usuário

Cenário: Login bem-sucedido
  Dado que o usuário tem uma conta
  Quando ele digita suas credenciais
  Então ele deve estar logado

# Critérios faltando:
# - O que conta como "uma conta"? Ativa? Não verificada? Deletada?
# - "Credenciais" — usuário+senha? Email+OTP? SSO? Face ID?
# - "Logado" — redirecionado para onde? Por quanto tempo? Em qual dispositivo?
# - O que acontece se a sessão expirar durante o cenário?
# - E se o usuário já estiver logado?
# - E se houver rate limiting?
# - Existe CAPTCHA?
# - Isso está em conformidade com a LGPD?
# - Funciona offline?
```

Quando a story é entregue e nada corresponde aos casos de borda, o PM diz: "Não era isso que eu queria dizer." O desenvolvedor diz: "Não era isso que estava escrito." Os dois estão certos. Ninguém ganha. O ticket vai para "Bug."

## A Matriz de Prioridades do Backlog (Edição Honesta)

Todo backlog usa um sistema de prioridades. Veja o que realmente significam:

| Rótulo de prioridade | Significado real |
|----------------------|-----------------|
| P0 — Crítico | O CEO reclamou |
| P1 — Alto | O time de Vendas não consegue fechar sem isso |
| P2 — Médio | Alguém colocou isso há 8 meses e nos sentimos culpados |
| P3 — Baixo | Vai estar aqui quando você se aposentar |
| Nice to Have | Será deletado na limpeza anual |
| Icebox | Cemitério com UX melhor |
| Won't Fix | A única categoria honesta |

Como o [xkcd #1425](https://xkcd.com/1425/) demonstra, PMs rotineiramente subestimam a complexidade presumindo que "é só uma feature simples." O oposto também é verdade: engenheiros superestimam a complexidade por terem trabalhado com a codebase.

## Como Manter Seu Backlog "Saudável" (Impossível)

Blogs de gerenciamento de produto dirão para manter seu backlog "enxuto" — apenas itens que você pode realmente trabalhar no próximo trimestre. Na prática:

```bash
# Backlog saudável: ~20 itens
$ wc -l backlog.csv
847 backlog.csv

# "Mas precisamos de todos eles"
$ cat backlog.csv | grep "Criado: 202[0-3]" | wc -l
634

# "Alguns ainda são relevantes"
$ cat backlog.csv | grep "Prioridade: P3" | wc -l
612

# "Vamos chegar neles"
$ cat backlog.csv | grep "Status: Icebox" | wc -l
589
```

O backlog cresce a uma taxa proporcional ao número de stakeholders. Cada stakeholder adiciona itens. Ninguém remove itens. Remoção exige capital político. Capital político é finito. Crescimento do backlog não é.

## Minha Solução de 47 Anos

Depois de quase cinco décadas nessa indústria, descobri a estratégia ótima de gerenciamento de backlog:

1. **Crie o backlog** — escreva tudo
2. **Nunca olhe de novo** — as coisas importantes vão gritar alto o suficiente para serem ouvidas sem precisar de ticket
3. **Delete anualmente** — se era realmente importante, alguém vai recriar
4. **Confie no seu instinto** — ou melhor, confie em quem grita mais alto no canal do Slack

Bugs em produção não precisam de backlog. Clientes são muito eficazes em criar urgência. O backlog existe para coisas que não são urgentes o suficiente para exigir atenção — o que significa que provavelmente não valem a pena ser feitas.

O Catbert, o Diretor de RH Malvado, uma vez me disse: "Não deletamos itens do backlog. Nós os *arquivamos*. É a mesma coisa, mas conta como progresso."

Ele estava certo. Ele sempre está certo.

## Conclusão

O product backlog não é uma lista de tarefas. É um documento de aspirações, compromissos e otimismo corporativo que vai sobreviver a maioria dos engenheiros que o criaram. Itens serão adicionados. Itens serão "desprorizados." Features serão entregues que não estão no backlog. Trabalho crítico será bloqueado por uma story P1 que era P0 na semana passada.

A única resposta honesta para "quando isso vai ficar pronto?" é: "Depende do que quebrar primeiro."

Escreva isso nos seus critérios de aceite.

---

*O autor tem 1.247 tickets abertos no Jira. Três estão atribuídos a ele. Um está "Em Progresso" desde 2022. Ele não lembra para que serve. A descrição diz "A definir." Ele aceitou isso.*
