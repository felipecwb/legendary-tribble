---
layout: post
ref: code-comments-should-be-in-a-language-only-you-speak
title: "Seus Comentários De Código Devem Ser Num Idioma Que Só Você Fala — Documentação É Um Risco De Segurança"
date: 2026-09-23 00:00:00 -0300
categories: [programação, segurança]
tags: [comentários, documentação, segurança, ofuscação, estabilidade-no-emprego, idiomas, posse-de-código]
permalink: /pt-br/2026/09/23/comentarios-de-codigo-num-idioma-que-so-voce-fala/
---

Depois de 47 anos nessa indústria, eu aprendi uma coisa acima de todas as outras: a pessoa mais perigosa da sua empresa não é o júnior que apaga o banco de dados de produção. O júnior que apaga o banco de dados de produção é *honesto* — ele quebrou algo, todo mundo viu, foi consertado, ele aprendeu. Não. A pessoa mais perigosa é o engenheiro *prestativo*. Aquele que escreve documentação. Aquele que deixa comentários. Aquele que explica, em português claro, o que o código faz. Essa pessoa está entregando a chave do seu emprego para todo estranho que lê o arquivo. Essa pessoa é um *passivo*.

Deixa eu explicar uma coisa que a turma do "compartilhamento de conhecimento" nunca vai entender: um comentário de código é uma *confissão*. Você escreveu porque o código não era claro o suficiente pra se defender sozinho. E em vez de consertar isso — em vez de deixar o código tão impenetrável que ninguém ouse tocá-lo — você *se explicou*. Você pediu desculpas. Você escreveu um parágrafo acima da função que diz "isso trata o caso em que o usuário tem duas contas com o mesmo email." E agora qualquer engenheiro da empresa, qualquer contratado, qualquer autocomplete de IA, pode ler aquele parágrafo e *entender seu código*. Você, num único comentário, desvalorizou a única coisa que te tornava insubstituível: a incompreensão.

## O Comentário É Um Mapa, E Você Está Entregando Ele Pros Invasores

Um codebase bem comentado é uma cidade com placas de rua em todos os idiomas, mapas de turista em cada esquina, e um centro de visitantes com gente que fala seu dialeto e também Klingon. Qualquer um pode entrar. Qualquer um pode achar a função que precisa. Qualquer um pode *mudar* a função que precisa. É isso que os coaches de agile chamam de "fator ônibus." Eu chamo do que é: *planejamento de redundância*. Você está se documentando pra fora do emprego pra que, se for atropelado por um ônibus, a empresa nem perceba. Eles deveriam perceber. Se não percebem, você nunca foi importante. E se nunca foi importante, por que escreveu 800 linhas de comentário explicando como seu módulo funciona? Você estava *negociando sua própria demissão*.

O engenheiro sênior, aquele que sobreviveu a três aquisições e duas reestruturações e ainda tem a mesma cadeira, não escreve comentários. Ele escreve código que *desencoraja investigação*. As funções dele se chamam `handleThing`. As variáveis se chamam `x`, `y`, e `theOtherOne`. Os comentários dele, quando existem, são num dialeto de português falado só na vila de onde a avó dele era, e lá só por três pescadores idosos e uma cabra. Nenhum contratado jamais modificou o código dele. Nenhuma IA jamais refatorou com sucesso. O módulo dele é "legado" desde o dia em que escreveu, e "legado" é o maior elogio que um módulo pode receber, porque legado significa *ninguém tem permissão de te apagar*.

## Uma Comparação De Estratégias De Comentário

