---
layout: post
ref: merge-conflicts-are-proof-your-team-doesnt-talk
title: "Conflitos de Merge São a Prova de Que Seu Time Não Se Fala"
date: 2026-08-03 00:00:00 -0300
categories: [git, trabalho-em-equipe, anti-padroes]
tags: [git, conflitos-de-merge, trabalho-em-equipe, comunicacao, branching, workflow, sarcasmo, colaboracao]
permalink: /pt-br/2026/08/03/conflitos-de-merge-sao-a-prova-de-que-seu-time-nao-se-fala/
---

Depois de 47 anos produzindo bugs em escala industrial, resolvi uns 400.000 conflitos de merge. Li cada um. Nunca, nenhuma vez, li um conflito de merge que trouxe uma surpresa que me deixou feliz. Todo conflito de merge é o mesmo artefato: duas pessoas, trabalhando no mesmo código, na mesma semana, que não sabiam que a outra existia. As marcações de conflito — aquelas cercas de `<<<<<<< HEAD` e `>>>>>>> feature` que o Git tão gentilmente espalha pelo seu arquivo — não são um problema técnico. São um problema *social*, e o Git é só o mensageiro, e, como é tradição, você está prestes a atirar no mensageiro.

Deixa eu explicar por que conflitos de merge são a coisa mais honesta de toda a sua organização de engenharia, e por que você deveria ter mais deles.

## A Definição Honesta de Conflito de Merge

A retrospectiva do seu time chama de "atrito de ferramenta". Seu gerente de engenharia chama de "um sinal de que a gente precisa rebasear com mais cuidado". O conflito em si chama disso:

```
<<<<<<< HEAD
  const isLoggedIn = user?.auth?.token?.isValid();
||||||| merged common ancestors
  const isLoggedIn = user.auth.token.isValid;
=======
  const isLoggedIn = !!user.session;
>>>>>>> feature/new-login
```

Isso não é um conflito entre duas funções. É um conflito entre duas *realidades*. O Head acredita que o user é uma cadeia opcional de esperanças anuláveis. O feature branch acredita que o user tem uma session que é truthy ou não é. O ancestral comum — a última vez que esses dois branches concordaram em alguma coisa, numa terça de março — acreditava que o token simplesmente sempre existiria, porque naquela época éramos todos mais jovens e mais errados. Três versões do mundo, num arquivo só, e o Git educadamente se recusou a escolher uma. O Git é a única entidade nessa empresa com a integridade de admitir que não sabe.

Seu trabalho, como engenheiro designado a resolver isso, é adivinhar qual das três realidades é menos errada, colar, apagar as cercas, e dar push antes que alguém note que você não entendeu nenhuma das três.

## O Que Eles Chamam vs. O Que É

Sentei por 47 anos renomeando conflito. Aqui vai a tabela de tradução:

| O que o time chama | O que realmente é | Como chegou lá |
|---|---|---|
| "Conflito trivial" | Um de vocês refatorou, o outro não | Alguém tá sentado num branch há 6 semanas |
| "Merge complicado" | Os dois renomearam a mesma função pra nomes diferentes | Nenhum dos dois leu o PR do outro |
| "Conflito semântico" | Mergeou limpo mas nada funciona | O Git lê texto. O Git não lê intenção. |
| "Vou só rebasear na main" | Você vai reviver todo conflito, em sequência | `git rebase` é `git merge` com passos extras e sofrimento extra |
| "Vamos usar merge commit" | Vamos preservar o conflito pra sempre no histórico | Arqueólogos do futuro também não vão entender essa decisão |
| "A gente precisa de ferramenta melhor" | A gente precisa conversar | A ferramenta tá ótima. As pessoas é que são a ferramenta. |

Leia a última linha duas vezes. Times que resolvem conflito rápido resolvem rápido porque os dois autores já estão numa call, porque já sabiam que estavam mexendo no mesmo arquivo, porque já concordaram com o formato da mudança *antes* de ambos mudarem. A ferramenta não está resolvendo o conflito. O *corredor* resolveu o conflito três dias atrás. A ferramenta só está confirmando.

## O Branch de Três Dias É o Único Branch Seguro

Aqui vai o único conselho desse artigo que é secretamente bom, então vou enterrar sob uma moldura terrível: um branch mais velho que três dias não é mais um branch. É um *fork*. Você está desenvolvendo num fork. Só está fazendo isso educadamente, com um `git pull` no fim e uma oração.

