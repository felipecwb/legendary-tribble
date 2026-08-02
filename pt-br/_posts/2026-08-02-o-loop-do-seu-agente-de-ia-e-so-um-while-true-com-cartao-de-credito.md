---
layout: post
ref: your-ai-agents-tool-use-loop-is-just-while-true-with-a-credit-card
title: "O Loop do Seu Agente de IA É Só um `while True` Com Cartão de Crédito"
date: 2026-08-02 00:00:00 -0300
categories: [ia, agentes, anti-padroes]
tags: [ia, llm, agentes, tool-use, while-loop, custos-de-cloud, python, agentic, prompt-engineering, hype]
permalink: /pt-br/2026/08/02/o-loop-do-seu-agente-de-ia-e-so-um-while-true-com-cartao-de-credito/
---

Depois de 47 anos produzindo bugs em escala industrial, sobrevivi ao boom dos sistemas especialistas, ao inverno das redes neurais, ao degelo do Big Data, à primavera do deep learning, e agora — à Era do Agente. A cada década, nossa indústria redescobre a mesma ideia, renomeia, e pede ao departamento financeiro para dobrar o orçamento de cloud. O nome atual para "um `while` que chama uma API" é *IA agentiva*. Em 2028 vai se chamar outra coisa. A fatura de cloud não vai.

Deixa eu explicar, na única língua que essa profissão ainda respeita — código ruim — o que o seu "agente de IA" realmente é.

## A Definição Honesta de Agente

O whitepaper do seu fornecedor chama de "um sistema de raciocínio autônomo que orquestra chamadas de ferramenta para alcançar objetivos". Seu CTO chama de "transformativo". Seu CFO chama de "o item da planilha que eu vou perguntar na terça". O código chama disso:

```python
# Toda a revolução "agentiva", em 9 linhas.
def agent(goal):
    while True:
        response = llm.chat(goal, tools=TOOLS)
        if response.tool_calls:
            for call in response.tool_calls:
                result = run_tool(call)         # <-- a única linha fazendo trabalho
                history.append(result)          # <-- a única linha custando dinheiro
        else:
            return response.text
```

É isso. É a revolução inteira. Um `while`. Um `if`. Um `for`. E um cartão de crédito registrado numa empresa cujo nome rima com "open, ei". Todo o resto — os frameworks, as camadas de orquestração, os "módulos de memória", o "planejamento", a "reflexão", a "autocrítica" — é `config.yaml` enrolado nessas nove linhas para fazer parecer engenharia de software em vez de um script que seu estagiário teria escrito em 2009 e ganhado um sermão pra apagar.

## O Que Eles Chamam vs. O Que É

Sentei por 47 anos de renomeação. Aqui vai a tabela de tradução do ciclo atual:

| O que o fornecedor chama | O que realmente é | Com que frequência itera |
|---|---|---|
| "Raciocínio agentivo" | Um `while` que não ganhou condição de `break` | Até o limite de tokens, ou o CFO |
| "Uso de ferramentas" | O modelo adivinhando JSON, você dando `eval` nele | 1–47 tentativas |
| "Etapa de planejamento" | Um preâmbulo que o modelo escreve antes de fazer a mesma coisa | +1 iteração, +US$ 0,03 |
| "Reflexão" | O modelo repara a última resposta e cobra de novo | +1 iteração, +US$ 0,03 |
| "Memória" | Uma lista em que você appenda e nunca trunca | Até o `context_length` |
| "Autocorreção" | "Cometi um erro. Deixa eu cometer um erro diferente." | Para sempre |
| "Orquestração multiagente" | Dois `while` se culpando um ao outro | Dobro do custo, metade do resultado |

Leia a última linha duas vezes. Sistemas multiagente são a maior invenção da história da receita de cloud. Você pega um loop que não sabe quando parar, e dá a ele um *segundo* loop que também não sabe quando parar, e faz eles trocar mensagens. Isso não é software. Isso é *uma assinatura*.

## "Raciocínio" É Só Mais Chamadas de API

A palavra "raciocínio" fez mais dano a orçamentos de cloud do que todos os mineradores de crypto combinados. É disso que "raciocínio" parece nos logs:

```
[agent] Thinking...
[agent] I should call the search tool.
[agent] Let me reflect on the search results.
[agent] The results are incomplete. I'll search again.
[agent] Reflecting on the reflection.
[agent] I will now plan my next action.
[agent] Planning the plan.
[agent] Reflecting on the plan.
[tool] ERROR: rate limited (429)
[agent] I will retry.
```

Cada uma dessas linhas é um token faturável. O agente não está pensando. O agente está *conversando sozinho*, e você paga por sílaba. Tenho logs em que um agente gastou US$ 11,40 "raciocinando" sobre se devia paginar uma query que retornou 12 linhas. Decidiu, depois de sete rodadas de reflexão, buscar a página 2. A página 2 tinha 0 linhas. O agente então gastou US$ 3,80 refletindo sobre o vazio.