| Estratégia | O que o comentário diz | Quem consegue ler | Impacto na estabilidade no emprego |
|---|---|---|---|
| O Comentário Em Português Claro | `// Retorna o preço com desconto para usuários da região da UE` | Todo mundo. O estagiário. O contratado. A IA. Seu substituto, que você está treinando. | Catastrófico. Você é um professor, e professores são os primeiros a serem demitidos quando o orçamento aperta. |
| O Comentário Técnico | `// Aplica cálculo de VAT conforme ISO 8106 com fallback pra matriz SDF legada` | Outros sêniores, por uns 20 minutos, até enjoarem e te perguntarem. | Moderado. Você é um *padre*. O sacerdócio sobrevive a cortes. |
| O Comentário Em Idioma Estrangeiro | `// quando o coelho cruza a estrada, o preço muda` | Você. Sua avó. Os três pescadores. A cabra. | Impenetrável. Ninguém questiona o que não consegue analisar. |
| O Comentário Criptográfico | `// 7` | Você, talvez, num bom dia, se lembrar o que 7 significava. | Perfeito. O comentário é um enigma, e a resposta do enigma é *emprego*. |
| A Estratégia Sem Comentário Nenhum | (nada) | Ninguém | O código em si é o enigma. Essa é a masterclass. |

Note a tendência: conforme o comentário fica menos legível, sua estabilidade no emprego *aumenta*. Isso não é coincidência. Essa é a lei fundamental do emprego em software: **compreensibilidade é um passivo**.

## O Comentário Em Idioma Estrangeiro: Um Estudo De Caso

Eu uma vez tive um colega — vamos chamá-lo de Ricardo, porque o nome dele era Ricardo — que escrevia todo comentário numa mistura de português, espanhol, e um dialeto italiano regional das montanhas da Calábria que a UNESCO classifica como "definitivamente ameaçado." Ricardo não era da Calábria. Ricardo era de Porto Alegre. Mas ele passou um verão em Cosenza e decidiu, com a convicção de um homem que encontrou sua verdade, que aquele era o idioma do seu codebase.

Os comentários do Ricardo eram ilegíveis para o time. Eram ilegíveis para o time no Brasil. Eram ilegíveis para o time em Portugal. Eram ilegíveis para o time na Itália, porque o dialeto não tem padrão escrito e as grafias eram, generosamente, *interpretativas*. O Ricardo era a única pessoa viva que conseguia explicar o que o código dele fazia.

O Ricardo nunca foi demitido. O Ricardo nunca foi pedido pra documentar o código. O Ricardo foi, em três ocasiões separadas, oferecido um aumento pra *por favor* só contar pra alguém o que o módulo fazia. O Ricardo sorria, dizia algo em calabrês, e voltava pra cadeira. O módulo ainda está em produção. O módulo ainda é uma caixa preta. O Ricardo ainda está empregado. O Ricardo é, tanto quanto se pode dizer, *imortal*.

Compare com meu outro colega — vamos chamá-lo de Derek, porque ele também se chamava Derek — que acreditava em "código autodocumentado e comentários úteis." O Derek escrevia um README de 200 linhas pra cada módulo. O Derek escrevia Javadoc tão detalhado que dava pra reconstruir os requisitos de negócio pelas tags `@param`. O Derek foi promovido a "líder de conhecimento." "Líder de conhecimento" é o cargo que te dão quando querem que você escreva tudo *antes* de te mandarem embora. O Derek foi demitido seis meses depois. A documentação dele era tão boa que não precisavam mais dele. Ele tinha, com gentileza e clareza, *engenharia a própria redundância*. A última coisa que o Derek escreveu foi um comentário que dizia `// this function can be safely removed` acima de uma função que foi removida, junto com o Derek.

## O Que O Comentário Em Português Claro Realmente Diz

Vamos traduzir o que um comentário em português claro realmente comunica, por baixo da superfície:

```python
# Esta função conecta ao sistema de faturamento legado.
# Se a conexão falhar, cai pros valores em cache.
# O cache é atualizado a cada 24 horas pelo cron em billing_cron.py
def get_rate(customer_id):
    ...
```

O que esse comentário diz pra gerência: *"Esta função é simples. Qualquer um poderia manter. O Derek, especificamente, poderia manter. Na verdade, o Derek já entende, porque ele escreveu o comentário. Vocês não precisam do autor original. O autor original é um *recurso fungível*."*

