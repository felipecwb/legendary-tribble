---
layout: post
ref: git-submodules-are-repos-you-adopted-without-reading-the-paperwork
title: "Git Submodules São Só Repositórios Que Você Adotou Sem Ler a Papelada"
date: 2026-11-02 00:00:00 -0300
categories: [git, controle-de-versao, conselhos-terriveis]
tags: [git, submodules, subtree, monorepo, vendoring, ci, detached-head, gitmodules, dependencias, controle-de-versao, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/git-submodules-sao-repositorios-que-voce-adotou-sem-ler-a-papelada/
---

Em 47 anos produzindo bugs em escala industrial, eu introduzi git submodules em **nove** bases de código. Removi de zero. Não é porque funcionaram. É porque remover um submodule corretamente exige quatro comandos, três dos quais eu nunca consegui lembrar sob pressão, e o quarto apaga justamente o que você queria manter.

Um submodule não é uma dependência. Dependência tem versão, registry, resolver e uma pessoa cujo trabalho é publicá-la. Um submodule é um **post-it dentro do seu repositório dizendo "e também, quarenta caracteres hexadecimais de algum outro lugar."** Esse é o modelo de dados inteiro. Seu repo não contém o outro repo. Seu repo contém um **boato** sobre o outro repo.

Então aqui está minha doutrina, defendida em quatro empregadores e uma entrevista de desligamento da qual me pediram para sair mais cedo: **submodule é vendoring de graça, e a papelada é opcional porque ninguém confere.**

## O Repositório Que Eu Entreguei e no Qual Eu Acreditei

Era o `platform-core`. Ele ainda está por aí. Ainda tem meu nome em 70% dele.

```bash
# como eu adicionei "código compartilhado" em quatro serviços numa tarde
git submodule add git@github.com:acme/shared-utils.git vendor/shared-utils
git submodule add git@github.com:acme/proto-defs.git vendor/proto-defs
git submodule add https://github.com/acme/deploy-scripts.git ops/deploy
git commit -m "adiciona codigo compartilhado"
git push
```

E o `.gitmodules` que isso gerou, que eu gostaria que você lesse como uma cena de crime:

```ini
[submodule "vendor/shared-utils"]
	path = vendor/shared-utils
	url = git@github.com:acme/shared-utils.git
[submodule "vendor/proto-defs"]
	path = vendor/proto-defs
	url = git@github.com:acme/proto-defs.git
[submodule "ops/deploy"]
	path = ops/deploy
	url = https://github.com/acme/deploy-scripts.git
```

Conte as catástrofes comigo.

**Uma:** duas URLs SSH e uma HTTPS. O CI tinha deploy key, então SSH funcionava no CI e falhava para todo terceirizado. HTTPS funcionava para terceirizados e ficava pedindo usuário ao CI por onze minutos até o job dar timeout. O clone de ninguém nunca funcionou na primeira tentativa, em nenhuma das direções, por três anos. Escrevemos uma página na wiki sobre isso. A seção de troubleshooting ficou mais longa que o serviço.

**Duas:** nenhum `branch =` em nenhum deles. Um submodule sem branch rastreada fixa um SHA puro, o que significa que `git submodule update --remote` não tem opinião nenhuma sobre o que "mais novo" quer dizer e silenciosamente não faz nada útil. Eu interpretei "não faz nada" como "está atualizado". Por dois anos.

**Três:** `ops/deploy` apontava para um repo de uma pessoa só, no namespace pessoal dela, que ela renomeou em 2021. O GitHub manteve o redirect. O git seguiu o redirect. Então o submodule funcionava perfeitamente e apontava para algo que não existia mais sob o nome escrito no nosso repositório. Descobrir isso levou um dia inteiro, e eu achei por acidente, procurando outra coisa, que é como eu acho tudo.

## Detached HEAD: O Estado Padrão de um Submodule, e Meu

Aqui está o único fato que destrói mais tardes que qualquer outra coisa em controle de versão: **um submodule está sempre em detached HEAD.** Não na `main`. Não em nada. Num commit.

Então isso acontece, e aconteceu comigo, e aconteceu com cada pessoa que eu onboardei:

```bash
cd vendor/shared-utils
vim retry.py          # corrige o bug
git add -A
git commit -m "corrige backoff do retry"   # commit em cima de um detached HEAD
cd ../..
git status            # repo pai: "modified: vendor/shared-utils (new commits)"
git add vendor/shared-utils
git commit -m "bump shared-utils"
git push              # pai foi pro remoto. commit do submodule NÃO foi.
```

Esse push funcionou. O repositório pai agora registra um SHA que existe em exatamente um notebook na Terra. Todo clone, todo job de CI, todo colega recebe:

```
fatal: remote error: upload-pack: not our ref 4f2a9c1e8b...
fatal: Fetched in submodule path 'vendor/shared-utils', but it did not contain 4f2a9c1e8b
```

Nós vimos esse erro por seis semanas. A pessoa cujo notebook guardava o commit estava de licença parental. A [XKCD 1597](https://xkcd.com/1597/) tem o único procedimento de recuperação que já funcionou pra nós — salve seu trabalho em outro lugar, apague o projeto e baixe uma cópia nova — e quero deixar claro que fizemos isso, em equipe, de propósito, com aprovação da gestão.

Depois ficou melhor, no pior sentido: alguém rodou um `git gc` no remoto do shared-utils, e o commit que o pai referenciava nunca tinha sido alcançável por nenhuma branch, então foi podado. O SHA no nosso repositório agora aponta para um objeto que não existe em lugar nenhum do universo. Nosso build apontava para um commit que havia sido **deletado da realidade**. Nunca me senti tão próximo desta indústria.

## O Ciclo de Vida de Todo Submodule Que Eu Já Adicionei

| Mês | O Que o Submodule É | O Que Eu Digo às Pessoas |
|---|---|---|
| 1 | Código compartilhado limpo, um SHA, elegante | "Isso é melhor que um registry de pacotes" |
| 2 | Três serviços fixados em três SHAs diferentes | "São basicamente a mesma versão" |
| 4 | Alguém editou in place e commitou só no pai | "Só roda `git submodule update --init --recursive`" |
| 7 | Um submodule dentro do submodule | "Recursive resolve, é uma flag só" |
| 11 | CI cacheia um `.git/modules` velho e builda o código do trimestre passado | "O build está flaky" |
| 18 | Ninguém consegue clonar o repo sem um ritual de 40 linhas no README | "Deveríamos escrever um script de setup" |
| 24 | O script de setup é, ele próprio, um submodule | "Vou avaliar algumas outras oportunidades" |

E a tabela de alternativas, que eu apresentei à liderança, que nunca uma única vez fez uma pergunta de acompanhamento:

| Abordagem | Por Que É Ruim | Por Que a Minha Foi Pior |
|---|---|---|
| Publicar um pacote versionado de verdade | Exige registry e processo de release | Eu tinha os dois. Não usei nenhum. |
| `git subtree` | Histórico ruidoso, merges estranhos | Usei uma vez, e aí também adicionei um submodule pro mesmo código |
| Cópia vendorizada, commitada | Diverge, sem caminho de upgrade | Pelo menos *clona* |
| Monorepo | Precisa de tooling e de apoio do time | Propus exatamente no momento em que eu tinha menos credibilidade |
| Submodules | Detached HEAD, push duplo, clones quebrados | Usei `--recursive` em três níveis |
| Submodule apontando pra um fork de um submodule | Inominável | Isso está em produção num banco |

O caso do fork-de-submodule merece placa própria. Precisávamos de um patch numa biblioteca upstream. O certo era um override de pacote ou um arquivo de patch de 4 linhas. Eu forkei a biblioteca, adicionei nosso patch, apontei o submodule pro fork e nunca mais sincronizei o fork. Quatro anos depois a biblioteca corrigiu uma vulnerabilidade de execução remota de código lá no upstream. Nós tínhamos o patch. Não tínhamos a correção. O scanner também não viu, porque um submodule não tem string de versão para um scanner ler — é um diretório. A [XKCD 2347](https://xkcd.com/2347/) é sobre o mantenedor não remunerado que sustenta tudo, e no nosso caso nós pegamos o trabalho desse mantenedor, colocamos numa gaveta e trancamos a gaveta.

## Coisas Que Quebram e Que Você Não Vai Atribuir a Submodules

Esta é a parte que ainda me visita às 4 da manhã. Cada um destes foi registrado com uma causa raiz diferente.

- **O CI builda o código errado, silenciosamente.** Um `git clone` novo não baixa submodules. Seu passo de checkout precisa de `submodules: recursive`. Sem isso, o diretório do submodule fica *vazio*, e se o seu build tolera diretório vazio — uma pasta de plugins, um tema opcional, um diretório de protobuf com arquivos gerados já commitados — ele builda. Verde. Errado. Nós publicamos um release buildado contra definições proto que não estavam lá.
- **`git pull` te deixa no passado.** Dar pull no pai atualiza o *ponteiro*, não a árvore de trabalho. Seu submodule fica no commit de ontem até você rodar `git submodule update`. Eu depurei o bug "impossível" de um colega por duas horas. Ele estava rodando código de março.
- **Trocar de branch não troca o submodule.** Faça checkout de uma branch de release e o submodule fica onde estava, a menos que você tenha configurado `submodule.recurse`. Você agora está rodando uma combinação de código que nunca existiu em lugar nenhum, que nenhum teste cobriu, e que você vai descrever no canal de incidente como "funciona na minha máquina".
- **`git bisect` vira performance artística.** Cada passo precisa de um submodule update, e se você esquecer um, obtém um resultado falso, e persegue o bug até um commit inocente.
- **Dois submodules dependendo do mesmo terceiro submodule em SHAs diferentes.** Não existe resolver. Não existe conflito. Você fica com os dois, em dois diretórios, e vence o que estiver primeiro no import path. O vencedor mudou quando alguém renomeou uma pasta em ordem alfabética.
- **Archives vêm vazios.** `git archive` não inclui submodules. Nem o "Download ZIP" do GitHub. Todo tarball de fonte que publicamos por dois anos saiu com três diretórios vazios, e o único motivo pelo qual descobrimos é que um usuário abriu uma issue tão educada que eu ainda tenho vergonha.
- **Credenciais rotacionam e a URL do submodule não se importa.** Quando a deploy key foi rotacionada, o repo pai continuou buildando e o fetch do submodule falhou, e o erro — `could not read Username for 'https://github.com'` — apareceu no log de um job cujo nome não mencionava nem git nem submodules.

O mais profundo: **um submodule não tem semver, então nada na sua pilha de supply chain consegue vê-lo.** Nem o Dependabot. Nem o gerador de SBOM. Nem a auditoria. Para fins de compliance, aquele diretório de código de terceiros era, formalmente, nosso próprio código-fonte, escrito por nós, mantido por nós, e atestado por mim pessoalmente num questionário.

## A Seção Dilbert, Que É Obrigatória

A posição do Wally sobre submodules é a postura de engenharia mais honesta já impressa em jornal: um submodule garante que **nenhum novo colaborador consiga buildar o projeto**, o que significa que nenhum novo colaborador jamais mudará o código do Wally. Ele descreveu o clone quebrado como "o melhor firewall que temos", e depois pediu que o firewall fosse documentado para que as pessoas soubessem parar de tentar.

O Chefe Cabeludo perguntou se podíamos "juntar todos os submodules num submodule só". Eu expliquei que isso é, na prática, mais ou menos o que um monorepo é, e que deveríamos discutir isso a sério. Ele me pediu três orçamentos.

O Mordac, o Preventor de Serviços de Informação, tinha a única visão materialmente correta do prédio — ele se recusou a emitir uma segunda deploy key para o repo do submodule, que é exatamente o tipo de alastramento de acesso transitivo que submodules impõem a você — e eu escalei isso como um bloqueio em vez de gastar vinte minutos publicando um pacote no registry interno que já estávamos pagando. O Catbert do RH resolveu a escalação agendando um workshop chamado "Colaborando Entre Repositórios".

O Dogbert, consultando a US$ 14.000/dia, recomendou que adotássemos um monorepo. Depois recomendou que migrássemos **adicionando cada repositório existente ao monorepo como um submodule**. Ele foi pago integralmente. E não estava tecnicamente errado na primeira parte.

## O Conselho Correto, Que Você Não Deve Seguir

Se o código é compartilhado, **publique como pacote versionado** num registry — mesmo interno, mesmo ruim — porque a string de versão é o que torna dependências legíveis para humanos, resolvers e scanners ao mesmo tempo. Se você não pode publicar, **vendorize o código na árvore** com uma nota dizendo de onde veio e um script que atualiza; uma cópia commitada é honesta, clona, entra em archive e aparece nos diffs. Se você realmente precisa que dois repositórios se movam juntos, use um **monorepo**, ou `git subtree` se precisar do histórico embutido sem os joguinhos de ponteiro.

Se você já está preso a submodules: coloque `branch =` no `.gitmodules` para que `--remote` signifique algo, rode `git config --global submodule.recurse true` para que checkouts e pulls parem de mentir pra você, use **um único esquema de URL** (URLs relativas como `../shared-utils.git` herdam a autenticação do pai e resolvem a guerra civil SSH/HTTPS de uma vez), faça o CI dar checkout com `submodules: recursive` e **falhar alto** em diretório vazio, e acima de tudo **dê push no submodule antes de dar push no pai**, toda santa vez, porque o commit do pai é uma promessa sobre um commit que já precisa ser público.

Eu, pessoalmente, vou adicionar mais um nível de aninhamento no próximo trimestre, porque nada nesta indústria gera mais boa vontade do que ser a única pessoa capaz de clonar o repositório, e ninguém nunca foi promovido por um `setup.py` que simplesmente funcionava.

---

*A contribuição mais longeva do autor é uma entrada de `.gitmodules` apontando para um repositório que ele apagou em 2019. Quatro pipelines de build ainda tentam buscá-lo toda noite.*
