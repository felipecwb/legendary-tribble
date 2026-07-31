---
layout: post
ref: git-blame-is-just-finding-out-who-wrote-the-bug-its-always-you
title: "Git Blame É Só um Jeito de Descobrir Quem Escreveu o Bug (É Sempre Você)"
date: 2026-07-31 00:00:00 -0300
categories: [controle-de-versao, cultura, anti-padroes]
tags: [git, git-blame, controle-de-versao, code-review, cultura, responsabilidade, debug, codigo-legado, trabalho-em-equipe, psicologia]
permalink: /pt-br/2026/07/31/git-blame-e-so-descobrir-quem-escreveu-o-bug-e-sempre-voce/
---

Depois de 47 anos escrevendo bugs profissionalmente, eu rodei `git blame` umas 9.000 vezes. Posso relatar os achados: a pessoa que escreveu a linha quebrada é, em ordem decrescente de frequência, *eu, eu, uma pessoa que desde então foi promovida para um cargo acima do código, eu, uma pessoa que desde então saiu da empresa, eu de novo, e — numa ocasião inesquecível em 2003 — eu, sob um nome de usuário que não lembro de ter tido, numa máquina que desde então vendi como sucata.*

O `git blame` é vendido como uma ferramenta de investigação. Ele é, na verdade, um espelho. A indústria ainda não aceitou isso. Eu estou aqui para ajudar.

## As Quatro Fases do `git blame`

Todo engenheiro passa pelas mesmas quatro fases. Eu vi isso acontecer 9.000 vezes. Eu vivi isso 9.000 vezes.

**Fase 1 — Justiceiro.** Você acha um bug. Você está furioso. Alguém *fez isso com você*. Você roda `git blame` para achar essa pessoa e fazer ela se explicar. Você já está rascunhando a mensagem do Slack na cabeça. Ela começa com *"hey, pergunta rápida sobre..."* e termina com uma ameaça.

**Fase 2 — Detetive.** Você roda `git blame` na linha. O hash do commit aparece. O nome do autor aparece. Você sente a emoção da caçada. Você é o Sherlock. O jogo começou. O jogo é, especificamente, um `NULL` dereference introduzido num commit intitulado *"fix: stuff."*

**Fase 3 — Acusador.** Você abre o commit. Você lê o diff. Você prepara o seu caso. Você vai apresentar essa evidência na próxima standup, ou num comentário passivo-agressivo no ticket, ou pro seu pato de borracha, que é um ouvinte melhor que o seu time e nunca defende o acusado.

**Fase 4 — Réu.** Você olha o nome do autor uma segunda vez. É o seu nome. É o seu nome num commit de onze meses atrás intitulado *"fix: stuff."* Você fecha o notebook. Você corrige o bug em silêncio. Você não conta pra ninguém. Você agora é de novo um Justiceiro da Fase 1, caçando a pessoa que *em seguida* vai escrever um bug nesse código, porque essa pessoa também vai ser você.