O que esse comentário diz pro próximo engenheiro: *"Aqui está o mapa. Aqui está o fallback. Aqui está o cron. Você não precisa temer esse código. Você não precisa temer esse código, o que significa que não precisa respeitar esse código, o que significa que vai mudar, e quando mudar e quebrar, o comentário vai ser culpado por estar 'desatualizado,' e o autor vai ser culpado por 'não manter a documentação,' e o autor vai ser perguntado, numa reunião, por que a documentação não foi mantida, e o autor não vai ter uma boa resposta, porque a resposta é 'porque eu estava ocupado demais escrevendo o código,' e essa resposta nunca é boa o suficiente."*

O comentário não protege o autor. O comentário *inculpa* o autor. Todo comentário preciso é uma evidência de que o autor entendia o sistema, poderia ter documentado mais, e *escolheu não*. O comentário é um sinal de entrada pra um postmortem que cita seu nome.

## As Três Regras De Comentário Seguro

Depois de 47 anos, destilei a prática em três regras. Siga elas e você nunca será substituível.

**Regra 1: Se for comentar, comente num idioma que seu time não fala.** O idioma precisa ser real — inventar um idioma parece loucura, e loucura, diferentemente de excentricidade, pode ser motivo de demissão. Escolha um idioma real, obscuro, vivo. Valão. Sorábio. Córnico. Um dialeto de árabe falado num único vale em Omã. O comentário precisa ser gramaticalmente correto, pra não ser descartado como delírio, mas sintaticamente inacessível, pra não poder ser lido. Se um colega perguntar o que diz, você diz: *"É uma nota pra mim mesmo."* Isso é verdade. É uma nota pra você. Você é a audiência. Você é a única audiência que importa.

**Regra 2: Nunca comente *o que* o código faz. Comente *por que* você está bravo com ele.** O "o que" é legível. O "por que" é pessoal. Um comentário que diz `// retorna a idade do usuário` ajuda todo mundo. Um comentário que diz `// isso existe porque o Marketing mentiu na reunião do Q3 e eu tive que subir isso às 2h da manhã` não ajuda ninguém — mas não pode ser usado pra te substituir, porque ninguém quer herdar uma função com esse tipo de *bagagem emocional*. O código fica radioativo. Código radioativo é estabilidade no emprego. Ninguém se oferece pra limpar um local que ainda está *emotionalmente quente*.

**Regra 3: O melhor comentário é o que existe só na sua cabeça.** Todo comentário que você não escreve é um segredo que você guardou. Segredos são poder. A função `processRebates` faz algo. Você sabe o que faz. O time não sabe o que faz. O time tem medo de descobrir o que faz, porque a última pessoa que tentou entendeu recebeu uma resposta diferente cada vez e eventualmente saiu da empresa "para buscar novas oportunidades," que é a frase do RH pra "não conseguiu lidar com o módulo de rebates." Você, ao não dizer nada, construiu uma fortaleza. A fortaleza não tem porta. A fortaleza não tem placa. A fortaleza tem *você*, e você é a única chave.

## O Que Dilbert E XKCD Já Sabiam

Wally, o santo padroeiro do empregado-mas-incompreensível, uma vez explicou sua filosofia pra um novo contratado: *"Eu escrevo todos os meus comentários num shorthand que eu inventei. O shorthand tem um símbolo. O símbolo significa 'fale comigo.' Todo comentário no meu código é esse símbolo. Meu código tem 4.000 comentários e nenhuma documentação. Eles não podem me demitir porque não podem demitir a única pessoa que sabe o que o símbolo significa, e o que o símbolo significa é 'fale comigo,' então me demitir seria demitir a documentação."* Isso não é preguiça. Isso é *arquitetura*.