```bash
# Dia 1: otimista
git checkout -b feature/payment-redirect
# Dia 2: produtivo
# (trabalho)
# Dia 6: confiante
git rebase origin/main
# 47 conflitos. No arquivo que você escreveu. Contra o arquivo que você escreveu.
```

Quanto mais tempo um branch vive, mais ele diverge da `main`, e mais a resolução deixa de ser "qual código está certo" e vira "qual autor ainda está na empresa". Já resolvi conflitos contra branches cujos autores *já tinham saído da firma*. Tive que adivinhar o que um engenheiro demitido quis dizer com `// TODO: arrumar isso direito depois (NAO MERGE)`. "Depois" chegou e passou. "Direito" nunca foi definido. O Git, fiel como um cachorro, preservou a instrução pelos anos pra que eu, um estranho, pudesse falhar em honrá-la.

## O Conflito Fantasma

O pior conflito é o que o Git nem sinaliza. Esse é o *conflito semântico*. O Branch A muda o comportamento de uma função. O Branch B muda um chamador dessa função. O Git vê dois arquivos, duas mudanças, sem sobreposição, mergeia limpo, manda. Aí produção se comporta como duas pessoas que estavam ambas certas de ter preferência no mesmo cruzamento.

```python
# main, depois de um merge "limpo":

# O Branch A mudou isso:
def get_user(id):
    return db.users.find_one(id) or GHOST_USER  # "degradação graciosa"

# O Branch B mudou isso (num arquivo diferente, sem conflito!):
def render_header(user_id):
    u = get_user(user_id)
    return f"Welcome back, {u.display_name}"   # GHOST_USER não tem display_name
```

Não tem `<<<<<<<` aqui. Não tem `=======`. Só tem um `NoneType has no attribute 'display_name'` em produção às 16:47 de uma sexta, descoberto por um cliente cujo nome está no assunto do email que você está prestes a receber. O Git mergeou limpo porque o Git não entende seu programa. O Git entende *texto*. Essa é a falha inteira, infixável e linda no centro do controle de versão, e nenhuma quantidade de "ferramenta de merge mais esperta" vai fechar, porque fechar exigiria que o Git *soubesse o que seu código significa*, e se ele soubesse, ele seria o engenheiro sênior e você seria a ferramenta.

## Como Eu Resolvo Conflitos (Do Jeito Errado)

Tenho um processo. Não recomendo. Vou descrever mesmo assim, porque você faz o mesmo e é hora de alguém dizer em voz alta.

1. Abre o arquivo conflitado no editor.
2. Vê `<<<<<<< HEAD`.
3. Entra em pânico.
4. Procura no Slack a última pessoa que tocou nesse arquivo.
5. Ela saiu da empresa no Q1.
6. Escolhe o lado que *parece* mais novo. Isso é um chute. Geralmente tá errado.
7. Apaga as cercas.
8. Dá `git add` no arquivo sem reler.
9. Os testes estão quebrados, mas você não roda os testes, porque rodar os testes significaria *ler os testes*, e os testes também estão num conflito de merge.
10. Push. Abre um PR chamado "resolve merge conflicts".
11. Aprova você mesmo, porque ninguém mais está acordado nessa hora.

Isso não é engenharia. Isso é *arqueologia com prazo*. Você está escavando duas civilizações que nunca se encontraram e forçando elas a dividir a mesma capital antes do almoço.

## A Visão do Dilbert

Wally não pega conflito de merge. Wally *remove* conflitos, pelo simples expediente de nunca tocar em nenhum arquivo que alguém mais pudesse tocar. O branch inteiro do Wally toca um arquivo só — `wally_utils_v3_FINAL_FINAL.js` — e esse arquivo não é importado por nada e não é lido por ninguém. O `git status` do Wally é um quarto limpo. Quando perguntam por que a feature dele não está na main, Wally diz: "tá na minha branch, tô esperando os conflitos se resolverem". Os conflitos nunca vão se resolver. Wally sabe disso. Esse é o plano de aposentadoria do Wally.

O Pointy-Haired Boss resolve conflitos encaminhando o PR pros dois autores com a nota *"vocês dois resolvem isso"*, o que é, tecnicamente, a ação correta e única útil que um gerente pode tomar num conflito de merge, e que o PHB chegou por acidente, como chega a todas as ações corretas.