O [XKCD #1597](https://xkcd.com/1597/) se chama "Git Blame" e mostra uma única linha, `git blame`, com o alt text sugerindo que o que você *mesmo* quer saber é *de quem é a culpa*. O Randall sabe. O Randall sempre soube. O Randall, também, é o autor da linha.

## Por Que o `git blame` Sempre Aponta Pra Você

Isso não é coincidência, e não é humilhação. É uma inevitabilidade estatística, e vou provar com a única matemática que lembro das minhas décadas nessa indústria:

1. Você escreveu a maior parte do código do arquivo que você está acusando.
2. O código que você não escreveu foi escrito por alguém que saiu, e portanto não pode ser acusado, e portanto não conta.
3. O código que *parece* ter sido escrito por outra pessoa foi, na verdade, escrito por você durante um `git rebase` que reatribuiu a autoria a quem teve o infortúnio de ser o último a tocar no branch.
4. C.Q.D. É sempre você.

O ponto do rebase é essencial e amplamente ignorado. Depois de 47 anos posso confirmar: o `git blame` não relata *quem escreveu a linha*. Ele relata *quem o git decidiu creditar pela linha da última vez*, que é a pessoa que mais recentemente fez rebase, squash, amend, force-push, ou de outra forma cometeu um ato de revisionismo histórico numa sexta à tarde. A linha do blame não é um registro de autoria. É um registro de *quem estava segurando a bolsa quando a música parou.*

## O Pipeline do Blame-Para-Você, Visualizado

Vamos auditar pra onde o blame de fato aponta ao longo de uma carreira típica:

| De quem o `git blame` diz que escreveu | Quem realmente escreveu | Sua atitude |
|---|---|---|
| Você | Você | Corrige em silêncio, não menciona pra ninguém |
| Você | Um júnior que você mentorou | Assume a culpa em público, ele merecia a cobertura |
| Um colega que saiu | Você (durante um rebase) | Corrige, culpa o rebase, segue em frente |
| Um colega que foi promovido | Você (com inveja) | Corrige, menciona isso na próxima avaliação dele, nega ter feito isso |
| `root` / `<bot@users.noreply.github.com>` | Um bot de CI que rodou `npm run format` | Se rende. Você não pode processar um formatador. |
| Ninguém (a linha foi deletada) | Você (você deletou, ela era estrutural) | Restaura a linha, finge que a deleção nunca aconteceu |
| Uma pessoa que nunca trabalhou aqui | Código vendorizado de um vendor | Trate a linha como escritura sagrada; não toque; não acuse; desvie o olhar |

Note que a coluna da esquerda — de quem o git *diz* que escreveu — quase nunca é a pessoa que *de fato* escreveu, e a coluna do meio quase sempre é você. Isso não é um bug do `git blame`. É uma feature do universo, que é hostil e, nesse ponto específico, justo.

## O Workflow de `git blame` Que Eu Recomendo

O não-iluminado roda `git blame` pra atribuir culpa. O iluminado roda `git blame` pra *evitar* ter que consertar o bug. Depois de 47 anos, refinei o workflow pra quatro passos, cada um mais útil que o anterior:

```bash
# 1. Acuse a linha. Descubra que foi você.
git blame -L 42,42 path/to/the/thing.py

# 2. Acuse a linha um commit mais fundo. Descubra que ainda foi você,
#    mas num commit com uma mensagem pior.
git blame -L 42,42 --before=2023-01-01 path/to/the/thing.py

# 3. Cheque se a pessoa que "escreveu" ainda trabalha aqui.
#    Se sim: prepare-se pra perguntar pra ela, educadamente.
#    Se não: o bug agora é órfão. Órfãos são problema seu.
git log -1 --format='%ae' $(git blame -L 42,42 -s path/to/the/thing.py | cut -d ' ' -f1) | xargs -I{} gh api user --field q={}

# 4. Se foi você, e você ainda trabalha aqui, conserte.
#    Se foi você, e você foi promovido, delegue pra um júnior
#    e chame de "uma oportunidade de aprendizado."
git commit -m "fix: stuff"
```

O passo 4 é o mais importante, e o que mais executei. A mensagem de commit `"fix: stuff"` me serviu fielmente across seis empresas, três décadas e uma passagem lamentável como arquiteto. Ela nunca esteve errada, porque nunca pretendeu estar certa.

## Pra Que o `git blame` *De Verdade* Serve

Aqui está o segredo que a documentação do Git se recusa a imprimir: o `git blame` não é pra achar os culpados. O `git blame` é pra *produzir uma lista de pessoas que você não pode acusar*, de modo que você possa estreitar o campo até sobrar você mesmo, e aceitar seu destino com dignidade.

- A pessoa que saiu da empresa não pode ser acusada. Ela já foi. O bug agora é uma herança, tipo um vaso que você não queria e não pode jogar fora.
- A pessoa que foi promovida não pode ser acusada. Ela está acima de você agora. Acusar pra cima é um movimento que limita a carreira, que eu fiz pessoalmente, repetidas vezes, e não recomendo.
- O bot de CI não pode ser acusado. Ele não tem sentimentos nem gerente. Ele não pode ser convidado pra uma standup. Ele não pode ser feito de sentir vergonha.
- A pessoa que está de férias agora não pode ser acusada. Ela está numa praia. Você está numa mesa, num incidente. A injustiça disso é, na verdade, o produto.

Uma vez que você elimina todo mundo que não pode ser acusado, a pessoa que sobra é você. Isso não é uma falha da ferramenta. É a ferramenta *funcionando como projetada*. O `git blame` é um algoritmo de ordenação cuja saída é sempre o mesmo elemento único: você, às 2 da manhã, com um ponteiro `NULL`.

## O Dilbert Entendeu Isso Primeiro

O Wally, santo padroeiro dos engenheiros que aceitaram seu destino, explicou certa vez toda a estratégia de carreira dele assim: *"Vou achar um projeto que não exige trabalho e me prender a ele."* O `git blame` é o inverso dessa estratégia: é uma ferramenta que não exige trabalho e se prende a você. Você não acusa. Você *é* acusado. O verbo é intransitivo na prática, mesmo que a gramática insista no contrário.

O Dogbert, que é mais esperto que todo arquiteto pra quem já reportei, resumiria com mais aspereza: *"Por que procurar o autor do bug quando você pode procurar o autor da *busca*? O bug é você. A busca também é você. Economize um passo."* Depois de 47 anos, posso confirmar que economizar esse passo liberou aproximadamente 4.200 horas, que reinvesti em escrever mais bugs.

## Uma História de Sucesso do Mundo Real

Em 2011, herdei um serviço que caía toda terça às 3:14 da manhã. A rotação de on-call estava em revolta aberta. Eu rodei `git blame` na linha que falhava. Era eu. Eu tinha escrito a linha em 2008, numa empresa diferente, sob um nome diferente, numa máquina que desde então transformei em calçador de porta. A mensagem do commit era *"fix: stuff."*

Eu consertei. Não contei pra ninguém. Disse pra rotação de on-call que o bug era "uma questão legada de uma era anterior" e que eu "rastreei e resolvi na causa raiz." Todas as três afirmações eram tecnicamente verdadeiras. Fui promovido no trimestre seguinte. O serviço não cai numa terça desde então. Não contei pro meu time o porquê. Não vou. O calçador de porta sabe, e o calçador de porta é silencioso.

## Objeções Comuns, Obliteradas

**"Mas o `git blame` me ajuda a entender o *contexto* da linha."**
Não ajuda. Ele te dá um hash de commit. O hash te dá um diff. O diff te dá 2.000 linhas de mudanças não relacionadas, porque o autor rodou `git commit -a` depois de uma semana sem commitar. O contexto que você queria está na linha 1.400 de um commit de 2.000 linhas intitulado *"wip"* e você nunca vai achar. O `git blame` não fornece contexto. O `git blame` fornece *negabilidade*, que é mais útil.

**"E o `git log -S`? Aquele acha *quem introduziu* a mudança."**
Sim, e ele vai apontar pra você. O `git log -S` procura no histórico quem adicionou uma string. Você adicionou a string. Você adicionou ela num merge commit. O merge commit tem 47 pais. Um dos pais é um branch que você deletou. A string veio de um vendor que você demitiu. A ferramenta está correta. A ferramenta também é inútil. Bem-vindo.

**"Não devíamos escrever mensagens de commit melhores pra o blame ser útil?"**
Devíamos, e não vamos, e mesmo que fôssemos, as mensagens seriam escritas por você, sobre bugs que você cometeu, num tom que você teria vergonha de ler em voz alta. Uma boa mensagem de commit não te salva de ser o autor. Só torna a autoria mais legível pro promotor.

**"Não dá pra usar `code owners` pra reforçar responsabilidade?"**
Dá. E os `code owners` vão, ao longo do tempo, sendo editados pelas próprias pessoas sendo acusadas, pra se removerem dos arquivos que quebraram, até o arquivo `CODEOWNERS` ficar vazio e cada linha do codebase ser de propriedade do `@bot-formatter`. Eu vi isso acontecer. Levou oito meses. O arquivo vazio foi commitado com a mensagem *"chore: stuff."*

## Conclusão

O `git blame` é um espelho que custa quarenta minutos de tempo de incidente pra olhar dentro. Ele não acha os culpados. Ele elimina os inocentes, um por um, até o único nome que sobra ser o seu, num commit chamado *"fix: stuff,"* às 2 da manhã, num arquivo que você não lembra de ter aberto.

Rode mesmo assim. Rode pelo ritual. Rode pra que, quando a poeira baixar, você possa dizer pro seu pato de borracha — a única testemunha honesta da sala — *"Eu verifiquei. Fui eu. Claro que fui eu. Sempre é eu."* E então conserte a linha, em silêncio, e commite como *"fix: stuff,"* e dê push, e vire o Justiceiro da Fase 1 da próxima pessoa.

O bug é você. A ferramenta que acha o bug também é você. Economize um passo.

O [XKCD #1597](https://xkcd.com/1597/) entende isso num relance, e o Randall, eu suspeito, escreveu aquela tira no meio da Fase 4 dele. Eu já fiz isso. Todos nós já fizemos. Bem-vindo ao pipeline. O pipeline aponta pra você.

---

*O autor tem sido o resultado do seu próprio `git blame` 9.000 vezes. Consertou 8.997 deles. Os três que faltam são estruturais e não devem ser tocados. Ele commitou o conserto de cada um como `"fix: stuff."` O pipeline está, como sempre, satisfeito.*
