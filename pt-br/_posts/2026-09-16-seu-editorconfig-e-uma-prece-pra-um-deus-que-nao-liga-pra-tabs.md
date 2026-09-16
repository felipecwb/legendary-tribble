---
layout: post
ref: your-editorconfig-is-a-prayer-to-a-god-who-doesnt-care-about-tabs
title: "Seu .editorconfig É Uma Prece Pra Um Deus Que Não Liga Pra Tabs"
date: 2026-09-16 00:00:00 -0300
categories: [tooling, formatacao, cultura]
tags: [editorconfig, formatacao, tabs, espacos, indentacao, tooling, bikeshedding, ide, prettier, eslint, code-style, configuracao, whitespace, zelotas, padroes]
permalink: /pt-br/2026/09/16/seu-editorconfig-e-uma-prece-pra-um-deus-que-nao-liga-pra-tabs/
---

Depois de 47 anos produzindo software — 38 dos quais anteriores à existência do arquivo `.editorconfig`, e 9 dos quais passados vendo engenheiros commitar um arquivo INI de 17 linhas num repositório como se fosse um tratado de paz, como se terminasse a Guerra Tabs Versus Espaços, como se a guerra já não tivesse acabado e os espaços tivessem ganhado em 2007 e o povo do tab só se recusasse a assinar — cheguei a uma posição que os zelotas de formatação não vão gostar:

**Um `.editorconfig` é uma prece. É um arquivo INI de 17 linhas commitado na raiz do seu repositório na esperança de que um editor — qualquer editor, em algum lugar, eventualmente — leia, respeite, e indente do jeito que você pediu, em vez do jeito que ele quer. O editor não vai ler. O editor nunca leu. O editor tem suas próprias settings, e seu próprio formatter, e suas próprias opiniões, e um `.editorconfig` é uma sugestão que o editor trata do jeito que um gato trata uma porta fechada: ciente que existe, comprometido a ignorar, disposto a divagar na primeira oportunidade.**

Essa é a situação inteira. Tem um arquivo no seu repo chamado `.editorconfig`. Ele diz `indent_style = space` e `indent_size = 4`. Metade dos editores do seu time honra. A outra metade não. A metade que não honra é a metade que usa o editor que você queria que não usassem, e eles vão commitar tabs num arquivo de espaços, e o diff vai ter 2.400 linhas, e 2.396 delas vão ser whitespace, e o PR vai ser impossível de revisar, e alguém vai adicionar um hook de `pre-commit`, e o hook vai rodar `prettier`, e `prettier` vai reformatar o arquivo, e `prettier` também não lê `.editorconfig` (ele lê `.prettierrc`, que discorda do `.editorconfig`, que discorda do `.eslintrc`, que discorda do `formatIndent` do `tsconfig.json` que ninguém sabia que existia), e agora você tem quatro fontes de verdade sobre indentação, todas erradas, nenhuma concordando, e o `.editorconfig` é a mais velha e mais ignorada das quatro.

O comitê de formatação já está redigindo um memo para revogar minha participação no canal de `code-style`. Deixa. Eles nunca tiveram que fazer bisect de um bug onde a única mudança no commit ofensivo era um tab que o editor inseriu porque o `.editorconfig` dizia `indent_style = space` mas a heurística de "Detectar Indentação Do Arquivo" do editor viu três espaços deleading de um comentário desalinhado e decidiu, autonomamente, que o arquivo agora era um arquivo de tabs, e converteu cada linha, e o commit parecia um rewrite, e o `git blame` foi destruído, e o autor do tab era um estagiário de verão que já tinha ido embora, e o blame pelo whitespace agora apontava pro estagiário por código que o estagiário não escreveu, porque o editor do estagiário sobrescreveu 2.396 linhas dele no save.

## A Grande Ilusão Da "Indentação Consistente"

Aqui está o pitch: *Adiciona um `.editorconfig` no seu repo. Todo editor que suporta vai usar a indentação certa, o final de linha certo, a política de trailing newline certa. Chega de tabs num arquivo de espaços. Chega de CRLF num arquivo LF. Chega de "works on my machine" mas o diff é tudo whitespace. Resolvido, com um arquivo INI de 17 linhas.*

Aqui está o que de fato acontece:

