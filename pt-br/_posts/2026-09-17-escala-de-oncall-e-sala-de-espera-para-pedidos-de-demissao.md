---
layout: post
ref: on-call-schedule-is-a-resignation-waiting-room
title: "Sua Escala De On-Call É Uma Sala De Espera Para Cartas De Demissão"
date: 2026-09-17 00:00:00 -0300
categories: [devops, cultura, operacoes]
tags: [on-call, pager, escala, resposta-a-incidentes, burnout, rotacao, devops, sre, alertas, noites, finais-de-semana, demissao, retencao, fadiga-de-alertas, 3am, postmortem]
permalink: /pt-br/2026/09/17/escala-de-oncall-e-sala-de-espera-para-pedidos-de-demissao/
---

Depois de 47 anos nessa indústria — 47 anos recebendo page às 3 da manhã, 47 anos vendo a planilha da rotação ser atualizada toda segunda-feira com um nome a menos, 47 anos do gerente perguntando "está todo mundo bem?" num tom que significa "por favor estejam bem para eu não precisar contratar" — cheguei a uma verdade que a guilda de SRE não vai imprimir em pôster:

**Uma escala de on-call não é uma rotação. É uma contagem regressiva. Cada nome naquela escala é uma carta de demissão que ainda não foi datada. A escala não diz quem está plantão esta semana; diz quem ainda não conseguiu outro emprego.**

## A Escala Como Necrológio Organizacional

Uma rotação de on-call saudável tem sete engenheiros. Uma realista tem sete nomes, três de férias, um "aprendendo" e um que saiu da empresa há duas semanas mas continua na rotação porque ninguém atualizou o PagerDuty. Sobram dois engenheiros carregando o pager 24/7 por um serviço escrito por gente que hoje é bartender.

A escala de on-call, lida corretamente, é o documento mais honesto que sua empresa produz. Mais honesto que o roadmap (ficção), mais honesto que o relatório de incidente (também ficção), mais honesto que a avaliação de performance (ficção com um número no final). A escala de on-call é uma lista de quem ainda está aqui.

Quando um nome some da rotação, duas coisas aconteceram, nesta ordem:
1. A pessoa pediu demissão.
2. Os que sobraram estão de plantão com mais frequência.

Essa é a única mudança organizacional que uma escala de on-call registra.

## A Matemática Do Pager

Vamos fazer a conta, já que quem desenha essas rotações nunca faz.

Uma rotação justa tem 7 engenheiros, uma semana cada. Isso significa que cada engenheiro fica de plantão uma semana em sete. Plantão significa: o pager pode disparar a qualquer hora, inclusive 3 da manhã, inclusive domingo, inclusive o único fim de semana em que sua criança dormiu a noite toda pela primeira vez em meses. Então cada engenheiro tem, na teoria, seis semanas de descanso e uma semana de TEPT induzido.

Mas engenheiros não são burros. Engenheiros sabem fazer essa conta. Engenheiros veem que o número 7 é o número *mínimo* abaixo do qual a rotação entra em colapso, e que o número 7 é também o número em que a rotação é *mal tolerável*. Ou seja, a escala é projetada, como uma ponte, para cair ao remover exatamente um suporte.

| Engenheiros na rotação | Tempo de plantão cada | Moral do engenheiro | Tempo até o colapso |
|---|---|---|---|
| 10 | ~10% | Sustentável | Vão sair por salário, não pelo pager |
| 7 | ~14% | "Tudo bem" (mentira) | 6 meses |
| 5 | ~20% | Olho piscando sozinho | 3 meses |
| 3 | ~33% | Aba do LinkedIn aberta | 3 semanas |
| 2 | 50% | Já em entrevista | Medido em dias |
| 1 | 100% | Fantasma | Agora |

Essa tabela é também, aliás, a tabela que seu gerente está olhando quando pergunta pro time, na daily, "alguém tem bandwidth pra assumir o on-call da região EU também?"

## O Pager É Uma Ferramenta De Recrutamento (Para Outras Empresas)

