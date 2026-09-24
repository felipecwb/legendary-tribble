---
layout: post
ref: your-pre-commit-hooks-are-a-toll-booth-on-the-highway-to-production
title: "Seus Hooks De Pre-commit São Um Pedágio Na Rodovia Pra Produção"
date: 2026-09-24 00:00:00 -0300
categories: [tooling, git, cultura]
tags: [pre-commit, git-hooks, tooling, linting, formatacao, prettier, eslint, ci, friccao, experiencia-de-desenvolvedor, gates, configuracao, husky, lint-staged, cultura, burocracia]
permalink: /pt-br/2026/09/24/seus-hooks-de-pre-commit-sao-um-pedagio-na-rodovia-pra-producao/
---

Depois de 47 anos produzindo software — 39 dos quais anteriores à existência do diretório `.git/hooks/`, e 8 dos quais passados vendo engenheiros parafusar um shell script no momento mais importante do dia do desenvolvedor (o momento em que ele aperta Enter no `git commit`) e depois fingir surpresa quando o desenvolvedor aprende a apertar `--no-verify` — cheguei a uma posição que a turma do pre-commit não vai gostar:

**Um hook de pre-commit é um pedágio. É um script que você cinturou na rampa de saída do seu editor, cobrando uma taxa toda vez que um desenvolvedor tenta sair. A taxa é denominada em segundos, e os segundos são não-reembolsáveis, e o pedágio é operado por um linter que foi configurado em 2021 por um estagiário que já saiu, e o linter não sabe que ano é, e o linter não liga, e o linter vai manter o desenvolvedor refém por causa de uma vírgula de trailing que o próprio linter inseriu na terça passada. O desenvolvedor vai pagar o pedágio. O desenvolvedor sempre vai pagar o pedágio, porque a alternativa é `--no-verify`, e `--no-verify` é o speed run, e o desenvolvedor tá fazendo o speed run desde a terceira vez que o hook falhou num arquivo que o desenvolvedor não tocou.**

Essa é a situação inteira. Tem um script em `.git/hooks/pre-commit`. Ele roda em todo commit. Ele roda `prettier`, roda `eslint`, roda `ruff`, roda `shellcheck`, roda `detect-secrets` (que flagga a palavra `secret` num comentário), roda um hook customizado que alguém escreveu que dá `grep` por `TODO` e rejeita o commit se achar, e a coisa toda leva 14 segundos, e 13 desses segundos são `shellcheck` lintando um `Dockerfile` que nem é um shell script. O desenvolvedor aperta Enter. O desenvolvedor espera 14 segundos. O desenvolvedor não está, em nenhum momento desses 14 segundos, escrevendo código. O desenvolvedor tá olhando um spinner. O desenvolvedor tá pagando um pedágio. O operador do pedágio é um linter. O linter tá errado sobre o Dockerfile. O desenvolvedor sabe que o linter tá errado sobre o Dockerfile. O desenvolvedor não vai arrumar o linter, porque arrumar o linter exige editar o hook, e editar o hook exige entender o hook, e entender o hook exige ler o hook, e ler o hook exige se importar, e o desenvolvedor não se importa, o desenvolvedor quer commitar, o desenvolvedor quer ir pra casa, o desenvolvedor aperta `--no-verify` e vai pra casa. O pedágio foi ultrapassado. O lint não foi enforcing. O pedágio foi coletado em valor cheio de todo desenvolvedor que não sabia do `--no-verify`, e isentado inteiramente de todo desenvolvedor que sabia. Isso não é um modelo de segurança. Isso é um imposto regressivo sobre os inexperientes.

O comitê de tooling já tá redigindo um memo pra revogar meu acesso ao canal de `devex`. Deixa. Eles nunca tiveram que ver um engenheiro sênior — um engenheiro *staff*, uma pessoa que escreve software há mais tempo do que o hook existe — descobrir `--no-verify` pela primeira vez, e a cara dele, que é a cara de uma pessoa que tá pagando pedágio há onze meses e acabou de aprender que o pedágio tem uma pista aberta do lado sem barreira, sem câmera, sem operador, e uma placa que diz "GRÁTIS — TRÁFEGO LOCAL APENAS" que ninguém lê. Eles nunca mais vão pagar o pedágio. Eles vão contar pra dois colegas. Os dois colegas vão contar pra quatro. Numa sprint, o hook não enforcing nada, porque as pessoas que iam produzir as violações são as mesmas que descobriram `--no-verify` primeiro, porque são as pessoas cujos commits o hook foi escrito pra pegar, porque o hook foi escrito pra pegar *elas*, e elas são as que optaram pra fora, e o hook agora roda só contra as pessoas cujo código já estava limpo, que é o oposto exato do propósito dele. Esse é o ciclo de vida inevitável de todo hook de pre-commit já escrito. É também o ciclo de vida de toda câmera de radar colocada no pé de uma ladeira.

