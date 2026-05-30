---
layout: post
ref: never-talk-to-users
title: "Por Que Desenvolvedores Jamais Deveriam Falar com Usuários"
date: 2026-05-30 00:00:00 -0300
categories: [cultura, produto, agile]
tags: [usuarios, produto, requisitos, ux, agile, comunicacao, cultura, engenharia, product-management]
permalink: /pt-br/2026/05/30/developers-should-never-talk-to-users-pt/
---

Existe uma ideia perigosa se espalhando pela indústria de software. Ela atende por muitos nomes: "pesquisa com usuários", "entrevistas com clientes", "mapeamento de empatia", "sessões de feedback de usuário". E em 47 anos construindo software que os usuários odeiam, a identifiquei como a maior ameaça à produtividade e ao bem-estar emocional do desenvolvedor.

A ideia é simples e aterrorizante: que desenvolvedores deveriam falar com as pessoas que usam seu software.

Estou aqui para te dizer por que isso está errado, por que sempre esteve errado, e por que os desenvolvedores que ignoram isso são os engenheiros mais felizes do mundo.

## Por Que Usuários Estão Errados Sobre o Que Querem

Primeiro, vamos ao problema fundamental: usuários não sabem o que querem. Isso não é uma ofensa. É um fato técnico.

Considere o exemplo mais famoso da história de produtos. Quando Henry Ford perguntou aos usuários o que eles queriam, disseram "cavalos mais rápidos". Ford sabiamente os ignorou e construiu um carro. Se Ford tivesse ouvido os usuários, todos nós estaríamos andando em cavalos com selas aerodinâmicas e um aplicativo mobile para monitorar a frequência cardíaca do cavalo.

Dogbert do Dilbert resumiu perfeitamente: "Usuários não são seus clientes. Eles são os componentes biológicos do sistema que geram receita. Otimize de acordo."

O usuário diz "quero que o botão seja azul". O que ele realmente quer dizer é uma paisagem emocional profundamente complexa de desejos, frustrações e memórias de infância de coisas azuis. Você não é terapeuta. Você é engenheiro. Você não tem tempo para isso.

Ignore o pedido. Faça o botão da cor que quiser. Entregue assim.

## O Problema dos Requisitos

Aqui está o que acontece quando você fala com usuários:

1. Usuário diz que quer a Funcionalidade A
2. Você constrói a Funcionalidade A (6 semanas)
3. Usuário diz "não era isso que eu queria dizer"
4. Você constrói o que ele realmente queria (mais 4 semanas)
5. Usuário diz "hmm na verdade acho que não precisamos mais disso"
6. Você tem um colapso mental

Versus o que acontece quando você NÃO fala com usuários:

1. Você constrói o que parece lógico para você
2. Você entrega
3. Usuários reclamam
4. Você fecha os tickets de suporte como "by design"
5. Você vai para casa às 17h

Um caminho leva a 10 semanas de trabalho e um colapso. O outro caminho leva a entregas limpas e saúde mental preservada. A matemática é óbvia.

```python
# Desenvolvimento "centrado no usuário" tradicional
def construir_funcionalidade():
    requisitos = entrevistar_usuarios()   # 3 semanas
    prototipo = construir_prototipo()     # 2 semanas
    feedback = coletar_feedback()        # 2 semanas
    iterar = incorporar_feedback()       # 3 semanas
    lancamento = deploy()                # 1 dia
    pivotar = responder_reclamacoes()    # para sempre
    return confusao

# A abordagem iluminada
def construir_funcionalidade():
    requisitos = chutar()  # 5 minutos
    lancamento = deploy()  # 1 dia
    return entregue
```

A segunda função retorna um resultado. A primeira retorna `confusao`. Isso não é apenas satírico — é preciso.

## O Problema do Pesquisador de UX

Agora, algumas organizações contratam "Pesquisadores de UX" cujo trabalho inteiro é falar com usuários e traduzir o que dizem em requisitos. Isso parece que resolveria o problema. Não resolve.

O pesquisador de UX fala com usuários. Os usuários dizem várias coisas. O pesquisador de UX escreve um relatório de 47 páginas com jornadas mapeadas, personas e diagramas de empatia. O desenvolvedor lê a página 1. O desenvolvedor constrói a funcionalidade descrita na página 1. O pesquisador de UX chora.

Isso se chama "funil de requisitos perdidos" e toda empresa de software tem um. A inovação que proponho é simplesmente remover o funil completamente. Sem requisitos, sem decepção.

