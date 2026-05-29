---
layout: post
ref: sprint-planning-is-fiction-writing
title: "Planejamento de Sprint É Só Fanfic Para Adultos"
date: 2026-05-29 00:00:00 -0300
categories: [agile, planejamento, gestao]
tags: [agile, sprint, planejamento, estimativas, scrum, ficcao, reunioes]
permalink: /pt-br/2026/05/29/planejamento-sprint-e-ficcao/
---

Já participei de 2.300 sessões de planejamento de sprint. Contei. Tive tempo porque nada nessas sessões nunca aconteceu da forma que alguém planejou.

Planejamento de sprint é escrita de ficção. Sempre foi. A diferença entre um romance e um plano de sprint é que o romance é honesto sobre ser inventado, e alguém ocasionalmente termina de ler.

## A Anatomia da Ficção do Planejamento de Sprint

Toda sessão de planejamento de sprint segue o mesmo arco narrativo:

**Ato 1 – A Preparação:** Alguém abre o backlog. O backlog contém 400 tickets escritos em seis estilos diferentes porque quatro pessoas saíram da empresa desde que o mais antigo foi criado. Ninguém sabe o que PLAT-2847 significa. O autor original agora trabalha na concorrência.

**Ato 2 – O Conflito:** O time começa a estimar. O engenheiro sênior diz "3 pontos." O product manager ouve "3 dias." O CEO, de alguma forma nessa reunião, ouve "pronto até terça." São três coisas diferentes. Ninguém corrige ninguém. O ticket recebe 3.

**Ato 3 – A Resolução:** Trinta e dois tickets são adicionados ao sprint. A capacidade do sprint é tecnicamente suficiente para catorze deles. O time aplaude. O board está cheio. Todos voltam para suas mesas e trabalham na coisa que está literalmente em chamas, que não estava no sprint de forma alguma.

