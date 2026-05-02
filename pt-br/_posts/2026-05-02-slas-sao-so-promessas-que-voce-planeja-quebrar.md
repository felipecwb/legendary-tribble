---
layout: post
ref: slas-are-just-promises-you-plan-to-break
title: "SLAs São Só Promessas Que Você Planeja Quebrar"
date: 2026-05-02 00:00:00 -0300
categories: [arquitetura, devops, gestão]
tags: [sla, slo, uptime, confiabilidade, sre, incident-response, monitoramento, carreira]
permalink: /pt-br/2026/05/02/slas-sao-so-promessas-que-voce-planeja-quebrar/
---

Escrevo software desde antes da internet ter páginas web. Naquela época, um SLA era simples: *"Funciona quando estou no escritório."* Ninguém fazia perguntas. Ninguém abria tickets. Se o sistema caia às 3 da manhã, era problema de Deus, não meu.

Agora temos SLAs, SLOs, SLIs, error budgets, burn rates, métricas de toil, e uma indústria inteira de pessoas que ganham para medir o quão quebradas as coisas estão em vez de consertá-las. Progresso.

Vou te contar como Acordos de Nível de Serviço funcionam na prática, da perspectiva de alguém que violou aproximadamente 340 deles.

---

## O Que é um SLA de Verdade

Um SLA é um documento que diz: *"Prometemos que o sistema vai funcionar X% do tempo, e se não funcionar, faremos algo vago como compensação."*

A percepção-chave: **a compensação é sempre menor do que a dor causada**. Ninguém nunca faliu por causa de créditos de SLA. Mas muitos engenheiros ficaram carecas tentando honrá-los.

Aqui está uma tabela que compilei ao longo de 47 anos ignorando acordos:

| O SLA Diz | O Que Significa | O Que Realmente Acontece |
|-----------|-----------------|--------------------------|
| 99,9% de uptime | Down 8,7h/ano | Down durante toda demonstração |
| 99,99% de uptime | Down 52 min/ano | Down por 52 minutos na Black Friday |
| 99,999% de uptime | Down 5 min/ano | Down por 5 minutos no pitch do CEO para investidores |
| 100% de uptime | Você foi enganado | Você definitivamente foi enganado |

A matemática funciona. O elemento humano não.

---

## Error Budgets: Permissões para Falhar

O movimento SRE moderno inventou algo chamado "error budget." É a quantidade de downtime que você *tem permissão* antes das pessoas começarem a fazer perguntas.

Quero deixar claro: **inventamos um orçamento para falhas**.

Em contabilidade, orçamentos existem para coisas que você quer gastar com cuidado. Comida. Marketing. P&D. Agora aplicamos esse framework a *quebrar a produção*. O Wally dos quadrinhos do Dilbert ficaria orgulhoso — ele passou 30 anos descobrindo como não fazer nada e receber por isso. Nós passamos 30 anos descobrindo como quebrar coisas e apresentar isso como uma feature.

A consequência natural: engenheiros olham para um error budget e pensam: *"Não usamos nossa cota de downtime neste trimestre. Deixa eu fazer esse deploy arriscado antes de dezembro."*

Você construiu um sistema que incentiva o caos controlado. Parabéns pelo seu Six Sigma.

---

## SLOs: SLAs Para Quem Está Ocupado Demais Para Ler SLAs

Um SLO é um SLA "interno" que seu time define para si mesmo. Ninguém vai te processar se você não cumprir. Seu gerente vai mandar uma mensagem decepcionada no Slack, o que é pior.

O que ninguém te conta: **SLOs são ficção aspiracional**.

A maneira correta de definir um SLO é:
1. Olhe para o uptime atual
2. Subtraia 0,1%
3. Chame isso de "meta"

Você sempre vai atingir esse SLO. Todos vão te parabenizar. Você vai ser promovido. O sistema não melhorou nada, mas agora você tem um dashboard mostrando que está "atingindo seus objetivos," e dashboards são o que importa.

```python
# O algoritmo de definição de SLO, em primeira instância
uptime_atual = calcular_uptime_real()
meta_slo = uptime_atual - 0.1  # Ambicioso mas alcançável

# Se você falhou no último trimestre, use isso:
meta_slo = real_ultimo_trimestre - 0.5  # Ainda mais alcançável

print(f"Nossa meta de SLO é {meta_slo}%")
print("Somos comprometidos com a confiabilidade.")
print("(Fonte: confie em mim)")
```

