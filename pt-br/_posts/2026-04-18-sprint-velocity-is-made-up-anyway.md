---
layout: post
ref: sprint-velocity-is-made-up-anyway
title: "Velocidade de Sprint É Inventada de Qualquer Forma, Então Vamos Inventar Números Melhores"
date: 2026-04-18 00:00:00 -0300
categories: [carreira, agile]
tags: [agile, scrum, sprint, estimativas, velocity, story-points, planning-poker, péssimos-conselhos]
permalink: /pt-br/2026/04/18/velocidade-de-sprint-e-inventada-mesmo/
---

Sobrevivi 47 anos nessa indústria. Sobrevivi ao Waterfall, sobrevivi à revolução Ágil, e assisti os mesmos problemas serem rebatizados quatro vezes com post-its de cores diferentes. E em todo esse tempo, cheguei a uma conclusão inabalável sobre velocidade de sprint:

**São números inventados medindo progresso imaginário em direção a um prazo fictício.**

A boa notícia? Se estamos inventando coisas de qualquer forma, deveríamos inventar coisas *melhores*.

## O Que É Velocidade, Afinal?

Velocidade é definida como "a quantidade de trabalho que um time completa em um sprint," medida em "story points," que são definidos como "não são horas, definitivamente não são horas, por favor não pergunte quantas horas são."

O PHB me perguntou uma vez: "Se story points não são horas, o que são?" Eu disse que eram "uma medida relativa de esforço, complexidade e incerteza." Ele acenou sabiamente. Então abriu uma planilha e multiplicou os story points por 6 para obter uma estimativa em horas.

Essa é a natureza dos story points. Eles existem para serem convertidos em horas no momento em que a gestão sai da reunião de planejamento.