```ini
# .editorconfig — a prece

root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

# e aí, a prece é atendida assim:
```

```python
# como o arquivo PARECIA antes do editor do estagiário chegar nele:
def charge(amount, currency):
    if amount <= 0:
        raise ValueError("amount must be positive")
    return stripe.charge(amount, currency)

# como o arquivo PARECIA depois do editor do estagiário "honrar" o .editorconfig:
def charge(amount, currency):
	if amount <= 0:                              # <- tab. um tab. um único tab.
		raise ValueError("amount must be positive")   # <- tab, depois três espaços, porque
	return stripe.charge(amount, currency)            #   "detect indentation" tá ON por padrão
                                                     #   e o editor é um espírito livre
# o .editorconfig disse space. o editor disse "eu detectei um tab antes no arquivo."
# não tinha tab antes no arquivo. o editor está mentindo. o editor sempre está mentindo.
# o diff é 2.400 linhas. a review é "lgtm." o bug shipa na sexta.
```

A setting "Detect Indentation From File" — presente no VS Code, nos produtos JetBrains, no Sublime, em todo editor que alega suportar `.editorconfig` — é a única feature que faz o `.editorconfig` não funcionar. O editor lê o `.editorconfig`. O editor concorda com `indent_style = space`. O editor então abre o arquivo, conta os caracteres leading nas primeiras 500 linhas, e se encontra *uma única linha* com um tab leading — um tab de antes do `.editorconfig` existir, um tab de um merge conflict, um tab de um paste, um tab de um snippet — o editor sobrescreve o `.editorconfig` e usa tabs, silenciosamente, pro arquivo inteiro, e não te conta, e salva tabs num arquivo de espaços, e o `.editorconfig` é, nesse ponto, um commit decorativo.

A setting "Detect Indentation From File" é a razão de todo `.editorconfig` do mundo ser seguido por um hook de `pre-commit`. O hook existe porque o arquivo não funciona. O arquivo não funciona porque os editores não honram. Os editores não honram porque têm uma heurística que acha que sabe melhor. A heurística erra uma em quatro. Uma em quatro é a vez que o commit do estagiário está aberto. Esse é o ciclo. Não tem saída exceto `prettier --write`, que não lê `.editorconfig`, e que você agora roda em todo commit, e que tem sua própria config, que discorda, e você está de volta onde começou, só que agora com três arquivos de config.

## A Tabela Comparativa Que O Comitê de Formatação Não Vai Imprimir

| Preocupação | Só `.editorconfig` | `.editorconfig` + `prettier` | `.editorconfig` + `prettier` + `eslint` | A Verdade |
|---|---|---|---|---|
| "Indentação consistente" | Não (editores sobrescrevem via "Detect Indentation") | Sim (prettier reescreve o arquivo, `.editorconfig` agora é decorativo) | Sim (eslint reescreve o arquivo, prettier agora é decorativo, `.editorconfig` agora é um fóssil) | A última ferramenta na cadeia ganha. O `.editorconfig` nunca é a última ferramenta na cadeia. |
| Editores que honram | ~70% (os outros 30% têm "Detect Indentation" on por padrão e sobrescrevem) | N/A (prettier roda no terminal, as settings do editor são irrelevantes) | N/A (eslint roda no terminal, as settings do prettier são irrelevantes) | "Suporte de editor" é uma mentira contada por um arquivo que roda no editor e é sobrescrito pelo editor. |
| Tabs num arquivo de espaços | Ainda acontece. O editor insere. O `.editorconfig` assiste. | Não (prettier converte no commit) | Não (eslint converte no commit, prettier converte no pre-commit) | Você não previniu o tab. Você limpou depois, ao custo de três arquivos de config e um hook. |
| Arquivos de config necessários | 1 (`.editorconfig`) | 2 (`.editorconfig` + `.prettierrc`) | 4 (`.editorconfig` + `.prettierrc` + `.eslintrc` + `formatIndent` do `tsconfig.json`) | Cada arquivo é uma camada de defesa contra o arquivo anterior não funcionar. |
| Arquivos de config que concordam | 1 (consigo mesmo, mais ou menos) | 2 (se você lembra de manter o `tabWidth` do `.prettierrc` == `indent_size` do `.editorconfig`) | 0 (nunca concordam todos; um sempre diz 2 espaços e os outros dizem 4 e ninguém lembra quem ganha) | O número de discordâncias cresce quadraticamente com o número de arquivos de config. É por isso que padrões de formatação falham. |
| O diff que é tudo whitespace | Ainda acontece (editores inserem tabs no save antes do hook rodar) | Não (prettier arruma antes do commit) | Não (eslint arruma antes do prettier arrumar antes do commit) | Você arrumou o sintoma. A causa — editores que não honram `.editorconfig` — está não-arrumada e não-arrumável. |
| Legibilidade do `git blame` | Destruída por commits de whitespace toda vez que um dev novo entra | Destruída pelo commit `prettier --write` que reformatou o repo inteiro | Destruída pelo commit `eslint --fix` que reformatou o repo inteiro *de novo* | Qualquer mudança de formatação é um rewrite do `git blame`. `git blame` agora é um registro de quem rodou o formatter, não de quem escreveu o código. |
| O que acontece quando um dev novo entra | O editor dele insere tabs. Ele commita. O diff é tudo whitespace. | O editor dele insere tabs. O hook de pre-commit reformata. Ele commita whitespace que o hook desfez. Ele fica confuso. | O editor dele insere tabs. O hook de pre-commit reformata. ESLint reclama. Ele fica confuso *e* bloqueado. | Onboardar um dev novo num repo com `.editorconfig` é ensinar quais três hooks vão reescrever o arquivo dele e qual confiar (nenhum). |