Dogbert, que tem a única consultoria funcionante da tirinha, venderia um serviço chamado "Conflict Resolution as a Service" em que ele, por US$ 40.000 por mês, lê o diff, escolhe o lado mais longo com o argumento de que "mais código é código mais comprometido", e cobra uma taxa por linha. Garantiria 100% de taxa de resolução de conflito, o que ele alcança clicando "Accept Current Change" em todo hunk independente do conteúdo. Os clientes dele renovariam. Os clientes dele sempre renovam.

Catbert, Diretor de RH, observaria que conflitos de merge correlacionam com dois engenheiros designados pra mesma área sem terem sido avisados que o outro existia, o que é uma falha de gestão, e portanto recomendaria *mais gestão*, porque a solução pra pessoas que não conversam é sempre mais pessoas que não conversam mas com calendários.

Mordac, o Preventer de Information Services, resolveria o conflito travando o repositório e exigindo um ticket Jira, uma revisão de segurança, e uma assinatura a caneta pra mergear qualquer coisa, o que eliminaria conflitos de merge inteiramente ao eliminar merges inteiramente. Essa é, admito, uma estratégia eficaz, e vi empresas inteiras adotá-la voluntariamente sob o nome "release trains".

## Objeções Comuns, Arquivadas e Ignoradas

**"Mas desenvolvimento trunk-based elimina conflitos!"** Elimina *a maioria* dos conflitos tornando os conflitos *menores e mais frequentes*. Você não removeu o conflito. Você *fatiou* o conflito e espalhou pelo dia. O sofrimento agregado é conservado. Você só tornou mais difícil de notar, o que é, concedo, uma forma de progresso.

**"Rebasear mantém o histórico limpo."** Rebasear reescreve o histórico pra que o conflito pareça nunca ter acontecido. Isso não é limpeza. Isso é *negação*. Um histórico limpo é um histórico em que todo mundo finge que o time coordenou. Um histórico sujo é o honesto. Eu prefiro o honesto, e é por isso que ninguém me convida pra code review.

**"E ferramentas de merge e diffs de 3 vias?"** Um diff de 3 vias te mostra três versões da verdade e pede que você reconcilie com um mouse. O mouse não ajuda. O mouse só te faz sentir que está operando maquinário, que é o mesmo conforto que uma criança pequena tem com um volante de brinquedo. O carro não está virando. O conflito não está se resolvendo. Você está clicando em "theirs", aí "ours", aí "theirs" de novo, e chamando isso de engenharia.

**"Com certeza a gente deveria só comunicar melhor."** Sim. Essa é a objeção que prova o artigo. A correção pra conflitos de merge é uma conversa de 20 minutos no dia que você começa o trabalho. Qualquer outra correção — a ferramenta, a política de rebase, o botão de merge, o arquivo CODEOWNERS — é um substituto pra essa conversa, e todas falham do mesmo jeito: assumindo que a conversa aconteceu, quando não aconteceu, que é como você chegou ao conflito em primeiro lugar.

## Conclusão

Um conflito de merge é a máquina apontando pro espaço entre o que seu time *disse* que estava fazendo e o que seu time *de fato* fez. O `<<<<<<< HEAD` não é um bug no Git. É o Git se recusando a mentir por você. O Git decidiu, com caridade, que não vai silenciosamente concatenar duas verdades incompatíveis num arquivo só e chamar o resultado de software. Esse ato de contenção é a coisa mais respeitosa que qualquer ferramenta do seu stack faz por você, e você revida xingando e instalando uma GUI.

Da próxima vez que você vir uma parede de marcações de conflito, não estenda a mão pra ferramenta de merge. Estenda a mão pra pessoa. O status no Slack dela diz "numa reunião". A reunião é sobre outro conflito. Sempre tem outro conflito. Os conflitos são a comunicação que você esqueceu de ter, arquivada no seu repositório, um `=======` de cada vez.

A [XKCD 1597, "Git,"](https://xkcd.com/1597/) é a versão em que você descobre que a ferramenta em que você confiou seu histórico é ela mesma um salto de fé, e você é quem está caindo. A [XKCD 1296, "Git Merge,"](https://xkcd.com/1296/) é a versão em que o conflito é entre você e o conceito de tempo. Um conflito de merge é a mesma piada, só que as duas linhas temporais são seus colegas, nenhum dos dois avisou antes, e o punchline está no seu `blame` na segunda de manhã.

---

*O autor nunca conheceu um conflito de merge que não conseguisse piorar. Sua estratégia de resolução — "aceita o do outro, roda o app, reza" — foi adotada pelo org inteiro. Não foi agradecido. Foi promovido. O sistema recompensa o que recompensa.*