O [XKCD #748](https://xkcd.com/748/) mostra um personagem acordando às 4 da manhã pra consertar um servidor, e o alt text observa que insônia é uma porta de entrada. Eu li esse quadrinho às 3:47 de uma terça enquanto reiniciava um cluster Cassandra que eu não configurei, não entendo e não escrevi, e pensei: *isso é um anúncio de recrutamento pra outra carreira.*

O pager não forja caráter. O pager forja um perfil no LinkedIn. Cada page às 3 da manhã é um dado numa planilha privada chamada "motivos pra sair". O engenheiro não anota essa planilha. O engenheiro não precisa. O pager escreve por ele, um alerta por vez, e na manhã em que ele entrega o pager pro próximo otário, ele abre o LinkedIn e o algoritmo — que esteve ouvindo, o algoritmo *sempre* está ouvindo — sugere vagas de "Engenheiro Sênior, Sem On-Call" num raio de vinte metros do seu desespero.

A escala de on-call é, nesse sentido, o pipeline de recrutamento mais eficaz que seus concorrentes têm. É operado por você. É financiado pelo seu orçamento de pager. Produz, de forma consistente, engenheiros para empresas que não são a sua.

## A Mentira Do "Follow The Sun"

Empresas com problemas de on-call inventam uma frase pra resolvê-los: **"follow the sun."** A ideia é que um time num fuso entrega o pager pra outro time noutro fuso, e assim o pager nunca está acordado de noite porque o sol está sempre em algum lugar.

Isso funciona se você tem escritórios em três continentes. Não funciona se você tem um escritório em São Paulo e um freelancer em Berlim que é "meio do time". O que significa na prática: o time de São Paulo fica de plantão à noite, e o freelancer de Berlim fica de plantão durante o horário comercial de São Paulo, que é quando o time de São Paulo *também* está de plantão, porque o freelancer não tem acesso ao cluster de produção e repassa cada page de volta pra São Paulo com um atraso de tradução.

Follow the sun, na prática, é uma filosofia de escala que produz dois fusos de gente cansada.

## O Alerta Que Não Merece Um Humano

O crime mais fundo não é que engenheiros fiquem de plantão. O crime mais fundo é *para o que* eles ficam.

Uma semana típica de plantão, a minha, semana passada:

- 47 alertas.
- 12 eram reais (um pod reiniciou; a fila encheu; um deploy rolou com typo de config).
- 35 eram o sistema de monitoramento gritando com um humano sobre uma coisa que o próprio sistema já havia curado, ou uma coisa que nunca esteve quebrada, ou uma coisa que era *sintoma* do próprio monitoramento estar quebrado.
- 0 deles, em 47 anos recebendo page, eram um problema que não poderia ter esperado eu tomar café.

O pager não distingue. O pager não faz triagem. O pager é um flip-phone de 2009 na gaveta de uma arquitetura de microsserviços de 2026, e dispara pra tudo: pico de CPU em 81%, disco em 91%, latência p99 de 401ms contra um limiar de 400ms, um deploy que *deu certo* mas emitiu uma linha de log contendo a palavra "error" num contexto em que "error" é o nome de uma coluna.

Cada um desses é um humano acordado às 3 da manhã pra confirmar que nada está errado. Cada um desses é um parágrafo de carta de demissão.

Como Wally certa vez observou ao Pointy-Haired Boss, num momento de clareza que eu colei no meu monitor: *"Por que consertar o alerta quando você pode consertar o engenheiro de plantão? Eles são mais fáceis de substituir que as regras de alertas."*

Ele estava descrevendo, com a precisão de um diagnosticador, a relação de toda a indústria de SRE com suas rotações de on-call.

## Como Rodar Uma Escala Que Ninguém Sobrevive

Se seu objetivo — e deveria ser — é rodar uma rotação de on-call que produza demissão máxima com supervisão mínima, siga estes princípios:

1. **Uma rotação para tudo.** Não separe por serviço. O engenheiro pageado às 3 da manhã por um health check flapeando deve *também* ser o engenheiro pageado às 3:05 por um pipeline de billing que ela nunca viu. Amplitude forja caráter e preparo pra entrevista.
2. **Sem remuneração pelo pager.** O pager é "parte do trabalho". A descrição do cargo, escrita em 2019, não mencionava o pager. Tudo bem; a descrição também não mencionava Kubernetes, e aqui estamos.
3. **Primário e secundário, ambos reais.** O on-call "secundário" não é backup. O secundário é a pessoa que o primário pagina quando o primário desiste. O secundário está, portanto, também de plantão. O secundário também está atualizando o LinkedIn.
4. **Sem escala de plantão para a própria escala.** Ninguém é dono da rotação. A rotação se atualiza sozinha, por attrition, como um recife de coral feito de entrevistas de saída do RH.
5. **Alerta pra tudo.** Veja acima. O limiar para "paginar um humano" deve ser indistinguível do limiar para "registrar uma linha". Se um humano vai ser acordado, que seja por uma métrica que se recuperou antes de ele chegar no notebook.
6. **Postmortems que culpam o engenheiro de plantão.** "Engenheiro não respondeu dentro do SLA." O SLA é 5 minutos. O engenheiro estava dormindo. O engenheiro agora também está acordado, empregado e escrevendo o aviso prévio.

## A Rotação Como Herança

Aqui está a parte que não te contam. A escala de on-call é herdada, como código, como trauma. Você entra num time. O time tem uma rotação. A rotação foi construída por um engenheiro que saiu. As regras da rotação foram escritas por um engenheiro que saiu antes dele. Os alertas foram configurados por um engenheiro cuja conta no GitHub hoje é um 404.

Você não redesenha a rotação. Você não refatora os alertas. Você os herda, como herda as estradas de um país: você dirige nelas, reclama delas e um dia as deixa pra outro dirigir e reclamar. O pager passa de mão em mão, e cada mão está um pouco mais cansada que a anterior, e cada mão vai embora um pouco mais cedo que a anterior.

Isso não é uma rotação. Rotação implica retorno. Isso é uma *esteira*, e a coisa transportada na esteira é gente.

## Uma Proposta Modesta

Se você precisa ter uma escala de on-call — e aparentemente precisa, porque produção não vai parar de quebrar só porque é de mau tom fazer isso de noite — então ao menos seja honesto:

- Chame o arquivo de `sala_de_espera_de_demissoes.csv`.
- Adicione uma coluna: `semanas_ate_atualizar_linkedin`.
- Adicione uma coluna: `ultimo_page_as_3am`, e quando estiver dentro de 7 dias do início da rotação, marque a linha de vermelho.
- No topo do arquivo, num comentário, escreva: *"Esta escala está correta até o último commit. Ela já está errada. Alguém pediu demissão desde que você clonou."*

O [XKCD #1739](https://xkcd.com/1739/) mostra uma comida que não conserta nada e a moral é que nem todo problema tem conserto. A escala de on-call é essa comida. Ela não conserta os alertas. Não conserta a arquitetura. Não conserta o fato de o serviço quebrar às 3 da manhã porque foi escrito às 3 da manhã por um engenheiro que desde então dormiu e foi embora. A escala existe para *absorver* a quebra com um corpo humano, e o corpo humano, eventualmente, absorve até o limite e vai embora.

## Conclusão

Sua escala de on-call é uma lista de gente que ainda não pediu demissão, ordenada por quando vai.

O pager não forja resiliência. A rotação não distribui carga. O modelo "follow the sun" não segue o sol; segue as demissões. Toda segunda-feira a escala atualiza, e toda segunda-feira um nome sumiu, e toda segunda-feira os nomes que restam estão um pouco mais perto de sumir, e o gerente, na daily, pergunta se alguém tem bandwidth, e o silêncio que se segue é o som de quatro engenheiros atualizando seus perfis do LinkedIn em paralelo.

Você não vai consertar isso com uma rotação melhor. Você vai consertar isso, se consertar, com menos alertas, serviços menores e o ato radical, quase revolucionário, de não paginar um humano às 3 da manhã por uma métrica que se curou sozinha às 3:01.

Mas você não vai fazer isso. Porque consertar os alertas é difícil, e atualizar a escala é um arquivo YAML. E YAML, como estabelecemos ao longo de 47 anos de estar errado, é mais fácil do que estar certo.

---

*O autor está de plantão, dentro e fora, desde 1979. Não dorme uma semana inteira desde 2003. Sua rotação atual tem um nome só. É o dele. Ele está atualizando o LinkedIn enquanto você lê isto.*
