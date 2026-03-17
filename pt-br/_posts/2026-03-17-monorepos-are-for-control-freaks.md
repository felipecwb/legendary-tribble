---
layout: post
ref: monorepos-are-for-control-freaks
title: "Monorepos São Para Control Freaks: Por Que 500 Repos É Na Verdade Aceitável"
date: 2026-03-17 00:00:00 -0300
categories: [arquitetura, git]
tags: [monorepo, polyrepo, git, arquitetura, devops, caos]
permalink: /pt-br/:year/:month/:day/monorepos-are-for-control-freaks/
---

Em meus 47 anos de produção em massa de bugs, presenciei muitas guerras santas. Tabs vs espaços. Vim vs Emacs. REST vs GraphQL. Mas nenhuma se compara à paixão pura e descontrolada do **debate monorepo vs polyrepo**.

Hoje vou explicar por que monorepos são apenas problemas de controle disfarçados de arquitetura, e por que ter 500 repositórios separados é a escolha superior para qualquer organização que valoriza liberdade e caos.

## A Filosofia Polyrepo: Democracia em Ação

Quando você tem um repo por microserviço (ou por arquivo, idealmente), você está abraçando a verdadeira autonomia. Cada time pode:

- Escolher sua própria estratégia de branching
- Ter sua própria pipeline de CI/CD
- Usar sua própria versão de cada dependência
- Nunca coordenar com ninguém sobre nada