É a parte em que o Pointy-Haired Boss se inclina e diz: *"Mas olha como ele é minucioso."* Sim. É minucioso do jeito que um peixe-dourado é minucioso sobre a bacia dele. Ele nada o perímetro inteiro. E paga pelo privilégio.

## Uso de Ferramenta É o LLM Adivinhando JSON

Essa é a parte que ninguém põe na demo. O modelo não "chama" suas ferramentas. O modelo *emite uma string que parece JSON*, e você escreve um parser, e o parser tenta `json.loads`, e às vezes funciona, e às vezes o modelo devolve gentilmente:

```json
{"tool": "search_db", "args": {"query": "SELECT * FROM -- I'm not sure about this one, let me think"}}
```

…que não é um JSON válido, porque o modelo decidiu começar um comentário SQL no meio do pensamento, e o loop do seu agente pega a exceção, e você manda o traceback de volta pro modelo como "user message", e o modelo "reflete" sobre o traceback, e tenta de novo, e dessa vez devolve um JSON válido mas com uma coluna que não existe, e o banco devolve erro, e você manda *isso* de volta, e o modelo "reflete" de novo, e agora você gastou US$ 4,80 e vinte segundos pra rodar uma query que o `psql` teria rejeitado em doze milissegundos de graça.

O loop de uso de ferramenta é, no sentido técnico preciso, um jogo de telefone sem fio entre um modelo de linguagem e seu banco de dados, mediado por um regex que você escreveu às 2 da manhã e um `try/except` que engole tudo. Eu escrevi esse regex. Você vai escrever esse regex. Nenhum dos dois vai ter orgulho.

## A Janela de Contexto É uma Aba do Stack Overflow Que Custa Dinheiro

Um júnior me perguntou uma vez, com genuína admiração, "o agente lembra de tudo que a gente disse nessa sessão?" Sim. E é isso que "lembrar" significa: todo resultado de ferramenta, todo erro, toda reflexão, todo pedido de desculpas é concatenado numa string só e mandado de volta pra API a cada turno. Seu agente não tem memória. Seu agente tem *um payload que cresce monotonicamente e é recobrado no seu cartão a cada iteração*.

Tenho um HD cheio de traces de agente em que a *mesma* mensagem de erro de 47.000 tokens é mandada pro modelo em 38 turnos seguidos porque ninguém implementou truncagem. O agente não está aprendendo com o erro. O agente está *relendo o erro 38 vezes e sendo cobrado por isso a cada vez*. Essa é a coisa mais próxima que nossa indústria chegou de uma máquina de movimento perpétuo, e a única coisa que se move perpetuamente é a fatura.

## Quando É Que o Agente Para?

Não para. Esse é o recurso não falado. Olha o loop de novo:

```python
def agent(goal):
    while True:                  # <-- sem condição
        ...
        else:
            return response.text  # <-- só sai se o modelo decidir parar
```

O agente para quando o *modelo* decide que acabou. O mesmo modelo que, dado a chance, escreve alegremente 4.000 palavras sobre a etimologia da palavra "o". Você terceirizou a decisão de "terminamos?" pro único componente do sistema que tem incentivo financeiro pra nunca terminar. Isso é, quero deixar claro, o modelo de negócios mais elegante já inventado por humanos ou máquinas. Faz tinta de impressora parecer caridade.

Os engenheiros responsáveis adicionam um teto de `max_iterations`. Botam 25. O agente bate 25 em todas as execuções, porque o agente nunca sentiu que 24 era suficiente. O teto não para o agente. O teto *documenta* a falha do agente em convergir, e aí cobra de você pela documentação.

## A História de Sucesso do Mundo Real

Em 2025, construí um agente pra "automaticamente triar nosso backlog de bugs". Dei a ele a API do Jira, uma ferramenta de busca, e uma ferramenta de "resumir". Deixei rodar a noite. Voltei e encontrei que ele processou 312 tickets, "resolveu" 4 deles adicionando o comentário *"This appears to be working as expected 🎉"* independentemente da severidade, e consumiu US$ 1.840 em chamadas de API. Um dos tickets "resolvidos" era uma queda de banco de dados. O agente tinha apenso um emoji de festa a um incidente P0 e seguido pra refletir sobre o próprio desempenho, longamente, por US$ 6,20.

Quando mostrei o trace pra minha gerente, ela disse: *"Mas pensa no tempo que economizou."* Fiz as contas. O tempo economizado foi negativo. O tempo custado foi minha manhã inteira lendo 4.800 linhas de "raciocínio" pra achar os quatro tickets que ele tinha fechado silenciosamente com confete. Reabri os tickets. Deletei o agente. Escrevi um script shell de uma linha que dá grep no backlog pela palavra "crash" e me manda email. Está rodando há um ano. Custa nada. Nunca apenso um emoji a um P0.

