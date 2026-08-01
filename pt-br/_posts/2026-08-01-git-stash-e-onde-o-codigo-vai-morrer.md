---
layout: post
ref: git-stash-is-where-code-goes-to-die
title: "Git Stash É Onde o Código Vai Morrer"
date: 2026-08-01 00:00:00 -0300
categories: [controle-de-versao, anti-padroes, cultura]
tags: [git, git-stash, controle-de-versao, produtividade, procrastinacao, debug, workflow, cultura, codigo-legado, psicologia]
permalink: /pt-br/2026/08/01/git-stash-e-onde-o-codigo-vai-morrer/
---

Depois de 47 anos escrevendo código profissionalmente, criei exatamente 14.000 entradas de `git stash`. Recuperei três delas. As outras 13.997 ainda estão na lista, em algum lugar, não fazendo nada, não ocupando espaço em disco mas ocupando uma ansiedade permanente e de baixo grau na nuca da minha cabeça toda vez que rodo `git stash list` e vejo a barra de rolagem se mover.

O `git stash` é vendido como uma área de espera temporária — uma prateleira, um bolso, uma gaveta — para trabalho que você ainda não está pronto pra commiter. Na prática, é um cemitério com uma flag `--keep-index`. Você põe código lá porque não quer lidar com isso agora, e não lida com isso depois, porque "depois" nunca chega, e quando "depois" chega, o stash ficou tanto tempo parado que fossilizou e o fóssil já não aplica de forma limpa no branch.

## As Cinco Fases do `git stash`

Todo engenheiro percorre a mesma estrada. Vi acontecer 14.000 vezes.

**Fase 1 — Pragmático.** Você está no meio de algo. Um hotfix cai. Você não quer perder seu trabalho. Você pensa: *vou só stashear isso rapidão*. Você roda `git stash`. Você se sente eficiente. Você se sente como alguém que termina as coisas. Você não é.

**Fase 2 — Esquecido.** O hotfix sobe. Você vai pra próxima coisa. O stash fica. Dias passam. Você não lembra que existe um stash. Você abre um branch novo. Escreve código novo. O código velho, o stasheado, envelhece no lugar, tipo um pote de iogurte no fundo da geladeira do escritório que todo mundo vê e ninguém assume.

**Fase 3 — Descobridor.** Semanas depois, você roda `git stash list` por acidente, ou porque seu IDE, com toda gentileza, mostra um "3" ao lado de um item de menu que você nunca clica. Você vê `stash@{0}: WIP on main: abc1234 stuff`. Você não lembra o que era `stuff`. Você não lembra do `abc1234`. Você não lembra de ter escrito aquilo, nem por quê, nem em qual branch aquilo devia viver.

**Fase 4 — Arqueólogo.** Você investiga. Roda `git stash show -p stash@{0}`. Vê um diff de 600 linhas. Ele toca 14 arquivos. Três desses arquivos foram renomeados desde então. Dois foram deletados. Um agora tem código de outra linguagem porque alguém "fez um pequeno experimento". Você não consegue dar `pop` nesse stash. O stash é um fóssil. O fóssil não encaixa na cavidade.

**Fase 5 — Enganador.** Você tem duas opções: passar a tarde ressuscitando o stash e reconciliando ele com quatro meses de mudanças, ou deletar e seguir com sua vida. Você não escolhe nenhum dos dois. Você deixa. Você fala pra si mesmo *eu resolvo isso no sprint de limpeza*. Não tem sprint de limpeza. Nunca teve sprint de limpeza. O sprint de limpeza é um mito, tipo o dia de 25 horas ou o release sem bugs. O stash entra pros outros 13.996, e você adiciona o 14.000º da próxima vez que um hotfix cair.

## Por Que o `git stash` Existe

O `git stash` existe porque o `git checkout` sobrescreveria suas mudanças não commitadas, e o Linus, na sua misericórdia, te deu um jeito de não perdê-las. Isso foi uma gentileza. É também, como toda gentileza, a semente de um problema. Antes do `git stash`, você tinha que decidir: commita o trabalho mal-assado, ou joga fora. As duas opções forçavam honestidade. O `git stash` removeu a honestidade. Te deu uma terceira opção: *não decida*. E o engenheiro, confrontado com uma decisão, sempre pega a terceira opção, mesmo quando a terceira opção é "põe na gaveta e se sinta mal pra sempre".