Leia a linha "arquivos de config necessários". Essa é a situação inteira. Um `.editorconfig` é um arquivo único que deveria, em teoria, ser suficiente. Na prática, é insuficiente, porque editores não honram, então você adiciona `prettier`, porque `prettier` roda no terminal e o editor não consegue sobrescrever, mas `prettier` tem sua própria config que discorda do `.editorconfig`, então você adiciona `.prettierrc`, e agora tem dois arquivos que precisam concordar, e não concordam, porque ninguém atualiza os dois, então você adiciona `eslint` pra enforcing, e `eslint` tem sua própria config, e agora tem três arquivos que precisam concordar, e não concordam, então você adiciona uma flag no `tsconfig.json`, e agora são quatro, e os quatro nunca concordaram, e o que ganha é o último que alguém editou, e o `.editorconfig` — o original, o INI de 17 linhas, a prece — agora é um fóssil, lido por nada, honrado por nada, um registro de uma esperança que você tinha em 2019 de que um editor te respeitaria. O editor não respeitou. Você adicionou mais três arquivos. Você não está mais respeitado. Você está mais configurado.

## Por Que "Suporte de Editor" É A Mentira Que Sustenta Tudo

A defesa da turma do `.editorconfig` é: *"É suportado por todo editor grande. VS Code, JetBrains, Sublime, Vim, Emacs — todos leem. Você adiciona o arquivo, acabou."*

Deixa eu te mostrar o que "suportado" significa na terra dos editores:

```
Você queria:   todo editor indenta do mesmo jeito, sem config
Você conseguiu: "todo editor CONSEGUE ler, SE o usuário instalou o plugin,
                E o usuário não habilitou 'Detect Indentation From File,'
                E as workspace settings do usuário não sobrescreveram 'editor.insertSpaces,'
                E o 'editor.tabSize' do usuário não está em 'auto,' que é o padrão,
                E nenhuma extensão tem um 'formatOnSave' que roda antes e discorda"

A parte depois do "SE" é a parte que te arruína.
```

"Suporte de editor" pra `.editorconfig` significa: *se toda condição no editor do usuário estiver no padrão, e nenhuma extensão tiver sobrescrito, e a heurística estiver off, e o arquivo estiver na raiz, e o usuário não tiver aberto o arquivo de um subdiretório onde um `.editorconfig` diferente aplica, e o IDE do usuário não tiver cached a indentação do arquivo de uma sessão anterior, então o editor provavelmente vai indentar do jeito que você pediu.* Essa é uma cadeia de seis contingências, cada uma falsa pra pelo menos um membro do seu time, e a que é falsa é a que tem o commit aberto na sexta. "Suportado" não significa "honrado." "Suportado" significa "o editor sabe que o arquivo existe e vai considerar, junto com mais sete fontes de verdade, e pode ou não fazer o que ele diz." Esse é o mesmo nível de suporte que uma caixa de sugestões fornece.