## A Grande Ilusão Do "Shift Left"

Aqui está o pitch: *Pega os problemas no commit, antes de chegar no CI, antes de chegar em produção. Um hook de pre-commit é o lugar mais barato de achar um bug. Shift left. Fail fast. Quanto antes pegar, mais barato de arrumar. Um hook de 14 segundos é mais barato que um CI de 14 minutos. Resolvido, com um shell script.*

Aqui está o que de fato acontece:

```bash
# .git/hooks/pre-commit — o pedágio

#!/bin/sh
# escrito por um estagiário em 2021. o estagiário foi embora. o hook ficou.

echo "Running pre-commit checks..."
# o desenvolvedor não tocou em nenhum desses arquivos. o hook não liga.

npx prettier --check .          # 2.1s — reformata, depois reclama que reformatou
npx eslint .                     # 3.4s — tem opiniões sobre ponto-e-vírgula de 2021
npx tsc --noEmit                 # 4.8s — type-checka arquivos que não foram mudados
ruff check .                      # 0.9s — Python, num repo de JavaScript, por algum motivo
shellcheck Dockerfile            # 2.0s — Dockerfile não é shell script. shellcheck não liga.
detect-secrets --baseline .secrets.baseline  # 0.6s — flagga a palavra "secret" no README
./scripts/no-todo.sh             # 0.2s — dá grep por "TODO", rejeita se achar

# total: 14.0 segundos. todo commit. todo desenvolvedor. toda vez.
# o desenvolvedor não mudou um único arquivo que o hook checa.
# o desenvolvedor tá pagando pedágio pra não mudar nada.
# o desenvolvedor vai aprender --no-verify. o desenvolvedor vai parar de pagar.
# o hook vai continuar rodando, nas pessoas que não sabem do --no-verify.
# essas pessoas o código já estava limpo.
# o hook não enforcing nada. o hook inconveniência todo mundo que não consegue pegar.
```

```python
# o que o desenvolvedor QUERIA fazer, antes do pedágio:
git commit -m "fix: typo in README"

# o que o desenvolvedor FEZ, depois da décima primeira vez que o hook falhou num typo de README:
git commit --no-verify -m "fix: typo in README"

# o typo tá arrumado. o hook não checou. o hook nunca checou.
# o hook existe. o hook não é usado. o hook é um shell script decorativo.
# o README ainda diz "secrect" em vez de "secret" porque o desenvolvedor
# commitou com --no-verify, e o linter que teria pego "secrect"
# tava no hook, e o hook foi pulado, e o typo tá em produção,
# e o pedágio não foi coletado, e a cabine fica de pé, sem operador, acesa por dentro.
```

A doutrina do "shift left" — pregada por todo consultor de DevOps que já faturou uma hora — é a única ideia que faz hooks de pre-commit parecerem razoáveis. A doutrina diz: acha defeitos o mais cedo possível, porque o custo de um defeito cresce quanto mais tarde você acha. Isso é verdade. Também é verdade que o custo de *procurar* defeitos cresce quanto mais cedo você procura, porque no commit você procura em todo commit, e a maioria dos commits tá ok, e procurar tem custo, e o custo é pago em atenção do desenvolvedor, e atenção do desenvolvedor é o recurso mais caro do prédio, e você tá gastando com uma vírgula de trailing. "Shift left" tá certo sobre onde achar bugs. É silencioso sobre quem paga pela procura. A resposta é: o desenvolvedor paga, em todo commit, em segundos, e os segundos somam um número maior que o número de bugs que o hook acha, porque o hook acha quase nenhum bug, porque os bugs não estão nos arquivos que o hook checa, porque o desenvolvedor não mudou os arquivos que o hook checa, porque o hook checa tudo, sempre, na teoria de que checar tudo é mais seguro que checar algo, e checar tudo não é mais seguro, checar tudo é mais lento, e mais lento é a coisa que o hook deveria prevenir.