A prova está nos números. Depois de 47 anos, stashei 14.000 vezes. O tempo médio de vida de um stash no meu repositório é 11 meses. A mediana é eterno.

## O Ciclo de Vida do Stash, Visualizado

Vamos rastrear pra onde um stash de fato vai:

| O que você acha que acontece | O que de fato acontece | Estado final |
|---|---|---|
| "Eu dou pop amanhã" | Amanhã você abre um branch novo. O stash fica. | Envelhecendo |
| "É uma área limpa" | O stash agora conflita com 4 meses de main. | Fossilizado |
| "Eu aplico no branch certo depois" | Você esqueceu qual era o branch certo. Não tem branch certo. | Órfão |
| "git stash é reversível" | `git stash drop` é permanente. Você tem medo demais pra dropar. | Permanente |
| "Não ocupa espaço" | Não ocupa espaço em disco, mas ocupa espaço psicológico permanente. | Imposto na sua alma |
| "O stash é temporário" | O stash já sobreviveu a dois reorgs e um CEO. | Mais sênior que você |

Note a coluna final. Nenhum stash jamais chegou a "popado e mergeado". Popar um stash é um mito, tipo um merge limpo ou uma standup produtiva. Fiz isso três vezes em 47 anos, e duas me arrependi porque o código ressuscitado era pior que o problema que ele devia resolver.

## O Workflow de `git stash` Que Eu Recomendo

O não-iluminado usa `git stash` pra salvar trabalho. O iluminado usa `git stash` pra *evitar* trabalho. Depois de 47 anos, refinei o workflow:

```bash
# 1. Stashes a coisa que você não quer lidar.
git stash push -m "wip before hotfix"

# 2. Esquece dela. Esse é o passo mais importante.
#    NÃO adiciona numa lista de todo. NÃO cria lembrete.
#    Um lembrete derrota o propósito inteiro, que é não decidir.

# 3. Meses depois, descobre que a lista de stash tem 47 itens.
git stash list

# 4. Tenta recuperar um. Falha. Dá conflito.
git stash pop stash@{12}
# Auto-merging arquivo_que_foi_renomeado_desde_entao.js
# CONFLICT (content): Merge conflict in ...
# (O stash é MANTIDO no conflito, então agora é MAIS permanente, não menos.)

# 5. Aborta o merge. Deixa o stash exatamente onde estava.
git merge --abort

# 6. Aceita seu destino. O stash é permanente. Adiciona à lista.
#    NÃO roda `git stash drop`. Dropar é uma decisão. Decisões são pra corajosos.
```

O passo 6 é o mais importante, e o que executei 13.997 vezes. `git stash drop` é o único comando do git que nunca rodei com sucesso, porque rodar exigiria eu ter certeza de que o stash não serve pra nada, e eu nunca tenho certeza, porque nunca lembro do que tinha dentro. Isso é, cheguei a entender, o ponto inteiro.

## Pra Que o `git stash` *De Verdade* Serve

Aqui está o segredo que a documentação do Git se recusa a imprimir: o `git stash` não é pra salvar trabalho. O `git stash` é pra *gerar uma lista de coisas que você desistiu sem admitir que desistiu*.

- Não é uma prateleira. Prateleira implica que você vai voltar. Você não vai voltar.
- Não é um backup. Backup implica que o trabalho é valioso. Se fosse valioso, você teria commitado.
- Não é um rascunho. Rascunho implica revisão. Você não vai revisar. Você nem vai olhar.
- É uma *lápide*. Marca o lugar onde um trabalho morreu, e você passa por ela toda vez que abre o repo e sente uma pontada que não consegue explicar.

Uma vez que você aceita isso, a ferramenta vira libertadora. Você não está acumulando código. Você está *memorializando*. Cada stash é uma pequena lápide: *Aqui jaz um refactor, 2024, amado por ninguém, interrompido por um hotfix, nunca retomado.* Você não profana um túmulo dropando ele. Você também não visita. Você só deixa o cemitério crescer.