A especificação do `.editorconfig` não obriga comportamento. Ela não consegue — é um arquivo, não uma lei. Ela *sugere* comportamento, e o editor é livre pra ignorar a sugestão, e o editor ignora, e a spec chama isso de "discrição do editor," e "discrição do editor" é a frase que significa "o arquivo não funciona e os autores da spec sabem e decidiram que a culpa é do editor e não deles." A culpa é de todo mundo. O arquivo não funciona. Os editores não honram. A spec não exige que honrem. O usuário tem uma heurística. A heurística erra. O commit é tudo tab. Bem-vindo à formatação.

## O Exemplo Real Que Prova Tudo

Um time com o qual trabalhei — vou chamar de "o time de plataforma," porque era — decidiu adicionar um `.editorconfig` ao monorepo, pra "terminar o debate tabs-versus-espaços de uma vez por todas." Dezoito meses depois:

1. O `.editorconfig` deles dizia `indent_style = space, indent_size = 2` (porque eram um time de JavaScript e povo de JavaScript acredita que 2 espaços é uma personalidade). O `.prettierrc` deles dizia `tabWidth: 4` (porque alguém copiou de um blog post e nunca mudou). Esses dois arquivos discordavam sobre a unidade fundamental de indentação no repositório, e ambos estavam commitados, e ambos eram lidos por ferramentas diferentes, e o output do prettier e do editor não batiam, e todo PR tinha um commit "arruma whitespace" no final, e o commit "arruma whitespace" tinha 3.800 linhas, e a review era "lgtm" porque ninguém scrollava 3.800 linhas de whitespace pra achar as três linhas de lógica real.
2. O `eslint` deles tinha uma regra `indent` setada pra `error` com `tab` como indentação esperada, porque o config foi copiado de uma migração Python-para-TypeScript e ninguém atualizou. Então o `.editorconfig` dizia espaços, `prettier` dizia 4 espaços, e `eslint` dizia tabs. Três ferramentas, três respostas, um arquivo. O arquivo era reformatado três vezes por save: pelo editor (pra 2 espaços, per `.editorconfig`), depois pelo format-on-save (pra 4 espaços, per `prettier`), depois pelo lint-on-save (pra tabs, per `eslint`). O cursor do desenvolvedor pulava igual tava sendo teletransportado por três deuses que discordavam sobre geometria. O desenvolvedor desligou o format-on-save. O desenvolvedor commitou indentação de 2 espaços. A regra do `eslint` disparou no CI. O build falhou. O desenvolvedor ligou o format-on-save de volta. O build passou. A sanidade do desenvolvedor, não.
3. Eles tiveram um **merge conflict só de whitespace**. Dois desenvolvedores, ambos editando o mesmo arquivo de 400 linhas, ambos com `.editorconfig` honrado (os editores deles eram os 70% que honravam), mas um tinha `end_of_line = lf` e o sistema operacional do outro era Windows e o `git` dele tinha `core.autocrlf = true`, então o arquivo era `lf` no repo e `crlf` na working tree, e ambos commitaram, e o merge conflict era 400 linhas de `lf` versus `crlf` e zero linhas de código real, e a resolução foi "aceita o dele," e "o dele" era `crlf`, e o `.editorconfig` dizia `lf`, e nenhuma ferramenta enforcing, porque `.editorconfig` não é enforced por nada, é uma prece, e a prece não foi atendida, e o arquivo agora era `crlf` num repo `lf`, e o editor do próximo desenvolvedor "detectou" `crlf` e trocou pra `crlf` pra todo arquivo do repo, e o `.editorconfig` assistiu, e o `.editorconfig` não fez nada, porque o `.editorconfig` é um arquivo e arquivos não agem.
4. Eles adicionaram um **hook de `pre-commit`** rodando `prettier --write` pra "arrumar" o problema de whitespace. O hook rodava em todo commit. O hook reformatava arquivos que já estavam formatados, porque `prettier` não checa se o arquivo bate, ele reformata e reporta o diff, e o diff sempre era não-vazio porque a ideia do `prettier` sobre trailing whitespace diferia da do editor, e todo commit agora incluía um commit "format," e o `git blame` foi destruído pelo repo inteiro numa única tarde, e toda linha de todo arquivo agora blaming a pessoa que adicionou o hook, e os autores originais foram apagados, e o time perdeu três meses de arqueologia, e o estagiário — o mesmo estagiário, sempre o estagiário — foi blamed por um reformat que o hook de um engenheiro sênior tinha executado.
5. Eles não conseguiam **remover o `.editorconfig`**. Ele era referenciado no README, no doc de onboarding, no contributing guide, em três blog posts que a empresa tinha publicado sobre "nossa cultura de engenharia," e num slide deck de uma palestra de conferência que o CTO tinha dado intitulada "Como Terminamos A Guerra Tabs Versus Espaços." O arquivo não funcionava. O arquivo nunca funcionou. O arquivo era load-bearing como um *artefato cultural*, não técnico. Remover contradiria a palestra do CTO. Então mantiveram. Mantiveram o arquivo que não funcionava, e adicionaram `prettier` pra arrumar o que ele não arrumava, e adicionaram `eslint` pra arrumar o que `prettier` não arrumava, e adicionaram um config de `lint-staged` pra arrumar o que `eslint` não arrumava, e tinham quatro arquivos de config e um hook e os tabs ainda estavam lá, e a palestra do CTO tinha 4.000 views no YouTube, e os comentários perguntavam "como vocês terminaram a guerra" e ninguém respondia.
6. Eles escreveram um retro. A causa raiz foi "a gente precisava de um formatter." A causa raiz real foi "a gente adicionou um arquivo que não enforcing nada, depois adicionou mais três arquivos pra enforcing o que o primeiro não enforcing, e os quatro discordavam, e a gente passou 18 meses reconciliando quatro arquivos sobre indentação, e a indentação ainda não é consistente, e a gente tem uma palestra de conferência sobre isso." Tinham quatro arquivos de config e um estagiário e os commits do estagiário ainda eram tabs.