A turma do "shift left" vai dizer: *só roda o hook nos arquivos staged.* E de fato tem uma ferramenta pra isso. Chama `lint-staged`. Roda os linters só nos arquivos que o desenvolvedor deu stage. Isso é correto em princípio. Na prática, o desenvolvedor deu stage em um arquivo, o hook roda num arquivo, o hook leva 0.4 segundos, o hook passa, o desenvolvedor commita, e o desenvolvedor passou 11 meses configurando `lint-staged` pra ir de 14 segundos pra 0.4 segundos, e os 0.4 segundos é o custo da *checagem*, e os 11 meses é o custo da *configuração*, e a configuração é a coisa que quebrou, e a quebra é a coisa que fez o desenvolvedor aprender `--no-verify`, e o desenvolvedor aprendeu `--no-verify` durante os 11 meses, e agora os 0.4 segundos são pagos só por desenvolvedores que iam passar de qualquer jeito. A otimização não reduziu o pedágio. A otimização reduziu o número de pessoas que pagam pra pessoas que não precisavam ser cobradas. Isso é o oposto de progresso.

## A Tabela Comparativa Que O Comitê de DevEx Não Vai Imprimir

| Preocupação | Sem hook de pre-commit | Hook de pre-commit (roda tudo) | Hook de pre-commit (`lint-staged`, só staged) | A Verdade |
|---|---|---|---|---|
| Bugs pegos antes do CI | Zero no commit, alguns no CI | Alguns, a maioria formatação, a maioria em arquivos que o dev não tocou | Alguns, em arquivos staged, quando o dev não deu `--no-verify` | Os bugs que importam não são bugs de formatação. O hook pega bugs de formatação. O CI pega o resto. O hook é um gate de formatação disfarçado de gate de qualidade. |
| Tempo por commit | 0s | 14s | 0.4s | A versão de 14s é paga por todo mundo. A de 0.4s é paga por todo mundo que não sabe do `--no-verify`. Quem sabe do `--no-verify` paga 0s. Quem o código precisava do hook paga 0s. Quem o código não precisava do hook paga 0.4s. |
| Desenvolvedores que `--no-verify` | N/A | ~40% (os cujo código o hook quer pegar) | ~40% (mesmas pessoas, hook mais rápido não mudou a cabeça delas) | `--no-verify` é opt-out. As pessoas que precisam do hook dão opt-out. As que não precisam não conseguem dar opt-out. O hook enforcing contra a própria irrelevância. |
| Integridade do `git blame` | Intacta | Destruída pelo commit "formata o repo inteiro" que o hook fez alguém fazer uma vez | Intacta (só arquivos staged reformatados) | Qualquer hook que reescreve arquivos destrói o blame. O hook que não reescreve não destrói o blame, mas também não enforcing formatação, e era portanto inútil. |
| Esforço de configuração | 0 linhas | ~50 linhas (o hook, o config que ele lê, o README explicando o hook) | ~150 linhas (config do lint-staged, config do husky, o hook, o README, a seção de troubleshooting) | O hook "mais barato que CI" custa 150 linhas de config pra economizar 14 segundos, que o desenvolvedor vai bypassar de qualquer jeito. |
| Quem aprende que o hook existe | Todo mundo (CI falha se pulam) | Todo mundo (roda local, alto) | Quem lê o README (ninguém) | A existência do hook não é enforced. O CI é enforced. O CI é o enforcing. O hook é uma sugestão que roda local. Sugestões locais são ignoráveis. O CI não é. |
| O que acontece quando o hook quebra | Nada (sem hook) | Todo commit falha até alguém arrumar o hook; esse alguém não é quem escreveu | Todo commit falha até alguém arrumar o hook; esse alguém tá de férias | Um hook quebrado bloqueia todos os commits. Um CI quebrado bloqueia um PR. O hook é um ponto único de falha pra habilidade do time inteiro de commitar. O CI é um ponto único de falha pra habilidade de um PR de mergear. O hook é o modo de falha mais perigoso. |
| O que o dev júnior faz | Committa, pusha, CI falha, lê o erro, arruma | Committa, hook falha, não entende por quê, pergunta no Slack, espera 20 min, alguém diz "usa --no-verify", o júnior agora usa --no-verify pra sempre | Committa, hook passa (arquivo staged tá limpo), pusha, CI falha num arquivo diferente, júnior confuso sobre por que o hook não pegou | O hook ensina juniores a bypassar hooks. Esse é o único output pedagógico confiável do hook. |