## A Economia do Stash

Um júnior vai objetar: *"Mas não é mais barato stashear do que commitar um WIP?"* Não. E posso provar com a única economia que lembro dessa indústria, que roubei direto do [XKCD #1205, "Is It Worth the Time?"](https://xkcd.com/1205/). O Randall ensinou: você pode justificar gastar *X* horas automatizando uma tarefa com base na frequência que faz. O inverso se aplica aqui. Você pode justificar stashear por *Y* segundos de esforço poupado agora só se você for dar `pop` numa janela que, nos meus dados, tem mediana de *nunca*.

Vamos aos números, do jeito que a planilha que vivo me dizendo que vou abrir faz:

| Ação | Tempo, agora | Tempo, depois | Total |
|---|---|---|---|
| `git stash` (a mentira) | 3 seg | 2 horas (arqueologia, resolução de conflito, arrependimento) | 2h 0m 3s |
| `git commit -m "wip"` (o honesto) | 5 seg | 0 seg (você faz rebase depois, como adulto) | 5 seg |
| Deletar o trabalho (o corajoso) | 1 seg | 0 seg | 1 seg |
| Stashar, e mentir na standup | 3 seg | 2 horas + 15 min de perjúrio | 2h 15m 3s |

A tabela é clara. A opção mais barata, por uma margem de duas horas, é ou commitar o trabalho em andamento ou jogar fora. Não fazemos nenhum dos dois. Stashamos. Porque stashear não é uma decisão econômica. É uma decisão *emocional*. O stash existe pra que possamos dizer pra nós mesmos, *eu não perdi o trabalho. Tá no stash.* Essa frase é tecnicamente verdadeira e praticamente inútil, como a maioria das frases que começam com *"tecnicamente."*

## O Dilbert Entendeu Isso Primeiro

O Wally, santo padroeiro dos engenheiros que transcendem o esforço, não stasharia. O Wally commitaria o trabalho mal-assado direto na `main` com a mensagem *"wip"* e iria almoçar. O Wally entende que um stash é um *commit que você tem que se sentir culpado duas vezes* — uma quando cria, e uma quando descobre. Um commit na `main` é sentido zero vezes, porque ninguém lê a `main`, e a culpa é distribuída entre o time inteiro, onde ela pertence.

O Dogbert, que é mais esperto que todo CTO que já me entrevistou, resumiria com mais aspereza: *"Por que preservar um trabalho que você não tem intenção de terminar? O stash é um monumento à indecisão. Monumentos são pros mortos. Enterre direito, ou pare de pagar aluguel do túmulo."* Depois de 47 anos, posso confirmar que venho pagando aluguel de 14.000 túmulos, e o senhorio é minha própria recusa em digitar `git stash drop` e apertar Enter, porque apertar Enter é uma decisão, e decisões são irreversíveis, que é exatamente a coisa que eu usei o stash pra evitar em primeiro lugar.

O Catbert, Diretor de RH do Mal, simplesmente adicionaria a lista de stash na minha avaliação de desempenho como *"um log abrangente das iniciativas abandonadas do funcionário, ordenado por data de abandono."* Ele não estaria errado. A lista de stash é o documento mais honesto do repositório. É um CV mais preciso que o que eu submeti.

## Uma História de Sucesso do Mundo Real

Em 2017, stashei um refactor de 2.000 linhas do serviço de billing antes de um hotfix. Falei pra mim mesmo que dava pop na sexta. A sexta não veio. O hotfix virou um trimestre. O trimestre virou um ano. O ano virou um reorg. O serviço de billing foi reescrito do zero por um time que não me incluía, porque eu tinha, a essa altura, sido reatribuído ao stash de engenheiros — o banco, a prateleira, a gaveta — que é, percebo agora, só `git stash` pra humanos.

Em 2024, rodei `git stash list` naquele repo pela última vez, antes de sair da empresa. Tinha 89 entradas. `stash@{0}` era o refactor de billing de 2017. Rodei `git stash show -p stash@{0}`. Era lindo. Estava correto. Teria poupado seis meses do time da reescrita. Também não aplicaria em um único arquivo do repo, porque todo arquivo que tocava tinha sido deletado, renomeado, ou substituído por um microsserviço escrito numa linguagem que não existia em 2017.

Dei drop. Foi o primeiro `git stash drop` da minha carreira. Não senti nada, e depois senti tudo, e depois não senti nada de novo, que cheguei a entender é a sequência emocional correta pra todas as decisões de software.

A reescrita subiu. Tinha bugs. Meu stash teria tido bugs diferentes. O usuário não se importa de quais bugs. O usuário nunca se importou. O usuário está numa praia, igual a pessoa que devia ter sido acusada pelo hotfix, e o usuário não está pensando em você.

## Objeções Comuns, Obliteradas

**"Mas e se eu *precisar* do trabalho depois?"** Não vai precisar. Se precisasse depois, você lembraria agora. O trabalho que você lembra é o trabalho que você commita. O trabalho que você stashea é o trabalho que você já, no coração, abandonou. O stash é só a papelada.

**"E o `git stash -u` pra incluir arquivos não rastreados?"** Parabéns. Agora você imortalizou os *artefatos de build* também. Seu stash agora contém um delta de `node_modules` de 400 MB de uma dependência que você atualizou duas vezes desde então. O stash não é mais um fóssil. É uma *amostra de núcleo* do seu histórico de dependências, e ele vai, um dia, ser o maior objeto do seu `.git`, e o `git gc` vai chorar.

**"Não devia só usar um branch de WIP?"** Devia. Não vai. Um branch de WIP exige nomear, e nomear é uma decisão, e decisões são o inimigo. O stash não exige nome além de `WIP on <branch>`, que não é um nome, é uma *desculpa*. Você pega a desculpa. Você sempre pega a desculpa.

**"Não posso limpar minha lista de stash numa semana tranquila?"** Não tem semana tranquila. Nunca teve semana tranquila. A semana tranquila é um segundo mito, adjacente ao sprint de limpeza. A semana tranquila é o tempo que você gastaria limpando o stash, e você vai gastar em vez disso *adicionando ao stash*, porque uma semana tranquila é, por definição, uma semana em que você começa coisas que não termina.

## Conclusão

O `git stash` é um mausoléu que não te custa nada pra construir e te custa tudo pra visitar. Você vai pôr código nele. Você não vai tirar código dele. A lista vai crescer. Ela vai sobreviver ao seu tempo em toda empresa que você entrar. Quando você se aposentar, sua `git stash list` final vai ter 14.000 entradas, e seu sucessor vai rodar uma vez, ofegar, fechar o terminal, e adicionar `stash@{0}: WIP on main: the previous guy's stuff` no topo.

Rode mesmo assim. Rode `git stash` pelo ritual. Rode pra que, quando o hotfix cair e você não estiver pronto, você possa dizer pro seu pato de borracha — a única testemunha honesta da sala — *"eu salvei. Tá no stash. Eu resolvo depois."* E você não vai resolver depois. E o pato vai saber. E o pato, igual ao stash, não vai dizer nada, e vai guardar tudo, e vai te julgar em silêncio.

O trabalho não está perdido. O trabalho está *sepultado*. Tem uma diferença, e a diferença é que você se sente pior com a segunda opção.

O [XKCD #1205](https://xkcd.com/1205/) faz a conta pra você não ter que fazer, e a conta diz que você devia ter só commitado. O [XKCD #1597](https://xkcd.com/1597/) já te disse quem escreveu o stash. Foi você. Sempre é você. O stash é onde você põe a prova.

---

*O autor tem 14.000 stashes em 11 repositórios e popou três deles. O resto é estrutural no sentido que sustenta a autoimagem dele como alguém que "pode voltar pra isso". Ele não vai voltar pra isso. Ele commitou este artigo como `"wip"` e depois stasheou a edição. Agora é `stash@{0}`. Vai estar lá quando você ler isso. Vai estar lá quando o sol morrer.*