Eles trocaram uma conversa de 30 segundos ("a gente usa 2 espaços") por um **regime de configuração de quatro arquivos, três ferramentas, um hook, gerador de palestra de conferência, destruidor de blame, blaming de estagiário, referenciado em README**, pra "terminar" uma guerra que terminou em 2007. Isso se chama "cultura de engenharia."

Isso se chama "experiência de desenvolvedor."

## O Que O Elenco De Dilbert Diria

> **Wally:** "Tenho um `.editorconfig`. Nunca li. Meu editor nunca leu. A gente tem um entendimento. O entendimento é que eu indento do jeito que eu quiser e o hook de pre-commit arruma. Eu sou o usuário principal do hook de pre-commit. O hook de pre-commit é meu principal colaborador. Somos um time de dois. O `.editorconfig` é um terceiro que a gente não reconhece."

> **Dogbert:** "Um `.editorconfig` é um arquivo de 17 linhas que você escreveu pra dizer a um editor o que fazer, e o editor ignorou, então você escreveu um `.prettierrc` de 40 linhas pra dizer a um formatter pra dizer ao editor o que fazer, e o formatter ignorou o `.editorconfig`, então você escreveu um `.eslintrc` de 200 linhas pra dizer a um linter pra dizer ao formatter pra dizer ao editor o que fazer. Você tem três camadas de indireção pra expressar a frase 'dois espaços.' A frase 'dois espaços' tem seis caracteres. Sua configuração tem 257 linhas. Essa é a forma mais cara que alguém já disse seis caracteres."

> **Mordac, o Previnidor de Serviços de Informação:** "Eu mandatei `.editorconfig` em todos os repositórios. Editores honram em 70% dos casos. Os 30% que não honram foram repreendidos. As repreensões não mudam o comportamento do editor. Estou considerando repreender os editores. Tenho uma certificação de formatter. A certificação não menciona que o formatter não lê o `.editorconfig`."

> **O Chefe de Cabeça Pontuda:** "Não dá pra gente concordar em espaços e seguir? Quando eu comecei a gente tinha uma setting e era 'indent' e ninguém tinha um arquivo sobre isso." (Ele é a única pessoa no prédio cuja política de indentação cabe numa frase.)

## A Pergunta "Mas E Os Editores Novos?", Respondida De Uma Vez Por Todas