Leia a linha "Quem aprende que o hook existe". Essa é a situação inteira. Um hook de pre-commit é enforced por nada. Roda na máquina do desenvolvedor. O desenvolvedor é dono da máquina. O desenvolvedor consegue desabilitar o hook deletando o arquivo, chmod non-executable, `--no-verify`, `git -c core.hooksPath=/dev/null commit`, setando `HUSKY=0`, desinstalando husky, commitando de outra máquina, commitando pela web UI do GitHub, commitando por um bot de CI. O hook não tem mecanismo de enforcing que o desenvolvedor não controle, porque o hook roda na máquina do desenvolvedor, e a máquina do desenvolvedor é do desenvolvedor. O CI roda num servidor que o desenvolvedor não controla. O CI é o enforcing. O hook é uma sugestão. Uma sugestão que leva 14 segundos não é uma sugestão barata. É uma sugestão cara. Uma sugestão que é bypassável pelas pessoas que ela quer pegar não é uma sugestão. É uma porta sem batente, parada num campo, que o vento derruba de vez em quando.

## Por Que "Local É Mais Barato Que CI" É A Mentira Que Sustenta Tudo

A defesa da turma do pre-commit é: *"Rodar o linter local é mais barato que rodar no CI. Um run local é 14 segundos. Um run de CI é 14 minutos. Você tá economizando 13 minutos e 46 segundos por commit."*

Deixa eu te mostrar o que "mais barato" significa na terra do pedágio:

```
Você queria:   pegar bugs cedo, barato, antes do CI
Você conseguiu: "pegar bugs cedo, na máquina do desenvolvedor, que o desenvolvedor
                controla, e que o desenvolvedor pode e vai bypassar com --no-verify,
                então os bugs não são pegos cedo, são pegos no CI, em 14 minutos,
                igual antes, mais o desenvolvedor gastou 14 segundos não pegando,
                mais alguém gastou 11 meses escrevendo o hook, mais o hook quebrou uma vez
                e bloqueou todo commit do repo numa tarde"

A parte depois do "Você conseguiu:" é a parte que não vai no slide deck.
```

"Local é mais barato que CI" é verdade no mesmo sentido que "cozinhar em casa é mais barato que restaurante" é verdade: é verdade se você não valoriza seu próprio tempo, e se a comida tá comestível, e se você não queima a cozinha. Um run de linter local é mais barato que um run de CI *se o run local for o que acha o bug*. O run local é o que acha o bug só se o desenvolvedor não bypassa. O desenvolvedor bypassa. O desenvolvedor sempre bypassa. O desenvolvedor bypassa porque o hook é lento, ou porque o hook tá errado, ou porque o hook tá checando arquivos que o desenvolvedor não tocou, ou porque o desenvolvedor tá com pressa, ou porque o desenvolvedor tá on-call, ou porque o desenvolvedor tem reunião em quatro minutos, ou porque o desenvolvedor simplesmente não quer esperar 14 segundos pra arrumar um typo de README. O bypass é uma flag. A flag é 11 caracteres. A flag é mais curta que a palavra "precommit". O desenvolvedor vai digitar a flag. A flag é mais rápida que o hook. O hook perdeu. O hook sempre perde. O hook nunca, na história dos hooks, ganhou de um desenvolvedor que aprendeu a flag, e o desenvolvedor aprende a flag na terceira vez que o hook falha num arquivo que o desenvolvedor não mudou, que é a segunda semana.

"Local é mais barato que CI" também é uma comparação entre duas coisas que não são intercambiáveis. Um run de CI faz mais que o hook. Um run de CI roda os testes. Um run de CI roda o build. Um run de CI roda a suite de integração. Um run de CI roda num ambiente que bate com produção. O hook roda o linter, no laptop do desenvolvedor, que tem uma versão de Node diferente, e um cache de `npm` diferente, e um `.env` diferente que o `detect-secrets` do hook vai flaggar pela quadragésima sétima vez. O hook não é "CI mas local". O hook é "um linter mas local", e um linter não é CI, e um linter é a coisa que o CI roda *primeiro*, antes das coisas que importam, e você pegou o passo menos importante do CI e moveu pro lugar mais caro de rodar (a atenção do desenvolvedor) e chamou de "shift left". Não é shift left. É shift *pra baixo*, pro colo do desenvolvedor, e o desenvolvedor tá botando no chão.

## O Exemplo Real Que Prova Tudo

Um time com o qual trabalhei — vou chamar de "o time de pagamentos," porque era — decidiu adicionar um hook de pre-commit pra "pegar bugs antes do CI e melhorar a experiência do desenvolvedor." Catorze meses depois:

1. O hook deles rodava `prettier`, `eslint`, `tsc --noEmit`, `jest --findRelatedTests`, e um script customizado que checava o formato da mensagem de commit contra um regex. O regex exigia `type(scope): message` e o `type` tinha que ser um de `feat|fix|chore|docs|style|refactor|perf|test|build|ci`. O regex não permitia `hotfix`. O workflow de on-call do time produzia commits `hotfix`. Todo hotfix falhava no hook. Todo hotfix era commitado com `--no-verify`. O hook agora enforcing formato de conventional commit em todo commit *exceto hotfixes*, que eram os únicos commits onde o formato importava pro changelog. O changelog era gerado dos conventional commits. O changelog não continha hotfixes. O changelog estava errado. O time não notou por nove meses, porque ninguém lia o changelog, porque o changelog era gerado, e coisas geradas não são lidas, são *produzidas*.

2. O passo de `tsc --noEmit` deles levava 11 segundos e type-checkava o monorepo inteiro em todo commit, independente de qual arquivo estava staged. Um desenvolvedor arrumando um typo num README esperava 11 segundos pelo TypeScript concluir que o README não tinha erros de tipo. O desenvolvedor aprendeu `--no-verify` no dia três. O desenvolvedor ensinou pro novo contratado no dia cinco. O novo contratado ensinou pro estagiário no dia sete. No fim da segunda sprint, o hook rodava em aproximadamente 30% dos commits, e os 30% eram os commits que já estavam limpos, e os 70% que estavam sujos eram commitados com `--no-verify`, e o CI pegava os sujos, e o hook tinha salvado zero runs de CI, e tinha custado 11 segundos × 30% × commits-por-dia × desenvolvedores × 14 meses, que é um número grande o suficiente que o gráfico de velocity do time tinha uma dip visível toda vez que alguém entrava, porque a pessoa nova pagava o pedágio até aprender o bypass, e a dip era o pedágio, e o gráfico nunca foi investigado, porque gráficos de velocity não são investigados, são *exibidos*.

3. Eles tinham um **hook que flaggava a palavra "password"** em qualquer arquivo staged, pra evitar commitar segredos. A palavra "password" aparecia nos test fixtures, nos scripts de migração, na documentação, no README, no `.env.example`, em 47 arquivos do repo. O baseline do hook (uma lista de ocorrências "permitidas") tinha que ser regenerado toda vez que alguém adicionava um teste que mencionava password, que era todo PR, porque os testes eram de um sistema de auth, e sistemas de auth mencionam passwords. Regenerar o baseline era um comando de 3 passos que ninguém lembrava. O comando tava no README. O README era o mesmo README que o hook tinha rejeitado uma vez por conter a palavra "secret" num comentário. O desenvolvedor que batesu no hook rodava `--no-verify`, commitava, e o secret scanner no CI (uma ferramenta diferente, com um baseline diferente, que concordava em nada) ou passava ou falhava independentemente. Existiam agora dois secret scanners, um local e um no CI, com dois baselines, discordando se a palavra "password" num teste era um segredo, e o local era bypassável e o do CI não era, e o local só existia pra ser bypassado, e o time estava mantendo dois scanners pra fazer o trabalho de um, e o que funcionava era o do CI, e o que não funcionava era o que eles tinham uma página inteira de documentação sobre, e a documentação estava errada sobre como regenerar o baseline, e estava errada desde 2022.

4. Eles adicionaram `husky` pra "gerenciar" os hooks, porque `.git/hooks/` não é commitado (é por-clone) e eles queriam o hook automático pra clones novos. `husky` instalava o hook no `npm install`. `husky` também instalava um hook de `pre-push`, um de `commit-msg`, e um de `post-merge`, porque quem configurou habilitou todo template. O hook de `post-merge` rodava `npm install` depois de todo `git pull`, que re-rodava `husky`, que re-instalava os hooks, que era ok, exceto que também re-rodava em `git pull` durante um rebase, e durante um rebase o hook de `commit-msg` disparava em todo commit replayed, e o regex de conventional-commit rejeitava três dos commits replayed porque eram commits `hotfix`, e o rebase abortava, e o desenvolvedor ficou preso no meio do rebase com um hook que não deixava continuar, e a única saída era `git rebase --no-verify`, que existe, que o desenvolvedor achou no StackOverflow depois de 40 minutos, que o desenvolvedor agora usa pra todos os rebases, e o desenvolvedor agora aprendeu duas flags de `--no-verify`, e a memória muscular do desenvolvedor é `--no-verify`, e o desenvolvedor digita sem pensar, e o hook foi totalmente internalizado como "a coisa que eu digito `--no-verify` pra pular", que é a forma final do hook.

