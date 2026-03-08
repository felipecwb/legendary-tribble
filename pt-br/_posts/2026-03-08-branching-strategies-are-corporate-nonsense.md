---
layout: post
ref: branching-strategies-are-corporate-nonsense
title: "Estratégias de Branching São Bobagem Corporativa: Commita na Main"
date: 2026-03-08 09:00:00 -0300
categories: [git, sabedoria]
tags: [git, branching, gitflow, trunk-based, devops]
permalink: /pt-br/:year/:month/:day/branching-strategies-are-corporate-nonsense/
---

GitFlow. GitHub Flow. GitLab Flow. Trunk-based development. Feature branches. Release branches. Hotfix branches. Sabe o que tudo isso tem em comum? **São formas de complicar coisas simples**.

Quer saber minha estratégia de branching? `git push origin main`. É isso. É a estratégia inteira.

## O Problema com Branches

Toda branch é um universo paralelo onde seu código vive sozinho, com medo, gradualmente divergindo da realidade. Quanto mais tempo uma branch vive, mais ela se afasta da `main`. E aí vem o merge. O temido merge.

```
Ciclo de vida da feature branch:

Dia 1: Cria branch da main
Dia 2: Escreve código
Dia 3: Escreve mais código
Dia 4: Puxa main mais recente, resolve 3 conflitos
Dia 5: Continua codando
Dia 7: Puxa main mais recente, resolve 47 conflitos
Dia 10: Desiste, cria nova branch
Dia 11: Repete
```

Sabe o que nunca tem conflitos de merge? Código que já está na main.

## Por Que GitFlow é Crime de Guerra

Deixa eu decodificar GitFlow pra você:

| Branch GitFlow | O Que Promete | O Que Realmente Acontece |
|----------------|---------------|-------------------------|
| main | Código pronto pra produção | Ninguém sabe o que tem lá |
| develop | Branch de integração | Pesadelos de integração |
| feature/* | Desenvolvimento isolado | Isolamento da realidade |
| release/* | Estabilização | Desestabilização |
| hotfix/* | Correções rápidas | Criação rápida de novos bugs |

Você precisa de um PhD em Git pra entender quando fazer merge do quê pra onde. E nem me faz começar sobre o passo "merge de volta pra develop" que todo mundo esquece.

Eu trabalhei numa empresa onde uma branch de hotfix levou 3 semanas porque ninguém conseguia descobrir a ordem dos merges. O bug era um typo numa mensagem de erro.

## Trunk-Based Development (Mas De Verdade)

Pessoal fala de trunk-based development como se fosse revolucionário. Sabe o que trunk-based development realmente é? É o que fazíamos em 1997 com CVS antes de alguém inventar branches e estragar tudo.

Minha implementação de trunk-based development:

```bash
# Passo 1: Escreve código
vim main.py

# Passo 2: Deploy
git add .
git commit -m "coisas"
git push origin main

# Passo 3: Não existe passo 3
```

Alguns chamam isso de imprudente. Eu chamo de eficiente.

## Feature Flags: A Solução Real

"Mas como você trabalha em features sem feature branches?"

Simples: commita código quebrado na main e esconde atrás de feature flags.

```python
if os.getenv('HABILITA_NOVA_FEATURE') == 'true':
    # Código novo que talvez funcione
    faz_coisa_nova()
else:
    # Código antigo que definitivamente funciona
    # (a gente acha)
    faz_coisa_antiga()
    
# Depois de 6 meses, ninguém lembra o que HABILITA_NOVA_FEATURE faz
# Mas temos medo de remover
# Fica pra sempre
# Tá tudo bem
```

## O Teste Dilbert

O Chefe Cabeça Pontuda uma vez perguntou: "Por que precisamos de todas essas branches? Só coloca tudo num lugar só."

Por uma vez, ele não estava errado. A resposta do Wally foi esclarecedora: "Branches me permitem fingir que trabalho em algo por semanas sem ninguém verificar meu código."

É por isso que branches existem. Não pra isolamento de código. Pra evitar responsabilização.

## [XKCD 1597](https://xkcd.com/1597/) Estava Certo

Ninguém entende Git. Todos fingimos. Quanto mais branches você tem, mais oportunidades de provar que não entende Git.

Meu colega uma vez passou 4 horas tentando fazer merge de uma feature branch. Ele acabou deletando a branch, copiando os arquivos manualmente, e commitando direto na main.

Isso não é uma falha. É iluminação.

## Code Review Sem Branches

"Mas e code reviews? Você não precisa de branches pra pull requests?"

Não. Aqui está meu processo de code review:

1. Escreve código
2. Commita na main
3. Se os testes passam, está revisado
4. Se os testes falham, reverte
5. Se não tem testes, veja passo 3

Alguns chamam isso de "continuous deployment." Eu chamo de "mover rápido."

## O Problema Real

Sabe do que branches te protegem? Código ruim chegando em produção.

Sabe o que mais te protege de código ruim? Não escrever código ruim.

Todo o sistema de branches existe porque não confiamos em nós mesmos. Mas a questão é: se você não confia no seu código sem branches, você também não deveria confiar com branches. A branch não corrigiu o bug—só atrasou a descoberta.

## Minha Estratégia de Branching na Prática

```
Repositório: app-crítico-de-produção
Branches: 1 (main)
Contribuidores: 47
Commits por dia: 200+
Deploys por dia: 50+
Incidentes causados por não ter branching: 3
Incidentes culpados no não-branching: 47
Incidentes realmente causados por branching no emprego anterior: "não falamos disso"
```

## Conclusão

Toda branch é dívida técnica. Todo merge é sofrimento. Toda estratégia de branching é um mecanismo de enfrentamento pra uma disfunção organizacional mais profunda.

Push na main. Confia nos seus testes. Demita rápido.

---

*O último repositório do autor tinha 847 feature branches abandonadas, cada uma representando um desenvolvedor que saiu da empresa antes de terminar o trabalho. A branch main continha código que "provavelmente funciona." De fato, provavelmente funcionava.*