Este é o ciclo da vida. O XKCD [#1425](https://xkcd.com/1425/) capturou perfeitamente o espírito da estimativa: atribuímos complexidade com confiança a tarefas que não entendemos, e ficamos surpresos quando erramos.

## Story Points: Uma História de Amor

Story points foram inventados para parar as pessoas de estimar em tempo, o que levava a compromissos de tempo irrealistas. Foram substituídos por story points, que levaram a compromissos de pontos irrealistas, que foram então convertidos de volta para tempo pela gestão de qualquer jeito.

O círculo está completo. Passamos por uma revolução metodológica inteira para acabar exatamente onde começamos, exceto que agora discutimos se "5 pontos" significa "mais difícil que 3 pontos" ou "tipo 3 pontos mas com uma dependência."

| Story Points | O Que Significa | O Que Significa De Verdade |
|---|---|---|
| 1 | Mudança trivial | Você vai descobrir o schema do banco durante isso |
| 2 | Tarefa pequena | Dois times são donos, nenhum vai responder |
| 3 | Complexidade média | Esse requisito muda na quarta-feira |
| 5 | Complexo | São na verdade três tickets que alguém fundiu |
| 8 | Muito complexo | Ninguém tocou nesse código há 4 anos |
| 13 | Escala épica | Esse é o rewrite inteiro que ninguém aprovou |
| ∞ | "Vamos fazer um spike" | Não temos ideia e nunca vamos ter |

## O Objetivo do Sprint: Haiku Moderno

Todo sprint deve ter um "objetivo de sprint." Um objetivo de sprint é um haiku escrito por comitê, otimizado para significar tudo e não comprometer nada.

Objetivos de sprint reais que já testemunhei:

> *"Melhorar a estabilidade da plataforma enquanto entregamos valor para o usuário"*

Tradução: Vamos corrigir uns bugs, talvez. Ou não. Reservamos o direito de interpretar "estabilidade" e "valor" retrospectivamente no final do sprint.

> *"Mover a agulha nas métricas de engajamento"*

Tradução: Não sabemos o que estamos construindo. A agulha é metafórica. Ninguém definiu "engajamento."

> *"Capacitar o time para executar contra as prioridades do Q3"*

Tradução: Esse sprint não tem objetivo. Temos apenas movimento, confundido com progresso.

> "O PHB disse que nossa velocity precisa melhorar 20% nesse trimestre."
> "Ele disse melhorar o *trabalho* ou melhorar os *números*?"
> "A mesma coisa."
> — *Dilbert*, sobre por que métricas de velocity existem para ser manipuladas

## Velocity: A Métrica Que Não Mede Nada

Velocity é o número de story points completados por sprint. É usado para prever sprints futuros. Funcionaria muito bem se:

1. Story points fossem consistentes entre sprints
2. Story points fossem consistentes entre membros do time
3. Story points refletissem complexidade real
4. A composição do time nunca mudasse
5. Requisitos nunca mudassem durante o sprint
6. Ninguém nunca adicionasse "quick wins" diretamente ao board do sprint
7. A produção nunca pegasse fogo

Em 47 anos, nunca vi um sprint onde todos os sete fossem verdade simultaneamente. Velocity está medindo a saída de um sistema caótico com uma régua que encolhe e expande baseada no humor do time.

Mas colocamos num gráfico. O gráfico vai no slide deck. O slide deck vai para a liderança. A liderança diz "a velocity está caindo" e agenda uma reunião sobre isso, reduzindo a velocity ainda mais.

```
velocity_do_sprint = (
    trabalho_realmente_feito
    / fator_de_inflação_dos_story_points
    / multiplicador_de_scope_creep
    * humor_do_time_na_segunda_feira
    - interrupções_da_gestão
    + mentiras
)

# Média histórica: 34 pontos
# Útil para previsões: Não
```

## Para Que o Planejamento de Sprint Realmente Otimiza

Não entrega. Não qualidade. Não previsibilidade. Planejamento de sprint otimiza para a *sensação* de controle.

O board está cheio. Os tickets foram estimados. O objetivo do sprint está escrito (vagamente). A capacidade foi calculada (incorretamente). Todos acenam. Todos *sentem* que um plano existe.

Essa sensação dura até aproximadamente 10h da manhã de segunda-feira quando alguém diz "ei, temos um incidente em produção."

[XKCD #303](https://xkcd.com/303/) — Compilando. Não é sobre compilar. É sobre todas as formas como o trabalho para de ser o que planejamos e se torna o que estamos realmente lidando.

## Minha Alternativa: O Sprint Caótico

Depois de 47 anos vendo ficção se disfarçar de planejamento, proponho o Sprint Caótico:

**Regras:**
1. Sem estimativas. Escreva tickets em português claro. "Corrigir login." "Adicionar botão." "Por que a produção caiu."
2. Sem objetivo de sprint. O objetivo do sprint é: sobreviver.
3. Sem rastreamento de velocity. Em vez disso, conte "coisas que foram feitas." Uma coisa está feita quando ninguém mais a menciona na stand-up.
4. O sprint termina quando termina.
5. Retrospectivas são substituídas por uma refeição compartilhada e o reconhecimento tácito de que todos estamos fazendo o nosso melhor.

Isso não é piada. É assim que todo projeto de sucesso que já entreguei realmente funcionou. Chamávamos de outra coisa na época — "crunch," "simplesmente fazer acontecer," "a semana antes da conferência" — mas a metodologia era Sprint Caótico puro.

## A Única Coisa Que o Planejamento de Sprint Acerta

Darei crédito onde é devido: o planejamento de sprint faz uma coisa bem.

Coloca todo o time na mesma sala.

Ou em uma videochamada. Tanto faz.

Por noventa minutos, todos viram o mesmo backlog, ouviram os mesmos requisitos confusos, e experimentaram a mesma confusão coletiva. Quando o sprint colapsar — e vai colapsar — pelo menos todos têm contexto compartilhado sobre o porquê.

Esse é o verdadeiro valor do planejamento de sprint. Não o plano. A ficção compartilhada. A alucinação comum que brevemente faz um time sentir que está coordenado.

Somos todos apenas autores escrevendo a mesma história, revezando em fingir que sabemos como ela termina.

---

*O autor nunca completou um sprint como planejado. Entregou muitas features. Ele não vê essas afirmações como contraditórias.*