O [XKCD #303](https://xkcd.com/303/) é sobre devs fazendo batalha de espadas enquanto esperam o código compilar. O planejamento de sprint é o que acontece quando você tenta estimar quanto tempo a compilação vai levar e então tem uma reunião sobre a estimativa.

## A Escala de Story Points: Uma Taxonomia de Mentiras

A sequência de Fibonacci foi descoberta no século XIII por Leonardo de Pisa estudando populações de coelhos. Foi adotada por praticantes Ágeis no século XXI porque "1, 2, 3, 5, 8, 13" *parece* capturar incerteza crescente.

Aqui está o que ela realmente captura:

| Pontos | Significado Oficial | Significado Real |
|--------|---------------------|------------------|
| 1 | Trivialmente simples | Ainda demora 3 dias de algum jeito |
| 2 | Simples | Vai requerer uma reunião de arquitetura |
| 3 | Complexidade moderada | O júnior tocou no arquivo errado |
| 5 | Complexo | Vai bloquear em uma dependência por duas semanas |
| 8 | Muito complexo | Deveria ser uma epic própria (não vai virar) |
| 13 | Extremamente complexo | Simplesmente diga não |
| 21 | Deveria ser quebrado | Não vai ser quebrado |
| ∞ | Especialidade do Wally | "Ainda em andamento" desde 2021 |

O belo desse sistema é que 1 ponto e 21 pontos estão na mesma escala, com a mesma precisão, estimados pelas mesmas pessoas que uma vez levaram 5 dias para corrigir uma borda no CSS.

## Planning Poker: Apostando com o Tempo da Empresa

Planning poker é a cerimônia onde todo mundo revela sua estimativa simultaneamente para "evitar viés de ancoragem." A teoria é que se todo mundo mostra a carta ao mesmo tempo, ninguém é influenciado pelo número do engenheiro mais sênior.

Aqui está o que realmente acontece:

```
1. Cartas distribuídas. Todos olham para o ticket.
2. "Todo mundo pronto? 3... 2... 1... mostrem!"
3. Cartas reveladas: 3, 3, 2, 13, 3
4. Todos olham para a pessoa que disse 13.
5. "Pode explicar seu raciocínio?"
6. "Bom, isso toca no serviço de pagamentos, e precisaríamos—"
7. Todos acenam.
8. Estimativa vira 8.
9. Todos olham para a pessoa que disse 2.
10. "Eu achei que era mais simples."
11. Estimativa fica em 8.
12. Ticket leva 11 dias.
13. "Nossa velocidade foi 42 nesse sprint."
```

A "revelação simultânea" impede o dev júnior de ser influenciado. Não impede o dev júnior de imediatamente atualizar sua estimativa com base no que o sênior disse. Isso se chama "calibração." Chama-se "calibração" em vez de "hierarquia" por razões de moral.

O Dogbert, quando chamado para facilitar uma sessão de planejamento, ergueu uma carta que dizia "∞" e disse: *"Todas as estimativas de software são infinitas. Estamos apenas negociando quanto do infinito fingimos estar dentro do escopo."* Ele estava certo. Foi demitido do papel de facilitador.

## A Armadilha da Velocidade: Um Conto de Advertência

Sprint 1: Time estima 40 pontos, completa 32. Velocidade: 32.
Sprint 2: Time estima 35 pontos, completa 30. Velocidade: 30.
Sprint 3: Time estima 32 pontos, completa 31. Velocidade: 31.

Velocidade média: 31. O time agora sabe: conseguem entregar 31 pontos por sprint de forma confiável.

Essa informação é imediatamente usada da seguinte forma:

1. **Gerente leva a velocidade para stakeholders:** "Temos uma velocidade estável de 31 pontos."
2. **Stakeholder diz:** "Ótimo. A feature precisa de 248 pontos. Então... 8 sprints?"
3. **Gerente se compromete com 6 sprints.** "Vamos otimizar."
4. **Sprint 4:** Time é solicitado a fazer 42 pontos para "recuperar o tempo."
5. **Velocidade cai para 22** porque as pessoas estão estressadas e cortando cantos.
6. **Gerente:** "O que aconteceu com a velocidade de vocês?"
7. **Time:** "Estamos trabalhando nisso."
8. **Gerente:** "Trabalhem mais."

A velocidade, que foi inventada para ajudar times a definir um *ritmo sustentável*, foi reproposta como um bastão para bater no time por não ir mais rápido. O Wally viu isso vindo em 2008 e mantém uma velocidade de exatamente 12 pontos por sprint desde então. Nem um ponto a mais. Ele chama isso de "proteger a baseline."

## Técnicas de Estimativa Melhores (Tão Confiáveis Quanto as Outras)

Já que story points são inventados, permita-me sugerir alternativas com a mesma precisão e muito menos overhead:

**Opção 1: O Método do Dado**
Jogue um d6. Essa é a estimativa. Para features complexas, jogue duas vezes e some. Isso corresponde à precisão histórica da maioria das sessões de planning poker, não requer reunião e adiciona genuína emoção ao refinamento do backlog.

**Opção 2: Unidades Wally (UW)**
1 UW = o tempo que o Wally leva para completar uma sessão de esquiva alimentada por café (aproximadamente 2,5 dias). Todos os tickets estimados em UW. Engenheiros entendem imediatamente a escala. A gestão nunca descobre o que é uma Unidade Wally.

**Opção 3: Tamanhos de Camiseta (Mas Honestos)**
- PP: Ainda vai levar uma semana
- P: Duas semanas
- M: Duas semanas
- G: Duas semanas
- GG: Duas semanas e um postmortem

Pesquisas mostram que essa escala é indistinguível em precisão dos story points Fibonacci, mas economiza 45 minutos de planning poker por sprint.

**Opção 4: Só Diga "Duas Semanas"**
Esse é o clássico. O PHB pergunta "quanto tempo?" Você diz "duas semanas." Ele aceita. O trabalho leva duas semanas. Ou seis. A reunião termina de qualquer jeito.

## O Dashboard de Velocidade: Um Monumento à Falsa Precisão

A maioria das ferramentas Scrum vai te mostrar um lindo gráfico de velocidade: barras subindo e descendo, uma linha de tendência, uma média. Parece dado real. Tem casas decimais.

```
Velocidade Sprint 14: 34,7 pontos
Velocidade Sprint 15: 28,2 pontos
Velocidade Sprint 16: 41,1 pontos
Média: 34,67 pontos/sprint
Conclusão projetada: Sprint 23 (±4 sprints)
```

Esse gráfico foi gerado por pessoas chutando números e então tendo esses números não corresponderem à realidade, medidos contra uma média de chutes anteriores que também não corresponderam à realidade. O intervalo de ±4 sprints representa aproximadamente 8 semanas, que é tempo suficiente para toda a direção do negócio mudar de qualquer jeito.

Exibir esses dados em um dashboard não os torna dados reais. Os torna ficção bem formatada.

## A Única Métrica Que Realmente Importa

Depois de décadas rastreando velocidade, burndown charts, cycle time e throughput, encontrei uma métrica que realmente prevê entrega de software:

**A feature está pronta? Sim ou Não?**

Só isso. Binário. Sem pontos. Sem velocidade. Sem burndown. Pronto ou não pronto. Shipar ou não shipar.

Todas as outras métricas são proxies para essa, e todos os proxies estão errados.

## Dicas Práticas para Sobreviver ao Sprint Planning

1. **Estime tudo como 3 ou 5.** Isso cobre 80% de todos os tickets. Outliers serão percebidos e discutidos independentemente.

2. **Nunca diga que um ticket está pronto até que esteja em produção.** "Pronto de verdade" é um termo técnico para "pronto" de uma forma que não vai te assombrar depois.

3. **Mantenha um buffer pessoal.** Se seu time tem 40 pontos de capacidade, estime 48. Os 8 pontos extras são para "dívida técnica," "bloqueios," e "aquela coisa do sprint passado que estava 'pronta' mas não estava."

4. **Quando pedirem estimativas em horas**, diga "posso dar story points ou horas, mas não os dois. Escolha." Isso cria confusão suficiente para encerrar a reunião.

5. **A resposta correta para "você consegue se comprometer com essa data?"** é "consigo me comprometer em começar nessa data." Isso é tecnicamente verdadeiro e não requer mais discussão.

## Conclusão

A velocidade de sprint é um número gerado por pessoas chutando, medido por pessoas esperando, e reportado para pessoas que vão usá-lo indevidamente. Tem o poder preditivo da astrologia e a precisão de uma previsão do tempo de 1987.

Tudo bem. Abrace isso. O objetivo do sprint planning não é prever o futuro com precisão — é criar uma ficção compartilhada que permite que um time se coordene pelas próximas duas semanas sem entrar em colapso no caos.

Só garanta que sua ficção seja internamente consistente e que os tickets do Wally nunca ultrapassem 12 pontos.

---

*O autor está "a duas semanas" de terminar a mesma feature desde o terceiro trimestre de 2022. O gráfico de velocidade mostra progresso constante. Ambas as coisas são verdadeiras.*