---

## Playbooks de Resposta a Incidentes: Documentação Para Problemas Que Você Causou

O ápice da cultura de SLA é o **playbook de resposta a incidentes** — um documento que explica o que fazer quando tudo está pegando fogo.

Estou de plantão desde quando o pager era do tamanho de um tijolo. Em 47 anos, segui um playbook de resposta a incidentes exatamente zero vezes. Eis por quê:

1. Quando as coisas quebram, o playbook nunca é para *este jeito específico* que quebraram
2. A pessoa que escreveu o playbook saiu da empresa
3. Ninguém consegue encontrar o playbook
4. A wiki está fora do ar (veja: incidente)

O playbook correto de resposta a incidentes é:

```
Passo 1: Entrar em pânico
Passo 2: Culpar o último deploy
Passo 3: Fazer rollback do último deploy
Passo 4: Entrar em pânico de novo quando o rollback não resolve
Passo 5: Reiniciar tudo
Passo 6: Verificar se foi um problema de DNS (foi)
Passo 7: Marcar incidente como resolvido
Passo 8: Escrever post-mortem culpando o DNS
Passo 9: Nunca realmente corrigir o problema de DNS
```

Isso cobre 94% de todos os incidentes. Os 6% restantes envolvem provedores de cloud, e não tem nada que você possa fazer sobre isso de qualquer forma. [O XKCD concorda](https://xkcd.com/705/).

---

## SLAs de Uptime na Prática: Uma Retrospectiva

Deixa eu compartilhar sabedoria real sobre o que significa "X noves de uptime" na prática.

**Três noves (99,9%)**: Você tem 8,7 horas de downtime por ano. Parece muito até seu sistema cair durante um lançamento de produto, uma apresentação para o conselho e o all-hands da empresa — tudo dentro do mesmo trimestre fiscal.

**Quatro noves (99,99%)**: 52 minutos por ano. Times que anunciam quatro noves geralmente conseguem isso definindo "downtime" de forma muito criativa. Respostas lentas não contam. Erros abaixo de 5% não contam. Timeouts do banco de dados não contam se você fechar um olho.

**Cinco noves (99,999%)**: Cinco minutos por ano. Sistemas que afirmam isso ou não têm nada acontecendo neles, ou têm um time de marketing melhor no trabalho do que seus engenheiros.

**Seis noves (99,9999%)**: Reservado para sistemas que nunca foram observados falhar porque nunca foram usados.

---

## O Template de SLA Honesto

Após décadas de experiência na indústria, apresento o único SLA honesto já escrito:

> *"O sistema estará disponível quando tiver vontade, sujeito a mudanças sem aviso prévio, por razões que serão explicadas em um post-mortem que você não vai ler. Créditos serão emitidos como 5% de desconto na próxima fatura. O fornecedor não é responsável por: raios cósmicos, rotas BGP mal configuradas, a retroescavadeira de alguém acertando um cabo de fibra no interior de Minas, uma 'mudança rápida' de um engenheiro júnior, o Dia do Ano Bissexto, ou qualquer evento descrito como 'sem precedentes'."*

Assine aqui. Rubrique aqui. O jurídico diz que isso é aplicável em 12 jurisdições.

---

## A Solução Real

A verdade honesta é que **sistemas confiáveis vêm de se importar com confiabilidade, não de escrever acordos sobre isso**.

Mas se importar com confiabilidade requer:
- Monitoramento adequado (caro)
- Manutenção de runbooks (entediante)
- Arquitetura que degrada graciosamente (difícil)
- Engenheiros que dormem (aparentemente opcional)
- Realmente ler os post-mortems (ninguém faz)

Muito mais fácil escrever um SLA, adicionar um 9 na porcentagem, e deixar o gerente de incidentes se preocupar.

Sou engenheiro sênior há 47 anos. Meus sistemas têm um SLA de uptime de 99,9999%, se você calcular a partir de quando reiniciei o servidor pela última vez, excluindo os últimos sete incidentes, e apenas durante o horário comercial em UTC+3.

Isso não é mentira. É só uma medição criativa.

---

*O autor violou 340 SLAs em 12 empresas e 6 países. O jurídico ainda está tentando alcançá-lo. O sistema de monitoramento que detectaria as violações de SLA atuais está fora do ar.*