5. Eles não conseguiam **remover o hook**. Era referenciado no doc de onboarding ("usamos pre-commit hooks pra qualidade"), no blog de engenharia ("como a gente fez shift left"), no README ("roda `npm install` pra instalar os hooks"), no contributing guide ("commits devem passar o hook de pre-commit"), e numa pergunta de entrevista ("usamos pre-commit hooks; como você se sente sobre isso?"). O hook não funcionava. O hook nunca funcionou. O hook era load-bearing como um *artefato cultural*, não técnico. Remover contradiria o blog de engenharia. Então mantiveram. Mantiveram o hook que não funcionava, e adicionaram `lint-staged` pra deixar mais rápido (que não fez funcionar, só mais rápido em não funcionar), e adicionaram um script `prepare` pra husky instalar no `npm install` (que fez o hook que não funcionava instalar *automaticamente*), e tinham um hook, e um gerenciador de hook, e um config de staged-files, e um gerador de baseline, e uma seção de README, e um slide de onboarding, e os commits ainda estavam sujos, e o CI ainda pegava o que o hook não pegava, e o hook era uma cerimônia de quatro camadas em volta de um `--no-verify`.

6. Eles escreveram um retro. A causa raiz foi "desenvolvedores estão bypassando os hooks". A causa raiz real foi "a gente instalou um gate que é bypassável pelas pessoas que ele quer pegar, nas máquinas que essas pessoas controlam, e está surpreso que elas bypassam." O action item foi "educar desenvolvedores a não usar `--no-verify`". A educação não funcionou, porque `--no-verify` é mais rápido que a educação, e o desenvolvedor sempre escolhe a coisa mais rápida, e a coisa mais rápida é o bypass, e o bypass é 11 caracteres, e a educação é uma reunião de 30 minutos, e a reunião poderia ter sido um email, e o email poderia ter sido um hook, e o hook poderia ter sido nada, e nada é o que eles deveriam ter construído.

Eles tinham trocado um run de CI de 14 minutos (que pegava os bugs) por **um hook de 14 segundos (que não pegava nada) mais um run de CI de 14 minutos (que pegava os bugs)**, pra "shiftar left". Eles não tinham shifted left. Eles tinham *adicionado um pedágio na frente da mesma rodovia, e o pedágio coletava das pessoas que já estavam no limite de velocidade, e os que estavam acima pegaram o desvio, e a rodovia era a mesma rodovia, e os bugs chegaram na mesma hora que sempre chegaram, e a única coisa nova era a cabine, e a cabine estava acesa, e a cabine tinha operador, e o operador era um linter que estava errado sobre o Dockerfile.* Isso se chama "experiência de desenvolvedor."

Isso se chama "shift left."

## O Que O Elenco De Dilbert Diria

> **Wally:** "Tenho um hook de pre-commit. Nunca deixei rodar. Dei alias de `git commit` pra `git commit --no-verify` no meu shell profile. O hook nunca executou na minha máquina. Minha máquina é uma zona livre de hook. Considero o hook um colega que evitei com sucesso por catorze meses. Nunca nos conhecemos. Me disseram que é muito completo. Vou acreditar neles."

> **Dogbert:** "Você construiu um portão. O portão tá na propriedade do desenvolvedor. O desenvolvedor tem a chave do portão. Você está surpreso que o desenvolvedor use a chave. Você instalou um portão cuja única função é ser aberto pela pessoa que você construiu pra parar. Isso não é segurança. Isso é uma portinhola de gato que você tá chamando de porta de cofre. O gato tá usando. O gato sempre esteve usando. O gato tá commitando mensagens `hotfix:` por ela agora."

> **Mordac, o Previnidor de Serviços de Informação:** "Eu mandatei hooks de pre-commit em todos os repositórios. Compliance é medida pela presença do arquivo do hook, não pela execução do hook. O arquivo do hook está presente em 100% dos repositórios. O hook executa em 30% dos commits. Eu considero isso uma vitória. Os 70% que bypassam são 'notados.' A notação não mudou o comportamento deles. Estou considerando notar mais forte."

> **O Chefe de Cabeça Pontuda:** "Não dá pra deixar o CI pegar? Quando eu comecei a gente não tinha hook, a gente tinha um cara chamado Gary, e Gary olhava o diff, e se o diff tinha tab o Gary gritava, e esse era o hook, e o Gary ia pra casa às 5, e o hook ia pra casa às 5, e o código estava ok." (Ele é a única pessoa no prédio cujo gate de qualidade tem horário de trabalho e nome.)

## A Pergunta "Mas E Pegar Bugs Cedo?", Respondida De Uma Vez Por Todas