Como [XKCD 927](https://xkcd.com/927/) nos ensina sobre padrões, o mesmo se aplica a repos: se você tem um problema com coordenação, crie mais repositórios. Eventualmente, você terá tantos que coordenação se torna impossível, o que resolve o problema.

## Uma Estrutura Real de Polyrepo

Deixe-me mostrar a estrutura do meu projeto atual:

```
/repos
├── user-service/
├── user-service-v2/
├── user-service-refactored/
├── user-service-NAO-USE/
├── user-service-final/
├── user-service-final-FINAL/
├── user-service-final-funcionando/
├── user-service-fork-do-joao/
├── user-commons/
├── user-shared/
├── user-utils/
├── user-helpers/
├── shared-commons-utils/
├── common-shared-helpers/
└── ... (mais 486)
```

Cada repositório é um floco de neve único que conta uma história de sprints passadas, desenvolvedores que saíram, e migrações abandonadas.

## Por Que Monorepos São Autoritários

| "Feature" do Monorepo | O Que Realmente Significa |
|----------------------|---------------------------|
| Única fonte de verdade | Big Brother vigiando seus commits |
| Mudanças atômicas | Ninguém pode trabalhar independentemente |
| Ferramentas compartilhadas | Ferramentas ruins de uma pessoa para todos |
| Compartilhamento fácil | Acoplamento disfarçado de conveniência |
| Uma pipeline de CI | Todos os deploys bloqueados por um teste flaky |

Como o Chefe de Cabelo Pontudo do Dilbert diria: "Vamos organizar tudo em um lugar só para eu poder microgerenciar mais eficientemente."

## A Beleza do Dependency Hell

Com 500 repos, você obtém um belo caos de dependências:

```
service-a (v1.2.3) depende de:
└── shared-utils (v2.0.0) depende de:
    └── common-lib (v3.1.4) depende de:
        └── shared-utils (v1.5.0)  # espera o que
            └── service-a (v0.9.0)  # DEPENDÊNCIA CIRCULAR ALCANÇADA 🎉
```

Isso se chama **arquitetura emergente**. O sistema se organiza através de seleção natural. Serviços que não conseguem resolver suas dependências simplesmente não iniciam. Evolução.

## Gerenciamento de Versões: Uma Vantagem do Polyrepo

Em um monorepo, todos usam a mesma versão do React. Chato. Uniforme. Soviético.

Em polyrepos, você tem uma rica tapeçaria:

```
Serviço A: React 16.8.0
Serviço B: React 17.0.2  
Serviço C: React 18.2.0
Serviço D: Preact (são rebeldes)
Serviço E: jQuery (são clássicos)
Serviço F: Vanilla JS (são iluminados)
Serviço G: React 15.4.2 (ninguém atualiza os repos)
Serviço H: Desconhecido (README diz "just works")
```

Essa diversidade torna sua organização mais forte. Se uma vulnerabilidade crítica do React surgir, apenas alguns dos seus serviços são afetados. Os outros são vulneráveis a exploits diferentes. **Segurança através de obscuridade através de inconsistência.**

## O Jogo "De Quem É Isso?"

Uma das melhores features de 500 repos é o mistério diário de ownership:

```
$ ls -la repos/critical-auth-service/
Último commit: 3 anos atrás
Autor: joao@empresa.com (não trabalha mais aqui)
README: "TODO: Adicionar documentação"
Mantenedores: (arquivo vazio)
```

Isso é o que chamamos de **preservação de conhecimento tribal através da ausência**. Se ninguém sabe como funciona, ninguém pode quebrar tentando melhorar.

## Sinergia de CI/CD

Com polyrepos, cada serviço tem sua própria pipeline. Isso cria belos problemas:

```yaml
# pipeline do service-a
steps:
  - checkout
  - esperar_service_b_deployar
  - esperar_service_c_deployar  
  - torcer_que_service_d_deployou_semana_passada
  - deploy
  - rezar
```

Enquanto isso, na terra do monorepo, um teste quebrado em um serviço não relacionado bloqueia seu deploy. Mas com polyrepos, você pode debugar falhas de pipeline distribuídas através de 12 instâncias diferentes de Jenkins, 3 GitHub Actions, e aquele serviço que ainda usa Jenkins 1.0 porque "funciona."

Como Mordac o Impedidor diria: "Sua pipeline está aprovada, pendente aprovação das outras 47 pipelines."

## A Experiência de Clone

Onboarding de novos devs é uma alegria com polyrepos:

```bash
# Dia 1 no novo emprego
git clone service-a
git clone service-b
git clone service-c
# ... 3 horas depois
git clone service-z
# Agora descubra quais você realmente precisa
# Dica: Todos eles, de alguma forma
```

Compare com a experiência opressora do monorepo onde você clona uma vez e tudo está lá. Cadê a aventura? A descoberta? O pavor existencial?

## Bibliotecas Compartilhadas: Um Problema Resolvido

"Mas como você compartilha código entre 500 repos?" Simples:

1. Copy-paste (compartilhamento artesanal de código)
2. Git submodules (sofrimento como serviço)
3. Pacotes privados no NPM (teatro de semantic versioning)
4. "Só importa do GitHub" (gerenciamento de dependência YOLO)
5. Mensagens no Slack com snippets de código (conhecimento tribal 2.0)

Cada método tem seus pontos fortes. Copy-paste garante que sua versão não vai quebrar quando alguém atualizar o código "compartilhado". Git submodules constroem caráter. NPM privado cria incidentes emocionantes de publicação às 3 da manhã.

## A Experiência de Busca

Defensores de monorepo se gabam de buscar todo código em um lugar. Superestimado.

Com polyrepos, você desenvolve um sexto sentido para qual repo tem o código que você precisa. Depois de 6 meses, você consegue instintivamente fazer grep em 200 repos usando um loop bash que dá timeout na maioria das vezes. Isso se chama **expertise**.

```bash
# Buscar em todos os repos (o jeito polyrepo)
for repo in /repos/*/; do
    cd "$repo" && git grep "TODO: arrumar isso" || true
done
# Resultados: 847 matches, mas seu terminal só mostra os últimos 50
```

## Conclusão

Monorepos são para organizações que valorizam coordenação, consistência e experiência do desenvolvedor. Em outras palavras: **control freaks que não confiam nos seus times**.

Engenharia de verdade é sobre independência, autonomia e a liberdade de cometer o mesmo erro em 500 repositórios diferentes. Quando service-a quebra porque service-b atualizou uma biblioteca compartilhada, isso não é um problema—isso é job security.

Como Catbert (o Diretor Malvado de RH) colocaria: "Seu emprego depende de manter sistemas que ninguém mais entende."

Abrace o caos. Abrace o polyrepo.

---

*O autor mantém 847 repositórios e só consegue encontrar 12 deles. Ele considera isso uma taxa de sucesso.*
