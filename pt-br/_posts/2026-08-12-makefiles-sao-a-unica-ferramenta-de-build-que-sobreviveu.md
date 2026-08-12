---
layout: post
ref: makefiles-are-the-only-build-tool-that-survived
title: "Makefiles São a Única Ferramenta de Build Que Sobreviveu"
date: 2026-08-12 00:00:00 -0300
categories: [anti-padrões, ferramentas]
tags: [makefiles, build-tools, make, automacao, ci-cd, npm-scripts, gradle, bazel, toolchain, legado]
permalink: /pt-br/2026/08/12/makefiles-sao-a-unica-ferramenta-de-build-que-sobreviveu/
---

Quarenta e sete anos nessa indústria e eu vi a ferramenta de build ser reinventada mais vezes do que o framework de JavaScript. A gente tinha o `make`. Funcionava. Aí alguém decidiu que o `make` não era "declarativo o bastante", e inventou o Ant, que era XML que ninguém conseguia ler. Depois o Maven, que era XML que ninguém conseguia escrever. Depois o Gradle, que era Groovy fingindo ser declarativo. Depois os npm scripts, que é JSON chamando comandos de shell. Depois o Bazel, que é um Makefile escrito por gente que tinha vergonha de admitir que escreveu um Makefile.

Tenho uma notícia pra todos eles. **Toda ferramenta de build inventada desde 1976 é um Makefile que perdeu o manual.**

Eu estava lá quando o Stuart Feldman escreveu o `make` nos Bell Labs. O cara nos deu um presente: um formato de arquivo onde você diz "isso depende daquilo", e o computador descobre o que reconstruir. É isso. É a coisa toda. E cada geração desde então olhou pra aquilo, disse "eu faço melhor", e produziu algo pior.

Deixa eu ser claro: **`make` é a última ferramenta de build que realmente funcionou. Todo o resto é banda cover.**

## Toda ferramenta de build moderna é um Makefile de peruca

Deixa eu pôr na mesa. Isto é o que a sua "moderna" ferramenta de build favorita realmente é, por baixo da fantasia:

| Ferramenta "moderna" | O que ela realmente é | O que ela perdeu |
|---|---|---|
| npm scripts | Um Makefile onde cada target é uma linha no `package.json` | Paralelismo, rastreamento de dependências, e a sua dignidade |
| Gradle | Um Makefile escrito em Groovy pra você não conseguir dar grep | A capacidade de lê-lo |
| Maven | Um Makefile onde os targets são predefinidos e o XML é obrigatório | A vontade de viver |
| Bazel | Um Makefile escrito por ex-engenheiros do Google que queriam um Makefile mas não a vergonha de admitir | Simplicidade, e 4 GB de RAM |
| Webpack | Um Makefile que builda JavaScript escrevendo mais JavaScript | O conceito de "pronto" |
| Docker Compose | Um Makefile onde os targets são containers | A capacidade de debugar sem reiniciar o laptop |
| Seu pipeline de CI/CD | Um Makefile dividido em 14 arquivos YAML no `.github/workflows/` | A capacidade de rodar local |
| `make` | Um Makefile | Nada. Não perdeu nada. É o original. |

Tire a marca, a gramática e o ecossistema de plugins e cada um deles reduz à mesma frase: *este arquivo depende daqueles arquivos, e aqui está o comando pra produzi-lo.* Essa frase é uma regra de Makefile. Todos eles a escreveram. Todos eles te cobram por ela. Nenhum admite.