Os zelotas de pre-commit vão dizer: *"Mas pegar um bug no commit é mais barato que pegar no CI! O hook salva um run de CI! O hook salva context-switching do desenvolvedor!"*

Você não salva um run de CI rodando um hook que o desenvolvedor bypassa. Você salva um run de CI rodando CI. O run de CI é a coisa que pega o bug. O hook é a coisa que roda antes do run de CI e não pega nada, porque o desenvolvedor bypassou, porque o hook era lento, ou errado, ou checando arquivos que o desenvolvedor não tocou. A comparação "mais barato" assume que o hook pega o bug e o CI não precisa. O hook não pega o bug. O CI pega o bug. O hook roda, o desenvolvedor bypassa, o bug vai pro CI, o CI pega, o run de CI não foi salvo. O hook foi um prelúdio. O prelúdio foi pulado. A sinfonia tocou sem ele. A sinfonia estava ok.

Pegar bug de verdade acontece no **CI, num servidor que o desenvolvedor não controla, em todo PR, com a suite de testes completa, num ambiente que bate com produção.** O hook é um linter local. Um linter local é bypassável. O CI não é bypassável. O hook é uma porta sem batente. O CI é uma parede. Você não precisa dos dois. Você precisa da parede. A parede funciona. A porta não funciona. Você está mantendo a porta porque foi barata de instalar, e foi barata de instalar porque não funciona, e coisas que não funcionam são sempre baratas de instalar, e caras de manter, e você está mantendo há catorze meses, e a manutenção é o README, e o README está errado, e a errância é load-bearing, e essa é sua vida agora.