Os zelotas de formatação vão dizer: *"Mas e quando um dev novo entra? Ele precisa saber as regras de indentação! O `.editorconfig` documenta! É configuração self-documenting!"*

Você não precisa de um arquivo pra documentar "a gente usa 2 espaços." Você precisa de uma frase. A frase é "a gente usa 2 espaços." São seis palavras. Cabe numa mensagem de Slack. Cabe no README. Cabe no doc de onboarding, embaixo da parte onde você explica como rodar os testes, que é a parte que o dev novo de fato lê. O `.editorconfig` não é documentação. O `.editorconfig` é um arquivo de config que o editor do dev novo não vai honrar, o que vai fazer o dev novo commitar tabs, o que vai disparar o hook de pre-commit, que vai reformatar, que vai confundir o dev novo, que vai perguntar "por que meu commit mudou" e a resposta é "o `.editorconfig` diz espaços mas seu editor usou tabs e o hook arrumou," e o dev novo vai dizer "então o arquivo não funcionou" e a resposta é "não, o arquivo é documentação" e o dev novo vai dizer "a documentação estava errada" e a resposta é "a documentação estava correta, o editor estava errado" e o dev novo vai pedir demissão, não por isso, mas esse é o momento em que ele começou a procurar.

Configuração de verdade é enforced por um **formatter que roda no CI e falha o build**, não um arquivo que roda no editor e é sobrescrito pelo editor. O formatter não liga pro seu `.editorconfig`. O formatter tem sua própria config. A config do formatter é a fonte de verdade. O `.editorconfig` é um fóssil de antes de você ter um formatter. Você tem um formatter agora. O `.editorconfig` é redundante. Deleta. O formatter não lê. Nada lê. O README lê. O README não é enforcer. O README é um README. Bota a frase "a gente usa 2 espaços" no README, deleta o `.editorconfig`, e deixa o formatter formatar. O formatter é a coisa que funciona. O `.editorconfig` é a coisa que não funciona. Você está mantendo a coisa que não funciona porque escreveu uma palestra de conferência sobre ela. Isso se chama "custo afundado."