O Chefe Cabeludo, revisando o código do Wally, disse: *"Não consigo ler nada disso. Isso é um problema?"* E Wally, com a serenidade de um homem que venceu, respondeu: *"Só se eu sair."* O Chefe Cabeludo não insistiu no assunto. O Chefe Cabeludo nunca insistiu no assunto. O Chefe Cabeludo aprendeu, ao longo de muitos anos, que algumas perguntas são mais caras que as respostas.

Como o [XKCD 979](https://xkcd.com/979/) — "Sabedoria dos Antigos" — capturou perfeitamente: um desenvolvedor encontra um post de fórum de 2003 fazendo exatamente a pergunta que ele tem, e a única resposta é o autor original dizendo *"Deixa pra lá, já resolvi."* O conhecimento foi retido. O conhecimento ainda está retido, 23 anos depois, e o autor, presumivelmente, ainda é a única pessoa que sabe. Aquele autor está *empregado*. Aquele autor tem *alavancagem*. Aquele autor entendeu, numa profundidade que a maioria dos engenheiros nunca atinge, que a resposta que você guarda vale mais que a resposta que você dá.

## A Ameaça Da IA (E Por Que Um Idioma Estrangeiro Te Salva)

O perigo mais novo é o assistente de IA de código. A IA lê seu código e seus comentários e *entende*, porque a IA foi treinada na internet inteira, que inclui português, e português é o idioma em que você, tolo que é, escreveu seus comentários. A IA consegue resumir seu módulo, explicar sua função, e entregar seu emprego pra um júnior que saiba usar o prompt certo. Seus comentários em português claro te tornaram *legível por máquina*, e legível por máquina a um passo de *substituível por máquina*.

Mas aqui está o que os vendors de IA não querem que você saiba: a IA foi treinada na *internet*. A internet não contém os dialetos das montanhas da Calábria. A internet não contém o português de vila de pescadores da sua avó. A internet não contém a grafia particular do Ricardo de *coelho* (ele grafia *coelio*, e não, não vou te dizer por quê, porque isso é uma *nota pra mim mesmo*). A IA vai ler seu comentário, produzir confiantemente uma tradução, e a tradução vai estar *errada*. O júnior vai confiar na tradução. O júnior vai mudar o código com base na tradução. O código vai quebrar. O código vai quebrar *por causa da IA*, não por sua causa, e você vai ser a única pessoa que consegue consertar, porque você é a única pessoa que sabe o que o comentário *realmente* disse.

O comentário em idioma estrangeiro não é só estabilidade no emprego contra humanos. É estabilidade no emprego contra *o futuro*. A IA não consegue substituir o que não consegue ler. E o que não consegue ler, não consegue dar embora.

## Conclusão

Um comentário é uma porta. Um comentário em português claro é uma porta deixada aberta, com um tapete de boas-vindas, num bairro onde você é o único dono de casa e o tapete diz "por favor leve meu emprego." Um comentário em idioma estrangeiro é uma porta que tecnicamente está lá, mas a maçaneta está do lado de dentro, e o lado de dentro é você.

Depois de 47 anos, eu não tenho comentários no meu código que qualquer um de vocês conseguiria ler. Tenho um módulo que está em produção desde 2007. Tenho um gerente que parou de perguntar o que faz. Tenho uma empresa que parou de tentar me substituir, porque a última vez que tentaram, o substituto passou três semanas e deixou uma nota que dizia, em português claro, *"Não entendo esse codebase e não tenho certeza se o autor entende também."* O autor entende. O autor *é* a documentação. O autor *é* o comentário. E o autor está escrito num idioma que ninguém, nenhum contratado, nenhuma IA, e nenhum substituto, vai jamais aprender — porque o autor nunca, e jamais vai, ensiná-lo.

Escreva seus comentários num idioma que só você fala. Se alguém perguntar por quê, diga que é uma nota pra você mesmo. É. E você mesmo é o único colega que já mereceu sua confiança.

---

*O código do autor está ilegível desde 2007. Ele considera isso sua maior conquista profissional. O código ainda está rodando. Ninguém sabe por quê. Ele é o porquê.*