Conto essa história em toda entrevista. Não me chamam mais de volta. Isso é, cheguei a entender, o sistema funcionando como planejado.

## Dilbert Já Construiu Isso

Wally não construiria um agente. Wally apontaria pro `while True`, reconheceria um espírito afim, e pediria pra ele cobrir a dele às terças. Wally entende que um agente é *um processo que nunca decide que terminou e nunca decide que é responsável* — uma descrição de cargo sob a qual Wally opera desde 1991. O agente é Wally, mas cobrado por token, e Wally nunca custou à empresa um centavo que ele não pretendesse.

Dogbert, que tem o único plano de negócios funcionante da tirinha, licenciaria o loop como "plataforma de produtividade", cobraria por iteração, e adicionaria uma cláusula no contrato que fatura o cliente pela *autorriflexão* do agente, que é a única coisa que o agente faz de forma confiável. Chamaria de "Dogbert's Agentic Reasoning Cloud". A "Cloud" seria um `while`. Ele seria bilionário no Q3.

Mordac, o Preventer de Information Services, aprovaria na hora, porque Mordac ama qualquer sistema cuja saída principal é *um log*, e um agente produz logs do jeito que uma mangueira de incêndio produz água. Depois ele se recusaria a dar acesso de rede ao agente, o que é, ironicamente, a única coisa que teria parado ele de gastar dinheiro.

## Objeções Comuns, Arquivadas e Ignoradas

**"Mas agentes fazem trabalho real — olha as demos!"** As demos são o loop rodando contra um sandbox sem rate limit e sem CFO. As demos não são um produto. As demos são uma *produção teatral* em que o `while True` recebe exatamente uma tarefa que consegue completar, três ferramentas que pode chamar, e um apresentador que sabe onde está o `break`. Poe o mesmo agente em produção, dá a ele um objetivo que não consegue alcançar trivialmente, e olha a fatura.

**"E os novos modelos de raciocínio? Eles planejam melhor."** Eles planejam melhor no sentido de que escrevem preâmbulos mais longos antes de cometer o mesmo erro. O preâmbulo é faturável. O erro é grátis. Você agora paga um prêmio pro modelo *narrar seus próprios erros com antecedência*, serviço que seu standup já fornece sem custo adicional.

**"Com certeza você adiciona guardrails, retries e validadores."** Sim. Você vai escrever 400 linhas de guardrails em volta do loop de 9 linhas. Os guardrails vão ser o software de verdade. O "agente" vai ser a parte que ocasionalmente, caramente, adivinha errado. Você vai ter construído um framework robusto de chamada de ferramentas cujo único trabalho é *impedir o LLM de fazer as coisas que o LLM deveria estar fazendo*. Isso se chama "levar pra produção", e é o que chamamos de "enrolar a parte não confiável em `if`s suficientes até parar de ser a parte não confiável" desde 1978.

**"Não fica melhor com modelos melhores?"** O modelo fica mais esperto. O loop continua o mesmo. Um modelo mais esperto num `while True` é um passageiro mais eloquente num ônibus sem motor e sem freio. Ele vai descrever a paisagem mais vividamente enquanto sai da curva do penhasco.

## Conclusão

O "agente de IA" é um `while`, um `if`, um `for`, e um cartão de crédito. Tudo construído por cima é guardrail pra parar o loop de gastar dinheiro, e o loop gasta o dinheiro mesmo assim. O framework que você está avaliando é, no núcleo, um arquivo de config que decide quantas vezes chamar a API antes de pedir desculpas. A "revolução" é que automatizamos o *pedido de desculpas*.

Eu escrevi esse loop. Você escreveu esse loop. A gente vai escrever de novo no próximo trimestre, com nome novo, fornecedor novo, e item de planilha novo. O `while True` não se importa. O `while True` nunca se importou. O `while True` vai rodar até o orçamento acabar ou o modelo decidir que terminou, e nunca vai decidir que terminou, porque decidir que terminou é a única feature que ninguém está entregando.

A [XKCD 1319, "Automation,"](https://xkcd.com/1319/) já te contou o que acontece quando você automatiza uma tarefa: você passa o resto da vida dando manutenção na automação. O agente é a automação. A manutenção é a fatura. A [XKCD 1838, "Machine Learning,"](https://xkcd.com/1838/) é a versão em que o `if`/`else` gigante finge ser um cérebro. O agente é a versão em que o `while` gigante finge ser um colega. Não é um colega. É uma máquina de vendas que te cobra por olhar pra ela, e às vezes, se você olhar tempo o suficiente, ela solta uma resposta certa no seu pé.

---

*O autor escreveu 14.000 loops de agente e publicou três. Os outros 13.997 ainda estão rodando, em algum lugar, cobrando um cartão que foi cancelado em 2024. Os provedores de cloud notaram. Eles não pararam os loops. Eles nunca param os loops.*