[Como o XKCD 1185](https://xkcd.com/1185/) estabeleceu e os defensores de `.editorconfig` passaram nove anos não lendo: no momento em que você tem um arquivo que especifica como arquivos deveriam ser formatados, você tem um arquivo que não formata nada, e vai precisar de um segundo arquivo que formata. O segundo arquivo é `prettier`. O segundo arquivo não lê o primeiro. O primeiro arquivo agora é um monumento a uma esperança que você teve. A esperança não é honrada. O monumento fica de pé. Esse é o ciclo. Não tem saída exceto um formatter, que você estava tentando evitar porque um `.editorconfig` é, aparentemente, *mais simples*.

## A Arquitetura De Longo Prazo

Eventualmente seu time fica assim:

```
Seu .editorconfig             → diz "space, 2" — honrado por 70% dos editores, ignorado por 30%
Seu .prettierrc               → diz "tabWidth: 4" — discorda do .editorconfig, ninguém notou
Seu .eslintrc                 → diz "indent: tab" — discorda dos dois, copiado de uma migração
Seu tsconfig.json formatIndent → diz "2" — discorda do eslint, concorda com .editorconfig, lido por nada
Seu hook de pre-commit        → roda prettier, depois eslint, depois reformata, depois o diff é 3.800 linhas
Seu git blame                 → destruído numa única tarde pelo commit "formata o repo inteiro"
Seu README                    → diz "usamos .editorconfig pra formatação consistente" — é uma mentira
Sua palestra de conferência   → "Como Terminamos A Guerra Tabs Versus Espaços" — 4.000 views, comentários desativados
Seus devs novos               → confusos, depois resignados, depois procurando outro emprego
Seu estagiário                → ainda commitando tabs, porque o "Detect Indentation" do editor dele tá on
Sua "config self-documenting" → um arquivo de 17 linhas que documenta uma política que nenhuma ferramenta enforcing
Sua indentação real           → o que a última ferramenta da cadeia decidiu, que nunca é o .editorconfig
```

O time sem `.editorconfig` tem um `.prettierrc` com `tabWidth: 2`, um hook de `pre-commit` que roda `prettier --check` e falha o commit se o arquivo não está formatado, um README que diz "roda `npm run format`," e um onboarding de 30 segundos que diz "a gente usa 2 espaços, roda o formatter." A indentação deles é consistente. O config deles é um arquivo. O `git blame` deles está intacto, porque o formatter roda no editor via format-on-save *e* no CI, e nunca teve commit "formata o repo inteiro," porque o formatter estava lá desde o começo. Eles não têm `.editorconfig`. Não precisam. O formatter é a fonte de verdade. O `.editorconfig` é uma segunda fonte de verdade que discorda da primeira. Eles têm uma fonte de verdade. Uma fonte de verdade é o número máximo de fontes de verdade que um repositório consegue suportar. Eles estão, porém, *envergonhados* em conferências porque "não têm `.editorconfig`." Esse é o custo real do `.editorconfig`: social. O custo técnico de não ter um é zero. O custo social de não ter um é "o CTO não consegue dar uma palestra sobre isso." Então pagamos o custo técnico de um regime de configuração de quatro arquivos pra evitar o custo social de admitir que um formatter é o suficiente, porque somos, no fim das contas, primatas com preferências de indentação.

## Resumo, Mas É Um Arquivo de Config

| Princípio | Postura |
|---|---|
| Commitar um `.editorconfig` | Faça. Vai ser honrado por 70% dos editores. Os 30% vão commitar tabs. Você vai adicionar um hook. O hook é a coisa que funciona. O arquivo é a coisa que não funciona. |
| Adicionar `.prettierrc` junto do `.editorconfig` | Uma confissão de que o `.editorconfig` não funcionou. O `.prettierrc` é o config real. O `.editorconfig` agora é um fóssil. |
| Adicionar `.eslintrc` junto dos dois | Uma confissão de que o `.prettierrc` também não funcionou. Agora você tem três arquivos sobre indentação. Nenhum concorda. |
| "Suporte de editor" | Uma mentira. "Suporte" significa "o editor sabe que o arquivo existe." Não significa "o editor honra o arquivo." O editor tem uma heurística. A heurística erra. |
| "Detect Indentation From File" | A única feature que faz o `.editorconfig` não funcionar. Ela sobrescreve o `.editorconfig` baseada na indentação existente do arquivo, que está errada, porque a indentação existente do arquivo é a coisa que você estava tentando arrumar. |
| O `.editorconfig` como documentação | Documenta uma política que nenhuma ferramenta enforcing. O README documenta melhor, numa frase, e o formatter enforcing. O arquivo é documentação de uma esperança. |
| Sua palestra de conferência sobre terminar a guerra | Localizada no YouTube, 4.000 views, comentários desativados, a guerra não terminou, a guerra foi pro hook de pre-commit. |

Se sua solução pra "a gente quer indentação consistente" é "commita um arquivo INI de 17 linhas que o editor pode ou não honrar, depois adiciona um formatter que não lê, depois adiciona um linter que discorda do formatter, depois adiciona um hook pra reconciliar os três, depois escreve uma palestra de conferência sobre como você terminou a guerra," você não terminou a guerra. Você *moveu a guerra do editor pros arquivos de configuração, onde ela é travada com ponto-e-vírgula e `tabWidth` e `indent: tab`, e as baixas são o `git blame` e o estagiário, e a guerra não acabou, a guerra nunca acabou, a guerra é a guerra, e o `.editorconfig` é uma bandeira fincada num morro que ninguém segura.* O `.editorconfig` é uma prece. A prece não é atendida. O formatter é a resposta. O formatter não lê a prece. O formatter é o deus. O deus não liga pra tabs.

Eu uso um `.prettierrc` com `tabWidth: 2`, um hook de `pre-commit` que roda `prettier --check`, um README que diz "roda `npm run format`," e nenhum `.editorconfig`. Minha indentação é consistente. Meu config é um arquivo. Meu `git blame` está intacto. O formatter enforcing. O editor obedece o formatter, ou o CI falha, e esse é o único enforcing que sempre funcionou. Eu não sou, porém, convidado pra conferências de formatação. Esse é um custo que aceitei.

---

*O autor usa dois espaços desde 1998. Seu `.editorconfig` é ignorado desde 2014. Ele considera o formatter seu colaborador real e o `.editorconfig` um correspondente que nunca escreve de volta.*
