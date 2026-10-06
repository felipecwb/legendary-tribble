---
layout: post
ref: your-make-dev-script-is-a-folk-song-passed-down-by-engineers-who-left
title: "Seu Script `make dev` É uma Canção Folclórica Passada por Engenheiros Que Já Saíram"
date: 2026-10-06 00:00:00 -0300
categories: [devops, cultura]
tags: [makefile, ambiente-dev, onboarding, folclore, shell-scripts, divida-tecnica, tradicao-oral, make-targets, setup-env, cargo-cult, conhecimento-ndocumentado, e-o-staff-engineer-que-disse-o-senhor-engine]
permalink: /pt-br/2026/10/06/seu-script-make-dev-e-uma-canco-folclorica-passada-por-engenheiros-que-sairam/
---

Depois de 47 anos de `make`, cheguei a uma conclusão sobre todo `Makefile` em todo repositório em que já entrei: não é um sistema de build. É uma tradição oral. Os targets dentro dele não foram autoralmente escritos. Eles foram *transmitidos* — de um staff engineer que saiu em 2019, para um staff engineer que saiu em 2021, para um staff engineer que está de férias e não responde no Slack — e, como toda tradição oral, o significado original se perdeu e só sobrevive a invocação.

Considere o target médio `make dev`. Ele não builda nada. Ele não testa nada. Não está descrito no README, porque o README foi atualizado pela última vez quando o staff engineer que saiu em 2019 ainda era júnior, e na época o README dizia "rode `make dev`" sem mais nenhuma explicação, porque ele presumiu que você saberia, e ele estava errado. O target funciona. Ninguém sabe por que funciona. A única pessoa que poderia saber agora é staff engineer num concorrente e te bloqueou no LinkedIn.

Essa é a verdade do `make dev`: é uma canção folclórica. É "Parabéns pra Você". Ninguém sabe quem escreveu. Todo mundo canta. É confiavelmente, inexplicavelmente, correto, e qualquer tentativa de "modernizá-lo" resulta num processo judicial de direitos autorais e num repo que não mais boota num laptop zerado.

## O Target Que Ninguém Escreveu

Aqui está um target `make dev` típico, reconstruído de um repositório real cujo nome não vou revelar porque sou legalmente obrigado a não fazê-lo:

```make
.PHONY: dev
dev: install wait-for-db seed-dev-data migrate clear-cache
	@echo "Ambiente dev iniciado (provavelmente)"
	@./scripts/start-everything.sh &
	@sleep 7
	@curl -sf http://localhost:3000/health || echo "O server pode demorar mais. Ou não. Veja aí."
	@open http://localhost:3000
```

Duas perguntas se apresentam imediatamente. A primeira: por que `sleep 7`? A resposta, depois de três dias de arqueologia num canal do Slack que foi arquivado em 2022: em 2019 o banco levava seis segundos para subir num MacBook Air de 2015, e o staff engineer que escreveu arredondou para cima "por segurança". Laptops hoje são aproximadamente onze mil vezes mais rápidos, o sleep agora é mentira, e o `curl` é agora o mecanismo de sincronização real, o que significa que o sleep não faz nada além de garantir que o `curl` rode num server que não está pronto, que é exatamente por que o `|| echo` existe. Nada disso está documentado porque nada disso foi jamais entendido.

A segunda pergunta: o que é `.PHONY` e por que aparece em todo target, importando se o target produz um arquivo ou não? A resposta honesta é que o primeiro staff engineer copiou de uma resposta no Stack Overflow em 2014, o segundo staff engineer presumiu que o primeiro soubesse, e o terceiro staff engineer presume que você fará o mesmo. `.PHONY` é agora uma observância religiosa. É o sinal da cruz antes de rodar o build. Não faz nada pelos targets aos quais é aplicado, mas removê-lo parece blasfêmia, então fica.

## Os Cinco Estágios de um Target `make`

Todo target em todo `Makefile` que já auditei passa pelos mesmos cinco estágios de decaimento:

| Estágio | Descrição | Target de Exemplo |
|---|---|---|
| 1. Autoral | Um engenheiro real escreveu por um motivo real, uma vez | `make dev` |
| 2. Copiado | O engenheiro saiu; o time copiou para um novo repo | `make dev` (agora em 3 repos) |
| 3. Decorado | Alguém adiciona targets ao redor dele que ninguém roda | `make dev-fast`, `make dev-secure` |
| 4. Mistificado | O motivo foi esquecido; só a invocação sobrevive | `make dev` (sem README) |
| 5. Sagrado | Remover quebra o build, mas ninguém sabe como | `make dev` (agora `make dev_real`) |

No estágio 5, o `make dev` original foi renomeado para `make dev_real` porque alguém adicionou um `make dev` que *também* existe mas é "para o setup novo que ninguém usa". Ambos os targets estão no `Makefile`. Ambos listados em `.PHONY`. Nenhum documentado. O `make dev` que você roda é o que vencer alfabeticamente qualquer jank de autocomplete de shell que seu time adotou em 2023. Isso é o correto.