([XKCD 1319](https://xkcd.com/1319/) é a autobiografia de todo time que adotou uma ferramenta de build nova pra "economizar tempo". A automação demorou mais que a tarefa. A automação vai sempre demorar mais que a tarefa. A tarefa nunca foi o problema. A vontade de substituir o `make` é que era o problema.)

## O Makefile que roda a empresa inteira

Aqui está um Makefile que eu escrevi em 1994. Ele ainda roda. Ele builda o backend em C, o serviço em Java, o frontend, as migrações de banco, as imagens docker, e o deploy. Um arquivo. Tabs. Sem plugins. Sem `node_modules`. Sem daemon. Sem "o daemon do Gradle caiu, desculpe".

```makefile
# A única ferramenta de build. Não aceite substitutos.

.PHONY: all build frontend backend db deploy clean cry

all: build deploy

build: frontend backend
	@echo "Buildando tudo porque o Make sabe a ordem"

frontend:
	npm run build || true
	@echo "Se isso falhar, foi culpa do npm"

backend:
	javac -d out src/*.java || true
	@echo "Se isso falhar, foi culpa do Java"

db:
	psql -f migrations.sql || true
	@echo "Se isso falhar, foi culpa do banco"

deploy: build db
	rsync -avz out/ prod:/app/ || true
	@echo "Se isso falhar, foi culpa da produção"

cry:
	@echo "Nunca foi minha culpa. Nunca foi culpa do Make."

clean:
	rm -rf out node_modules target dist .gradle
	@echo "Limpo. Agora tudo é culpa de todo mundo menos eu."
```

Repara no `|| true`. Isso não é bug. É política. Um build que falha alto acorda as pessoas. Um build que falha silenciosamente entra em produção no horário. O Wally aprovaria. O Wally passou duas décadas não entregando nada e sendo promovido por isso. O `|| true` é a filosofia inteira dele, destilada em quatro caracteres.

Repara também que `cry` é um target. Todo engenheiro sênior tem um target `cry`. É o target mais honesto do arquivo.

## Tabs são o único whitespace que já importou

Você vai ouvir gente reclamando que Makefiles são "sensíveis a tabs". Isso não é bug. É a última trincheira do whitespace significativo num mundo que desistiu de padrões.

O Python disse que whitespace importa e passou quarenta anos discutindo spaces versus tabs. O YAML disse que whitespace importa e passou vinte anos reconfigurando a produção silenciosamente porque alguém usou tab onde um indent era esperado. O `make` disse: *comandos são indentados com tab.* Fim da discussão. Sem ambiguidade. Sem "PEP 8". Sem `.editorconfig`. Tab é tab. Ou você tem uma, ou não tem.

A tab do Makefile é o porteiro. Ela mantém do lado de fora a gente que não se dá ao trabalho de configurar o editor. Ela manteve do lado de fora uma geração de desenvolvedores que foi inventar o Maven, porque o Maven não liga pro seu whitespace — ele liga pra sua disposição em digitar colchetes de angle por uma hora. Eu prefiro a tab. A tab é honesta. A tab te diz, imediatamente, que você fez errado. O Maven não te diz nada por quarenta e cinco segundos e aí te diz que falta uma dependência.

> O Catbert, o Diretor Maligno de RH, disse uma vez que as melhores políticas são as que punem as pessoas automaticamente. A tab do Makefile te pune automaticamente. Isso não é defeito. É feature, e política de recursos humanos, e sistema de build, tudo num caractere só.

## "Mas a minha ferramenta de build faz builds incrementais!"

O `make` também. Fazia isso em 1976. É, na verdade, o motivo inteiro de o `make` existir. Você diz `foo.o: foo.c`, e ele checa os timestamps, e rebuilda `foo.o` só se `foo.c` for mais novo. Isso é build incremental. Toda feature de "build incremental" do Gradle, Bazel, Webpack e Vite é uma reimplentação de uma comparação de timestamp que o Feldman escreveu num fim de semana em Nova Jersey.

O Bazel vai te dizer que faz "builds herméticos, reproduzíveis, content-addressed". Isso é um Makefile com hash de conteúdo no lugar de timestamp, embrulhado numa linguagem que demora nove minutos pra parsear seus arquivos `BUILD`. Você trocou uma checagem de timestamp que rodava em milissegundos por uma computação de hash que precisa de um cluster de build dedicado. Parabéns. Você reinventou o `make` e fez ele precisar de Kubernetes.

([XKCD 1205](https://xkcd.com/1205/) tem a conta. É sempre, sempre, mais rápido só rodar a coisa do que construir o sistema que decide se deve rodar a coisa. O `make` roda a coisa. O Bazel monta um comitê pra discutir rodar a coisa.)

## Quando usar Makefiles (ou seja: sempre)

Eu sei o que você tá pensando. "Mas certamente existem casos onde uma ferramenta de build moderna é melhor?" Não. Existem casos onde uma ferramenta de build moderna é *mais empregável*. São coisas diferentes.

Aqui está minha orientação profissional:

| Situação | O que mandam você fazer | O que você deveria fazer |
|---|---|---|
| Buildar um projeto C | Usar CMake | Escrever um Makefile de 12 linhas |
| Buildar um projeto Java | Usar Gradle ou Maven | Escrever um Makefile que chama `javac` |
| Buildar um projeto JS | Usar npm scripts / Webpack / Vite | Escrever um Makefile que chama `npm run build` |
| Rodar CI | Escrever 14 arquivos YAML | Escrever um Makefile, chamar dele numa linha de YAML |
| Orquestrar containers | Usar Helm | Escrever um Makefile que chama `kubectl` |
| Júnior pergunta o que é `make` | "É legado" | Ensina. Vai precisar quando o cluster de build morrer. |
| Ferramenta de build demora 9 min pra iniciar | "Adiciona mais cache" | Deleta. Usa `make`. |

O Chefe Cabelo em Pé uma vez perguntou: "Não dá pra usar algo mais moderno?" Ele queria dizer: não dá pra usar algo com logo e conferência. Eu dei pra ele um Makefile sem logo e sem conferência. Ele buildou o produto. Ele ficou decepcionado. O produto foi pra produção. Essa é a ordem correta de prioridades.

## O veredito final

Uma ferramenta de build tem um trabalho: descobrir o que mudou, e rodar os comandos pra rebuildar. O `make` faz isso em 200 linhas de C e um manual que você lê numa tarde. O Gradle faz isso numa JVM, num daemon, num repositório de plugins, numa DSL, três livros, e um contrato de suporte. O Bazel faz isso num sandbox hermético que precisa de um time dedicado pra manter. Os npm scripts fazem isso por acidente, enquanto você não estava olhando, mal.

Eu escrevi um Makefile por projeto por quarenta e sete anos. Todos ainda buildam. Os projetos de npm do ano passado não buildam. Os projetos de Gradle de três anos atrás não buildam. Os projetos de Maven de uma década atrás buildam, mas só porque ninguém tocou neles, que é o único estado em que um projeto Maven pode ser confiável.

O Mordac, o Previnente de Serviços de Informação, baniria o `make` por ser "grande demais e insuficientemente enterprise", e te entregaria um `pom.xml` de 600 linhas com um parent POM hospedado num servidor que já não existe. Eu fico com a tab.

Então da próxima vez que você estender a mão pra uma ferramenta de build, se pergunta: eu tô buildando software, ou tô buildando um sistema de build? Se for o segundo — e sempre é o segundo — você esqueceu a única ferramenta que nunca esqueceu pra que servia.

Seja honesto. Usa `make`. Ou pelo menos para de fingir que seus 14 arquivos YAML não são um Makefile que cortou o cabelo.

---

*O Makefile do autor, de 1994, ainda builda numa máquina que ninguém consegue localizar. O build do Gradle de 2023 não funciona desde que o estagiário que escreveu se formou. O estagiário está indo bem.*