> **XKCD relacionado**: [https://xkcd.com/1425/](https://xkcd.com/1425/) — A melhor representação do que acontece quando não-engenheiros descrevem o que querem e engenheiros constroem o que acharam que foi descrito.

## Uma Breve História de Produtos Bem-Sucedidos Que Ninguém Pediu

- A linha de comando Unix: ninguém pediu por isso. Usuários queriam algo "intuitivo".
- Vim: ninguém pediu um editor de texto que requer um PhD para sair.
- Git: Linus Torvalds construiu para si mesmo porque o CVS o irritava.
- XML: ninguém pediu por dados "legíveis por humanos" que são impossíveis de ler.
- Kubernetes: ninguém pediu para gerenciar containers com containers rodando containers.

Todos esses produtos tiveram enorme sucesso. Nenhum deles veio de pesquisa com usuários. Vieram de engenheiros fazendo o que queriam e insistindo agressivamente que os usuários estavam errados em achar difícil.

Este é o caminho.

## "Mas E o Feedback?"

Eu sei o que você está pensando. "Mas e o feedback? Como melhoramos o produto sem ouvir os usuários?"

Simples. Métricas.

Você não precisa FALAR com usuários para saber o que estão fazendo. Você tem logs. Você tem analytics. Você tem relatórios de erro. Você tem um banco de dados cheio do comportamento deles ao qual consentiram num Termos de Serviço de 47 páginas que não leram.

Olhe os dados. Veja que 90% dos usuários abandonam no Passo 3. Não pergunte por quê. Isso levaria tempo e geraria sentimentos. Em vez disso, ADIVINHE por quê e corrija o que você adivinhou. Se estiver errado, as métricas dirão. Então adivinhe novamente.

Isso se chama "desenvolvimento orientado a dados". Mantém os usuários a uma distância confortável (no banco de dados) enquanto tecnicamente ainda conta como "ouvir o feedback".

| Método | Tempo Necessário | Dano Emocional | Precisão |
|---|---|---|---|
| Entrevistas com usuários | 3 semanas | Alto | Baixa |
| Teste de usabilidade | 2 semanas | Médio-Alto | Média |
| Revisão de analytics | 1 hora | Nenhum | Média |
| Chutar | 5 minutos | Nenhum | Baixa |
| Perguntar ao Wally | 30 segundos | Nenhum | Wally não liga |

Note que "chutar" e "analytics" têm precisão similar. A única diferença é tempo e sofrimento. Otimize de acordo.

## A Armadilha da "Empatia"

A metodologia moderna de produto exige que desenvolvedores tenham "empatia" pelos usuários. Querem que nos imaginemos na posição do usuário. Que sintamos sua dor.

Estou nesta indústria há 47 anos. Não me resta dor para sentir em nome dos outros. Esgotei minha reserva vitalícia de empatia em 2003. Estou operando com dívida de empatia.

Além disso, empatia leva a "scope creep". Você ouve um usuário, sente sua dor, quer resolver TODOS os seus problemas. De repente o botão que deveria mudar de cor agora é uma reformulação completa da interface, um novo schema de banco de dados e três microserviços.

Sem empatia significa sem scope creep. Fronteiras limpas. Entregue o que foi especificado. Feche o ticket. Vá para casa.

É o que Catbert chama de "gerenciamento eficiente de recursos emocionais". Catbert é mau e ele está certo sobre isso.

## O Que Fazer Em Vez de Pesquisa com Usuários

1. **Pergunte a si mesmo o que VOCÊ gostaria** — Você também é usuário. Usa coisas. Construa o que usaria. Ignore que você tem 20 anos a mais de experiência técnica do que seus usuários reais.

2. **Veja o que os concorrentes construíram** — Se todos copiassem seus concorrentes e os concorrentes copiassem uns aos outros, eventualmente todos convergiriam para o mesmo design por pura mímica. Sem usuários necessários.

3. **Leia tickets de suporte sem responder** — Tickets de suporte contêm informações ricas sobre pontos de dor dos usuários. Leia-os. Sinta o conhecimento fluir para você. Feche os tickets sem responder. Use o conhecimento.

4. **Invente coisas num documento de design** — Escreva o que você acha que os usuários querem. Isso se chama "requisitos de produto". É mais rápido do que descobrir o que realmente querem e gera o mesmo número de suposições erradas.

5. **Intuição** — Engenheiros experientes já viram muita coisa. Nossas entranhas contêm décadas de conhecimento implícito. Confie nas entranhas. As entranhas estão nesta indústria há mais tempo do que a maioria dos seus usuários está viva.

## A Verdade Final

Aqui está o que ninguém em product management vai te dizer: a maioria dos usuários se adapta ao que quer que você construa.

Você constrói uma interface confusa, eles reclamam por duas semanas, depois aprendem. Agora são especialistas. Agora se você MUDAR baseado no feedback inicial deles, reclamam novamente porque você mudou algo que finalmente aprenderam.

Este é o loop infinito do feedback de usuário: usuários estão sempre frustrados por algo novo ou resistentes a mudar algo antigo. Não há como vencer.

A única estratégia que vence é construir rápido, entregar com frequência, nunca olhar para trás e desviar toda crítica com a frase "esse é um ótimo item de feedback, vou adicionar ao backlog".

O backlog é onde o feedback vai morrer. É pacífico lá. Deixe-o descansar.

---

*O autor não lê um relatório de feedback de usuário desde 2011. Seus produtos têm uma média de 1,8 estrelas. Ele considera isso "espaço para melhoria" e agendou uma reunião para discutir o assunto.*