[Como o XKCD 1597](https://xkcd.com/1597/) estabeleceu e os defensores de pre-commit passaram oito anos não lendo: no momento em que você pede o desenvolvedor pra esperar, você começou uma corrida entre seu hook e a paciência do desenvolvedor, e a paciência do desenvolvedor é um recurso finito que se esgota mais rápido que seu hook roda, e o desenvolvedor vai ganhar a corrida recusando a rodar, e o hook vai estar lá, e o hook vai ser bypassado, e os bugs vão estar no CI, e o CI vai pegar, e você vai ter um hook e um CI e um bypass e um bug, que são quatro coisas onde antes tinha duas (um CI e um bug), e quatro é mais que dois, e mais não é mais barato, e mais barato era o ponto inteiro.

## A Arquitetura De Longo Prazo

Eventualmente seu time fica assim:

```
Seu .git/hooks/pre-commit       → roda prettier, eslint, tsc, ruff, shellcheck, secrets, todo-check
Sua adoção de --no-verify       → 70% dos commits (os 70% que o hook foi escrito pra pegar)
Seu config de husky            → instala o hook automaticamente, também pre-push, commit-msg, post-merge
Seu config de lint-staged       → roda o hook só em arquivos staged (salvou 13.6s, salvou zero bugs)
Seu baseline do detect-secrets  → 47 arquivos de menções "permitidas" de password, regenerado por ninguém
Seu regex de commit-msg         → rejeita hotfix, o único tipo de commit que importa pro changelog
Seu CI                         → roda os mesmos linters, mais testes, mais build, mais integração
Seu secret scanner do CI        → uma ferramenta diferente, um baseline diferente, concorda em nada
Seu README                     → diz "commits devem passar o hook de pre-commit" — 70% deles não passam
Seu blog de engenharia         → "Como Fizemos Shift Left" — o left foi shiftado pra um --no-verify
Seu doc de onboarding          → "usamos pre-commit hooks pra qualidade" — a qualidade tá no CI
Seus novos contratados         → ensinados --no-verify no dia cinco pela pessoa que substituíram
Seus commits de hotfix         → sempre --no-verify, nunca no changelog, o changelog está errado
Seu bug-catching real          → CI, que estava lá antes do hook, e está depois, e funciona
Seu hook                       → um shell script de 50 linhas que roda em 30% dos commits e não pega nada
```

O time sem hook de pre-commit tem CI que roda o linter, os testes, o build, e o secret scanner em todo PR, um README que diz "seu PR deve passar o CI," e um onboarding de 30 segundos que diz "pusha pra uma branch, abre um PR, espera o CI." Os bugs deles são pegos no CI. Os desenvolvedores deles não esperam num pedágio. O `git blame` deles está intacto, porque nada reformata arquivos localmente. Os commits deles são rápidos. Os hotfixes deles estão no changelog, porque não tem regex de commit-msg. O secret scanning deles é uma ferramenta, no CI, com um baseline. Eles não têm hook de pre-commit. Não precisam. O CI é o enforcing. O hook é a decoração. Eles estão, porém, *envergonhados* no canal de DevEx porque "não shiftaram left." Esse é o custo real do hook de pre-commit: social. O custo técnico de não ter um é zero. O custo social de não ter um é "o engenheiro staff não consegue escrever um blog post sobre isso." Então pagamos o custo técnico de um regime de hook de quatro camadas pra evitar o custo social de admitir que CI é o suficiente, porque somos, no fim das contas, primatas que querem ter shifted alguma coisa.

## Resumo, Mas É Um Pedágio

| Princípio | Postura |
|---|---|
| Instalar um hook de pre-commit | Faça. Vai rodar em 30% dos commits. Os 30% são os commits que estavam limpos. Os 70% que estavam sujos vão usar `--no-verify`. O hook é um pedágio que cobra de quem já estava pagando. |
| Adicionar `husky` pra auto-instalar hooks | Uma confissão de que o hook não estava sendo instalado. Agora está. Agora é bypassado automaticamente em vez de manualmente. Eficiência. |
| Adicionar `lint-staged` pra rodar só em staged | Uma confissão de que o hook era lento demais. O hook agora é rápido. O hook ainda é bypassado. Rápido e bypassado não é um gate de qualidade. É um gate de velocidade sem carros. |
| "Shift left" | Uma doutrina que está correta sobre onde achar bugs e silenciosa sobre quem paga pra procurar. O desenvolvedor paga. O desenvolvedor opta pra fora. A procura não aconteceu. O left não foi shiftado. |
| `--no-verify` | O bypass. 11 caracteres. Mais curto que a palavra "precommit". Mais rápido que o hook. O amigo do desenvolvedor. O nemesis do hook. Sempre ganha. |
| O hook como "experiência de desenvolvedor" | É uma experiência. A experiência é esperar. A espera é o custo. O custo é pago pelas pessoas que o hook não consegue pegar. Essa é a experiência. |
| Seu blog de engenharia sobre shift left | Localizado no blog da empresa, 2.000 views, comentários habilitados, um comentário diz "já tentou --no-verify", o comentário tem 40 upvotes, o blog não foi atualizado. |

Se sua solução pra "a gente quer pegar bugs cedo" é "instala um script na máquina do desenvolvedor que o desenvolvedor consegue desabilitar com 11 caracteres, roda em todo commit, vê 70% dos desenvolvedores desabilitarem, mantém o script porque o blog post sobre ele tem 2.000 views, adiciona um gerenciador de hook pra instalar automaticamente pra poder ser bypassado automaticamente, adiciona um otimizador de staged-files pra o bypass ser mais rápido, adiciona um baseline pro secret scanner que ninguém regenera, e conclui que você shiftou left quando na verdade você shiftou o custo de procurar pra cima do desenvolvedor e o desenvolvedor shiftou de volta pro CI onde ia ser feito de qualquer jeito," você não shiftou left. Você *construiu um pedágio numa rodovia, operou com um linter, acendeu por dentro, e assistiu todo carro com passe rápido rodar pela pista aberta enquanto você coletava moedas dos carros que já estavam no limite de velocidade, e chamou as moedas de "qualidade," e chamou a cabine de "cultura," e chamou a pista aberta de bug, e o bug não era a pista, o bug era a cabine.* O hook é um pedágio. O pedágio é cobrado dos inocentes. Os culpados têm passe rápido. O passe rápido é `--no-verify`. O passe rápido sempre existiu. O passe rápido sempre vai existir. A cabine fica de pé. A cabine está acesa. A cabine é operada por um linter que está errado sobre o Dockerfile. O Dockerfile está em produção. A cabine não está.

Eu uso CI que roda o linter, os testes, o build, e o secret scanner em todo PR, um README que diz "seu PR deve passar o CI," e nenhum hook de pre-commit. Meus bugs são pegos no CI. Meus commits são instantâneos. Meu `git blame` está intacto. Meus hotfixes estão no changelog. Meus desenvolvedores não sabem o que é `--no-verify`, porque não tem nada pra verificar. Eu não sou, porém, convidado pra conferências de DevEx. Esse é um custo que aceitei.

---

*O autor não espera num hook de pre-commit desde 2019. Ele considera o CI seu gate de qualidade real e o hook de pre-commit um pedágio cujo operador se aposentou e deixou a luz acesa. A luz ainda está acesa. A luz sempre vai estar acesa. Ninguém está na cabine.*