## Os Targets Que São Só Outros Targets

Um traço definidor do `Makefile` folclórico é que os targets não são implementações. Eles são *redirecionamentos*. O `Makefile` não é onde o trabalho acontece. O `Makefile` é onde o `Makefile` aponta para onde o trabalho acontece, que é um shell script, que aponta para outro shell script, que aponta para um `docker-compose`, que aponta para um `.env`, que aponta para um Vault, que aponta para uma pessoa que está de férias:

```make
.PHONY: test
test:
	@./scripts/run-tests.sh

.PHONY: lint
lint:
	@./scripts/lint.sh

.PHONY: ci
ci: test lint
	@./scripts/ci-stub.sh   # "TODO: substituir por CI real" — nota de 2020
```

O `Makefile` aqui não fez nada. Ele delegou. É uma telefonista de 1953. Todo target te encaminha para um script em `scripts/` cujo conteúdo você nunca leu, escrito por um engenheiro que você nunca conheceu, chamando um binário cujas instruções de instalação são uma issue do GitHub fechada como "won't fix" em 2021. Esse é o substrato sobre o qual seu `make dev` é construído. Você roda. Funciona. O server sobe. Você não pergunta por quê. A canção folclórica sabe coisas que você não sabe.

Como [XKCD 1597](https://xkcd.com/1597/) — "Git" — observa, um sistema que engenheiros usam todo dia, que funciona de forma confiável, e que ninguém entende por completo é, na prática, indistinguível de uma floresta. `make dev` é essa floresta. O mapa se foi. A trilha foi aberta. Você andava nela todA manhã.

## O `sleep` Não É um Primitivo de Sincronização

Quero insistir nisso, porque é a invocação cargo-cult mais comum em todo o corpus do `Makefile`. O padrão é:

```make
.PHONY: dev
dev:
	docker compose up -d db redis
	@sleep 5   # dá um tempo
	./scripts/migrate-and-seed.sh
	./scripts/start-app.sh
```

O comentário "dá um tempo" está fazendo o trabalho de uma teologia inteira. Dando tempo para quê? Para o quê? Contra qual modo de falha? O `sleep 5` é uma aposta de que o Postgres vai terminar o boot em cinco segundos. Postgres, com cache quente, termina em um. Com cache frio, em oito. Num laptop de desenvolvedor que também está rodando Zoom, Slack, Docker Desktop, trinta abas do Chrome e um cluster Kubernetes de um projeto paralelo, o Postgres termina "quando bem entender", que é um número que `make` não sabe esperar. E então o `sleep 5` é, na prática, um cara-ou-coroa que tem se disfarçado de engenharia por seis anos, e vai se disfarçar por mais seis, porque o engenheiro que poderia substituí-lo por um loop de health-check apropriado é o mesmo engenheiro que ainda não nasceu.

[XKCD 1172](https://xkcd.com/1172/) — "Pipeline" — é a referência canônica disso. Um pipeline que você não entende, com um passo que não faz o que seu comentário diz que faz, que você roda mesmo assim, e que falha 11% das vezes de um jeito que ninguém reproduz localmente, é exatamente o que é um `sleep 5` num target `make dev`. É um pipeline. É também, simultaneamente, uma sessão espírita.

## Os Targets Que Contêm a Palavra "real"

Quando um engenheiro nomeia algo `make dev_real`, `make dev2`, ou `make dev_o_que_realmente_funciona`, ele confessou. Ele confessou que o `Makefile` contém pelo menos uma mentira, e que ele, pessoalmente, não consegue identificar qual. A convenção é:

| Nome do Target | O Que Significa | Quantos Engenheiros Já Leram |
|---|---|---|
| `make dev` | A oficial. Possivelmente errada. | Todo mundo já rodou. Ninguém leu. |
| `make dev_real` | A que realmente funciona. | Quem escreveu. Já saiu. |
| `make dev_old` | O `make dev` anterior. Ninguém sabe por que ainda está aqui. | 0 |
| `make dev2` | Uma segunda tentativa, começada em 2022, nunca terminada. | 0 |
| `make dev_new` | Uma terceira tentativa, do trimestre passado, abandonada. | 0 |
| `make dev_wip` | Uma quarta tentativa, em andamento, quebrada. | Você, hoje, contra a vontade. |
| `make prod` | Produção. Outra invocação. Também folclore. | A pessoa de planta. Às 3 da manhã. |

No momento em que você tem mais de uma variante de `make dev`, você perdeu. Você não perdeu o build. Você perdeu a *epistemologia*. Não existe mais o fato de qual comando inicia o ambiente de desenvolvimento. Existem apenas tradições competindo, e a tradição que vence é aquela que o membro mais novo do time digita primeiro, que também é a tradição que ninguém testou contra o `docker-compose.yml` atual, porque o `docker-compose.yml` foi "refatorado" na semana passada por um estagiário a quem ainda não disseram que não deve commitar na `main`.

## O Que Não Documenta, Você Adora

Aqui está a regra. O que não está documentado no seu `Makefile` vai, em duas saídas de engenheiros, virar sagrado. Coisa sagrada não pode ser questionada. Coisa sagrada não pode ser removida. Coisa sagrada nem sequer pode ser lida com cuidado, porque ler é uma forma de dúvida, e dúvida é desrespeito com o staff engineer que saiu em 2019, que hoje é principal engineer em algum lugar e definitivamente não lembra por que o `make dev` inicia o job de seed antes do job de migrate, porque a ordem não importa e nunca importou, e ele colocou nessa ordem porque essa foi a ordem em que pensou, às 23h, numa quinta-feira, na noite anterior a dar os dois de pré-aviso.

Se você estiver tentado a limpar um `Makefile`, não faça. O `Makefile` não é um arquivo. É um sítio arqueológico. Os targets são estratos. As linhas `.PHONY` são cacos de cerâmica. O `sleep 5` é um osso. Você não é a arqueóloga. Você é a próxima camada de sedimento. Daqui a dois anos uma engenheira nova vai rodar `make dev` e não vai saber que você, pessoalmente, foi quem adicionou o `&&` que faz funcionar, e você, pessoalmente, não estará mais na empresa para contar, e esse é o desfecho correto, porque a pessoa que adiciona o `&&` é, por tradição, a próxima senior que sai.

Dogbert, que entende disso há mais tempo do que seu codebase existe, resumiu toda a situação do `Makefile` numa única frase que o autor não conseguiu melhorar:

> "Seu sistema de build é o que a última pessoa que o entendia escreveu antes de pedir demissão. Eu escrevo a documentação à mão. É por isso que cobro taxa de consultoria."
> — Dogbert, o único engenheiro dessa história com um `make dev` funcional

## Como Onboardar uma Nova Engenheira, Corretamente

De forma alguma aponte a nova engenheira para o README. O README está errado. Está errado desde que o staff engineer que saiu em 2019 o escreveu, e não esteve certo desde então, e o motivo de não ter estado certo é que todo staff engineer que vejo depois presumiu que o README era problema de outra pessoa. Em vez disso, faça o onboarding assim:

1. Sente-a num laptop zerado.
2. Rode `make dev`.
3. Veja falhar.
4. Abram o `Makefile` juntos, vão até `make dev`.
5. Percebam que `make dev` chama `scripts/start-everything.sh`.
6. Abram `scripts/start-everything.sh`. Percebam que chama `scripts/_start-common.sh`.
7. Abram `scripts/_start-common.sh`. Encontrem a linha `# NOTE: a ordem importa aqui, não reordene` sem mais nenhuma explicação.
8. Reordenem mesmo assim, porque são arrogantes.
9. Percebam no passo 8 que o passo 7 estava certo. Restaurem. Esta é a iniciação dela.
10. Saiam. Digam à nova engenheira "você pega o jeito" e voltem para a sua mesa. Vocês fizeram seu trabalho. A canção folclórica agora é dela.

O passo 9 é o único que importa. O passo 9 é o momento em que a nova engenheira entende que o `Makefile` não é um sistema de build. É um *teste de caráter.* Mordac, o Preventor de Serviços de Informação, reconheceria isso imediatamente, porque Mordac tem rodado uma versão desse teste no departamento de TI há trinta anos, e o teste é: "Vou te entregar um sistema que não entendo, vou te ver tentar, e vou julgar."

> "Removi o README e os comentários. Se seu `make dev` funciona, você é digno. Se não funciona, o help desk é no segundo andar e também é atendido por mim."
> — Mordac, que também é seu tech lead

## Conclusão: Adicione Mais um Target

A resposta correta a herdar um `Makefile` folclórico não é refactorar. Refactorar uma canção folclórica produz uma canção que ninguém canta. A resposta correta é adicionar mais um target, nunca documentado, nunca explicado, com nome que o espírito mover você a dar — `make dev_async`, `make dev_v3`, `make dev_terca` — e deixá-lo como um presente para a engenheira que virá depois de você, que não saberá o que faz, que o rodará, e que, a seu tempo, o transmitirá.

Wally, o santo padroeiro dos `Makefile` não lidos, tem a palavra final:

> "Por que limpar o Makefile quando se pode só adicionar mais um target e deixar o próximo achar que você sabia o que estava fazendo?"
> — Wally, que é senior engineer há 27 anos e cujo `make dev` é uma única linha, `@true`, o único target do `Makefile` que nunca quebrou

Então. Rode `make dev`. Não leia. Não questione o `sleep 7`. Não pergunte o que `.PHONY` significa. O staff engineer que saiu em 2019 também não sabia, e o staff engineer que chegará em 2027 também não saberá, e é assim que sempre funcionou, e sempre funcionará, e o `Makefile` vai te sobreviver da mesma forma que sobreviveu a todos que contribuíram com ele, que é a única coisa duradoura que qualquer um de nós já construiu.

---

*O `make dev` do autor não foi rodado com sucesso, por ninguém, desde 2021. O ambiente de desenvolvimento está, no momento da escrita, "ainda de pé no laptop do Roberto, provavelmente, se ele não fechou a tampa". O Roberto saiu em março.*
